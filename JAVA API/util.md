## 一、简介

工具类：

1. 集合框架
2. 日期时间
3. 事件模型
4. 随机数生成
5. 其他



---



## 二、日期日历

#### 2.1 Date

构造方法：

- `Date()`：创建一个表示当前日期和时间的Date对象
- `Date(long date)`：据指定的毫秒数创建一个Date对象，该毫秒数表示自1970年1月1日00:00:00 GMT以来的毫秒数

```
// 创建表示当前日期和时间的Date对象
Date currentDate = new Date();
// 创建一个指定日期和时间的Date对象
Date specificDate = new Date(1630454400000L); // 对应2022年9月2日12:00:00
// 判断日期先后
if (currentDate.after(specificDate)) {
	System.out.println("当前日期晚于指定日期");
}
```



**`SimpleDateFormat`**：

常用SimpleDateFormat 格式码：

- y：年份（例如，“yy” 表示年份的后两位，“yyyy” 表示完整的年份）。
- M：月份（1 到 12 或 01 到 12）。
- d：日期（1 到 31 或 01 到 31）。
- H：小时（0 到 23 或 00 到 23）。
- h：小时（1 到 12 或 01 到 12）。
- m：分钟（0 到 59或00到59）。
- s：秒（0 到 59 或 00 到 59）。
- S：毫秒。

```
// 格式化：
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date now = new Date(); // 获取当前日期和时间
String formattedDate = dateFormat.format(now);
// 解析：
String dateStr = "2023-09-01 12:30:45";	//必须和格式化时定义的一致，否则抛ParseException
try {
Date parsedDate = dateFormat.parse(dateStr);
    System.out.println(parsedDate);
} catch (ParseException e) {
    e.printStackTrace();
}
```



#### 2.2 Calendar

Calendar 是一个抽象类, 无法通过直接实例化得到对象

Calendar 提供了一个方法 getInstance,来获得一个Calendar对象, 得到的 Calendar 由当前时间初始化

​	`Calendar cal=Calendar.getInstance();//默认当前时间`

```
// 获取
Calendar cal = Calendar.getInstance();
System.out.print("年：" + cal.get(Calendar.YEAR));	
System.out.print("月：" + (cal.get(Calendar.MONTH)+1));// 月份默认从0开始，需要加1
System.out.println("日：" + cal.get(Calendar.DAY_OF_MONTH));
// 时HOUR_OF_DAY，分MINUTE，秒SECOND，星期DAY_OF_WEEK
```

```
// 设置
calendar.set(2023,2,5,15,30,50);	// 一次性设置
calendar.set(Calendar.DAY_OF_MONTH,15);	// 单独设置
```

```
// 计算
cal.add(Calendar.YEAR, 1);	// 在当前日期上加 1 年
```



---



## 三、随机

#### 3.1 Random

```
Random random = new Random();
int randomInt = random.nextInt();	//生成一个随机整数
boolean randomBoolean = random.nextBoolean();	// 生成一个随机boolean值
int randomIntWithinRange = random.nextInt(100);	// // 生成一个0到99之间的随机整数

// 使用自定义种子值创建Random对象
Random random1 = new Random(123);
Random random2 = new Random(123);
// 再次生成随机整数，由于种子值相同，结果也将相同
System.out.println(random1.nextInt() + " = " + random2.nextInt());
```



#### 3.2 UUID

**通用唯一标识符：**标准化的32位字符格式，用于在分布式系统中唯一标识信息

```
// 生成一个随机的UUID(极低概率重复)
UUID uuid = UUID.randomUUID();
// 将UUID转换为字符串表示形式
String uuidString = uuid.toString();
System.out.println("Random UUID: " + uuidString);
```



---



## 四、集合框架

存储多个数据的容器。根据不同存储方式形成的体系结构，就叫做集合框架体系

### 4.1 主要接口和类

#### 4.1.1 Collection 单列集合

- Collection是Iterable接口的子接口，包含一个iterator()方法，用以返回一个实现了Iterator接口的迭代器对象；
- Iterator对象称为迭代器，用于遍历Collection集合中的元素；

##### 4.1.1.1 List

- **有序，可重复**

1. ArrayList

   由数组实现存储，线程不安全

   ```
   // ArrayList 底层是数组实现，自动扩容(倍增)
   // 无泛型的情况下，集合可以存放任意类型元素
   List list = new ArrayList();
   list.add(1);list.add(1);list.add("ABCD");list.add(true);
   for(int i = 0; i < list.size(); i++) {
       System.out.println(list.get(i));
   }
   ```

2. LinkedList

   链表，线程不安全

3. Vector

   数组，**线程安全**（性能相对不足



##### 4.1.1.2 Set

**`HashSet`和`LinkedHashSet`调用`equals()`判断是否重复**，因此实际开发时如果希望筛掉重复值则需要重写equals方法

