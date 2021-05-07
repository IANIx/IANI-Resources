# Tiercel一个简单易用的Swift下载框架

官方介绍：

> Tiercel 是一个简单易用、功能丰富的纯 Swift 下载框架，支持原生级别后台下载，拥有强大的任务管理功能，可以满足下载类 APP 的大部分需求。

GitHub：https://github.com/Danie1s/Tiercel

## 目录结构

Sources
├── Extensions
│   ├── Array+Safe.swift
│   ├── CodingUserInfoKey+Cache.swift
│   ├── Data+Hash.swift
│   ├── DispatchQueue+Safe.swift
│   ├── Double+TaskInfo.swift
│   ├── Int64+TaskInfo.swift
│   ├── OperationQueue+DispatchQueue.swift
│   ├── String+Hash.swift
│   ├── UIDevice+Free.swift
│   └── URLSession+ResumeData.swift
├── General
│   ├── Cache.swift
│   ├── Common.swift
│   ├── DownloadTask.swift
│   ├── Executer.swift
│   ├── Notifications.swift
│   ├── Protector.swift
│   ├── SessionConfiguration.swift
│   ├── SessionDelegate.swift
│   ├── SessionManager.swift
│   ├── Task.swift
│   ├── TiercelError.swift
│   └── URLConvertible.swift
├── Info.plist
├── Tiercel.h
└── Utility
    ├── FileChecksumHelper.swift
    └── ResumeDataHelper.swift

## 源码解析

### SessionManager

`SessionManager`是核心class，通过`SessionManager`的实例去管理下载任务

```swift
var sessionManager = SessionManager("ViewController1", configuration: SessionConfiguration())
```

开始下载任务

```swift
sessionManager.download(URLString)?.progress { [weak self] (task) in
          self?.updateUI(task)
      }.completion { [weak self] (task) in
          self?.updateUI(task)
          if task.status == .succeeded {
              // 下载成功
          } else {
              // 其他状态
          }
      }.validateFile(code: "9e2a3650530b563da297c9246acaad5c", type: .md5) { [weak self] (task) in
          self?.updateUI(task)
          if task.validation == .correct {
              // 文件正确
          } else {
              // 文件错误
          }
      }
```

暂停下载任务

```swift
sessionManager.suspend(URLString)
```

取消下载任务

```swift
sessionManager.cancel(URLString)
```

删除下载任务

```swift
sessionManager.remove(URLString, completely: false)
```

清除下载任务

```swift
sessionManager.cache.clearDiskCache()
```

###### 方法解析：

#### Init

初始化方法，必传参数：`identifier`(标识符)、`configuration`(配置信息)，非必传：`logger`(日志) `cache`(缓存) `operationQueue`(操作队列)，`operationQueue`默认为DispatchQueue(label:"com.Tiercel.SessionManager.operationQueue", autoreleaseFrequency: .workItem)) 

在init中，除了做一些初始化赋值操作，还有一个关键点就是生成配置protectedState。protectedState是一个私有对象，为Protector的实例，以State为泛型。它的主要作用就是用于读写的安全操作，去加锁，具体后面说。

在配置完成后，在当前操作队列中执行同步操作，分别有`createSession`(建立session) 和`restoreStatus`(重置状态)。

```swift
public init(_ identifier: String,
                configuration: SessionConfiguration,
                logger: Logable? = nil,
                cache: Cache? = nil,
                operationQueue: DispatchQueue = DispatchQueue(label: "com.Tiercel.SessionManager.operationQueue",
                                                              autoreleaseFrequency: .workItem)) {
        let bundleIdentifier = Bundle.main.bundleIdentifier ?? "com.Daniels.Tiercel"
        self.identifier = "\(bundleIdentifier).\(identifier)"
        protectedState = Protector(
            State(logger: logger ?? Logger(identifier: "\(bundleIdentifier).\(identifier)", option: .default),
                  configuration: configuration)
        )
        self.operationQueue = operationQueue
        self.cache = cache ?? Cache(identifier)
        self.cache.manager = self
        self.cache.retrieveAllTasks().forEach { maintainTasks(with: .append($0)) }
        succeededTasks = tasks.filter { $0.status == .succeeded }
        log(.sessionManager("retrieveTasks", manager: self))
        protectedState.write { state in
            state.tasks.forEach {
                $0.manager = self
                $0.operationQueue = operationQueue
                state.urlMapper[$0.currentURL] = $0.url
            }
            state.shouldCreatSession = true
        }
        operationQueue.sync {
            createSession()
            restoreStatus()
        }
    }
```

#### download

首先在开始下载前先验证url, 并进行异常捕获。

```swift
do {
    let validURL = try url.asURL()
     } catch {
    log(.error("create dowloadTask failed", error: error))
    return nil
}
```

验证成功后创建一个DownloadTask 局部变量，并开启当前串行队列的一个同步任务。

```swift
var task: DownloadTask!
operationQueue.sync {}
```

在同步任务中，首先在taskMapper字典中根据url区查找对应的task。

```swift
 public func fetchTask(_ url: URLConvertible) -> DownloadTask? {
        do {
            let validURL = try url.asURL()
            return protectedState.read { $0.taskMapper[validURL.absoluteString] }
        } catch {
            log(.error("fetch task failed", error: TiercelError.invalidURL(url: url)))
            return nil
        }
    }
```

已有该任务就执行更新操作

```swift
task.update(headers, newFileName: fileName)
```
没有就创建一个新的task对象
创建完毕后将该task加入到taskMapper

```swift
task = DownloadTask(validURL,
                    headers: headers,
                    fileName: fileName,
                    cache: cache,
                    operationQueue: operationQueue)
task.manager = self
task.session = session
maintainTasks(with: .append(task))
```

处理完毕后，将task存到到cache中, 并开启任务

```swift
storeTasks()
start(task, onMainQueue: onMainQueue, handler: handler)
```

### DownloadTask

downloadTask也是下载任务的实际处理类，包含对下载任务的基本操作：

`download` `start` `suspend` `cancel` `remove` `update`

最终都是通过sessionTask: URLSessionDownloadTask去处理。 