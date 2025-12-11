# 函数式编程

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



# Stream API

此`Stream`不同于`java.io`的`InputStream`和`OutputStream`，它代表的是任意Java对象的序列

|      | java.io                  | java.util.stream           |
| ---- | ------------------------ | -------------------------- |
| 存储 | 顺序读写的`byte`或`char` | 顺序输出的任意Java对象实例 |
| 用途 | 序列化至文件或网络       | 内存计算／业务逻辑         |



|      | java.util.List           | java.util.stream     |
| ---- | ------------------------ | -------------------- |
| 元素 | 已分配并存储在内存       | 可能未分配，实时计算 |
| 用途 | 操作一组已存在的Java对象 | 惰性计算             |



```java
// 要表示一个全体自然数的集合，用List是不可能写出来的，因为自然数是无限的
List<BigInteger> list = ??? // 全体自然数?

// Stream可以做到
Stream<BigInteger> naturals = createNaturalStream(); // 全体自然数

// 且不考虑createNaturalStream()的实现，关注怎么使用这个 Stream
// 可以对每个自然数做一个平方，这样我们就把这个Stream转换成了另一个Stream
Stream<BigInteger> streamNxN = naturals.map(n -> n.multiply(n)); // 全体自然数的平方

// 惰性计算
Stream<BigInteger> s2 = naturals.map(n -> n.multiply(n)); // 不计算
Stream<BigInteger> s3 = s2.limit(100); // 不计算
s3.forEach(System.out::println); // 计算

// 链式
Stream<BigInteger> naturals = createNaturalStream();
naturals.map(n -> n.multiply(n)) // 1, 4, 9, 16, 25...
        .limit(100)
        .forEach(System.out::println);

// Stream API的基本用法就是：创建一个Stream，然后做若干次转换，最后调用一个求值方法获取真正计算的结果
int result = createNaturalStream() // 创建Stream
             .filter(n -> n % 2 == 0) // 任意个转换
             .map(n -> n * n) // 任意个转换
             .limit(100) // 任意个转换
             .sum(); // 最终计算结果
```



**`Stream`**: 

- 可以存放有限个或无限个元素：元素有可能已经全部存储在内存，也有可能是根据需要实时计算
- 一个`Stream`可以轻易地转换为另一个`Stream`，不修改原`Stream`本身
- 真正的计算通常发生在最后结果的获取，也就是惰性计算
  - 一个`Stream`转换为另一个`Stream`时，实际上只存储了转换规则，并没有任何计算发生



### 创建 Stream

1. 直接用`Stream.of()`静态方法，传入可变参数即创建了一个能输出确定元素的`Stream`

```
Stream<String> stream = Stream.of("A", "B", "C", "D");
// forEach()方法相当于内部循环调用，
// 可传入符合Consumer接口的void accept(T t)的方法引用：
stream.forEach(System.out::println);
```



2. 基于数组或Collection创建`Stream`

```java
// 把一个现有的序列变为Stream，它的元素是固定的
Stream<String> stream1 = Arrays.stream(new String[] { "A", "B", "C" });
Stream<String> stream2 = List.of("X", "Y", "Z").stream();
stream1.forEach(System.out::println);
stream2.forEach(System.out::println);
```



3. 基于`Supplier`创建`Stream`

```java
// 通过Stream.generate()方法创建Stream 需要传入一个Supplier对象
Stream<String> s = Stream.generate(Supplier<String> sp);

// 这样创建的Stream会不断调用Supplier.get()方法来不断产生下一个元素
// 这种Stream保存的不是元素，而是算法，它可以用来表示无限序列

// 模拟一个无限序列（当然受int范围限制不是真的无限大）
public class Main {
    public static void main(String[] args) {
        Stream<Integer> natual = Stream.generate(new NatualSupplier());
        // 注意：无限序列必须先变成有限序列再打印:
        natual.limit(20).forEach(System.out::println);
    }
}
// 不断生成自然数的Supplier
class NatualSupplier implements Supplier<Integer> {
    int n = 0;
    public Integer get() {
        n++;
        return n;
    }
}
```