```
Set set = new HashSet();
set.add(1);set.add(1);set.add("ABCD");set.add(true);
// HashSet是无序的不重复的，没有get(i)方法获取元素
// 使用迭代器来访问元素
Iterator iterator = set.iterator();
// while(set.iterator().hasNext()) {	这里会再每次执行时创建新的迭代器，导致调用next()一直取到第一个元素，使得输出不符合预期
while(iterator.hasNext()) {
    System.out.println(iterator.next());
}
System.out.println("-------");
for(Object obj : set) { System.out.println(obj);}	// 增强for循环
System.out.println("-------");
set.forEach(System.out::println);	// forEach + lambda 表达式
```

1. HashSet
   - HashSet底层是HashMap
   - 可以存放null值，但是只能有一个null
   - HashSet**无序**（不保证元素是有序的，取决于hash后，再确定索引的结果（即，不保证存放元素的顺序和取出顺序一致）
   - **不重复**
2. LinkedHashSet
   - LinkedHashSet是HashSet的子类;
   - LinkedHashSet底层是一个LinkedHashMap,底层维护了一个数组+双向链表+红黑树;
   - LinkedHashSet**有序**（根据元素的hashcode值来决定元素的存储位置,同时使用链表维护元素的次序（图）,是元素看起来以插入顺序保存;
   - LinkedHashSet**不允许添加重复元素**
3. TreeSet
   - **排序，不重复（通过比较机制去重**
   - TreeSet 底层维护的是一个 TreeMap

```
// 存入TreeSet的元素要实现排序：要么实现Comparable接口，要么在实例TreeSet的时候传入Comparator
// 匿名内部类
Set<Person> set = new TreeSet<>(new Comparator<Person>() {
	@Override
	public int compare(Person p1, Person p2) {
		return Integer.compare(p1.getId(), p2.getId());
	}
});
// Lambda表达式
Set<Person> set = new TreeSet<>((p1, p2) -> Integer.compare(p1.getId(), p2.getId()));
//(p1, p2) -> p1.getId() - p2.getId()存在整数溢出的风险
Set<Person> set = new TreeSet<>(Comparator.comparingInt(p -> p.getId()));
//方法引用
Set<Person> set = new TreeSet<>(Comparator.comparingInt(Person::getId));
```



#### 4.1.2 Map 双列集合：存放键值对

- **无序**
- **key不可重复**

1. HashMap

   - 底层维护了Node类型的数组table，默认为null

   - 当创建对象时，将加载因子（loadfactor）初始化为0.75

   - 当添加key-val时，通过key的哈希值得到在table的索引。然后判断该索引处是否有元素，如果没有元素直接添加。如果该索引处有元素，判断该元素的key和准备加入的key是否相等。如果相等，则直接替换val，如果不相等需要判断是树结构还是链表结构从而做出相应处理。如果添加时发现容量不够，则需要扩容。

   - 第1添加，则需要扩容table容量为16，临界值（threshold）为12（16*0.75）

   - 以后再扩容，则需要扩容table容量为原来的2倍（32），临界值为原来的2倍，即24。依此类推

   - 在Java8中，如果一条链表的元素个数超过TREEIFY_THRESHOLD（默认是8），并且table的大小 >= MIN_TREEIFY_CAPACITY（默认64），就会进行树化（红黑树）

   ```
   Map<Integer, String> map = new HashMap<>();
   map.put(9001,"张三");
   map.put(9002,"李四");
   map.put(9003,"张三");
   map.put(9004,"王五");
   map.put(9004,"名字"); // 替换
   
   // 返回键集合 set 
   Set<Integer> keys = map.keySet();
   keys.forEach(System.out::println);
   // 返回值集合 Collection
   Collection<String> values = map.values();
   values.forEach(System.out::println);
   // 返回键值对象※※※
   Set<Map.Entry<Integer, String>> entrys = map.entrySet();
   entrys.forEach(entry->System.out.printf("%d:%s\n",entry.getKey(),entry.getValue()));
   ```

2. LinkedHashMap

​	有序 不重复 参考 LinkedHashSet

3. TreeMap

   - 排序

   - 不重复

   - 底层：红黑树

4. HashTable

   - 存放的元素是键值对：即 K-V

   - Hashtable的**键和值都不能为null**，否则会抛出NullPointerException（HashMap允许为空

   - Hashtable 使用方法基本上和HashMap一样

   - Hashtable 是线程安全的（synchronized），HashMap是线程不安全的

5. Properties

   - Properties类继承自Hashtable类并且实现了Map接口，也是使用键值对的形式来保存数据（以字符串形式存储键值对

   - 它的使用特点和Hashtable类似

   - Properties 更多用于 从xxx.properties文件中，加载数据到Properties类对象，并进行读取和修改【说明：xxx.properties文件通常作为配置文件】

