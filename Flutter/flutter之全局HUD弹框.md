---
title: Flutter之全局HUD弹框
date: 2020-09-09 16:36:06
tags: [flutter]
---

## 前言

做过原生APP的小伙伴们都知道，在原生中想要随时随地弹出一个loading或者toast弹框是很方便的，毕竟视图层次是很清晰的，直接获取当前window在window上加载自己的hud视图就可以了。但是在flutter中，首先想要拿到window的context及其不方便，其次因为 flutter 应用是**响应式/声明式**的，不太适合native的**命令式**方案。

首先先梳理下我们想要达到的效果：

- 调用简单便捷：如iOS中的`[SVProgressHUD show]` 就可以在任何地方弹出loading。
- 减少代码冗余和耦合：不需要嵌套某个`widget`去展示，也不要传什么`context`，比如我在业务层也可以去弹出loading。

## HUD实体类

根据以上需求，我们先建立一个实体类`pregress_hud.dart`，并创建class `JQZHUD`，由于要在全局引用，所以要以单例模式去实现所需方法。

```dart
class JQZHUD {

  static JQZHUD _instance = JQZHUD._internal();

  factory JQZHUD() => _instance;



  JQZHUD._internal() {

​    if (null == _instance) {

​      _instance = new JQZHUD();

​    } 

  }

}
```

接下来就是`showLoading`的实现了，其实在当前视图弹出一个新视图常规做法就是`showDialog`，本身`showDialog`的做法就是Navigator去push一个新页面，所以在`showLoading`中为了一些定制化效果我们可以用`showGeneralDialog`，`showDialog`本身也是对`showGeneralDialog`的二次封装。

```dart
static showLoading() {

​    showGeneralDialog(

​      context: _context,

​      barrierDismissible: true,

​      barrierColor: Colors.black.withOpacity(.5),

​      barrierLabel: '',

​      transitionDuration: Duration(milliseconds: 300),

​      pageBuilder: (context, animation, secondaryAnimation) {

​        return Center(

​          child: CircularProgressIndicator(),

​        );

​      },

​    );

  }
```

此时问题又来了，这也是`showLoading`中的重点问题。那就是上下文`context`，前面已经说了首先loading调用时可能不在page页面，而在某个很小的小组件内，此时想要获取当前page的`context`是非常复杂的。另外在业务和UI层分离的情况下，UI层只关心某个请求的结果，至于这个请求的异常弹框逻辑处理一般在业务层，此时业务层是没有办法拿到`context`的。那就还有一种办法，在app主程序入口main函数中我们返回了`MaterialApp`，既然`JQZHUD`是单例，可以在APP启动时拿到当前APP主窗口的`context`并保存下来，后续一直用该`context`去跳转不就可以了吗？？

所以我们继续在`JQZHUD`中增加一个私有`context`并添加一个`init`方法，并将`context`通过此方法传递进来。

```dart
 static BuildContext _context;

 static void init({@required BuildContext context}) {

​    _context = context;

  }
```

接下来在`main.dart`中调用：

```dart
return MaterialApp(

​	....

​      home: MyHomePage(),

​    );

class MyHomePage extends StatelessWidget with WidgetsBindingObserver {

  @override

  Widget build(BuildContext context) {

​    JQZHUD.init(context: context);

​    ScreenUtil.init(context, width: 375, height: 812, allowFontScaling: false);

​    if (JQZDebugUtil.isNativeMode) {

​      return Scaffold(

​        body: Container(),

​      );

​    } else {

​      return JQZEntrancePage();

​    }

  }

}
```



如此就可以正常弹出一个loading页面，那loading有弹出就有隐藏，本来`showGeneralDialog`就是一个router新页面，那返回就直接用

`Navigator.of(context).pop();` 就可以了，但是Navigator pop也是需要传递上下文啊.... 而之前我们保存的_context是app主窗口的，所以我们需要pop的context的这个新的弹框页面的，好在系统已经给我们返回了这个弹框页面的context( `pageBuilder: (context, animation, secondaryAnimation) {}`)。

接下来的事情就好办了，再加一个私有的hudContext， 并用它去pop。

```dart
  static BuildContext _hudContext;

 pageBuilder: (context, animation, secondaryAnimation) {

​        _hudContext = context;

​        return Center(

​          child: CircularProgressIndicator(),

​        );

​      },

static hidddenLoading() {

​    if (Navigator.canPop(_hudContext)) {

​      Navigator.of(_hudContext).pop();

​    }

  }
```



至此，整个hud的初级骨架已完成。

## Usage

hud类完成后，现在我们可以在各种地方去使用它。

```dart
    JQZHUD.showLoading();
```

```dart
    JQZHUD.hidddenLoading();
```

