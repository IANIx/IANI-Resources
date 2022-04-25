## Block(闭包)

### OC

```objective-c
/// 自定义block规则
typedef <#returnType#>(^<#name#>)(<#arguments#>);
 
/************定义block************/
/// 无需传参以及返参的Block
typedef void(^TestBlock)(void);

/// 或者可以使用系统的dispatch_block_t
typedef void (^dispatch_block_t)(void);
/// 需要传参的Blck
typedef void(^TestBlock)(NSString*);
typedef void(^TestBlock)(NSString*, NSInteger);
/// 需要返参的Block
typedef NSInteger(^TestBlock)(void);
typedef NSString*(^TestBlock)(void);
/// 传参并且返参的Block
typedef NSString*(^TestBlock)(NSString*, NSInteger);

/************创建block************/
@property (nonatomic, copy) TestBlock testBlock;
@property (nonatomic, copy) dispatch_block_t testBlock;

/************实现block************/
/// 无参
self.testBlock = ^{
    <#code#>
};
/// 入参
self.testBlock = ^(NSString *) {
    /// do something
};
self.testBlock = ^(NSString *, NSInteger) {
    /// do something
};
/// 返参
self.testBlock = ^NSString *{
  	/// do something
    /// return string
};
/// 传参并且返参
self.testBlock = ^NSString *(NSString *, NSInteger) {
    /// do something
    /// return string
 };

/************调用block************/
/// 无参
if (self.testBlock != nil) {
    self.testBlock();
}
/// 入参
if (self.testBlock != nil) {
    self.testBlock(@"test");
}
if (self.testBlock != nil) {
    self.testBlock(@"test", 11);
}
/// 返参
if (self.testBlock != nil) {
    NSString * result = self.testBlock();
}
if (self.testBlock != nil) {
    NSString * result = self.testBlock(@"test", 11);
}

/************局部block************/
/// 局部变量
void (^Block)(void) = ^{
};
/// 与方法搭配使用
+ (void)open:(NSString *)name completion:(void (^)(BOOL))completion;
```

### Dart 

```dart
/// 自定义回调规则
typedef <#name#> = <#returnType#> Function(<#arguments#>);

/************定义回调************/
typedef TestCallBack = void Function();
typedef TestCallBack = void Function(String result);
typedef TestCallBack = void Function(dynamic result, String e);
typedef TestCallBack = String Function();
typedef TestCallBack = void Function(int result);

/************创建回调************/
TestCallBack callBack;

/************实现回调************/
TestCallBack callBack = () {
		/// do something
 };
TestCallBack callBack = (String result) {
		/// do something
 };
TestCallBack callBack = (dynamic result, String e) {
		/// do something
 };
TestCallBack callBack = () {
		/// do something
  	/// return string
 };
TestCallBack callBack = (int result) {
		/// do something
	  /// return string
 };

/************调用回调************/
if (callBack != null) {
    callBack();
}
if (callBack != null) {
    callBack("test");
}
if (callBack != null) {
    callBack(item, "test");
}
if (callBack != null) {
    String str = callBack();
}
if (callBack != null) {
     String str = callBack(111);
}
```

### Swift

```swift
/// 自定义闭包规则
{(parameters) -> return type in
   statements
}

/// 最简单的无入参出参
let test = { print("test") }
test()

/// 包含入参出参
let sort = {(s1: String, s2: String) -> Bool in
            return s1 > s2
       }
let result = sort("a":"b")

/// 类型定义
typealias testBlock = (_ str: String) -> Void
typealias testBlock = () -> Void

/// 尾随闭包
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // 函数体部分
}

// 以下是不使用尾随闭包进行函数调用
someFunctionThatTakesAClosure(closure: {
    // 闭包主体部分
})

// 以下是使用尾随闭包进行函数调用
// 如果闭包表达式是函数或方法的唯一参数，则当你使用尾随闭包时，你甚至可以把 () 省略掉：
someFunctionThatTakesAClosure() {
    // 闭包主体部分
}
```


