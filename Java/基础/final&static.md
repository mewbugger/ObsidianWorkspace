#### final
##### 数据
声明数据位常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。
- 对于基本类型，final使数值不变
- 对于引用类型，final使引用不变，也就不能引用其他对象，但是被引用的对象本身是可以改变的
对于上述第二点，代码如下：
``` java
class Person {  
    String name;  
  
    Person(String name) {  
        this.name = name;  
    }  
}
final Person person = new Person("Alice");
person = new Person("Bob"); // 编译错误：不能改变final变量的引用
person.name = "Bob"; // 正确：改变了person对象的内部状态
```
##### 方法
声明方法不能**被子类重写**。
`private`方法隐式地被指定为`final`，如果在子类中定义的方法和基类中的一个`private`方法**签名相同**，此时子类的方法**不是重写基类方法**，而是在子类中**定义了一个新的方法**。
##### 类
声明类不允许被继承
#### static
##### 静态变量
- 静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它；静态变量在内存中只存在一份
- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。
##### 静态方法
静态方法在类加载（[类加载机制](../JVM/类加载机制.md)）的时候就存在了，它不依赖与任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。
静态方法只能访问所属类的静态字段和静态方法，方法中不能有`this`和`super`关键字。
##### 静态语句块
静态语句块在类初始化时**运行一次**
``` java
public class A {  
    static {  
        System.out.println("123");  
    }  
  
    public static void main(String[] args) {  
        A a1 = new A();  
        A a2 = new A();  
    }  
}

// 会直接输出123
```
##### 静态内部类
非静态内部类依赖于外部类的实例，而静态内部类不需要
示例如下：
``` java
public class OuterClass {  
    class InnerClass {  
    }  
  
    static class StaticInnerClass {  
    }  
  
    public static void main(String[] args) {  
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context  
        OuterClass outerClass = new OuterClass();  
        InnerClass innerClass = outerClass.new InnerClass();  
        StaticInnerClass staticInnerClass = new StaticInnerClass();  
    }  
}
```
静态内部类不能访问外部类的非静态变量和方法
##### 静态导包
在使用静态变量和方法时不用再指明ClassName，从而简化代码，但可读性大大降低

``` java
import static com.xxx.ClassName.*
```
##### 初始化顺序
[3.类的初始化](../JVM/类加载机制.md#3.类的初始化)