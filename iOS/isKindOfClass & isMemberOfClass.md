# isKindOfClass和isMemberOfClass

首先看苹果的官方解释：

- isKindOfClass:
  Returns a Boolean value that indicates whether the receiver is an instance of given class or an instance of any class that inherits from that class. (required)
  这个方法用来判断一个对象是否是指定类或者某个从该类继承类的实例对象。
- isMemberOfClass:
  Returns a Boolean value that indicates whether the receiver is an instance of a given class. (required)
  这个方法用来判断一个对象是否是指定类的实例对象。

通过一个经典面试题，来加深对`isa` & `继承关系` & `类结构`的理解

```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        //iskindOfClass & isMemberOfClass 类方法调用
        BOOL re1 = [(id)[NSObject class] isKindOfClass:[NSObject class]];  
        BOOL re2 = [(id)[LGPerson class] isKindOfClass:[LGPerson class]];     
        BOOL re3 = [(id)[NSObject class] isMemberOfClass:[NSObject class]];        
        BOOL re4 = [(id)[LGPerson class] isMemberOfClass:[LGPerson class]];     
        NSLog(@" re1 :%hhd\n re2 :%hhd\n re3 :%hhd\n re4 :%hhd\n",re1,re2,re3,re4);
        
        //iskindOfClass & isMemberOfClass 实例方法调用
        BOOL re5 = [(id)[NSObject alloc] isKindOfClass:[NSObject class]];        
        BOOL re6 = [(id)[LGPerson alloc] isKindOfClass:[LGPerson class]]; 
        BOOL re7 = [(id)[NSObject alloc] isMemberOfClass:[NSObject class]];        
        BOOL re8 = [(id)[LGPerson alloc] isMemberOfClass:[LGPerson class]];   
        NSLog(@" re5 :%hhd\n re6 :%hhd\n re7 :%hhd\n re8 :%hhd\n",re5,re6,re7,re8);
    }
    return 0;
}
```

输出结果

```css
re1 :1
re2 :0
re3 :0
re4 :0

re5 :1
re6 :1
re7 :1
re8 :1
```

## 源码解析

#### isKindOfClass 源码解析（实例方法 & 类方法）

```objectivec
//--isKindOfClass---类方法、对象方法
//+ isKindOfClass：第一次比较是 获取类的元类 与 传入类对比，再次之后的对比是获取上次结果的父类 与 传入 类进行对比
+ (BOOL)isKindOfClass:(Class)cls {
    // 获取类的元类 vs 传入类
    // 根元类 vs 传入类
    // 根类 vs 传入类
    // 举例：LGPerson vs 元类 (根元类) (NSObject)
    for (Class tcls = self->ISA(); tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}

//- isKindOfClass：第一次是获取对象类 与 传入类对比，如果不相等，后续对比是继续获取上次 类的父类 与传入类进行对比
- (BOOL)isKindOfClass:(Class)cls {
/*
获取对象的类 vs 传入的类 
父类 vs 传入的类
根类 vs 传入的类
nil vs 传入的类
*/
    for (Class tcls = [self class]; tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}
```

#### isMemberOfClass 源码解析（实例方法 & 类方法）

```php
//类方法
//+ isMemberOfClass : 获取类的元类，与 传入类对比
+ (BOOL)isMemberOfClass:(Class)cls {
    return self->ISA() == cls;
}
//实例方法
//- isMemberOfClass : 获取对象的类，与 传入类对比
- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}
```

源码分析总结：
 isKindOfClass

> 类方法：`元类 --> 根元类 --> 根类 --> nil 与 传入类的对比`
> 实例方法：`对象的类 --> 父类 --> 根类 --> nil 与 传入类的对比`

isMemberOfClass

> 类方法： `类的元类` 与 `传入类` 对比
> 实例方法：`对象的父类` 与 `传入类` 对比

透过上面的源码解析总结，我们分析我们的示例代码

> ret1 = 1 : 传入的cls 为 NSobject， self 指向 NSobject，进入循环
>  第一次循环： tcls 为 NSobject meta ，cls 为 NSobject ；执行判断条件if (tcls == cls) ，不相等；执行 tcls = tcls->superclass  ，此时 tcls 指向 NSobject meta 的父类 ，即 NSObject。进入第二次循环。
>  第二次循环：此时 tcls 为 NSobject，cls 依然是 NSobject，执行判断条件 if (tcls == cls) 相等，return YES。
>  所以 re1 的结果为 1。

> ret2 = 0 : 传入的cls为 NSobject，self指向 Person，进入循环
>  第一次循环：tcls 为 Person meta，cls 为 Person类； 执行判断条件 if (tcls == cls) ，不相等，执行 tcls = tcls->superclass ，此时 tcls 指向 NSobject metal。进入第二次循环。
>  第二次循环： tcls 为 NSobject meta ，cls 为 Person类；不相等，执行 tcls = tcls->superclass ，此时 tcls 指向 NSObject。进入第三次循环。
>  第三次循环： tcls 为 NSobject ，cls 为 Person类；不相等，执行 tcls = tcls->superclass ，此时 tcls 指向 nil。不满足for循环执行条件 tcls。结束循环。
>  所以 re2 的结果为 0。

> ret3 = 0 : 传入的cls 为 NSObject， self 指向 NSObject
>  self->ISA( ) ，self的 isa 指向 NSObject meta ；NSObject meta 与 NSObject 不相等。
>  所以 re3 的结果为 0。

> ret4 = 0 : 传入的cls 为 Person， self 指向 Person
>  self->ISA( ) ，self的 isa 指向 Person meta ；Person meta 与 Person 不相等。
>  所以 re4 的结果为 0。

> ret5 = 1 : 传入的cls 为 NSObject 类，self 指向 NSObject 的 实例对象
>  第一次循环：tcls 指向 NSObject 类，cls 为 NSObject 类，执行判断 if (tcls == cls) ，相等，return YES，结束循环。
>  所以 re5 返回 1。

> ret6 = 1 : 传入的cls 为 Person 类，self 指向 Person 的 实例对象
>  第一次循环：tcls 指向 Person 类，cls 为 Person 类，执行判断 if (tcls == cls) ，相等，return YES，结束循环。
>  所以 re6 返回 1。

> ret7 = 1 : 传入的cls 为 NSObject， self 指向 NSObject 对象
>  [self class] 为 NSObject 类 ；与 cls 相等。
>  所以 re7 的结果为 1。

> ret8 = 1 : 传入的cls 为 Person， self 指向 Person 对象
>  [self class] 为 Person 类 ；与 cls 相等。
>  所以 re8 的结果为 1。

