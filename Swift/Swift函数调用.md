## Swift的函数调用

与 Objective-C 不同，Swift 的函数调用存在三种方式，分别是：基于 Objective-C 的消息机制、基于虚函数表的访问、以及直接地址调用。

### Objective-C 的消息机制

什么情况下 Swift 的函数调用是借助 Objective-C 的消息机制。如果方法通过 `@objc` `dynamic` 修饰，那么在编译后将通过 objc_msgSend 的来调用函数。

```swift
class MyTestClass :NSObject { 
    @objc dynamic func helloWorld() { 
        print("call helloWorld() in MyTestClass") 
    } 
} 
 
let myTest = MyTestClass.init() 
myTest.helloWorld() 
```

编译后其对应的汇编为:

```cpp
0x1042b8824 <+120>: bl     0x1042b9578               ; type metadata accessor for SwiftDemo.MyTestClass at <compiler-generated> 
0x1042b8828 <+124>: mov    x20, x0 
0x1042b882c <+128>: bl     0x1042b8998               ; SwiftDemo.MyTestClass.__allocating_init() -> SwiftDemo.MyTestClass at ViewController.swift:22 
0x1042b8830 <+132>: stur   x0, [x29, #-0x30] 
0x1042b8834 <+136>: adrp   x8, 13 
0x1042b8838 <+140>: ldr    x9, [x8, #0x320] 
0x1042b883c <+144>: stur   x0, [x29, #-0x58] 
0x1042b8840 <+148>: mov    x1, x9 
0x1042b8844 <+152>: str    x8, [sp, #0x60] 
0x1042b8848 <+156>: bl     0x1042bce88               ; symbol stub for: objc_msgSend 
0x1042b884c <+160>: mov    w11, #0x1 
0x1042b8850 <+164>: mov    x0, x11 
0x1042b8854 <+168>: ldur   x1, [x29, #-0x48] 
0x1042b8858 <+172>: bl     0x1042bcd5c               ; symbol stub for: 
```

从上面的汇编代码中我们很容易看出调用了地址为0x1042bce88的objc_msgSend 函数。

### 虚函数表的访问

虚函数表的访问也是动态调用的一种形式，只不过是通过访问虚函数表的方式进行调用。

假设还是上述代码，我们将 @objc dynamic 去掉之后，并且不再继承自 NSObject。

```swift
class MyTestClass { 
    func helloWorld() { 
        print("call helloWorld() in MyTestClass") 
    } 
} 
 
let myTest = MyTestClass.init() 
myTest.helloWorld() 
```

汇编代码变成了下面这样👇

```cpp
0x1026207ec <+120>: bl     0x102621548               ; type metadata accessor for SwiftDemo.MyTestClass at <compiler-generated> 
0x1026207f0 <+124>: mov    x20, x0 
0x1026207f4 <+128>: bl     0x102620984               ; SwiftDemo.MyTestClass.__allocating_init() -> SwiftDemo.MyTestClass at ViewController.swift:22 
0x1026207f8 <+132>: stur   x0, [x29, #-0x30] 
0x1026207fc <+136>: ldr    x8, [x0] 
0x102620800 <+140>: adrp   x9, 8 
0x102620804 <+144>: ldr    x9, [x9, #0x40] 
0x102620808 <+148>: ldr    x10, [x9] 
0x10262080c <+152>: and    x8, x8, x10 
0x102620810 <+156>: ldr    x8, [x8, #0x50] 
0x102620814 <+160>: mov    x20, x0 
0x102620818 <+164>: stur   x0, [x29, #-0x58] 
0x10262081c <+168>: str    x9, [sp, #0x60] 
0x102620820 <+172>: blr    x8 
0x102620824 <+176>: mov    w11, #0x1 
0x102620828 <+180>: mov    x0, x11 
```

从上面汇编代码可以看出，经过编译后最终是通过 blr 指令调用了 x8 寄存器中存储的函数。

### 直接地址调用

假设还是上述代码，我们再将 Build Setting 中Swift Compiler - Code Generaation -> Optimization Level 修改为 Optimize for Size[-Osize]，汇编代码变成了下面这样👇

```cpp
0x1048c2114 <+40>:  bl     0x1048c24b8               ; type metadata accessor for SwiftDemo.MyTestClass at <compiler-generated> 
0x1048c2118 <+44>:  add    x1, sp, #0x10             ; =0x10  
0x1048c211c <+48>:  bl     0x1048c5174               ; symbol stub for: swift_initStackObject 
0x1048c2120 <+52>:  bl     0x1048c2388               ; SwiftDemo.MyTestClass.helloWorld() -> () at ViewController.swift:23 
0x1048c2124 <+56>:  adr    x0, #0xc70c               ; demangling cache variable for type metadata for Swift._ContiguousArrayStorage<Any> 
```

这是大家就会发现bl 指令后跟着的是一个常量地址，并且是 SwiftDemo.MyTestClass.helloWorld() 的函数地址。

## 最后

| 声明位置  | @Objc | dynamic | 调用方式     |
| --------- | ----- | ------- | ------------ |
| Struct    | 否    | 否      | 直接调用     |
| Class     | 否    | 否      | V-Table 调用 |
| Extension | 否    | 否      | 直接调用     |
| Extension | 是    | 否      | objc_msgSend |
| Class     | 是    | 是      | objc_msgSend |

