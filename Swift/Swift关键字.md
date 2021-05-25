Swift关键字

常见的关键字有以下4种：

- 与声明有关的关键字：class、deinit、enum、extension、func、import、init、let、protocol、static、struct、subscript、typealias和var。
- 与语句有关的关键字：break、case、continue、default、do、else、fallthrough、if、guard、in、for、return、switch、where和while。
- 表达式和类型关键字：as、dynamicType、is、new、super、self、Self、Type、**COLUMN**、**FILE**、**FUNCTION**和**LINE**。
- 在特定上下文中使用的关键字：associativity、didSet、get、infix、inout、left、mutating、none、nonmutating、operator、override、postfix、precedence、prefix、rightset、unowned、unowned(safe)、unowned(unsafe)、weak和willSet。

## 声明关键字一览图

swift常见的声明关键字整理如下

![img](https:////upload-images.jianshu.io/upload_images/767049-419f301eb2146dc4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

swift声明关键字

## 声明关键字详解

### 1、class

在swift中，我们使用`class`关键字去声明一个类或者类方法。

```swift
class Person: NSObject {

    
    /// add `class` key word before function, this function become a class function
    class func work(){
        print("everyone need work!")
    }
}
```

这样我们就声明了一个Person类。

### 2、let

swift里有`let`关键字声明一个常量，及我们不可以对他进行修改。（注意：**我们用let修饰的常量是一个类, 我们可以对其所在的属性进行修改**）

```swift
class iOSer: Person{
    let name: String = "ningjianwen"
    var age: Int = 30
    var height: Float = 170
}

let ITWork: iOSer = iOSer()
ITWork.age = 25
print("老子希望永远25岁")
```

在iOSer类中`let`声明的name不可修改，`var`声明的**age**&**height**可以修改。同时`let`关键字声明的ITWork实例不可变，但是内部的`var`关键字声明的刷新是可以修改的。

### 3、var

swift中`var`修饰的变量是一个可变的变量，可以对她的值进行修改。
 **注意：我们不会用var去引用一个类, 也没有必要。**

```swift
func iOSerClassFunction(){
        let ITWork: iOSer = iOSer()
        ITWork.age = 25
        print("老子希望永远\(ITWork.age)岁")
        
        let iOS1 = ITWork
        iOS1.age = 18
        print("iOS1 age =\(iOS1.age)")
        print("ITWork age = \(ITWork.age)")
    }
    /** 打印结果
    老子希望永远25岁
    iOS1 age =18
    ITWork age = 18
    */
```

从结果可以看出对**iOS1**的修改同样影响了**ITWork**，说明两个对象指向同一块内存空间。

### 4、struct

在Swift中, 我们使用`struct`关键字去声明结构体，Swift中的结构体并不复杂，与C语言的结构体相比，除了成员变量，还多了成员方法。使得它更加接近于一个类。可以把**struct**看作是类的一个轻量化实现。

```swift
struct Student {
    var name: String
    var age: Int
    
    func introduce(){
        print("我叫：\(name),今年\(age)岁")
    }
}
```

从上方的代码可以看出，结构体和类拥有相同的功能，可以定义属性和方法。但是二者的内存管理方式有所不同，class属于引用类型，而struct属于值类型。
 swift中的结构体语法上与C语言或者OC类似，只不过Swift中的结构体，在定义成员变量时一定要注明类型。

### 5、enum

在Swift中, 我们使用**enum**关键字去声明枚举。枚举是一种常见的数据类型，他的主要功能就是将某一种有固定数量可能性的变量的值，以一组命名过的常数来指代。比如正常情况下方向有四种可能，东，南，西，北。我们就可以声明一组常量来指代方向的四种可能。使用枚举可以防止用户使用无效值，同时该变量可以使代码更加清晰。

```swift
enum Orientation: Int{
    case East
    case South
    case West
    case North
        
        /**
         或者
         enum Orientation:Int{
         case East,South,West,North
         }
         */
}

print(Orientation.East.rawValue)
/**
 输出结果 0
 
 */
```

注意：**我们在定义枚举时，一定要指定类型，否则在使用时就会报错。枚举类型的值如果没有赋值，他就按照默认的走，可以赋予我们自己想要的值。**

### 6、final

final关键字可以在**class**、**func**、**var**前修饰，表示不可重写，可以把类或者类中的部分实现保护起来，从而避免子类破坏。

```swift
class Fruit : NSObject {
    //修饰词 final 表示 不可重写 可以将类或者类中的部分实现保护起来,从而避免子类破坏
    final func price(){
        //something price code here
        //...
    }
}

class Apple: Fruit {
//    此处重写`final`修饰的price方法，报错 “Instance method overrides a 'final' instance method”
//    override func price(){
//
//    }
}
```

### 7、override

在Swift中, 如果我们要重写某个方法, 或者某个属性的话, 我们需要在重写的变量前增加一个**override**关键字。

```swift
class Fruit : NSObject {
    
    var sellPrice: Double = 0.0
    var name: String = "fruit"
    func info(){
        print("this fruit name is fruit")
    }
    
    //修饰词 final 表示 不可重写 可以将类或者类中的部分实现保护起来,从而避免子类破坏
    final func price(){
        //something price code here
        //...
    }
}

class Apple: Fruit {
    func eat(){
        print("eat fruit")
    }
    //重写info方法
    override func info() {
        print("this fruit name is \(super.name)")
    }
    //重写name属性
    override var name: String{
        set{
            super.name = newValue
        }
        get{
           return super.name
        }
    }
}
```

### 8、subscript

在swift中，**subscript**关键字表示下标，可以让**class**、**struct**、以及**enum**使用下标访问内部的值。其实就可以快捷方式的设置或者获取对应的属性, 而不需要调用对应的方法去获取或者存储, 比如官网的一个实例:

```swift
struct Matrix {
    let rows: Int, columns: Int
    var grid: [Double]
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        grid = Array(repeating: 0.0, count: rows * columns)
    }
    
    
    func indexIsValid(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
    //实现`subscript`方法
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
}

func matrixTest(){
        var matrix = Matrix(rows: 2, columns: 2)
        matrix[0, 1] = 1.5
        matrix[1, 0] = 3.2
        print("matrix == \(matrix)")
        /**
         打印结果：
         matrix == Matrix(rows: 2, columns: 2, grid: [0.0, 1.5, 3.2, 0.0])
         */
    }
```

### 9、static

在swift中，我们用**static**关键字声明静态变量或者函数，它保证在对应的作用域当中只有一份, 同时也不需要依赖实例化。
 注意：（**用static关键字指定的方法是类方法，他是不能被子类重写的**）

### 10、mutating

**mutating**关键字指的是可变即可修改。用在structure和enumeration中,虽然结构体和枚举可以定义自己的方法，但是默认情况下，实例方法中是不可以修改值类型的属性。为了能够在实例方法中修改属性值，可以在方法定义前添加关键字mutating。

### 11、typealias

使用关键字typealias定义类型别名(typealias就相当于objective-c中的typedef)，就是将类型重命名，看起来更加语义化。说人话就是：**起别名**。

```swift
typealias Width = Float
typealias Height = Float
func rectangularArea(width:Width, height:Height) -> Double {
            return Double(width*height)
        }
```

### 12、lazy

lazy关键修饰的变量, 只有在第一次被调用的时候才会去初始化值（即懒加载）。这个在定义属性经常会用到，为了提高程序的性能，我们把它定义成lazy的，等待真正需要的时候采取加载。

```swift
lazy var titleLabel: UILabel = {
       var lab = UILabel()
       lab.frame = CGRect(x: 50, y: 100, width: 200, height: 20)
       lab.textAlignment = .center
       lab.font = UIFont.systemFont(ofSize: 18)
       lab.textColor = UIColor.blue
       return lab
    }()
```

### 13、init

**init**关键字也表示构造器，初始化方法，在init后面加个”?”号, 表明该构造器可以允许失败。

```swift
class PerSon {
        var name:String
        init?(name : String) {
            if name.isEmpty { return nil }
            self.name = name
        }
    }
```

### 14、required

**required**是用来修饰init方法的，说明该构造方法是必须实现的。

```swift
class Father: NSObject {
    var name: String?
    required init(name: String) {
        self.name = name
    }
}

class Son: Father {
    required init(name: String) {
        super.init(name: name)
        self.name = name
    }
}
```

从上面的代码示例中不难看出，如果子类需要添加异于父类的初始化方法时，必须先要实现父类中使用required修饰符修饰过的初始化方法，并且也要使用required修饰符而不是override。

使用**required**的注意点：

> 1. required修饰符只能用于修饰类初始化方法。
> 2. 当子类含有异于父类的初始化方法时（初始化方法参数类型和数量异于父类），子类必须要实现父类的required初始化方法，并且也要使用required修饰符而不是override。
> 3. 当子类没有初始化方法时，可以不用实现父类的required初始化方法。

### 15、extension

**extension与Objective-C的category有点类似，但是extension比起category来说更加强大和灵活**，它不仅可以扩展某种类型或结构体的方法，同时它还可以与protocol等结合使用，编写出更加灵活和强大的代码。它可以为特定的`class`, `strut`, `enum`或者`protocol`添加新的特性。当你没有权限对源代码进行改造的时候，此时可以通过extension来对类型进行扩展。extension有点类似于OC的类别 -- category，但稍微不同的是category有名字，而extension没有名字。

extension也是swift开发经常用到的，它可以扩展扩展以下几个：

- 1. 定义实例方法和类型方法
- 1. 添加计算型属性和计算静态属性
- 1. 定义下标
- 1. 提供新的构造器
- 1. 定义和使用新的嵌套类型
- 1. 使一个已有类型符合某个接口

```swift
// MARK: - 添加计算属性
extension Double{
    var km: Double { return self * 1_000.0}
    var m: Double { return self}
    var cm: Double { return self / 100.0}
}


// MARK: - 为person 添加方法
extension Person{
    
    func run(){
        print("人有行走的属性")
    }
}

调用方法：
func extensionTest(){
        
        let oneInch = 25.4.km
        print("One inch is\(oneInch) meter")
        
        let njw = Person()
        njw.run()
    }
    /** 打印结果
    One inch is25400.0 meter
    人有行走的属性
    */
```

### 16、convenience

使用`convenience`修饰的构造函数叫做**便利构造函数** 。便利构造函数通常用在对系统的类进行构造函数的扩充时使用。
 便利构造函数的特点如下：

- 1. 便利构造函数通常都是写在extension里面
- 1. 便利函数init前面需要加载convenience
- 1. 在便利构造函数中需要明确的调用self.init()

### 17、deinit

deinit属于**析构函数**，当对象结束其生命周期时（例如对象所在的函数已调用完毕），**系统自动执行析构函数**。和OC中的dealloc 一样的。
 我们通常在deinit函数中进行一些资源释放和通知移除等。
 列举如下：

- 1. 对象销毁
- 1. KVO移除
- 1. 移除通知
- 1. NSTimer销毁

### 18、fallthrough

在swift中，fallthrough的作用是就是在**switch-case**中执行完当前case,继续执行下面的case.

### 19、protocol

在swift中，protocol也是定义协议的，用法和OC类似。

### 20、open

open修饰的对象表示可以被任何人使用，包括override和继承。

### 21、public

在swift中，`public`表示公有访问权限，**类或者类的公有属性或者公有方法可以从文件或者模块的任何地方进行访问。但在其他模块(一个App就是一个模块，一个第三方API, 第三等方框架等都是一个完整的模块)不可以被override和继承，而在本模块内可以被override和继承。**

### 22、internal

在swift中，`internal`表示内部的访问权限。即有着internal访问权限的属性和方法说明在模块内部可以访问，超出模块内部就不可被访问了。**在swift中默认就是internal的访问权限。**

### 23、private

在swift中，**private**表示私有访问权限。**被private修饰的类或者类的属性或方法可以在同一个物理文件中访问**。如果超出该物理文件，那么有着private访问权限的属性和方法就不能被访问。

### 24、fileprivate

在swift中，**fileprivate访问级别所修饰的属性或者方法在当前的Swift源文件里可以访问。**

下面的代码是对`internal`,`private`,`fileprivate`的一个简单展示。

```swift
/// 默认是internal的访问权限，在模块内部可以访问
class ParentClass: NSObject {
    
    ///这个方法在如何地方可以被`override`
    func speak(){
        print("这是一个说话属性，子类可以进行复写")
    }
    
    /// 这个方法是秘密，只有父类拥有，子类不可修改
    private func secret(){
        print("这是一个秘密，只有我自己知道")
    }
    
    /// 这是本类的秘密，出了该类就看不到了
    fileprivate func localSecret(){
        print("这是本类的秘密，出了该类就看不到了")
    }
}

class FirstSon: ParentClass {
    
    /// 长子说话
    override func speak() {
        print("我是长子")
    }
//长子也不能修改老爸的秘密
//    override func secret(){
//
//    }
    
    override func localSecret() {
        print("儿子把家里的秘密说出去了")
    }
}
//方法调用与及结果打印
func parentClassTest(){
        
        let parent: ParentClass = ParentClass()
        parent.speak()
//        parent.secret() //老爸的秘密不能对外
//        parent.localSecret() //家里的秘密也不能对外
        let oldSon: FirstSon = FirstSon()
        oldSon.speak()
//        oldSon.secret() //儿子不能说老爸的秘密
        oldSon.localSecret() //儿子把家里的秘密说出去了
        /** 打印结果
         
         这是一个说话属性，子类可以进行复写
         我是长子
         儿子把家里的秘密说出去了
         */
    }
```

说明：5种修饰符访问权限排序**open > public  > internal > fileprivate > private**.
 从类的安全性上来说，**访问权限越小越好**。

### 25、guard

如果未满足一个或多个条件，则使用保护语句将程序控制转移到作用域外。

```swift
guard condition else {
    statements
}
```

 在guard语句中任何条件的值必须是 Bool 类型或桥接到 Bool 的类型。该条件也可以是optional binding declaration.

从 guard 语句条件中的可选绑定声明分配值的任何常量或变量都可用于 guard 语句的封闭作用域的其余部分。

guard 语句的else是必需的，并且必须使用以下语句之一调用"Nerver"类型的函数或将程序控制权转移到保护语句的封闭范围之外：

- return
- break
- continue
- throw

e.g.：

```swift
func nonguardSubmit() {
    if let name = nameField.text {
        if let address = addressField.text {
            if let phone = phoneField.text {
                sendToServer(name, address: address, phone: phone)
            } else {
                show("no phone to submit")
            }
        } else {
            show("no address to submit")
        }
    } else {
        show("no name to submit")
    }
}
```

为了解决这个问题，我们可以使用guard语句

```swift
func submit() {
    guard let name = nameField.text else {
        show("No name to submit")
        return
    }

    guard let address = addressField.text else {
        show("No address to submit")
        return
    }

    guard let phone = phoneField.text else {
        show("No phone to submit")
        return
    }

    sendToServer(name, address: address, phone: phone)
}
```

作者：iCloudEnd
链接：https://www.jianshu.com/p/bd7f4c8f4946
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

作者：无神
链接：https://www.jianshu.com/p/de24119b7e91
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。