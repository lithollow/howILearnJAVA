## 一、简介

Java.lang包是Java语言体系中其他所有类库的基础，已经内嵌到Java虚拟机中，而且以对象的形式创建好了，所以，我们在使用Java.lang包时不需要再使用import将其导入，可以直接使用Java.lang包中的所有类以及直接引用某个类中的敞亮、变量和操作方法。



## 二、Object

所有类的根类。

每个Java类都直接或间接地继承自Object类。Object类的实例方法可以被所有Java对象使用。

1. `toString()`:返回字符串（类路径@hash码）；直接打印对象，Java默认调用此方法

2. `hashCode()`:`native`修饰的方法，返回每个对象十进制的hash值

​	hash值是由hash算法根据对象的地址、对象中的字符串、数字等计算出来的

3. `equals(obj)`:用于判断调用equals方法的对象和形参obj所引用的对象是否是同一对象，即指向了内存中的同一块存储单元地址

​	自反、对称、传递、一致性

​	如果a.equals(b)，那么 a 和 b 的 hashCode 相同；反之不一定，因为存在 hash 冲突

```
Person p1 = new Person("zs"，8);
Person p2 = new Person("zs"，8);
// 默认 equals 就是 == 
System.out.println(p1 == p2);		// false
System.out.println(p1.equals(p2));	// false
// --------------------------------------------------------
// 重写equals()和hashCode()
@Override
public int hashCode() {
    return Objects.hash(age, name);
}

@Override
public boolean equals(Object obj) {
    if (this == obj)
        return true;	// 是否是当前对象
    if (obj == null)
        return false;	// 是否为空
    if (getClass() != obj.getClass())
        return false;	// 是否为同一个类
    Person other = (Person) obj;	// 强转
    return age == other.age && Objects.equals(name, other.name);	// 先比较简单的属性，然后比较难的（String）
}
// 这时p1.equals(p2)为true
```

4. `clone()`:native方法，创建复制出当前类对象的一个副本，得到一个复制对象

​	首先分配一个和源对象(调用clone方法的对象)同样大小的内存空间，在这个内存空间中会创建出一个新对象；然后再使用源对象中对应的各个成员，填充新对象的成员，填充完成之后，clone方法会创建返回一个新的相同对象供外部引用

```
// 需要重写才能调用clone()；重写clone()需要implements Cloneable
@Override
protected Object clone() throws CloneNotSupportedException{
    return super.clone();
}
try {
    Person p3 = (Person)p1.clone();	// 需要做类型转换
    System.out.println(p1 == p3);	// false
    System.out.println(p1.equals(p3));	// 重写equals后为true
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
```

5. `wait()`、`wait(long)`、`wait(long, int)`、`notify()`、`notifyAll()`

​	这几个函数体现的是Java的多线程机制，一般是结合synchronize语句使用。

- wait()用于让当前线程失去操作权限，当前线程进入等待序列；
- notify()用于随机通知一个持有对象的锁的线程获取操作权限；
- wait(long) 和wait(long,int)用于设定下一次获取锁的距离当前释放锁的时间间隔；
- notifyAll()用于通知所有持有对象的锁的线程获取操作权限。



## 三、包装类

java针对基本类型提供了它们对应的包装类，八大基本数据类型，对应了八种包装类，以对象的形式来调用。

包装类有了类的特点，使我们可以调用包装类中的方法。

Byte、Short、*Integer*、Long、*Character*、Float、Double、Boolean



java设计包装类的原因

1. 面向对象完整性 - 让基本类型融入对象体系
2. 集合框架支持 - 解决集合不能存储基本类型的问题
3. 功能扩展 - 为基本类型添加工具方法和常量
4. 泛型支持 - 提供类型安全的泛型编程
5. 空值表示 - 允许表示"未知"或"未设置"状态
6. 序列化支持 - 支持对象序列化机制
7. 框架集成 - 便于与各种框架和库集成
8. 性能优化 - 通过缓存机制优化常见值



- 装箱 : 基本类型 ——> 包装类型（或者叫对象类型，引用类型)
- 拆箱 : 包装类型 ——> 基本类型

不建议使用构造函数创建包装类对象 `Integer i = new Integer(1);`

装箱：`Integer i = Integer.valueOf(1);`

拆箱：`int j = i.intValue();`

拆箱同时可以转换成其他类型：字符串转int：`j = Integer.valueOf("100"); j = Integer.parseInt("99")`

自动装箱：`Integer n = 100;`

自动拆箱：`int m = n;`

```
// 包装器类型的缓存特性
Integer a = Integer.valueOf(127);
Integer b = Integer.valueOf(127);
Integer c = Integer.valueOf(128);
Integer d = Integer.valueOf(128);
System.out.println("a == b :" + (a == b));	// true
System.out.println("a.equals(b) :" + a.equals(b));	// true
System.out.println("c == d :" + (c == d));	// false
System.out.println("c.equals(d) :" + c.equals(d));	// true
/* 在Integer中的valueOf()方法中，将-128到127之间的数值都存储在一个catch数组中，该数组相当于一个缓存，当我们在-128到127之间进行自动装箱时，就直接返回该值在数组中的地址，所以在-128到127之间的数值用==比较是相等的；而不在这个区间的数，则需要new开辟一个内存空间，所以不相等 */

// 其他包装类有点也有类似的机制：Character：0 到 127
Character c1 = Character.valueOf('A'); // ASCII 65
Character c2 = Character.valueOf('A');
System.out.println(c1 == c2); // true
Character c3 = Character.valueOf('啊'); // 中文字符，超出127
Character c4 = Character.valueOf('啊');
System.out.println(c3 == c4); // false
```

