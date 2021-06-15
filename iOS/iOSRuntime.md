# iOS Runtime机制浅析

Objective-C 是一门动态语言，它会将一些工作放在代码运行时才处理而并非编译时。也就是说，有很多类和成员变量在我们编译的时是不知道的，而在运行时，我们所编写的代码会转换成完整的确定的代码运行。

因此，编译器是不够的，我们还需要一个运行时系统(Runtime system)来处理编译后的代码。

Runtime 基本是用 C 和汇编写的，由此可见苹果为了动态系统的高效而做出的努力。苹果和 GNU 各自维护一个开源的 Runtime 版本，这两个版本之间都在努力保持一致。

## Runtime 的作用

Objc 在三种层面上与 Runtime 系统进行交互：

1. 通过 Objective-C 源代码
2. 通过 Foundation 框架的 NSObject 类定义的方法
3. 通过对 Runtime 库函数的直接调用

### Objective-C 源代码

多数情况我们只需要编写 OC 代码即可，Runtime 系统自动在幕后搞定一切，还记得简介中如果我们调用方法，编译器会将 OC 代码转换成运行时代码，在运行时确定数据结构和函数。

### 通过 Foundation 框架的 NSObject 类定义的方法

Cocoa 程序中绝大部分类都是 NSObject 类的子类，所以都继承了 NSObject 的行为。(NSProxy 类时个例外，它是个抽象超类)

一些情况下，NSObject 类仅仅定义了完成某件事情的模板，并没有提供所需要的代码。例如 `-description` 方法，该方法返回类内容的字符串表示，该方法主要用来调试程序。NSObject 类并不知道子类的内容，所以它只是返回类的名字和对象的地址，NSObject 的子类可以重新实现。

还有一些 NSObject 的方法可以从 Runtime 系统中获取信息，允许对象进行自我检查。例如：

- `-class`方法返回对象的类；
- `-isKindOfClass:` 和 `-isMemberOfClass:` 方法检查对象是否存在于指定的类的继承体系中(是否是其子类或者父类或者当前类的成员变量)；
- `-respondsToSelector:` 检查对象能否响应指定的消息；
- `-conformsToProtocol:`检查对象是否实现了指定协议类的方法；
- `-methodForSelector:` 返回指定方法实现的地址。

### 通过对 Runtime 库函数的直接调用

Runtime 系统是具有公共接口的动态共享库。头文件存放于/usr/include/objc目录下，这意味着我们使用时只需要引入`objc/Runtime.h`头文件即可。

许多函数可以让你使用纯 C 代码来实现 Objc 中同样的功能。除非是写一些 Objc 与其他语言的桥接或是底层的 debug 工作，你在写 Objc 代码时一般不会用到这些 C 语言函数。

## Runtime的术语

要想全面了解 Runtime 机制，我们必须先了解 Runtime 的一些术语，他们都对应着数据结构。

### SEL

它是`selector`在 Objc 中的表示(Swift 中是 Selector 类)。selector 是方法选择器，其实作用就和名字一样，日常生活中，我们通过人名辨别谁是谁，注意 Objc 在相同的类中不会有命名相同的两个方法。selector 对方法名进行包装，以便找到对应的方法实现。它的数据结构是：

```c
typedef struct objc_selector *SEL;
```

它是个映射到方法的 C 字符串，你可以通过 Objc 编译器器命令`@selector()` 或者 Runtime 系统的 `sel_registerName` 函数来获取一个 `SEL` 类型的方法选择器。

### id

id 是一个参数类型，它是指向某个类的实例的指针。定义如下：

```c
typedef struct objc_object *id;
struct objc_object { Class isa; };
```

以上定义，看到 `objc_object` 结构体包含一个 isa 指针，根据 isa 指针就可以找到对象所属的类。

> 注意：
> isa 指针在代码运行时并不总指向实例对象所属的类型，所以不能依靠它来确定类型，要想确定类型还是需要用对象的 `-class` 方法。