4. 其他：通过一些API提供的接口，直接获得`Stream`

```java
Files类的lines()方法可以把一个文件变成一个Stream，每个元素代表文件的一行内容
try (Stream<String> lines = Files.lines(Paths.get("/path/to/file.txt"))) { ...}

正则表达式的Pattern对象有一个splitAsStream()方法，可以直接把一个长字符串分割成Stream序列而不是数组
Pattern p = Pattern.compile("\\s+");
Stream<String> s = p.splitAsStream("The quick brown fox jumps over the lazy dog");
s.forEach(System.out::println);
```



**基本类型**

Java的泛型不支持基本类型，所以无法使用`Stream<int>`

保存`int`，只能使用`Stream<Integer>`；这样会产生频繁的装箱、拆箱操作

为了提高效率，Java标准库提供了`IntStream`、`LongStream`和`DoubleStream`这三种使用基本类型的`Stream`

```java
// 将int[]数组变为IntStream:
IntStream is = Arrays.stream(new int[] { 1, 2, 3 });
// 将Stream<String>转换为LongStream:
LongStream ls = List.of("1", "2", "3").stream().mapToLong(Long::parseLong);
```



### **`Stream.map()`**

`Stream`最常用的一个转换方法，把一个`Stream`转换为另一个`Stream`

`map`操作，就是把一种操作运算，映射到一个序列的每一个元素上

```java
Stream<Integer> s = Stream.of(1, 2, 3, 4, 5);
Stream<Integer> s2 = s.map(n -> n * n);
```

`Stream.map()`：接收`Function`接口对象，它定义了一个`apply()`方法，把一个`T`类型转换成`R`类型

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

@FunctionalInterface
public interface Function<T, R> {
    // 将T类型转换为R:
    R apply(T t);
}
```



### **`Stream.filter()`**

`filter()`操作，就是对一个`Stream`的所有元素一一进行测试，不满足条件的被“滤掉”，剩下满足条件的元素构成一个新的`Stream`

```java
IntStream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
        .filter(n -> n % 2 != 0)
        .forEach(System.out::println);
```

```java
// filter()方法接收的对象是Predicate接口对象，它定义了一个test()方法，负责判断元素是否符合条件
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```



### **`Stream.reduce()`**

把一个`Stream`的所有元素按照聚合函数聚合成一个结果

```java
int sum = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(0, (acc, n) -> acc + n);
// 首先初始化结果为指定值（0），对每个元素依次调用(acc, n) -> acc + n，acc是上次计算的结果
System.out.println(sum); // 45
```

```java
// reduce()方法传入的对象是BinaryOperator接口
// 定义了一个apply()方法，把上次累加的结果和本次的元素 进行运算，并返回累加的结果
@FunctionalInterface
public interface BinaryOperator<T> {
    // Bi操作：两个输入，一个输出
    T apply(T t, T u);
}
```



```java
// 将配置文件的每一行配置通过map()和reduce()操作聚合成一个Map<String, String>
List<String> props = List.of("profile=native", "debug=true", "logging=warn", "interval=500");
Map<String, String> map = props.stream()
                                // 把k=v转换为Map[k]=v:
                                .map(kv -> {
                                    String[] ss = kv.split("\\=", 2);
                                    return Map.of(ss[0], ss[1]);
                                })
                                // 把所有Map聚合到一个Map:
                                .reduce(new HashMap<String, String>(), (m, kv) -> {
                                    m.putAll(kv);
                                    return m;
                                });
