# Dart语法

## **混入mixins**（关键字 `with`）

`mixins`是要通过非继承的方式来复用类中的代码, 要注意的是**mixins不是多继承**。

`mixins`要用到的关键字 `with`

这里举个例子，有一个类A，A中有一个方法a()，还有一个方法B，也想使用a()方法，而且不能用继承，那么这时候就需要用到mixins,类A就是mixins类(混入类)，类B就是要被mixins的类，对应的Dart代码如下:

```dart
class A {

String name = 'name is 呵呵哒';

void testa() {

  print('testa');
  }
}

class B with A {}

/// 调用
B b = new B();

print(b.name);
b.testa();


/// 输出
flutter: name is 呵呵哒
flutter: testa
```

将类A `mixins` 到 B，B可以使用A的属性和方法，B就具备了A的功能

1. `mixins`的对象是类

2. `mixins`绝不是继承，也不是接口，而是一种全新的特性

3. 可以`mixins`多个类

4. `mixins`的使用需要满足一定条件

混合使用范例：

```dart
class A {

void a(){

print("a");
}
}


class A1 {

void a(){

print("a1");
}
}


class B with A,A1{
}


class B1 with A1,A{
}


class B2 with A,A1{

void a(){

print("b2");
}
}


class C {

void a(){

print("a1");
}
}


class B3 extends C with A,A1{
}


class B4 extends C with A1,A{
}


class B5 extends C with A,A1{

void a(){

print("b5");
}
}

/// 调用
B b = new B();

B1 b1 = new B1();

B2 b2 = new B2();

B3 b3 = new B3();

B4 b4 = new B4();

B5 b5 = new B5();


b.a();
b1.a();
b2.a();
b3.a();
b4.a();
b5.a();

/// 输出结果
flutter: a1
flutter: a
flutter: b2
flutter: a1
flutter: a
flutter: b5
```



## **继承**（关键字 `extends`）

Flutter中的继承和Java中的继承是一样的：

1. Flutter中的继承是单继承
2. 构造函数不能继承
3. 子类重写超类的方法，要用@override
4. 子类调用超类的方法，要用super

Flutter中的继承也有和Java不一样的地方：

1. Flutter中的子类可以访问父类中的所有变量和方法，因为Flutter中没有公有、私有的区别

```dart
class Television {
  void turnOn() {
    _illuminateDisplay();
  }
  
  void _illuminateDisplay(){
  }
}


class SmartTelevision extends Television {
  void turnOn() {
    super.turnOn();
    _bootNetworkInterface();
  }
  
  void _bootNetworkInterface(){
  }
}
```



## **接口实现**（关键字 `implements`）

Flutter是没有interface的，但是Flutter中的每个类都是一个隐式的接口，这个接口包含类里的所有成员变量，以及定义的方法。

如果有一个类 A,你想让类B拥有A的API，但又不想拥有A里的实现，那么你就应该把A当做接口，类B implements 类A.

所以在Flutter中:

1. class 就是 interface
2. 当class被当做interface用时，class中的方法就是接口的方法，需要在子类里重新实现，在子类实现的时候要加@override
3. 当class被当做interface用时，class中的成员变量也需要在子类里重新实现。在成员变量前加@override
4. 实现接口可以有多个

```dart
class Television {
  void turnOn() {
    _illuminateDisplay();
  }
  
  void _illuminateDisplay(){
  }
}


class Carton {
  String cartonName = "carton";


  void playCarton(){


  }
}


class Movie{
  void playMovie(){


  }
}


class Update{
  void updateApp(){


  }
}


class Charge{


  void chargeVip(){


  }
}


class SmartTelevision extends Television with Update,Charge implements Carton,Movie {
  @override
  String cartonName="SmartTelevision carton";


  void turnOn() {
    super.turnOn();
    _bootNetworkInterface();
    updateApp();
    chargeVip();
  }
  
  void _bootNetworkInterface(){
  }


  @override
  void playCarton() {
    // TODO: implement playCarton
  }


  @override
  void playMovie() {
    // TODO: implement playMovie
  }

}
```