```
String getProperty(String key)：用指定的键在此属性列表中搜索属性。
String getProperty(String key, String defaultValue)：用指定的键在属性列表中搜索属性，如果未找到，则返回默认值。
void setProperty(String key, String value)：设置属性，调用Hashtable的put方法，但要求键和值都是字符串。
void load(InputStream inStream)：从输入流中读取属性列表。
void store(OutputStream out, String comments)：将属性列表写入输出流，并带上注释。
void loadFromXML(InputStream in)：从XML输入流中读取属性。
void storeToXML(OutputStream os, String comment)：将属性写入XML输出流。
```



### 4.2 泛型

- 使用一个标识符来表示未知的数据类型，然后在使用该类或接口时指定该未知类型的真实类型。

```
定义在容器的后面，并用 < > 进行标注
泛型只能用于对象类型，不能用于基本数据类型
泛型的擦除：编码时期所指定的泛型，只在代码编译时期可见；当我们编写的类生成字节码文件之后，加入的泛型 < 数据类型 > 就会消失，不在字节码中体现出来
由于类型擦除，Java 不允许创建泛型类型的数组
```

- 泛型可用到的接口、类、方法中，将数据类型作为参数传递，其实更像是一个数据类型模板。

```
// 类
public class Box<T> {
    private T item;
    public void set(T item) { this.item = item; }
    public T get() { return this.item; }
}
// 使用
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
// ---------------------------------------------
// 方法
public static <T> void printArray(T[] inputArray) {
    for (T element : inputArray) {
        System.out.println(element);
    }
}
// 使用
GenericMethodTest.printArray(new Integer[]{1, 2, 3});
GenericMethodTest.printArray(new String[]{"A", "B", "C"});
// ---------------------------------------------
// 接口
public interface Pair<K, V> {
    K getKey();
    V getValue();
}
public class OrderedPair<K, V> implements Pair<K, V> {
    private K key;
    private V value;
    public OrderedPair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    public K getKey() { return key; }
    public V getValue() { return value; }
}
// 使用
Pair<String, Integer> pair = new OrderedPair<>("One", 1);
System.out.println(pair.getKey() + ": " + pair.getValue()); 
```

- 如果不使用泛型，从容器中获取出元素，需要做类型强转，也不能限制容器只能存储相同类型的元素

```
// 不定义泛型							|	//定义泛型
List list = new ArrayList();			|	List<String> list1 = new ArrayList<>();	
list.add("A");							|	list1.add("A");
String ele = (String) list.get(0);		|	String ele = list1.get(0);

// 在添加数据之前，泛型会看看你要添加的数据类型是否与标注的泛型类型相匹配，不匹配则不存入。若匹配，在存入之后，容器底层还是会把存入的所有数据类型当作 Object 类型保存；取数据的时候，再做一个强转，从 Object 类型强转变成泛型对应的类型
```

```
上界通配符 < ? extends T >，表示参数类型必须是 T 或 T 的子类	|不能往里存，只能往外取
下界通配符 < ? super T > 表示参数类型必须是 T 或 T 的父类		 |不影响往里存，但往外取只能返回 Object
无界通配符 <?>，接受任何类型								 |不能往里存，往外取只能返回 Object
```



### 4.3 Iterable 与 Iterator

- Iterable 接口是 Java 集合框架中最顶层的接口, 任何实现这个接口的类, Java 允许该类对象成为for-each循环语句的目标
- Iterable接口定义了三个方法，其中两个提供了默认实现，只有**`iterator()`是要求实现类必须实现**的方法。

```
@Override
public Iterator<T> iterator() {
    // 使用匿名内部类
    return new Iterator<T>() {
        private int index = 0;
        @Override
        public boolean hasNext() {
            return index < items.size();
        }
        @Override
        public T next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            return items.get(index++);
        }
    };
}
```



Iterable的迭代方法：

1. for-each循环
2. 获取 Iterable 实现类对象的迭代器 Iterator
3. 调用Iterable的forEach()方法



迭代器接口Iterator：

在Java中，Iterator接口提供了一种用于遍历集合元素的统一方式。通过实现Iterator接口，我们可以对集合中的元素进行迭代访问，而不需要了解集合的具体实现方式。

- Iterator对象称为迭代器，主要用于遍历Collection集合中的元素。
- 所有实现了Collection接口的集合类都有一个iterator()方法，用以返回一个实现了Iterator接口的对象，即可以返回一个迭代器。
- Iterator仅用于遍历集合，Iterator本身并不存放对象。

方法：

- boolean hasNext(): 如果仍有元素可以迭代，则返回true，否则返回false。
- E next(): 返回迭代的下一个元素。
- void remove(): 从迭代器指向的集合中移除迭代器最后返回的元素（可选操作）