PS:KVO 的实现机理就是将被观察对象的 isa 指针指向一个中间类而不是真实类型，详见:[KVO章节](http://lizhaoloveit.com/2014/05/11/KVO/)。

### Class

```
typedef struct objc_class *Class;
```

`Class` 其实是指向 `objc_class` 结构体的指针。`objc_class` 的数据结构如下：

```c
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;   //类的isa指针，指向其所属的元类（meta）

#if !__OBJC2__
    Class super_class  //父类指针                       OBJC2_UNAVAILABLE;
    const char *name   //类名                           OBJC2_UNAVAILABLE;
    long version  //是类的版本信息                       OBJC2_UNAVAILABLE;
    long info  //是类的详情                             OBJC2_UNAVAILABLE;
    long instance_size  //该类的实例对象的大小            OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars  //成员变量列表         OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists  //方法列表    OBJC2_UNAVAILABLE;
    struct objc_cache *cache  //缓存                    OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols  //协议列表     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```

从 `objc_class` 可以看到，一个运行时类中关联了它的父类指针、类名、成员变量、方法、缓存以及附属的协议。

Class 也有一个 isa 指针，指向其所属的元类（meta）。

·super_class：指向其超类。

·name：是类名。

·version：是类的版本信息。

·info：是类的详情。

·instance_size：是该类的实例对象的大小。

·ivars：指向该类的成员变量列表。

·methodLists：指向该类的实例方法列表，它将方法选择器和方法实现地址联系起来。methodLists 是指向 ·objc_method_list 指针的指针，也就是说可以动态修改 *methodLists 的值来添加成员方法，这也是 Category 实现的原理，同样解释了 Category 不能添加属性的原因。

·cache：Runtime 系统会把被调用的方法存到 cache 中（理论上讲一个方法如果被调用，那么它有可能今后还会被调用），下次查找的时候效率更高。

·protocols：指向该类的协议列表。

其中 `objc_ivar_list` 和 `objc_method_list` 分别是成员变量列表和方法列表：

```c
// 成员变量列表
struct objc_ivar_list {
    int ivar_count                                           OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_ivar ivar_list[1]                            OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;

// 方法列表
struct objc_method_list {
    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;

    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}
```

由此可见，我们可以动态修改 `*methodList` 的值来添加成员方法，这也是 Category 实现的原理，同样解释了 Category 不能添加属性的原因。这里可以参考下美团技术团队的文章：[深入理解 Objective-C: Category](http://tech.meituan.com/DiveIntoCategory.html)。

`objc_ivar_list` 结构体用来存储成员变量的列表，而 `objc_ivar` 则是存储了单个成员变量的信息；同理，`objc_method_list` 结构体存储着方法数组的列表，而单个方法的信息则由 `objc_method` 结构体存储。

值得注意的时，`objc_class` 中也有一个 isa 指针，这说明 Objc 类本身也是一个对象。为了处理类和对象的关系，Runtime 库创建了一种叫做 Meta Class(元类) 的东西，类对象所属的类就叫做元类。Meta Class 表述了类对象本身所具备的元数据。

我们所熟悉的类方法，就源自于 Meta Class。我们可以理解为类方法就是类对象的实例方法。每个类仅有一个类对象，而每个类对象仅有一个与之相关的元类。

当你发出一个类似 `[NSObject alloc](类方法)` 的消息时，实际上，这个消息被发送给了一个类对象(Class Object)，这个类对象必须是一个元类的实例，而这个元类同时也是一个根元类(Root Meta Class)的实例。所有元类的 isa 指针最终都指向根元类。

所以当 `[NSObject alloc]` 这条消息发送给类对象的时候，运行时代码 `objc_msgSend()` 会去它元类中查找能够响应消息的方法实现，如果找到了，就会对这个类对象执行方法调用。

![img](https://upload-images.jianshu.io/upload_images/1330553-81f64a11ad20c764.png?imageMogr2/auto-orient/strip%7CimageView2/2)

 

上图实现是 `super_class` 指针，虚线时 `isa` 指针。而根元类的父类是 `NSObject`，`isa`指向了自己。而 `NSObject` 没有父类。

最后 `objc_class` 中还有一个 `objc_cache` ，缓存，它的作用很重要，后面会提到。

### Method

Method 代表类中某个方法的类型

```
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}
```

`objc_method` 存储了方法名，方法类型和方法实现：

- 方法名类型为 `SEL`
- 方法类型 `method_types` 是个 char 指针，存储方法的参数类型和返回值类型
- `method_imp` 指向了方法的实现，本质是一个函数指针

### Ivar

`Ivar` 是表示成员变量的类型。

```
typedef struct objc_ivar *Ivar;

struct objc_ivar {
    char *ivar_name                                          OBJC2_UNAVAILABLE;
    char *ivar_type                                          OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}
```

其中 `ivar_offset` 是基地址偏移字节

### IMP

IMP在objc.h中的定义是：

```
typedef id (*IMP)(id, SEL, ...);
```

它就是一个函数指针，这是由编译器生成的。当你发起一个 ObjC 消息之后，最终它会执行的那段代码，就是由这个函数指针指定的。而 `IMP` 这个函数指针就指向了这个方法的实现。

如果得到了执行某个实例某个方法的入口，我们就可以绕开消息传递阶段，直接执行方法，这在后面 `Cache` 中会提到。

你会发现 `IMP` 指向的方法与 `objc_msgSend` 函数类型相同，参数都包含 `id` 和 `SEL` 类型。每个方法名都对应一个 `SEL` 类型的方法选择器，而每个实例对象中的 `SEL` 对应的方法实现肯定是唯一的，通过一组 `id`和 `SEL` 参数就能确定唯一的方法实现地址。

而一个确定的方法也只有唯一的一组 `id` 和 `SEL` 参数。

### Cache

Cache 定义如下：

```
typedef struct objc_cache *Cache

struct objc_cache {
    unsigned int mask /* total = mask + 1 */                 OBJC2_UNAVAILABLE;
    unsigned int occupied                                    OBJC2_UNAVAILABLE;
    Method buckets[1]                                        OBJC2_UNAVAILABLE;
};
```

Cache 为方法调用的性能进行优化，每当实例对象接收到一个消息时，它不会直接在 isa 指针指向的类的方法列表中遍历查找能够响应的方法，因为每次都要查找效率太低了，而是优先在 Cache 中查找。

Runtime 系统会把被调用的方法存到 Cache 中，如果一个方法被调用，那么它有可能今后还会被调用，下次查找的时候就会效率更高。就像计算机组成原理中 CPU 绕过主存先访问 Cache 一样。

### Property

```
typedef struct objc_property *Property;
typedef struct objc_property *objc_property_t;//这个更常用
```

可以通过`class_copyPropertyList` 和 `protocol_copyPropertyList` 方法获取类和协议中的属性：

```
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
```

> 注意：
> 返回的是属性列表，列表中每个元素都是一个 `objc_property_t` 指针

```
#import <Foundation/Foundation.h>

@interface Person : NSObject

/** 姓名 */
@property (strong, nonatomic) NSString *name;

/** age */
@property (assign, nonatomic) int age;

/** weight */
@property (assign, nonatomic) double weight;

@end
```

以上是一个 Person 类，有3个属性。让我们用上述方法获取类的运行时属性。

```
    unsigned int outCount = 0;

    objc_property_t *properties = class_copyPropertyList([Person class], &outCount);

    NSLog(@"%d", outCount);

    for (NSInteger i = 0; i < outCount; i++) {
        NSString *name = @(property_getName(properties[i]));
        NSString *attributes = @(property_getAttributes(properties[i]));
        NSLog(@"%@--------%@", name, attributes);
    }
```

打印结果如下：

```
2014-11-10 11:27:28.473 test[2321:451525] 3
2014-11-10 11:27:28.473 test[2321:451525] name--------T@"NSString",&,N,V_name
2014-11-10 11:27:28.473 test[2321:451525] age--------Ti,N,V_age
2014-11-10 11:27:28.474 test[2321:451525] weight--------Td,N,V_weight
```

`property_getName` 用来查找属性的名称，返回 c 字符串。`property_getAttributes` 函数挖掘属性的真实名称和 `@encode` 类型，返回 c 字符串。

```
objc_property_t class_getProperty(Class cls, const char *name)
objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
```

`class_getProperty` 和 `protocol_getProperty` 通过给出属性名在类和协议中获得属性的引用。

## 消息

一些 Runtime 术语讲完了，接下来就要说到消息了。体会苹果官方文档中的 messages aren’t bound to method implementations until Runtime。消息直到运行时才会与方法实现进行绑定。

这里要清楚一点，`objc_msgSend` 方法看清来好像返回了数据，其实`objc_msgSend` 从不返回数据，而是你的方法在运行时实现被调用后才会返回数据。下面详细叙述消息发送的步骤(如下图)：

![img](https://upload-images.jianshu.io/upload_images/1330553-87d03a7c0971c730.gif?imageMogr2/auto-orient/strip)

 

1. 首先检测这个 `selector` 是不是要忽略。比如 Mac OS X 开发，有了垃圾回收就不理会 retain，release 这些函数。
2. 检测这个 `selector` 的 target 是不是 `nil`，Objc 允许我们对一个 nil 对象执行任何方法不会 Crash，因为运行时会被忽略掉。
3. 如果上面两步都通过了，那么就开始查找这个类的实现 `IMP`，先从 cache 里查找，如果找到了就运行对应的函数去执行相应的代码。
4. 如果 cache 找不到就找类的方法列表中是否有对应的方法。
5. 如果类的方法列表中找不到就到父类的方法列表中查找，一直找到 NSObject 类为止。
6. 如果还找不到，就要开始进入**动态方法解析**了，后面会提到。

在消息的传递中，编译器会根据情况在 `objc_msgSend` ， `objc_msgSend_stret` ， `objc_msgSendSuper` ， `objc_msgSendSuper_stret` 这四个方法中选择一个调用。如果消息是传递给父类，那么会调用名字带有 Super 的函数，如果消息返回值是数据结构而不是简单值时，会调用名字带有 stret 的函数。

### 方法中的隐藏参数

> 疑问：
> 我们经常用到关键字 `self` ，但是 `self` 是如何获取当前方法的对象呢？

其实，这也是 Runtime 系统的作用，`self` 实在方法运行时被动态传入的。

当 `objc_msgSend` 找到方法对应实现时，它将直接调用该方法实现，并将消息中所有参数都传递给方法实现，同时，它还将传递两个隐藏参数：

- 接受消息的对象(`self` 所指向的内容，当前方法的对象指针)
- 方法选择器(`_cmd` 指向的内容，当前方法的 SEL 指针)

因为在源代码方法的定义中，我们并没有发现这两个参数的声明。它们时在代码被编译时被插入方法实现中的。尽管这些参数没有被明确声明，在源代码中我们仍然可以引用它们。

这两个参数中， `self`更实用。它是在方法实现中访问消息接收者对象的实例变量的途径。

这时我们可能会想到另一个关键字 `super` ，实际上 `super` 关键字接收到消息时，编译器会创建一个 `objc_super` 结构体：

```
struct objc_super { id receiver; Class class; };
```

这个结构体指明了消息应该被传递给特定的父类。 `receiver` 仍然是 `self` 本身，当我们想通过 `[super class]` 获取父类时，编译器其实是将指向 `self` 的 `id` 指针和 `class` 的 SEL 传递给了 `objc_msgSendSuper` 函数。只有在 `NSObject` 类中才能找到 `class` 方法，然后 `class` 方法底层被转换为 `object_getClass()`， 接着底层编译器将代码转换为 `objc_msgSend(objc_super->receiver, @selector(class))`，传入的第一个参数是指向 `self` 的 `id` 指针，与调用 `[self class]` 相同，所以我们得到的永远都是 `self` 的类型。因此你会发现：

```
// 这句话并不能获取父类的类型，只能获取当前类的类型名
NSLog(@"%@", NSStringFromClass([super class]));
```

### 获取方法地址

`NSObject` 类中有一个实例方法：`methodForSelector`，你可以用它来获取某个方法选择器对应的 `IMP` ，举个例子：

```
void (*setter)(id, SEL, BOOL);
int i;

setter = (void (*)(id, SEL, BOOL))[target
    methodForSelector:@selector(setFilled:)];
for ( i = 0 ; i < 1000 ; i++ )
    setter(targetList[i], @selector(setFilled:), YES);
```

当方法被当做函数调用时，两个隐藏参数也必须明确给出，上面的例子调用了1000次函数，你也可以尝试给 `target` 发送1000次 `setFilled:` 消息会花多久。

虽然可以更高效的调用方法，但是这种做法很少用，除非时需要持续大量重复调用某个方法的情况，才会选择使用以免消息发送泛滥。

> 注意：
> `methodForSelector:`方法是由 Runtime 系统提供的，而不是 Objc 自身的特性

------

## 动态方法解析

你可以动态提供一个方法实现。如果我们使用关键字 `@dynamic` 在类的实现文件中修饰一个属性，表明我们会为这个属性动态提供存取方法，编译器不会再默认为我们生成这个属性的 setter 和 getter 方法了，需要我们自己提供。

```
@dynamic propertyName;
```

这时，我们可以通过分别重载 `resolveInstanceMethod:` 和 `resolveClassMethod:` 方法添加实例方法实现和类方法实现。

当 Runtime 系统在 Cache 和类的方法列表(包括父类)中找不到要执行的方法时，Runtime 会调用 `resolveInstanceMethod:` 或 `resolveClassMethod:` 来给我们一次动态添加方法实现的机会。我们需要用 `class_addMethod` 函数完成向特定类添加特定方法实现的操作：

```
void dynamicMethodIMP(id self, SEL _cmd) {
    // implementation ....
}
@implementation MyClass
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically)) {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSEL];
}
@end
```

上面的例子为 `resolveThisMethodDynamically` 方法添加了实现内容，就是 `dynamicMethodIMP` 方法中的代码。其中 `"v@:"` 表示返回值和参数，这个符号表示的含义见：[Type Encoding](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)

> 注意：
> 动态方法解析会在消息转发机制侵入前执行，动态方法解析器将会首先给予提供该方法选择器对应的 `IMP` 的机会。如果你想让该方法选择器被传送到转发机制，就让 `resolveInstanceMethod:` 方法返回 `NO`。

------

## 消息转发

![img](https://upload-images.jianshu.io/upload_images/1330553-400bc5bde4db1725.png?imageMogr2/auto-orient/strip%7CimageView2/2)

 

### 重定向

消息转发机制执行前，Runtime 系统允许我们替换消息的接收者为其他对象。通过 `- (id)forwardingTargetForSelector:(SEL)aSelector` 方法。

```
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(mysteriousMethod:)){
        return alternateObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

如果此方法返回 `nil` 或者 `self`，则会计入消息转发机制(`forwardInvocation:`)，否则将向返回的对象重新发送消息。

### 转发

当动态方法解析不做处理返回 `NO` 时，则会触发消息转发机制。这时 `forwardInvocation:` 方法会被执行，我们可以重写这个方法来自定义我们的转发逻辑：

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([someOtherObject respondsToSelector:
            [anInvocation selector]])
        [anInvocation invokeWithTarget:someOtherObject];
    else
        [super forwardInvocation:anInvocation];
}
```

唯一参数是个 `NSInvocation` 类型的对象，该对象封装了原始的消息和消息的参数。我们可以实现 `forwardInvocation:` 方法来对不能处理的消息做一些处理。也可以将消息转发给其他对象处理，而不抛出错误。

> 注意：参数 `anInvocation 是从哪来的？`
> 在 `forwardInvocation:` 消息发送前，Runtime 系统会向对象发送`methodSignatureForSelector:` 消息，并取到返回的方法签名用于生成 NSInvocation 对象。所以重写 `forwardInvocation:` 的同时也要重写 `methodSignatureForSelector:` 方法，否则会抛异常。

当一个对象由于没有相应的方法实现而无法相应某消息时，运行时系统将通过 `forwardInvocation:` 消息通知该对象。每个对象都继承了 `forwardInvocation:` 方法。但是， `NSObject` 中的方法实现只是简单的调用了 `doesNotRecognizeSelector:`。通过实现自己的 `forwardInvocation:` 方法，我们可以将消息转发给其他对象。

`forwardInvocation:` 方法就是一个不能识别消息的分发中心，将这些不能识别的消息转发给不同的接收对象，或者转发给同一个对象，再或者将消息翻译成另外的消息，亦或者简单的“吃掉”某些消息，因此没有响应也不会报错。这一切都取决于方法的具体实现。

> 注意：
> `forwardInvocation:`方法只有在消息接收对象中无法正常响应消息时才会被调用。所以，如果我们向往一个对象将一个消息转发给其他对象时，要确保这个对象不能有该消息的所对应的方法。否则，`forwardInvocation:`将不可能被调用。

### 转发和多继承

转发和继承相似，可用于为 Objc 编程添加一些多继承的效果。就像下图那样，一个对象把消息转发出去，就好像它把另一个对象中的方法接过来或者“继承”过来一样。

![img](https://upload-images.jianshu.io/upload_images/1330553-c7ef6392ecc9ee9d.gif?imageMogr2/auto-orient/strip)

 

这使得在不同继承体系分支下的两个类可以实现“继承”对方的方法，在上图中 `Warrior` 和 `Diplomat` 没有继承关系，但是 `Warrior` 将 `negotiate` 消息转发给了 `Diplomat` 后，就好似 `Diplomat` 是 `Warrior` 的超类一样。

消息转发弥补了 Objc 不支持多继承的性质，也避免了因为多继承导致单个类变得臃肿复杂。

### 转发与继承

虽然转发可以实现继承的功能，但是 `NSObject` 还是必须表面上很严谨，像 `respondsToSelector:` 和 `isKindOfClass:` 这类方法只会考虑继承体系，不会考虑转发链。

如果上图中的 `Warrior` 对象被问到是否能响应 `negotiate`消息：

```
if ( [aWarrior respondsToSelector:@selector(negotiate)] )
    ...
```

回答当然是 `NO`， 尽管它能接受 `negotiate` 消息而不报错，因为它靠转发消息给 `Diplomat` 类响应消息。

如果你就是想要让别人以为 `Warrior` 继承到了 `Diplomat` 的 `negotiate` 方法，你得重新实现 `respondsToSelector:` 和 `isKindOfClass:` 来加入你的转发算法：

```
- (BOOL)respondsToSelector:(SEL)aSelector
{
    if ( [super respondsToSelector:aSelector] )
        return YES;
    else {
        /* Here, test whether the aSelector message can     *
         * be forwarded to another object and whether that  *
         * object can respond to it. Return YES if it can.  */
    }
    return NO;
}
```

除了 `respondsToSelector:` 和 `isKindOfClass:` 之外，`instancesRespondToSelector:` 中也应该写一份转发算法。如果使用了协议，`conformsToProtocol:` 同样也要加入到这一行列中。

如果一个对象想要转发它接受的任何远程消息，它得给出一个方法标签来返回准确的方法描述 `methodSignatureForSelector:`，这个方法会最终响应被转发的消息。从而生成一个确定的 `NSInvocation` 对象描述消息和消息参数。这个方法最终响应被转发的消息。它需要像下面这样实现：

```
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];
    if (!signature) {
       signature = [surrogate methodSignatureForSelector:selector];
    }
    return signature;
}
```

------

## 健壮的实例变量(Non Fragile ivars)

在 Runtime 的现行版本中，最大的特点就是健壮的实例变量了。当一个类被编译时，实例变量的内存布局就形成了，它表明访问类的实例变量的位置。实例变量一次根据自己所占空间而产生位移：

![img](https://upload-images.jianshu.io/upload_images/1330553-bcbc243a281ef8d9.png?imageMogr2/auto-orient/strip%7CimageView2/2)

 

上图左是 `NSObject` 类的实例变量布局。右边是我们写的类的布局。这样子有一个很大的缺陷，就是缺乏拓展性。哪天苹果更新了 `NSObject` 类的话，就会出现问题：

![img](https://upload-images.jianshu.io/upload_images/1330553-33263710847f6d86.png?imageMogr2/auto-orient/strip%7CimageView2/2)

 

我们自定义的类的区域和父类的区域重叠了。只有苹果将父类改为以前的布局才能拯救我们，但这样导致它们不能再拓展它们的框架了，因为成员变量布局被固定住了。在脆弱的实例变量(Fragile ivar)环境下，需要我们重新编译继承自 Apple 的类来恢复兼容。

在健壮的实例变量下，编译器生成的实例变量布局跟以前一样，但是当 Runtime 系统检测到与父类有部分重叠时它会调整你新添加的实例变量的位移，那样你再子类中新添加的成员变量就被保护起来了。

> 注意：
> 在健壮的实例变量下，不要使用 `siof(SomeClass)`，而是用 `class_getInstanceSize([SomeClass class])` 代替；也不要使用 `offsetof(SomeClass, SomeIvar)`，而要使用 `ivar_getOffset(class_getInstanceVariable([SomeClass class], "SomeIvar"))` 来代替。



