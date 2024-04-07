#### 概览
`String`被声明为`final`，因此它不可被继承。

内部使用`char`数组(`value`)存储数据，该数组被声明为`final` [final&static](final&static.md)，这意味着`value`数组初始化之后就不能再引用其他数组。并且`String`内部**没有改变**`value`数组的方法，因此可以保证`String`不可变。
``` java
public final class String  
        implements java.io.Serializable, Comparable<String>, CharSequence {  
    /**  
     * The value is used for character storage.     */    private final char value[];  
}
```
#### 不可变的好处
##### 可以缓存`hash`值
因为String的hash经常被使用，例如String用做HashMap的key。不可变性可以使得hash值也不可变，因此只需要进行一次计算。
##### `String Pool`的需要（[字符串常量池](../JVM/内存区域.md#字符串常量池)）
如果一个`String`对象已经被创建过了，那么就会从`String Pool`中取得引用。只有`String`是不可变的，才可能使用`String Pool`。
![](../../img/Pasted%20image%2020240126202643.png)

##### 安全性
`String`经常作为参数，`String`不可变性可以保证参数不可变。例如在作为网络连接参数的情况下，如果`String`是可变的，那么在网络连接过程中，如果`String`在连接建立后被改变，则会影响到现有的连接。
示例如下：
``` java
public class NetworkConnection {
    private String hostname;
    private int port;

    public NetworkConnection(String hostname, int port) {
        this.hostname = hostname;
        this.port = port;
    }

    public void connect() {
        // 连接到 hostname:port
    }
}

public class Main {
    public static void main(String[] args) {
        String host = "example.com";
        int port = 80;

        NetworkConnection connection = new NetworkConnection(host, port);
        connection.connect();

        // 假设 String 是可变的
        host = "malicious.com"; // 如果可以改变，这将影响到现有的 connection 对象

        // 但实际上，String 是不可变的，所以 connection 的 hostname 保持不变
    }
}

```
##### 线程安全
`String`不可变性天生具备线程安全，可以在多个线程中安全地使用。

#### `String`,`StringBuffer` and `StringBuilder`
##### 可变性
- `String`不可变
- `StringBuffer`和`StringBuilder`可变
##### 线程安全
- `String`不可变，因此是线程安全的
- `StringBuilder`不是线程安全的
- `StringBuffer`是线程安全的，内部使用synchronized进行同步

#### `String.intern()`
**如果字符串常量池中已经存在一个等于该字符串的对象，intern()方法会返回这个已存在的对象的引用**
**如果池中没有等于该字符串的对象，intern()方法会将该字符串添加到池中，并返回新添加的字符串对象**
使用`String.intern()`可以保证相同内容的字符串变量引用同一的内存对象。
示例如下：
``` java
String s1 = new String("aaa");  
String s2 = new String("aaa");  
System.out.println(s1 == s2);           // false  
/*
	intern()方法首先把s1引用的对象放到String Pool中，然后返回这个对应的引用
*/
String s3 = s1.intern();  
System.out.println(s1.intern() == s3);  // true
```
如果使用"abc"这种双引号的形式创建字符串实例，会自动将新建的对象放入`String Pool`中。
``` java
String s4 = "bbb";  
String s5 = "bbb";  
System.out.println(s4 == s5);  // true
```
`new String("a")`确保了在堆上创建了一个新的`String`对象`s1`，当出现"a"这类的字面值的时候，会将a直接加入到字符串常量池中。
s1.intern()没有重新赋值，所以s1还是堆里面的对象
s2 = "a"，获取的是池中的引用对象

字符串"aa"是通过字符串对象的连接操作动态生成的，这意味着在编译时，它不会被加入到字符串常量池中。**所以**，s3.intern()后会把s3的引用放入字符串常量池，此时字符串常量池中的引用是s3
``` java
String s1 = new String("a");
s1.intern();
String s2 = "a";
System.out.println(s1 == s2);  // false

String s3 = new String("a") + new String("a");
s3.intern();
String s4 = "aa";
System.out.println(s3 == s4);  // true
```