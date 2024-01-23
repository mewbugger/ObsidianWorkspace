#### 概览
``` java
public final native Class<?> getClass()  
  
public native int hashCode()  
  
public boolean equals(Object obj)  
  
protected native Object clone() throws CloneNotSupportedException  
  
public String toString()  
  
public final native void notify()  
  
public final native void notifyAll()  
  
public final native void wait(long timeout) throws InterruptedException  
  
public final void wait(long timeout, int nanos) throws InterruptedException  
  
public final void wait() throws InterruptedException  
  
protected void finalize() throws Throwable {}
```
#### equals()
##### equals()与==
- 对于基本类型，`==` 判断两个值是否相等，基本类型没有`equals()`方法。
- 对于引用类型，`== `判断两个变量是否引用同一个对象，而`equals()`判断引用的对象是否等价
示例代码如下：
``` java
Integer x = new Integer(1);  
Integer y = new Integer(1);  
System.out.println(x.equals(y)); // true  
System.out.println(x == y);      // false
```
`equals()`不能用于判断基本数据类型的变量，只能用来判断两个对象是否相等。`equals()`方法存在于`Object`类中，而`Object`类时所有类的直接或间接父类，因此所有的类都有`equals()`方法。
`Object`类的`equals()`方法：
``` java
public boolean equals(Object obj) {
     return (this == obj);
}
```
`equals()`方法存在两种使用情况：
- 类没有重写`equals()`方法：通过`equals()`比较该类的两个对象时，等价于通过`==` 比较这两个对象，使用的默认是`Object`类`equals()`方法。
- 类重写了`equals()`方法：一般我们都重写`equals()`方法来比较两个对象中的属性是否相等；若属性相等，则返回`true`
`String` 中的 `equals` 方法是被重写过的，因为 `Object` 的 `equals` 方法是比较的对象的内存地址，而 `String` 的 `equals` 方法比较的是对象的值。
#### clone()
##### cloneable
`clone()`是`Object`的`protected`方法，它不是`public`，一个类不显式去重写`clone()`,其他类就不能直接去调用该类实例的`clone()`方法。（protected修饰符的特性：[访问权限](继承.md#访问权限)）
示例如下：
``` java
package package1;

public class ClassA {
    // ClassA inherits the protected clone() method from Object
}

package package2;

import package1.ClassA;

public class ClassB {
    public void method() {
        ClassA a = new ClassA();
        ClassA aClone = a.clone();  // 编译错误：clone() 在 ClassA 中是 protected 访问权限
    }
}

// 重写后的clone()方法
package package1;
/*
	如果一个类没有实现Cloneable接口又调用了clone()方法，就会抛出 CloneNotSupportedException
*/
public class ClassA implements Cloneable {
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

```
##### 浅拷贝
浅拷贝会在对上创建一个新的对象（区别与引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，浅拷贝会复制内部对象的引用地址，也就是说拷贝对象和原对象公用同一个内部对象。
代码示例如下：
``` java
class ShallowCopyExample implements Cloneable {
    int a;
    Integer b;

    public ShallowCopyExample(int a, Integer b) {
        this.a = a;
        this.b = b;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        ShallowCopyExample obj1 = new ShallowCopyExample(1, new Integer(2));
        ShallowCopyExample obj2 = (ShallowCopyExample) obj1.clone();

        System.out.println("Before change:");
        System.out.println(obj1.b); // 输出 2
        System.out.println(obj2.b); // 输出 2

        obj1.b = 3;

        System.out.println("After change:");
        System.out.println(obj1.b); // 输出 3
        System.out.println(obj2.b); // 也输出 3
    }
}

```
##### 深拷贝
深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。拷贝对象和原始对象的引用类型引用不同对象。
代码示例如下:
``` java
class DeepCopyExample implements Cloneable {
    int a;
    Integer b;

    public DeepCopyExample(int a, Integer b) {
        this.a = a;
        this.b = b;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        DeepCopyExample copy = (DeepCopyExample) super.clone();
        copy.b = new Integer(b);
        return copy;
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        DeepCopyExample obj1 = new DeepCopyExample(1, new Integer(2));
        DeepCopyExample obj2 = (DeepCopyExample) obj1.clone();

        System.out.println("Before change:");
        System.out.println(obj1.b); // 输出 2
        System.out.println(obj2.b); // 输出 2

        obj1.b = 3;

        System.out.println("After change:");
        System.out.println(obj1.b); // 输出 3
        System.out.println(obj2.b); // 仍然输出 2
    }
}

```
![](../../img/Pasted%20image%2020240123213823.png)