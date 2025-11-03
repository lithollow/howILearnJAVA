```
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

```
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

