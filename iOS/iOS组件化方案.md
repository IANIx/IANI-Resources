# iOS组件化方案

### 前言

组件化已经是一个老生常谈的问题了，目前市场上也有着各种成熟的组件化方案。我做过大大小小的项目中也有采用过很多不同的组件方式，本篇文章主要是记录不同方案之间的区别，以及在实际项目中如果挑选最适合自己的。

### 一、为什么要组件化

1. 模块解耦

   通常一个工程中会有多个模块，而模块之间会有依赖关系，比如A调用B，那么在A模块中就会引用B模块的头文件，同时可能B模块又会依赖C模块，C模块又会依赖A模块等等，最终的结果是各模块高度耦合，特别是大型的工程，耦合特别严重。

   为了避免耦合，需要设计一种结构去避免各模块间的耦合，主要方式就是建立Mediator模块，ABC模块各自依赖Mediator，由Mediator在中间进行调度。

2. 减少冲突

   在每次发版以后，release分支会向dev分支合并代码，由于各组件会提交许多代码，不同的开发者可能会同时修改同一个文件，引起冲突，这样一个工程中就会出现多处的合并冲突，比如一个几十人的团队，这个冲突的概率非常大，需要各组件的人去dev分支修改自己组件的冲突，如果冲突没有解决，整个工程都无法运行，影响整个团队的开发，非常影响效率。

3. 方便拆分组件

   假设一个项目中有多个组件，相互耦合，这个时候想单独将一个组件拆分出来，提供给它人使用，几乎是不可能的，而组件化接触这种耦合之后，我们可以直接将某个组件单独提供给它人使用，各个组件可以像积木一样，相互组合起来，形成一个新的APP对外付能。

### 二、组件化方案

#### [HHRouter](https://github.com/lightory/HHRouter)

`HHRouter`是我早期使用的一个解耦页面依赖的URL路由，URL Router 即将 UIViewController 映射成 URL，从而支持通过URL进行界面跳转。就和Web一样。

注册

```objective-c
[[HHRouter shared] map:@"one" toControllerClass:[OneViewController class]];
[[HHRouter shared] map:@"/user/:userId/" toControllerClass:[UserViewController class]];
```

使用

```objective-c
ViewController *vc = [[HHRouter shared] matchController:@"one"];
[self.navigationController pushViewController:vc animated:YES];

ViewController *vc = [[HHRouter shared] matchController:@"/user/1/"];
[self.navigationController pushViewController:vc animated:YES];
```

优势就在于使用简单，快速灵活，重在解耦UIViewController间的耦合。但缺点也很明显，首先所有的UIViewController需要统一注册（大多在Appdelegate中），那注册类对所有UIViewController还是强耦合状态，其次也无法对ViewController对象作出自定义操作。

------

#### [CTMediator](https://github.com/casatwy/CTMediator)

`CTMediator`主要是基于Mediator模式和Target-Action模式，中间采用了runtime来完成调用。通过mediator的Category来输出组件的对外调用方法。使用同一种方案，实现组件间调用。具体可参考 **casatwy**的这篇Blog https://limboy.me/2016/03/10/mgj-components。

使用CTMediator组件结构如下：

```
project
│   README.md
│   Appdelegate.h
│   Appdelegate.m
|		....
└───Modules
    │
    └───CTMediator
    │   │   CTMediator.h
    │   │   CTMediator+CTMediatorModuleAActions.h
    │   │   CTMediator+CTMediatorModuleBActions.h
    │   │   ...
    │
    └───ModuleA
    │		│		
   	│		└───Actions
   	│		│		│		Target_A.h
   	│		│		ModuleAViewController.h
   	│		│		...
   	│
   	└───ModuleB
    │		│		
   	│		└───Actions
   	│		│		│		Target_B.h
   	│		│		ModuleBViewController.h
   	│		│		...
   	│
   	│		...

```

调用方式:

```
if (indexPath.row == 0) {
        UIViewController *viewController = [[CTMediator sharedInstance] CTMediator_viewControllerForDetail];

        // 获得view controller之后，在这种场景下，到底push还是present，其实是要由使用者决定的，mediator只要给出view controller的实例就好了
        [self presentViewController:viewController animated:YES completion:nil];
    }

    if (indexPath.row == 1) {
        UIViewController *viewController = [[CTMediator sharedInstance] CTMediator_viewControllerForDetail];
        [self.navigationController pushViewController:viewController animated:YES];
    }

    if (indexPath.row == 2) {
        // 这种场景下，很明显是需要被present的，所以不必返回实例，mediator直接present了
        [[CTMediator sharedInstance] CTMediator_presentImage:[UIImage imageNamed:@"image"]];
    }

    if (indexPath.row == 3) {
        // 这种场景下，参数有问题，因此需要在流程中做好处理
        [[CTMediator sharedInstance] CTMediator_presentImage:nil];
    }

    if (indexPath.row == 4) {
        [[CTMediator sharedInstance] CTMediator_showAlertWithMessage:@"casa" cancelAction:nil confirmAction:^(NSDictionary *info) {
            // 做你想做的事
            NSLog(@"%@", info);
        }];
    }
```

Mediator做为中间件完全和其他Modules解除耦合关系，通过runtime去调用到对应的Target-Action，也无需注册URL，并且支持对各种复杂参数的调用，完全可以在任何复杂的场景下使用。缺点也很明显，无论是target还是action都是需要硬编码去完成。

------

#### [MGJRouter](https://github.com/meili/MGJRouter)

`MGJRouter` 是蘑菇街组件化方案，也是URL Router。主要通过url-block使用。具体介绍可参考limboy的Blog:https://limboy.me/2016/03/10/mgj-components/

最基本的使用方案

```
[MGJRouter registerURLPattern:@"mgj://foo/bar" toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"routerParameterUserInfo:%@", routerParameters[MGJRouterParameterUserInfo]);
}];

[MGJRouter openURL:@"mgj://foo/bar"];
```

使用url-block的方案的确可以组建间的解耦，但是还是存在其它明显的问题，比如：

1、需要在内存中维护url-block的表，组件多了可能会有内存问题

2、url的参数传递受到限制，只能传递常规的字符串参数，无法传递非常规参数，如UIImage、NSData等类型

3、没有区分本地调用和远程调用的情况，尤其是远程调用，会因为url参数受限，导致一些功能受限

4、组件本身依赖了中间件，且分散注册使的耦合较多

5、需要列出每个组件中有什么URL可供调用，并且传参格式不明确，需要有一个类似后端API文档去制定url与对应的params

------

####  [BeeHive](https://github.com/search?q=BeeHive)

`BeeHive`是用于阿里团队开发的`App`模块化编程的框架实现方案，吸收了`Spring`框架`Service`的理念来实现模块间的`API`耦合。采用是protocol-service方式。

定义protocol

```
@protocol HomeServiceProtocol <NSObject, BHServiceProtocol>

- (void)registerViewController:(UIViewController *)vc title:(NSString *)title iconName:(NSString *)iconName;

@end
```

注册service

```
-(void)modInit:(BHContext *)context
{
    //注册模块的接口服务
    [[BeeHive shareInstance] registerService:@protocol(HomeServiceProtocol)
    																 service:[BHUserTrackViewController class]];
}
```

调用service

```
#import "BHService.h"

id< HomeServiceProtocol > homeVc = [[BeeHive shareInstance] createService:@protocol(HomeServiceProtocol)];
```

