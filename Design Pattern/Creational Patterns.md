# Creational Patterns (创建型模式)

在创建对象的同时**隐藏创建逻辑**，使得创建对象的过程与使用对象的过程分离

使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活，提高代码的可维护性和可扩展性

## Factory Pattern (工厂模式)

**`new `**+ 构造函数的方式暴露了实现细节，导致客户程序与具体实现类**紧耦合**

```java
// 构造函数的名字与类名一致
// 使用构造函数，意味着在编译时就依赖了具体的类，后期想要切换就只能修改源码
Student student = new Student();
```

**静态方法**与对象创建，仍然无法解决耦合的问题

```java
// 可以根据场景选择合适的命名，规避了构造函数表现力不足的问题
public class Student {
	public static Student createStudentA(...){ return ...}
	public static Student createStudentB(...){ return ...}
}
// JDK 中
Integer i = Integer.valueOf("1");
Boolean b = Boolean.valueOf(false);
```



### Simple Factory (简单工厂模式)

使用一个单独的工厂类来创建不同的对象，根据传入的参数决定创建哪种类型的对象

用静态方法做一层**函数抽象**，将对象的创建与使用相分离

```java
public class SimpleFactory {
	public static Product create(Integer type) {
		if (type == 1) {
			return new ProductA();
		} else if (type == 2) {
			return new ProductB();
		} else if (...) {
			// ...
		}
		throw new RuntimeException("类型错误");
	}
}

public class Client {
	public void method(Integer type) {
		Product product = SimpleFactory.create(type);
	}
}
```

缺点

- 无法使用多态机制（静态方法无法重写），灵活性欠佳
- 新增“Product类型”需要修改源码（增加if-else分支），不符合开闭原则
- 多写一个工厂类（实际业务并不存在Factory抽象，它被设计用于承接“对象创建”的职责）



### Factory Method (工厂方法)

为每一种Product都设计一个对应的Factory，专门用于构建该对象，本质是**把Product的新增转换为对应的Factory的新增**

```
┌─────────────┐      ┌─────────────┐
│   Product   │      │   Factory   │
└─────────────┘      └─────────────┘
       ▲                    ▲
       │                    │
┌─────────────┐      ┌─────────────┐
│ ProductImpl │◀─ ─ ─│ FactoryImpl │
└─────────────┘      └─────────────┘
```

```java
public interface AbstractFactory {
	AbstractProduct create();
}

public class FactoryA implements AbstractFactory {
	@Override
	public AbstractProduct create() {
		ProductA productA = new ProductA();
		productA.setXXX(...);
		return productA;
	}
}

public class Client {
	private AbstractFactory factory;
	
	public Client(AbstractFactory factory) {
		this.factory = factory;
	}
	
	public void method() {
		AbstractProduct product = this.factory.create(); 
	}
}
```

避免了客户程序与**具体产品**的紧耦合，却又与**具体工厂**发生耦合

```java
public class ClientTest {
	public void useClient() {
		// Client的创建依赖于具体的工厂
		AbstractFactory factory = new FactoryA();
		Client client = new Client(factory);
	}
}
```

**配置文件** + **反射**

```xml
<!-- 配置信息：在XML中定义Bean -->
<bean id = "factory" class = "com.demo.designpattern.FactoryA"/>
```

```java
public class ClientTest {
	public void useClient() {
		// 反射具体的factory
		AbstractFactory factory = XmlUtil.getBean("factory");
		Client client = new Client(factory);
        client.method();
	}
}
```

将变化转移到配置文件中是工程意义上封装变化的最终解，通常也是最优解。

实际开发时推荐使用Apollo或Nacos等中间件进行配置。



### Abstract Factory (抽象工厂)

```
                                ┌────────┐
                             ─ ▶│ProductA│
┌────────┐    ┌─────────┐   │   └────────┘
│ Client │─ ─▶│ Factory │─ ─
└────────┘    └─────────┘   │   ┌────────┐
                   ▲         ─ ▶│ProductB│
           ┌───────┴───────┐    └────────┘
           │               │
      ┌─────────┐     ┌─────────┐
      │Factory1 │     │Factory2 │
      └─────────┘     └─────────┘
           │   ┌─────────┐ │   ┌─────────┐
            ─ ▶│ProductA1│  ─ ▶│ProductA2│
           │   └─────────┘ │   └─────────┘
               ┌─────────┐     ┌─────────┐
           └ ─▶│ProductB1│ └ ─▶│ProductB2│
               └─────────┘     └─────────┘
```

```java
// 提供一个Markdown文本转换为HTML和Word的服务
public interface AbstractFactory {
    // 创建Html文档:
    HtmlDocument createHtml(String md);
    // 创建Word文档:
    WordDocument createWord(String md);
}

// 两个抽象产品
// Html文档接口:
public interface HtmlDocument {
    String toHtml();
    void save(Path path) throws IOException;
}

// Word文档接口:
public interface WordDocument {
    void save(Path path) throws IOException;
}

// 现在市场上有两家供应商：FastDoc Soft，GoodDoc Soft
public class FastFactory implements AbstractFactory {
    public HtmlDocument createHtml(String md) {
        return new FastHtmlDocument(md);
    }
    public WordDocument createWord(String md) {
        return new FastWordDocument(md);
    }
}
public class FastHtmlDocument implements HtmlDocument {
    public String toHtml() {
        ...
    }
    public void save(Path path) throws IOException {
        ...
    }
}
public class FastWordDocument implements WordDocument {
    public void save(Path path) throws IOException {
        ...
    }
}
// 使用FastDoc Soft的服务了
// 创建AbstractFactory，实际类型是FastFactory:
AbstractFactory factory = new FastFactory();
// 生成Html文档:
HtmlDocument html = factory.createHtml("#Hello\nHello, world!");
html.save(Paths.get(".", "fast.html"));
// 生成Word文档:
WordDocument word = factory.createWord("#Hello\nHello, world!");
word.save(Paths.get(".", "fast.doc"));
...
```

