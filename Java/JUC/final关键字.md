#### 修饰类
当某个类的整体定义为final的时候，表明不能继承该类，即这个类是不能有子类的。
**注意**：final类中的所有方法都隐式为final，因为无法覆盖他们，所以在final类中给任何方法添加final关键字是没有意义的。
##### final修饰的类要如何拓展
**问题**：String是final类型，如果想写一个类（MyString）复用所有String中的方法（由于String这个类是final修饰的，所以并不能让MyString继承String），同时增加一个新的toMyString()的方法，应该如何做。
设计模式中重要的两种关系：
1. 继承/实现
2. 组合
由此可见，当继承/实现不可操作的时候，我们应该考虑组合设计模式
示例如下：
``` java
class MyString{
    private String innerString;
    // ...init & other methods
    // 支持老的方法
    public int length(){
        return innerString.length(); // 通过innerString调用老的方法
    }
    // 添加新方法
    public String toMyString(){
        //...
    }
}
```
#### 修饰方法([方法](../基础/final&static.md#方法))
- private方法是隐式的final
- final方法是可以被重载([重载](../基础/继承.md#重载))的
##### private&final
- `private`关键字：`private`关键字用于限制对成员变量和方法的访问仅限于声明它们的类内部。即，一个类中的方法被声明为`private`，无法从该类的外部直接访问这个方法，包括从该类派生的任何子类。
- `final`关键字：当`final`修饰方法的时候，组织方法被覆盖（`override`）。即，一个类中的方法被声明为final，那么任何试图继承该类的子类无法重写这个方法。
类中所有private方法都隐式地指定为final的，由于无法取用private方法，所以也就不能覆盖它。可以对private方法增添final关键字，但这样做并没有什么好处。
**解析**：
- 由于`private`方法只能在其所属的类内部访问，这自然意味着它们在子类中是不可见的。因此，子类无法直接访问（或“看到”）父类的`private`方法，更不用说覆盖它们了。这与`final`方法的特性相似，`final`方法不能被子类覆盖。因此，可以说`private`方法在**行为上**是隐式`final`的。
- 尽管技术上可以在`private`方法声明中加上`final`关键字，但这并不会改变方法的行为，因为`private`方法本来就不能被覆盖。因此，添加`final`关键字对`private`方法来说是多余的。
#### 修饰参数
Java允许在参数列表中以声明的方式将参数指明为final，这意味你**无法在方法中更改参数引用所指向的对象**。这个特性主要用来向匿名内部类传递数据。
#### 修饰变量
##### **问题**：所有的final修饰的字段都是编译期常量吗？ **答**：否
- **编译期常量**：在编译时就能确定其值的常亮。这些常亮的值不及在编译时是已知的，而且也会被Java编译器内联到任何使用它们的地方。如果要修改该常亮的值，则使用该常亮的类也需要 重新编译，否则它们会继续使用旧值。编译期常量通常是`static final`修饰的
``` java
public class CompileTimeConstants {
    public static final int CONSTANT_NUMBER = 10; // 编译期常量
    public static final String CONSTANT_STRING = "Hello"; // 编译期常量
}
```
- **非编译器常量**：在编译时无法确定其值的常量。可能依赖于运行时才能知道的计算或方法调用结果。虽然这些字段也可以被声明为final，但是它们的值不是在编译时内联而是在运行时确定的。即，如果这些常亮的值发生变化，只有在运行时才能反映出来。
``` java
public class NonCompileTimeConstants {
    public static final int NON_CONSTANT_NUMBER = new Random().nextInt(100); // 非编译期常量
    public static final String NON_CONSTANT_STRING = System.getProperty("user.name"); // 非编译期常量
}
```
##### static final
一个既是static又是final的字段只占据一段不能改变的存储空间，它必须在定义的时候进行赋值，否则编译器将不予通过。
``` java
import java.util.Random;
public class Test {
    static Random r = new Random();
    final int k = r.nextInt(10);
    static final int k2 = r.nextInt(10);
    public static void main(String[] args) {
        Test t1 = new Test();
        System.out.println("k="+t1.k+" k2="+t1.k2);
        Test t2 = new Test();
        System.out.println("k="+t2.k+" k2="+t2.k2);
    }
}
==================================================
输出内容如下：
k=2 k2=7
k=8 k2=7
```
**解析**：根据上面的输出结果可以看出，不同对象的k值是不同的，但是k2的值是相同的。因为static关键字锁修饰的字段并不属于一个对象，而是属于这个类的（[3.类的初始化](../JVM/类加载机制.md#3.类的初始化)）。
##### blank final
Java允许生成空白final，也就是说被神明为final但又没有给定值的字段，但是必须在该字段被 使用之前被赋值，这个给予我们两种选择：
- 在定义处进行赋值（这不是空白final）
- 在构造器中进行赋值，保证了该值在被使用前赋值
构造器中赋值实例
``` java
public class Test {
    final int i1 = 1;
    final int i2;//空白final
    public Test() {
        i2 = 1;
    }
    public Test(int x) {
        this.i2 = x;
    }
}
```
可以看出，通过构造器赋值，更加灵活。但是要**注意**：如果字段由final和static修饰，仅能在声明时赋值或神明后再静态代码块中赋值，因为该字段不属于对象，属于这个类。
#### final域重排序规则