```
char ch = 'A';
Character cha = ch;
System.out.println(Character.isDigit(ch));	// 数字 false
System.out.println(Character.isLetter(ch));	// 字母 true
System.out.println(Character.isUpperCase(ch));	// 大写 true
System.out.println(Character.isLowerCase(ch));	// 小写 false
```



## 四、String

String是引用行数据类型，本身并不存储字符串，String类型的引用只用于存储对象的地址，以及调用相应的方法进行对对象的操作

String的底层有一个Value数组，用于存储字符串，也就是说虽然Java定义了字符串类型，但本质是还是用数组进行存储字符串的

在JDK1.8中，String的底层Value数组是char类型的，但是在JDk17中，Sting的底层Value数组是byte类型的。但两者并没有太大的区别，字符本质上就是数字，可以理解为String类当中的任何方法，都是对value数组进行操作的

```
使用常量串构造：
String s = "abcdef";
直接new String对象：
String s = new String("abcdef");
使用字符数组进行构造：
char[] ch = {'a' , 'b' , 'c' , 'd' , 'e' , 'f'};
String s = new String(ch);
```

比较：`equals(String str)`, `compareTo(String str)`

查找：`indexOf`, `lastIndexOf`

转化：数值和字符串之间的转化：`valueOf()`;大小写的转化：`toUpperCase()`，` toLowerCase()`;

替换：`replace("old","new")`

拆分：`split(";")`按“；”拆分；`split(";",2)`拆成两个

截取：`String.substring()`一个参数则截取该位置到字符串结尾的内容；两个参数则表示截取范围内内容，并且是左闭右开

```
String a = "abc";
String b = new String("abc");
System.out.println(a == b);	// false
System.out.println(a.equals(b));	// true
```



String的**不可变性**：

- 字符串中的内容是不可变的，一旦初始化完成后就无法改变了。使用“+=”进行追加，实际上是new了几个新对象进行追加成为一个新的对象最后再返回变成String类型  

- String类被final修饰，表示String类型不可以被继承
- String类的字符实际是保存在内部维护的value字符数组中，并且该数组也被final修饰，表示value本身的引用对象不可以修改，即不可以引用其他的字符数组，但是value引用的数组内容是可以修改的
- String所有涉及到字符串内容修改的方法，都是创建一个新对象，并进行改变，最终返回新对象



StringBuilder 和 StringBuffer

- 实现可变字符串功能，功能上两者是一样的（调用append()，向字符串追加内容时，不new新的对象
- 后者多线程安全，而前者不



## 五、Math



## 六、Class

Java程序在运行时，Java运行时系统一直对所有的对象进行所谓的运行时类型标识（RTTI），它纪录了每个对象所属的类

虚拟机通常使用运行时类型信息选准正确方法去执行，用来保存这些类型信息的类是Class类

Class类封装一个对象和接口运行时的状态，当装载类时，Class类型的对象自动创建



- Class类也是类的一种
- Class类的对象内容是你创建的类的类型信息，比如你创建一个shapes类，那么，Java会生成一个内容是shapes的Class类的对象
- Class类的对象不能像普通类一样，以 new shapes() 的方式创建，它的对象只能由JVM创建，因为这个类没有public构造函数
- Class类的作用是运行时提供或获得某个对象的类型信息。这些信息也可用于反射。



```
// 获取Class对象
Class cl1 = Integer.class;
Class cl2 = Class.forName("java.lang.Integer");
Integer i = 1;
Class cl3 = i.getClass();

//System.out.println(cl1 == cl2);	// true
//System.out.println(cl2 == cl3);	// true
```



```
@SuppressWarnings("rawtypes")
public static void main(String[] args) throws ClassNotFoundException {
    showClassInfo(100);	// 自动装箱
    showClassInfo('A');	// 自动装箱
    showClassInfo("Hello");
    showClassInfo(new A());
    showClassInfo(new B());
} 

@SuppressWarnings("rawtypes")
static void showClassInfo(Object obj) {
    // 获取 obj 的类名，属性数量，方法数量
    Class cls = obj.getClass();
    System.out.println("obj的类型是: " + cls);

    // 获取定义的属性
    Field[] fields1 = cls.getDeclaredFields();	// 获取公有的属性getFields()
    System.out.println("定义的属性: "+Arrays.toString(fields1));

    // 获取对象定义的方法
    cls.getDeclaredMethods();	// 获取对象公有的方法getMethods()

    try{
        Method method = cls.getMethod("hello", String.class, int.class);
        // 通过反射动态执行方法
        method.invoke(obj, "张三", 1);
    } catch (NoSuchMethodException e) {
        System.out.println(cls + " 没有该方法: hello()");
    } catch (IllegalAccessException e) {
        // 访问异常,例如: 调用私有方法
        e.printStackTrace();
    } catch (IllegalArgumentException e) {
        // 参数错误
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        // 方法内部错误
        e.printStackTrace();
    }
}

/*
*	控制台的部分输出：
*	obj的类型是: class lesson02.A
*	定义的属性: [private int lesson02.A.a, char lesson02.A.b, protected long lesson02.A.c, public float	
*	lesson02.A.d]
*	你好 a张三   1
*	obj的类型是: class lesson02.B
*	定义的属性: []
*	你好 a张三   1
*/
```