| 模式         | 工厂生产范围          | 使用场景                     |
| ------------ | --------------------- | ---------------------------- |
| 工厂方法模式 | 只生产 **一个产品**   | 需要解耦“单一产品”的创建逻辑 |
| 抽象工厂模式 | 生产 **多个相关产品** | 需要保证“产品族”整体的一致性 |



## Singleton (单例模式)

保证在一个进程中，某个类有且仅有一个实例；

单例自己创建自己的唯一实例；

给所有其他对象提供这一实例

```java
public class Singleton {
    // 静态字段引用唯一实例
    private static final Singleton INSTANCE = new Singleton();

    // 通过静态方法返回实例 供外部调用方获得
    public static Singleton getInstance() {
        return INSTANCE;
    }

    // private构造方法保证外部无法实例化
    private Singleton() {
    }
}
```



```java
// 1. 延迟加载：必须加锁，影响并发性能
public synchronized static Singleton getInstance() {
    if (INSTANCE == null) {
        INSTANCE = new Singleton();
    }
    return INSTANCE;
}

// 2. 双重检查：由于Java的内存模型，双重检查在这里不成立
// 要真正实现延迟加载，只能通过Java的ClassLoader机制完成
// 没有特殊的需求，使用Singleton模式的时候，最好不要延迟加载，这样会使代码更简单
public static Singleton getInstance() {
    if (INSTANCE == null) {
        synchronized (Singleton.class) {
            if (INSTANCE == null) {
                INSTANCE = new Singleton();
            }
        }
    }
    return INSTANCE;
}

// 3. 利用Java的enum实现Singleton：Java保证枚举类的每个枚举都是单例
// 同时避免了第一种方式实现Singleton的一个潜在问题：即序列化和反序列化会绕过普通类的private构造方法从而创建出多个实例
public enum World {
    // 唯一枚举:
	INSTANCE;
	private String name = "world";
	public String getName() {
		return this.name;
	}
	public void setName(String name) {
		this.name = name;
	}
}

```



## Builder (建造者模式)

使用多个“小型”工厂来最终创建出一个完整对象：创建这个对象的步骤比较多，每个步骤都需要一个零部件，最终组合成一个完整的对象



以Markdown转HTML为例：每一行根据语法，使用不同的转换器：（简化语法，把每一行视为可以独立转换

- 如果以`#`开头，使用`HeadingBuilder`转换
- 如果以`>`开头，使用`QuoteBuilder`转换
- 如果以`---`开头，使用`HrBuilder`转换
- 其余使用`ParagraphBuilder`转换

```
public class HtmlBuilder {
    private HeadingBuilder headingBuilder = new HeadingBuilder();
    private HrBuilder hrBuilder = new HrBuilder();
    private ParagraphBuilder paragraphBuilder = new ParagraphBuilder();
    private QuoteBuilder quoteBuilder = new QuoteBuilder();

    public String toHtml(String markdown) {
        StringBuilder buffer = new StringBuilder();
        markdown.lines().forEach(line -> {
            if (line.startsWith("#")) {
                buffer.append(headingBuilder.buildHeading(line)).append('\n');
            } else if (line.startsWith(">")) {
                buffer.append(quoteBuilder.buildQuote(line)).append('\n');
            } else if (line.startsWith("---")) {
                buffer.append(hrBuilder.buildHr(line)).append('\n');
            } else {
                buffer.append(paragraphBuilder.buildParagraph(line)).append('\n');
            }
        });
        return buffer.toString();
    }
}
```



## Prototype (原型模式)

创建新对象的时候，根据现有的一个原型来创建

```java
// 实现 Cloneable 接口 标识一个对象是“可复制”的
public class Student implements Cloneable {
    private int id;
    private String name;
    private int score;

    // 复制新对象并返回:
    public Object clone() {
        Student std = new Student();
        std.id = this.id;
        std.name = this.name;
        std.score = this.score;
        return std;
    }
}

// 使用时，因为clone()的方法签名是定义在Object中，返回类型也是Object，所以要强制转型
Student std1 = new Student();
std1.set...
// 复制新对象:
Student std2 = (Student) std1.clone();
System.out.println(std1 == std2); // false
// 更好的方式是定义一个copy()方法，返回明确的类型
public Student copy() {
    Student std = new Student();
    std.id = this.id;
    std.name = this.name;
    std.score = this.score;
    return std;
}
```

原型模式应用不是很广泛，因为很多实例会持有类似文件、Socket这样的资源，而这些资源是无法复制给另一个对象共享的，只有存储简单类型的“值”对象可以复制



