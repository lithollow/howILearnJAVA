```java
class ByTrain implements Runnable{
	@Override
	public void run() {
		System.out.println("坐火车...");
	}
}
class ByAir implements Runnable{
	@Override
	public void run() {
		System.out.println("坐飞机...");
	}
}

class ByCarClass{
	public void byCarMethod() {
		System.out.println("开车...");
	}
	public static void byTaxiStaticMethod(){
		System.out.println("坐网约车...");
	}
}

public class MyThread {

	// 成员变量
    private Runnable target;

    public MyThread() {}
    
    // 构造方法，接收外部传递的出行策略
    public MyThread(Runnable target) {
        this.target = target;
    }
    
    // MyThread自己的run()，现在基本不用了
    public void run() {
        System.out.println("步行...");
    }
    
    // 如果外部传递了出行策略，就会调用该策略里的run()
    public void start() {
        if (target != null) {
            target.run();
        } else {
            this.run();
        }
    }
    
	public static void main(String[] args) {
		
		// 策略模式，接口多态
		ByTrain byTrain = new ByTrain();
        new MyThread(byTrain).start();
        ByAir byAir = new ByAir();
        new MyThread(byAir).start();
        
        // 匿名内部类优化，阻止类的暴增
        new MyThread(new Runnable() {
            @Override
            public void run() {
                System.out.println("骑电瓶车...");
            }
        }).start();
        
        // Lambda
        // 前提：函数式接口Runnable，MyThread有构造方法接收一个Runnable接口的子类对象（匿名类对象）
        new MyThread(() -> {
            System.out.println("闪现...");
        }).start();
        
        
        // 方法引用
        // - 实例方法引用 - 封装了要调用的实例方法的类的对象::实例方法
        ByCarClass byCarObject = new ByCarClass();
        new MyThread(byCarObject::byCarMethod).start();
        // - 静态方法引用 - 封装了要调用的静态方法的类::静态方法
        new MyThread(ByCarClass::byTaxiStaticMethod).start();
        
	}
}
```

more:

```java
public class Demo {
    public static void main(String[] args) {
        String str1 = "abc";
        String str2 = "abcd";

		// 原始
        int i = compareString(str1, str2, new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return s1.length() - s2.length();
            }
        });
        
        // 改进 1
        Comparator<String> comparator = (String s1, String s2) -> {
            return s1.length() - s2.length();
        };
        int i = compareString(str1, str2, comparator);
        
        // 改进 2
        int i = compareString(str1, str2, (str1, str2) -> s1.length() - s2.length());
        
        // 改进 3（方法引用）
        int i = compareString(str1, str2, Comparator.comparingInt(String::length));
    }

    public static int compareString(String str1, String str2, Comparator<String> comparator) {
        return comparator.compare(str1, str2);
    }
}
```



**lambda表达式不能修改外部局部变量**

```java
public void test() {
    final int[] array = {1, 2, 3};
    Runnable r = () -> {
        array[0] = 10; // 允许 修改数组内容
        // array = new int[5]; // 编译错误 不能重新赋值引用
    };
}
```

使用数组来传递结果，以绕过lambda表达式对局部变量修改限制

```java
public void test() {
    final int[] array = {1, 2, 3};	// Java 8之前，要求在匿名内部类中访问的局部变量必须是final的
    // Java 8之后可以访问effectively final的局部变量（不需要声明final）（变量在初始化后没有修改过，可认为等效final）
    Runnable r = () -> {
        array[0] = 10; // 允许 修改数组内容
        // array = new int[5]; // 编译错误 不能重新赋值引用
    };
}
```

使用 Atomic 类

```java
public void test() {
    AtomicBoolean bool = new AtomicBoolean(false);
    
    Runnable r = () -> {
        bool = true;	// 错误 不能修改外部局部变量的引用（即不能对bool重新赋值）
        bool.set(true); // 正确：修改AtomicBoolean对象的值，而不是改变bool变量的引用
    };
}
```

这样设计的原因：

1. 线程安全考虑

   ```java
   public void threadSafetyExample() {
       int localVar = 0; // 栈上的变量
       int[] array = {0}; // 堆上的对象
       
       // 如果允许修改 localVar，多个线程会有不一致的副本
       new Thread(() -> {
           // 每个线程有自己的 localVar 副本
           // 如果允许修改，会造成数据混乱
       }).start();
       
       // 但 array 在堆上，所有线程共享同一个对象
       new Thread(() -> {
           array[0] = 1; // 所有线程看到相同的修改
       }).start();
   }
   ```

2. 生命周期管理

​	局部变量在方法执行结束后就销毁了，但lambda可能在方法返回后继续执行。如果允许修改局部变量，会导致访问已销毁的变量
