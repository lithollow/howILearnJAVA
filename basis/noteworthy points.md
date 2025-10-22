```
interface TestA { 
    String func(); 
}
public class Test {
    public static void main(String[] args) {
        System.out.println(new TestA() {
            public String func() { return “test”; }
    }
}
输出test
new TestA()这里是一个匿名内部类，它implement了TestA
```

---

```
小数默认是double类型
float f = 1.0f;
float f = 1e2f;	//1e2是科学计数法100.0，它是一个double类型的字面量
float f = 0x123	//16进制
```

```
int x = 8;  
byte b = 127; 
b = x;	// 错误。byte的范围比int小，编译器不允许直接将int赋值给byte，可能会丢失精度。需要强			 制类型转换：b = (byte)x;          
x = ‘a’;	// 正确。char类型，可以自动转换为int（得到其ASCII值97）       
long y = b;	// 正确。byte可以自动转换为long（小范围转大范围）    
float z = (int)6.89;	// 正确。先将6.89强制转换为int（得到6），然后自动转换为float

byte b = ‘a’;	// 正确。'a' 的 ASCII 值是 97，而 byte 的范围是 -128 到 127
b = b + 1;	// 错误。1是int 类型，b+1的结果是int类型。将int类型赋值给byte需要显式转换

short st = 127 + 128;	// 合法。127 和 128 都是整数字面量，默认类型为 int，它们的和 255 也是一个 int 类型的常量表达式。虽然 int 类型不能直接隐式转换为 short 类型（因为 short 的范围是 -32768 到 32767），但 Java 编译器对常量表达式有特殊处理：如果常量表达式的值在 short 的范围内，编译器会自动进行隐式转换。

5.0/2 + 10 的结果是double型数据 // 正确。double和int运算，int提升为double）
(short)10 + ‘a’ 的结果是short型数据	// 错误。(short)10是short，'a'是char（也是整型，值97），short和char相加，都会提升为int
```

---

```
局部变量没有默认值，必须给它赋
```

---

```
class Animal {	
	protected String name = "Animal";
	
	public void eat() {
     System.out.println("Animal is eating");
 }
}

class Dog extends Animal {	
	public String name = "Dog";
	
	public void eat() {
     System.out.println("Dog is eating");
 }

	public void bark() {
     System.out.println("Dog is barking");
 }
}

public class ExtendsDemo {
 public static void main(String[] args) {
     // 向上转型：子类引用转为父类引用
     Animal animal = new Dog(); // 隐式向上转型
     animal.eat(); // 输出: Dog is eating，方法调用是由实际对象类型决定的（运行时绑定/动态绑定）
     System.out.println("name of animal is : " + animal.name);	// 输出Animal，属性访问是由引用类型决定的（编译时绑定）
     // animal.bark();	// 编译报错 The method bark() is undefined for the type Animal
     // Dog d = new Animal();	// cannot convert from Animal to Dog

     // 向下转型：将父类引用转回子类引用
     if (animal instanceof Dog) {
         Dog dog = (Dog) animal; // 显式向下转型
         dog.bark(); // 输出: Dog is barking
     }
 }
}
```

---

```
构造方法不能使用final修饰。因为构造方法不能被子类继承，所以用final修饰构造方法是没有意义的，而且会导致编译错误
```

---

```
interface A{
	public void func();	// 仍会默认为abstract
}
```

---

```
重写：1、方法名、参数列表必须完全相同
	 2、访问修饰符不能比父类方法的访问修饰符更严格
	 3、返回类型必须相同或是父类方法返回类型的子类(不能返回void
	 4、子类重写的方法不能抛出比父类方法更宽泛的检查异常（子类方法可以不抛出异常
	 5、非静态方法不能重写静态方法，静态方法也不能重写非静态方法
	 6、final方法不能被重写
	 7、重写的方法可以使用@Override注解
	 8、同步性（synchronized）和严格性（strictfp）不是方法重写的条件，即重写方法时可以添加或移除synchronized和strictfp修饰符，但这可能会影响方法的行为
	 
重载：1、方法名相同，参数列表必须不同
	 2、返回类型不限
	 3、访问权限可以改改变

```

---

```
下列判断语句中a和b的值相等的有？ BC     
A. int a=0; int b=‘0’;     
	a的值为0，b的值为字符'0'的ASCII码（48），因此a和b的值不相等
B. int a=123;  char b=‘\u007B’; 
	a的值为123，b的Unicode码点'\u007B'对应十进制123，因此a和b的值相等
C. int a=3+‘5’; char b=’8’;
	a的值为3 + '5'的ASCII码（53） = 56，b的值为字符'8'的ASCII码（56），因此a和b的值相等
D. char a=‘\u0000’; char b=‘0’; 
	a的值为Unicode码点0（空字符），b的值为字符'0'的ASCII码（48），因此a和b的值不相等
```

---

```
public class Demo{  
	public static void main(String[] args) { 
		System.out.println(Test.PI);
	}
}
class Test { 
	public static final double PI = 3.14; 
	static{ 
		System.out.print("PI="); 
	} 
} 
结果：3.14
--------------------------------------------
public class Demo{  
	public static void main(String[] args) { 
		System.out.println(Test.RATE); 
	} 
} 
class Test { 
	public static final double RATE = Math.random(); 
	static{ 
		System.out.print("RATE="); 
	} 
} 
结果：RATE=<一个随机浮点型数>

在第一个例子中，PI 是一个编译时常量，因为它的值是字面量 3.14。所以当在 Demo 类中访问 Test.PI 时，实际上是在编译时直接将 3.14 替换到 System.out.println(Test.PI) 中，因此不需要初始化 Test 类。所以 Test 的静态代码块没有执行。
在第二个例子中，RATE 不是编译时常量，因为它的值是通过 Math.random() 方法调用得到的，这在编译时无法确定。所以当访问 Test.RATE 时，需要初始化 Test 类，这会导致静态代码块的执行。
-----------------------------------------------
编译时常量须满足的条件：
1、是基本类型或 String
2、被声明为 final
3、在声明时初始化
4、初始化表达式是常量表达式

```

