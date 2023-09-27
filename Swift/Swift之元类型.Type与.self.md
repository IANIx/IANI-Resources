# 理解 Swift 中的元类型：.Type 与 .self



### 先明确swift中对“任意”类型的几个概念

- Any :可以表示任何类型，包括基础数据类型、枚举类型、结构体、函数类型等；
- AnyObject: 可以表示任何类类型的对象实例，所有类都隐式地遵循 AnyObject；
- AnyClass: 表示类的元类型，是AnyObject.Type的别名：定义typealias AnyClass = AnyObject.Type；

> 我们可以说AnyObject是Any的子集，Any和AnyObject都是Swift 的不确定的类型。通过 `AnyObject.Type` 这种方式所得到是一个元类型 (Meta)，元类型就是类型的类型。

# .Type 与 .self

 Swift 中的元类型用 .Type 表示。比如 Int.Type 就是 Int 的元类型。
 类型与值有着不同的形式，就像 Int 与 5 的关系。元类型也是类似，.Type 是类型，类型的 .self 是元类型的值。

```swift
let intMetatype: Int.Type = Int.self
```

可能大家平时对元类型使用的比较少，加上这两个形式有一些接近，一个元类型只有一个对应的值，所以使用的时候常常写错：

```swift
 types.append(Int.Type)
 types.append(Int.self)
```

如果分清了 Int.Type 是类型的名称，不是值就不会再弄错了。因为你肯定不会这么写：

```swift
 numbers.append(Int)
```

当我们访问静态变量的时候其实也是通过元类型的访问的，只是 Xcode 帮我们省略了 .self。下面两个写法是等价的。如果可以不引起歧义，我想没人会愿意多写一个 self。

```swift
Int.max
Int.self.max
```

# type(of:) vs .self

前面提到通过 `type(of:)` 和 `.self`都可以获得元类型的值。那么这两种方式的区别是什么呢？

```swift
let instanceMetaType: String.Type = type(of: "string")
let staicMetaType: String.Type = String.self
```

`.self` 取到的是静态的元类型，声明的时候是什么类型就是什么类型。`type(of:)` 取的是运行时候的元类型，也就是这个实例 的类型。

```swift
let myNum: Any = 1 
type(of: myNum) // Int.type
```

# Protocol

很多人对 Protocol 的元类型容易理解错。Protocol 自身不是一个类型，只有当一个对象实现了 protocol 后才有了类型对象。所以 Protocol.self 不等于 Protocol.Type。如果你写下面的代码会得到一个错误：

```swift
protocol MyProtocol { }
let metatype: MyProtocol.Type = MyProtocol.self
```

正确的理解是 MyProtocol.Type 也是一个有效的元类型，那么就需要是一个可承载的类型的元类型。所以改成这样就可以了：

```swift
struct MyType: MyProtocol { }
let metatype: MyProtocol.Type = MyType.self 
```

那么 Protocol.self 是什么类型呢？为了满足你的好奇心苹果为你造了一个类型：

```swift
let protMetatype: MyProtocol.Protocol = MyProtocol.self
```



# Reference


[ANYCLASS，元类型和 .SELF](https://swifter.tips/self-anyclass/)