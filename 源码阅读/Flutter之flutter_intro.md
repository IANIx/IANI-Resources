# flutter_intro - A Guide Flutter Package

官方介绍：

> A better way for new feature introduction and step-by-step users guide for your Flutter project.

Github: https://github.com/tal-tech/flutter_intro

## 目录结构

lib
├── delay_rendered_widget.dart
├── flutter_intro.dart
├── intro_status.dart
├── step_widget_builder.dart
├── step_widget_params.dart
└── throttling.dart

## 源码解析

### Intro

Intro是main class，使用方式为实例化`Intro`对象时，传入`widgetBuilder`和`stepCount`，再从`Intro.keys`中获取`GlobalKey`并将其添加到需要添加指南页面的Widget中，最后执行`start`方法，参数是当前的`BuildContext`。

`stepCount`是引导页数

`widgetBuilder`是生成指南页面的方法，并返回`Widget`作为指南页面

```dart
final Intro intro = Intro(
   stepCount: 4,
   widgetBuilder: widgetBuilder,
 );

 Container(
   key: intro.keys[0],
 );

 Text(
   'need focus widget',
   key: intro.keys[1],
 );

 intro.start(context);
```

###### 参数解析：

***Public***

`maskColor`：引导背景颜色 default is [Color.fromRGBO(0, 0, 0, .6)]

```dart
final Color maskColor;
```

`stepCount`： 一共有多少步骤

```dart
  final int stepCount;
```

 `widgetBuilder`： 生成指南页面内容的方法，将在出现指南页面时由`Intro`内部调用。并将通过`StepWidgetParams`在当前页面上传递一些参数。

```dart
  final Widget Function(StepWidgetParams params) widgetBuilder;
```

  `padding`：内边距 default is [EdgeInsets.all(8)]

```dart
  final EdgeInsets padding;
```

  `borderRadius`：圆角 default is [BorderRadius.all(Radius.circular(4))]

```dart
  final BorderRadiusGeometry borderRadius;
```

***Private***

`_widgetWidth` `_widgetHeight`：widget宽高

`_currentStepIndex`：当前分步索引

`_stepWidget`：当前的分步widget

`_configMap`：配置表

`_lastScreenSize`：最后屏幕尺寸大小

```dart
bool _removed = false;
double _widgetWidth;
double _widgetHeight;
Offset _widgetOffset;
OverlayEntry _overlayEntry;
int _currentStepIndex = 0;
Widget _stepWidget;
List<Map> _configMap = [];
List<GlobalKey> _globalKeys = [];
Duration _animationDuration;
Size _lastScreenSize;
final _th = _Throttling(duration: Duration(milliseconds: 500));
```



###### 方法解析：

***Public***

`start`: 开始展示引导页

```dart
void start(BuildContext context) {
    ...
    _createStepWidget(context);
    _showOverlay(
      context,
      _globalKeys[_currentStepIndex],
    );
  }
```

除了一些初始化配置操作外，主要核心有两个：`_createStepWidget`去生成分步指引的widget和`_showOverlay`去显示指引。

***Private***

`_createStepWidget`：生成分步widget

```dart
void _createStepWidget(BuildContext context) {
    _getWidgetInfo(_globalKeys[_currentStepIndex]);
    Size screenSize = MediaQuery.of(context).size;
    Size widgetSize = Size(_widgetWidth, _widgetHeight);

    _stepWidget = widgetBuilder(StepWidgetParams(
      screenSize: screenSize,
      size: widgetSize,
      onNext: _currentStepIndex == stepCount - 1
          ? null
          : () {
              _onNext(context);
            },
      onPrev: _currentStepIndex == 0
          ? null
          : () {
              _onPrev(context);
            },
      offset: _widgetOffset,
      currentStepIndex: _currentStepIndex,
      stepCount: stepCount,
      onFinish: _onFinish,
    ));
  }
```

这个方法中主要操作有两点：

1. `_getWidgetInfo`根据当前的分步索引获取`widgetInfo`。
2. 配置`StepWidgetParams`通过`widgetBuilder`去生成widget，并赋给`_stepWidget`。

`_showOverlay`：展示引导

```dart
void _showOverlay(
    BuildContext context,
    GlobalKey globalKey,
  ) {
    _overlayEntry = new OverlayEntry(
      builder: (BuildContext context) {
        Size screenSize = MediaQuery.of(context).size;

        if (screenSize.width != _lastScreenSize.width &&
            screenSize.height != _lastScreenSize.height) {
          _lastScreenSize = screenSize;
          _th.throttle(() {
            _createStepWidget(context);
            _overlayEntry.markNeedsBuild();
          });
        }

        return _DelayRenderedWidget(
          removed: _removed,
          childPersist: true,
          duration: _animationDuration,
          child: Material(
            color: Colors.transparent,
            child: Stack(
              children: [
                ColorFiltered(
                  colorFilter: ColorFilter.mode(
                    maskColor,
                    BlendMode.srcOut,
                  ),
                  child: Stack(
                    children: [
                      _maskBuilder(
                        backgroundBlendMode: BlendMode.dstOut,
                        left: 0,
                        top: 0,
                        right: 0,
                        bottom: 0,
                      ),
                      _maskBuilder(
                        width: _widgetWidth,
                        height: _widgetHeight,
                        left: _widgetOffset.dx,
                        top: _widgetOffset.dy,
                        borderRadiusGeometry: _configMap[_currentStepIndex]
                                ['borderRadius'] ??
                            borderRadius,
                      ),
                    ],
                  ),
                ),
                _DelayRenderedWidget(
                  duration: noAnimation
                      ? Duration(milliseconds: 0)
                      : Duration(milliseconds: 200),
                  child: _stepWidget,
                ),
              ],
            ),
          ),
        );
      },
    );
    Overlay.of(context).insert(_overlayEntry);
  }
```

在此方法中，首先生成一个蒙层控件OverlayEntry，再展示蒙层`Overlay.of(context).insert(_overlayEntry)`。

蒙层中主要用到的一个自定义widget：`_DelayRenderedWidget`，主要用于动画展示，核心是AnimatedOpacity包装子widget。

`_getWidgetInfo`：获取当前widget信息

```dart
RenderBox renderBox = globalKey.currentContext.findRenderObject();
```

主要还是通过globalKey去拿去renderBox，最后计算结果得出。

## 总结

总体来说还是一个比较简单的引导组件，调用方式也很简单，通过globalKey去绑定widget，根据currentIndex绘制相应的组件，最后展示蒙层。不过在使用上来说，过多的animal导致引导切换有些不太流畅，要求比较高的情况下最好取消animal。