// 打印结果:
map.forEach((k, v) -> {
    System.out.println(k + " = " + v);
});
```

对一个`Stream`做聚合计算后，结果就不是一个`Stream`，而是一个其他的Java对象



### 输出集合

1. 输出为 List：聚合操作，它会强制`Stream`输出每个元素

```
Stream<String> stream = Stream.of("Apple", "", null, "Pear", "  ", "Orange");
List<String> list = stream.filter(s -> s != null && !s.isBlank()).collect(Collectors.toList());
System.out.println(list);
```

2. 数组

```java
List<String> list = List.of("Apple", "Banana", "Orange");
String[] array = list.stream().toArray(String[]::new);
```

3. Map

```java
Stream<String> stream = Stream.of("APPL:Apple", "MSFT:Microsoft");
Map<String, String> map = stream
        .collect(Collectors.toMap(
                // 把元素s映射为key:
                s -> s.substring(0, s.indexOf(':')),
                // 把元素s映射为value:
                s -> s.substring(s.indexOf(':') + 1)));
System.out.println(map);
```

4. 分组输出

```java
List<String> list = List.of("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
Map<String, List<String>> groups = list.stream()
        .collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));
System.out.println(groups);

分组的key，这里是 s -> s.substring(0, 1)，首字母相同的String分到一组
分组的value，这里是 Collectors.toList()，表示输出为List
运行结果：
    A=[Apple, Avocado, Apricots],
    B=[Banana, Blackberry],
    C=[Coconut, Cherry]
```



### 其他操作

1. **排序**：`sorted()`方法，转换操作，返回一个新的`Stream`

```java
// 要求`Stream`的每个元素必须实现`Comparable`接口
List<String> list = List.of("Orange", "apple", "Banana")
                        .stream()
                        .sorted()
                        .collect(Collectors.toList());
System.out.println(list);

// 要自定义排序，传入指定的Comparator即可
List<String> list = List.of("Orange", "apple", "Banana")
                        .stream()
                        .sorted(String::compareToIgnoreCase)
                        .collect(Collectors.toList());
```

2. **去重**：`distinct()`方法

```java
List.of("A", "B", "A", "C", "B", "D")
    .stream()
    .distinct()
    .collect(Collectors.toList()); // [A, B, C, D]
```

3. **截取**：把一个无限的`Stream`转换成有限的`Stream`
   1. 用`skip()`跳过当前`Stream`的前N个元素
   2. 用`limit()`截取当前`Stream`最多前N个元素

```java
List.of("A", "B", "C", "D", "E", "F")
    .stream()
    .skip(2) // 跳过A, B
    .limit(3) // 截取C, D, E
    .collect(Collectors.toList()); // [C, D, E]
```

4. **合并**：`Stream`的静态方法`concat()`

```java
Stream<String> s1 = List.of("A", "B", "C").stream();
Stream<String> s2 = List.of("D", "E").stream();
// 合并:
Stream<String> s = Stream.concat(s1, s2);
System.out.println(s.collect(Collectors.toList())); // [A, B, C, D, E]
```

5. **`flatMap()`**：把`Stream`的每个元素（比如`List`）映射为`Stream`，然后合并成一个新的`Stream`

```java
Stream<List<Integer>> s = Stream.of(
        Arrays.asList(1, 2, 3),
        Arrays.asList(4, 5, 6),
        Arrays.asList(7, 8, 9));
// 转换为Stream<Integer>
Stream<Integer> i = s.flatMap(list -> list.stream());
```

6. **并行处理**：通常情况下，对`Stream`的元素进行处理是单线程的，即一个一个元素进行处理

```java
// 用parallel() 把一个普通Stream转换为可以并行处理的Stream
Stream<String> s = ...
String[] result = s.parallel() // 变成一个可以并行处理的Stream
                   .sorted() // 可以进行并行排序
                   .toArray(String[]::new);

经过parallel()转换后的Stream只要可能，就会对后续操作进行并行处理。我们不需要编写任何多线程代码就可以享受到并行处理带来的执行效率的提升
```

7. 其他

- `forEach()`，循环处理`Stream`的每个元素

- `count()`：用于返回元素个数
- `max(Comparator<? super T> cp)`：找出最大元素
- `min(Comparator<? super T> cp)`：找出最小元素

针对`IntStream`、`LongStream`和`DoubleStream`，还额外提供了以下聚合方法：

- `sum()`：对所有元素求和
- `average()`：对所有元素求平均数

...
