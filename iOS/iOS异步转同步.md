## iOS异步转同步

在iOS中，很多时候我们去调用一些API的时候，结果都是采用block来返回。block的虽然好用但是在某些时候却给编码带来一些麻烦。比如你需要在某个函数return返回参数的时候，但因为在block中不能去直接return，这种时候大多数使用dispatch_semaphore_t来进行线程同步。所以接下来我主要梳理一下平时如何去同步的方式，来做以记录。

### 1

首先，在主线程中直接调用，并且不去做任务线程的转换。

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    [self f1];
}

- (int)f1 {
    NSLog(@"semaphore---begin");
    NSLog(@"f1 currentThread---%@",[NSThread currentThread]);  // 打印当前线程

    __block int number = 0;
    NSLog(@"111111");
    [self f2:^(int i) {
        NSLog(@"44444");
        number = i;
    }];
    NSLog(@"22222");
    NSLog(@"semaphore---end, %d ",number);
    return number;
}

- (void)f2:(void(^)(int))block {
    NSLog(@"f2 currentThread----%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"33333");
    sleep(2);
    block(2);
}
```

最后日志输出：

> 2021-05-07 15:31:50.669518+0800 TestDemo[88601:1881101] semaphore---begin
> 2021-05-07 15:31:50.669644+0800 TestDemo[88601:1881101] f1 currentThread---<NSThread: 0x600000c28300>{number = 1, name = main}
> 2021-05-07 15:31:50.669723+0800 TestDemo[88601:1881101] 111111
> 2021-05-07 15:31:50.669833+0800 TestDemo[88601:1881101] f2 currentThread----<NSThread: 0x600000c28300>{number = 1, name = main}
> 2021-05-07 15:31:50.669909+0800 TestDemo[88601:1881101] 33333
> 2021-05-07 15:31:52.671129+0800 TestDemo[88601:1881101] 44444
> 2021-05-07 15:31:52.671383+0800 TestDemo[88601:1881101] 22222
> 2021-05-07 15:31:52.671603+0800 TestDemo[88601:1881101] semaphore---end, 2

根据日志可以梳理一下执行过程：

f1在main thread执行  --> 执行f2（因为主线程为串行，所以会等待f2执行完毕再执行下一步）--> sleep(2) --> block(2) --> number赋值 -- > --> end

可以看出，因为主线程是串行队列，所以一定会在f2执行完毕之后在执行下一步，所以这里不需要做线程转换。

### 2

在主线程中直接调用，子任务异步执行，这也是最常见情况

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    [self f1];
}

- (int)f1 {
    NSLog(@"semaphore---begin");
    NSLog(@"f1 currentThread---%@",[NSThread currentThread]);  // 打印当前线程

    __block int number = 0;
    NSLog(@"111111");
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self f2:^(int i) {
            NSLog(@"44444");
            number = i;
        }];
    });
    NSLog(@"22222");
    NSLog(@"semaphore---end, %d ",number);
    return number;
}

- (void)f2:(void(^)(int))block {
    NSLog(@"f2 currentThread----%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"33333");
    sleep(2);
    block(2);
}
```

执行结果：

> 2021-05-07 15:38:52.773300+0800 TestDemo[89131:1888185] semaphore---begin
> 2021-05-07 15:38:52.773405+0800 TestDemo[89131:1888185] f1 currentThread---<NSThread: 0x6000001bc600>{number = 1, name = main}
> 2021-05-07 15:38:52.773457+0800 TestDemo[89131:1888185] 111111
> 2021-05-07 15:38:52.773552+0800 TestDemo[89131:1888185] 22222
> 2021-05-07 15:38:52.773579+0800 TestDemo[89131:1888281] f2 currentThread----<NSThread: 0x6000001f0d80>{number = 5, name = (null)}
> 2021-05-07 15:38:52.773620+0800 TestDemo[89131:1888185] semaphore---end, 0
> 2021-05-07 15:38:52.773630+0800 TestDemo[89131:1888281] 33333
> 2021-05-07 15:38:54.774022+0800 TestDemo[89131:1888281] 44444

可以看到，因为f2在异步处理后，这次没有等待f2结果，所以导致得出number为0。所以这种情况就需要semaphore来线程同步。

改后的f1为：

```objective-c
- (int)f1 {
    NSLog(@"semaphore---begin");
    NSLog(@"f1 currentThread---%@",[NSThread currentThread]);  // 打印当前线程

    __block int number = 0;
    NSLog(@"111111");
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self f2:^(int i) {
            NSLog(@"44444");
            number = i;
            dispatch_semaphore_signal(semaphore);
        }];
    });
    NSLog(@"22222");
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"semaphore---end, %d ",number);
    return number;
}
```

这次执行结果为：

> 2021-05-07 15:43:14.185740+0800 TestDemo[89441:1892113] semaphore---begin
> 2021-05-07 15:43:14.185865+0800 TestDemo[89441:1892113] f1 currentThread---<NSThread: 0x6000039b04c0>{number = 1, name = main}
> 2021-05-07 15:43:14.185956+0800 TestDemo[89441:1892113] 111111
> 2021-05-07 15:43:14.186083+0800 TestDemo[89441:1892113] 22222
> 2021-05-07 15:43:14.186127+0800 TestDemo[89441:1892289] f2 currentThread----<NSThread: 0x6000039e8c80>{number = 6, name = (null)}
> 2021-05-07 15:43:14.186213+0800 TestDemo[89441:1892289] 33333
> 2021-05-07 15:43:16.188570+0800 TestDemo[89441:1892289] 44444
> 2021-05-07 15:43:16.188896+0800 TestDemo[89441:1892113] semaphore---end, 2

可以看到虽然f2异步执行了，但是dispatch_semaphore_wait卡住当前主线程，在等待f2执行完毕block返回后signal+1，主线程又可以通过，最终得到number为2。

### 3

第三种情况，在主线程中直接调用，子任务异步执行，但子任务在主线程返回结果。

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    [self f1];
}

