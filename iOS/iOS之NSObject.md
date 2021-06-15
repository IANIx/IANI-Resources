# 深入探究NSObject对象

## NSObject对象有哪几种类型？

三种：分别是实例对象(instance)、类对象(class)、元类对象(meta-class)

#### 实例对象(instance)

就是我们通常的类的实例化的对象。

```cpp
//类的实例
typedef struct objc_object *id;
/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
这里有个isa 指针，指向的就是 当前实例的类对象
```

```objective-c
NSObject *object1 = [[NSObject alloc] init];
NSObject *object2 = [[NSObject alloc] init];
```

object1, object2都是NSObject的`instance`对象(**实例**对象)

它们是不同的两个对象,分别占据着两块不同的内存

`instace`对象在内存中存储的信息包括

- `isa`指针
- 其它`成员变量`

#### 类对象(class)

其实类也是一个对象，每个类在内存中有且只有一个class对象。

```cpp
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY; //
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;//父类
    const char *name                                         OBJC2_UNAVAILABLE;//类名
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;//实例大小
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;//成员变量列表
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;//方法列表
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;//方法缓存
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;//协议列表
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

这里也是有个isa 指针，其实这个isa 指向的就是 元类
```

```objc
NSObject *object1 = [[NSObject alloc] init];
NSObject *object2 = [[NSObject alloc] init];
Class objectClass1 = [object1 class];
Class objectClass2 = [object2 class];
Class objectClass3 = [NSObject class];
Class objectClass4 = object_getClass(object1); // Runtime
Class objectClass5 = object_getClass(object2); // Runtime
```

objectClass1 ~ objectClass5都是NSObject的`class`对象(类对象),class方法返回的一直是`class`对象

它们都是同一个对象.每个类在内存中有且只有一个`class`对象

`class`对象在内存中存储的信息主要包括

- `isa`指针
- `superclass`指针
- 类的**属性**信息(`@property`),类的**对象方法**信息(`instance method`)
- 类的**协议**信息(`protocol`),类的**成员变量**信息(`ivar`)

#### 元类对象(meta-class)

类对象的isa指向的类。每个类在内存中有且只有一个元类对象。

```kotlin
Class objectMetaClass = object_getClass([NSObject class]);
```

- objectMetaClass是`NSObject`的`meta-class`对象
- 每个类在内存中有且只有一个`meta-class`对象
- `meta-class`对象和`class`对象的内存结构是一样的,但是用途不一样,在内存中存储的信息主要包括
  - `isa`指针
  - `superclass`指针
  - 类的类方法信息(`class method`)
- 查看`Class`是否为`meta-class`

```objc
BOOL result = class_isMetaClass([NSObject class]); // Runtime
```

## isa

`instance`对象的`isa`指向`class`
当调用`对象方法`时,通过`instance`的`isa`找到`class`,最后找到`对象方法`的实现进行调用
`class`的`isa`指向`meta-class`
当调用`类方法`时,通过`class`的`isa`找到`meta-class`,最后找到`类方法`的实现进行调用
`meta-class`的`isa`指向基类的`meta-class`

## superclass

- `class`的`superclass`指向`meta-class`，如果没有父类,`superclass`指针为`nil``

- ``meta-class`的`superclass`指向**基类**的`meta-class`

- 基类的`meta-class`的`superclass`指向基类的`class`

## 最后

`intance`调用对象方法的轨迹

- `isa`找到`class`,方法不存在,就通过`superclass`找父类,一直找,直到所有的父类找完都没有这个对象方法的实现时,再经过runtime的动态方法解析和消息转发,如果都没有,就会报`unrecognized selector sent to instance 0xxxxxxxxx`

`class`调用类方法的轨迹

- `isa`找`meta-class`,方法不存在,就通过`superclass`找父类
- 这种有一种特殊情况,找到`meta-class`的都没有找到时,因为`meta-class`的`superclass`指向基类的`meta-class`的,所以会调用基类的`meta-class`的类方法.这里不用奇怪明明调用的是类方法,最后却调用了基类的对象方法.



