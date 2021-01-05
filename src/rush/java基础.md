### Java 基础

- 面向对象特性：封装，多态（动态绑定，向上转型），继承
- 泛型，类型擦除
- 反射，原理，优缺点
- `static`，`final` 关键字
- `String`，`StringBuffer`，`StringBuilder`底层区别
- BIO、NIO、AIO
- `Object` 类的方法
- 自动拆箱和自动装箱

### Synchronized

from:[javaDoop](https://javadoop.com/post/Threads-And-Locks-md)

- 简要介绍

  - > java中的**每个对象都关联了一个监视器**，线程可以对其进行加锁和解锁操作。在同一时间，只有一个线程可以拿到对象上的监视器锁。如果其他线程在锁被占用期间试图去获取锁，那么将会被阻塞直到成功获取到锁。同时，监视器锁可以重入，也就是说如果线程 t 拿到了锁，那么线程 t 可以在解锁之前重复获取锁；每次解锁操作会反转一次加锁产生的效果。
    >
    > synchronized 有以下两种使用方式：
    >
    > 1. synchronized 代码块。synchronized(object) 在对某个对象上执行加锁时，会尝试在该对象的监视器上进行加锁操作，只有成功获取锁之后，线程才会继续往下执行。线程获取到了监视器锁后，将继续执行 synchronized 代码块中的代码，如果代码块执行完成，或者抛出了异常，线程将会自动对该对象上的监视器执行解锁操作。
    > 2. synchronized 作用于方法，称为同步方法。同步方法被调用时，会自动执行加锁操作，只有加锁成功，方法体才会得到执行。如果被 synchronized 修饰的方法是实例方法，那么**这个实例的监视器**会被锁定。如果是 static 方法，线程会锁住相应的 **Class 对象的监视器**。方法体执行完成或者异常退出后，会自动执行解锁操作。

  ​                  

  - 小知识点：**对 Class 对象加锁、对对象加锁，它们之间不构成同步**。synchronized 作用于静态方法时是对 **Class 对象**加锁，作用于实例方法时是对实例加锁。


---------------------------

### 面向对象特性

#### 封装

> 百科：隐藏对象的属性和实现细节，仅对外公开接口，控制在程序中属性的读取和修改的访问级别。

​	封装在工程上，最大的意义就是作功能/能力的隔离，使得相邻层之间的调用不需要关注内部实现逻辑，只需要按照接口的定义传入参数即可，此处可引申出：

- *面向接口开发*
- 部分对应设计模式的原则中的
  - *开闭原则*（扩展底层方法的实现，来在不修改上层代码的情况下扩展上层调用方的获取信息能力）
  - *依赖倒置原则*（面向接口开发的原因）：高层模块不应该依赖低层模块，两者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。
  - tbd（待复习[设计模式](http://c.biancheng.net/view/1330.html)）

#### 多态

>引用Charlie Calverts对多态的描述——多态性是允许你将父对象设置成为一个或更多的他的[子对象](https://baike.baidu.com/item/子对象/11001276)相等的技术，赋值之后，父对象就可以根据当前赋值给它的子对象的特性以不同的方式运作

##### 动态绑定

> 百科定义：
>
> 动态绑定（后期绑定）是指：在程序运行过程中，根据具体的实例对象才能具体确定是哪个方法。
>
> 动态绑定是多态性得以实现的重要因素，它通过方法表来实现：每个类被加载到虚拟机时，在方法区保存元数据，其中，包括一个叫做方法表（methodtable）的东西，表中记录了这个类定义的方法的指针，每个表项指向一个具体的方法代码。如果这个类重写了父类中的某个方法，则对应表项指向新的代码实现处。从父类继承来的方法位于子类定义的方法的前面。
>
> 我们假设Fatherft=newSon();ft.say();Son继承自Father，重写了say()。
>
> ### 编译
>
> 我们知道，向上转型时，用父类引用执行子类对象，并可以用父类引用调用子类中重写了的同名方法。但是不能调用子类中新增的方法。
>
> 在代码的编译阶段，编译器通过声明对象的类型（即引用本身的类型）在方法区中该类型的方法表中查找匹配的方法（最佳匹配法：参数类型最接近的被调用），如果有则编译通过。（这里是根据声明的对象类型来查找的，所以此处是查找Father类的方法表，而Father类方法表中是没有子类新增的方法的，所以不能调用。）
>
> 编译阶段是确保方法的存在性，保证程序能顺利、安全运行。 [2] 
>
> ### 运行
>
> ft.say()调用的是Son中的say()，这里就是动态绑定机制的真正体现。
>
> 编译阶段在声明对象类型的方法表中查找方法，只是为了安全地通过编译（也为了检验方法是否是存在的）。而在实际运行这条语句时，在执行Fatherft=newSon();这一句时创建了一个Son实例对象，然后在ft.say()调用方法时，JVM会把刚才的son对象压入操作数栈，用它来进行调用。而用实例对象进行方法调用的过程就是动态绑定：根据实例对象所属的类型去查找它的方法表，找到匹配的方法进行调用。我们知道，子类中如果重写了父类的方法，则方法表中同名表项会指向子类的方法代码；若无重写，则按照父类中的方法表顺序保存在子类方法表中。故此：动态绑定根据对象的类型的方法表查找方法是一定会匹配（因为编译时在父类方法表中以及查找并匹配成功了，说明方法是存在的。这也解释了为何向上转型时父类引用不能调用子类新增的方法：在父类方法表中必须先对这个方法的存在性进行检验，如果在运行时才检验就容易出危险——可能子类中也没有这个方法）。 [2] 
>
> ### 区分
>
> 程序在JVM运行过程中，会把类的类型信息、static属性和方法、final常量等元数据加载到方法区，这些在类被加载时就已经知道，不需对象的创建就能访问的，就是静态绑定的内容；需要等对象创建出来，使用时根据堆中的实例对象的类型才进行取用的就是动态绑定的内容。 [2]

​	

动态绑定最粗浅的解释：

```````````````JAVA

class Human{
    public void eat(){
        System.out.println("I'm full");
    }
}

class Fat extends Human{
    public void eat(){
        System.out.println("gain fat");
    }
    
public static void main(String[] args) {
        Human human = new Fat();
        human.eat();
    }
```````````````

- fat继承了Human并重写了Human的eat方法，因此控制台输出：**gain fat**
  - 但human**对象**不能调用子类中在父类不存在的方法属性。
  - todo：学习jvm相关绑定部分

##### 向上转型(**upcasting**) &&向下转型(**downcasting**)

- 把子类对象直接赋给父类引用叫upcasting向上转型，向上转型不用强制转换。-
  - 例子：上述code中的human = new Fat()

- 把指向子类对象的父类引用赋给子类引用叫向下转型(downcasting)，要强制转换。

  - 例子：

    ```java
    	human =  new Human();
        human.eat();//正常输出
        ((Fat)human).eat();//抛出ClassCastException
    
        Human human = new Human();
        human.eat();
        Fat fat = new Fat();
        human = fat;
        human.eat();
        ((Fat) human).go();//向下转型并强制转型后，可以调用子类中的方法
        fat.eat();
    ```

#### 继承

- 接口和抽象类
  - 接口：java8中接口已经可以通过*default*修饰方法，来实现方法并提供给子类默认使用。但接口中
  - 抽象类：
- 继承抽象类后是否能正常调用内部静态类？
- 

----------------------------

### 关键字

#### static

##### 静态内部类是共享的吗？

- 静态内部类的**非静态变量**在每个父对象中都有一个独立副本，生命周期内是**隔离的**。
- 静态内部类的**静态变量**是共享的。

public class StaticInnerClassTest {

```java
public void init(){
    InnerStaticTest.setI(10);
}

public void print(){
    System.out.println(InnerStaticTest.getI());
}

@Data
private static class InnerStaticTest{
    private static int i ;

    public static int getI() {
        return i;
    }

    public static void setI(int i) {
        InnerStaticTest.i = i;
    }
}

public static void main(String[] args) {
    StaticInnerClassTest test1 = new StaticInnerClassTest();
    test1.init();
    test1.print();
    StaticInnerClassTest test2 = new StaticInnerClassTest();
    test2.print();
}
```
console输出：10 10 