- (int)f1 {
    NSLog(@"semaphore---begin");
    NSLog(@"f1 currentThread---%@",[NSThread currentThread]);  // 打印当前线程

    __block int number = 0;
    NSLog(@"111111");
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self f2:^(int i) {
            NSLog(@"44444");
            number = i;
            dispatch_semaphore_signal(semaphore);
        }];
    });
    NSLog(@"22222");
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"semaphore---end, %d ",number);
    return number;
}

- (void)f2:(void(^)(int))block {
    NSLog(@"f2 currentThread----%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"33333");
    sleep(2);
    dispatch_async(dispatch_get_main_queue(), ^{
        block(2);
    });
}
```

日志输出：

> 2021-05-07 15:47:34.504680+0800 TestDemo[89744:1895978] semaphore---begin
> 2021-05-07 15:47:34.504775+0800 TestDemo[89744:1895978] f1 currentThread---<NSThread: 0x6000037241c0>{number = 1, name = main}
> 2021-05-07 15:47:34.504827+0800 TestDemo[89744:1895978] 111111
> 2021-05-07 15:47:34.504911+0800 TestDemo[89744:1895978] 22222
> 2021-05-07 15:47:34.504939+0800 TestDemo[89744:1896157] f2 currentThread----<NSThread: 0x60000377d1c0>{number = 6, name = (null)}
> 2021-05-07 15:47:34.504997+0800 TestDemo[89744:1896157] 33333

Bingo，主线程果然卡死。因为dispatch_semaphore_wait已经卡住当前主线程，所以在f2中又切到主线程去执行block，所以是不会响应的。那么如何解决呢？

看到网上有人采用在一个同步并行队列中去执行f2

```objective-c

- (int)f1{
    __block int c=0;
    dispatch_sync(dispatch_get_global_queue(0, 0), ^{ //解决的关键
        dispatch_semaphore_t sema = dispatch_semaphore_create(0);
        [self f2:^(int a) {
            c=a;
            dispatch_semaphore_signal(sema);
        }];
        dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    });
   
    return c;
}

- (void)f2:(void(^)(int))block{
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
      sleep(2);
       dispatch_async(dispatch_get_main_queue(), ^{
            block(2);
        });
    });
}
```

But ,主线程依然是被锁状态。因为这个同步任务设定在全部并行队列中，但同步并不具备开启新线程的能力导致这段代码依旧在主线程中执行。

暂时还没找到好的解决方案，所以在使用信号量时一定要确定该方法中是以哪种方式去处理任务的，并且执行结果是如何返回的。否则很容易就会卡死主线程。