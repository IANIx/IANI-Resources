# Tiercel一个简单易用的Swift下载框架

官方介绍：

> Tiercel 是一个简单易用、功能丰富的纯 Swift 下载框架，支持原生级别后台下载，拥有强大的任务管理功能，可以满足下载类 APP 的大部分需求。
>

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

`SessionManager`中有两个核心函数：`download` 开启一个下载任务，`multiDownload`批量开启多个下载任务, 所有任务都会并发下载，在这两个函数最终返回的都是单个或多个`DownloadTask`数组。

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

验证成功后创建一个DownloadTask 局部变量，并开启串行队列的一个同步任务。

`operationQueue`是在manager `init`的传递参数，默认为`DispatchQueue = DispatchQueue(label: "com.Tiercel.SessionManager.operationQueue", autoreleaseFrequency: .workItem)`， 在init的时候将该queue全局保存。

```swift
var task: DownloadTask!
operationQueue.sync {}
```

在同步任务中，首先在taskMapper字典中根据url区查找对应的task。

```swift
/// 已有该任务就执行更新操作
task.update(headers, newFileName: fileName)
```

```swift
/// 没有就创建一个新的task对象
/// 创建完毕后将该task加入到taskMapper
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

