# Structural Patterns (结构型模式)

关注对象之间的组合和关系，旨在解决如何构建灵活且可复用的类和对象结构



## Adapter / Wrapper (适配器模式)

充当两个不兼容接口之间的桥梁

通过一个中间件（适配器）将一个类的接口转换成客户期望的另一个接口，使原本不能一起工作的类能够协同工作

```java
// Task 类 实现 Callable 接口
public class Task implements Callable<Long> {
    private long num;
    public Task(long num) {
        this.num = num;
    }

    public Long call() throws Exception {
        long r = 0;
        for (long n = 1; n <= this.num; n++) {
            r = r + n;
        }
        System.out.println("Result: " + r);
        return r;
    }
}

Callable<Long> callable = new Task(123450000L);
// Thread thread = new Thread(callable); // 报错：Thread接收Runnable接口，但不接收Callable接口
// 解决：不该写Task类，而是用一个Adapter，把这个Callable接口“变成”Runnable接口
Thread thread = new Thread(new RunnableAdapter(callable));
thread.start();

// Adapter 接收一个Callable，输出一个Runnable
public class RunnableAdapter implements Runnable {
    // 引用待转换接口:
    private Callable<?> callable;

    public RunnableAdapter(Callable<?> callable) {
        this.callable = callable;
    }

    // 实现指定接口:
    public void run() {
        // 将指定接口调用委托给转换接口调用:
        try {
            callable.call();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```



## Bridge (桥接模式)

通过将一个对象的抽象部分与它的实现部分分离，把抽象化与实现化解耦，使得二者可以独立变化

核心思想：不要过度使用继承，而是优先拆分某些部件，使用组合的方式来扩展功能

```java
// 某个汽车厂商生产三种品牌的汽车：Big、Tiny和Boss，每种品牌又可以选择燃油、纯电和混合动力
                   ┌───────┐
                   │  Car  │
                   └───────┘
                       ▲
    ┌──────────────────┼───────────────────┐
┌───────┐          ┌───────┐          ┌───────┐
│BigCar │          │TinyCar│          │BossCar│
└───────┘          └───────┘          └───────┘
    ▲                  ▲                  ▲
    │ ┌───────────────┐│ ┌───────────────┐│ ┌───────────────┐
    ├─│  BigFuelCar   │├─│  TinyFuelCar  │├─│  BossFuelCar  │
    │ └───────────────┘│ └───────────────┘│ └───────────────┘
    │ ┌───────────────┐│ ┌───────────────┐│ ┌───────────────┐
    ├─│BigElectricCar │├─│TinyElectricCar│├─│BossElectricCar│
    │ └───────────────┘│ └───────────────┘│ └───────────────┘
    │ ┌───────────────┐│ ┌───────────────┐│ ┌───────────────┐
    └─│ BigHybridCar  │└─│ TinyHybridCar │└─│ BossHybridCar │
      └───────────────┘  └───────────────┘  └───────────────┘
// 使用传统的集继承方法会使子类众多，且扩展困难
// 引入桥接模式：
	   ┌───────────┐
       │    Car    │
       └───────────┘
             ▲
       ┌───────────┐       ┌─────────┐
       │RefinedCar │ ─ ─ ─▶│ Engine  │
       └───────────┘       └─────────┘
             ▲                  ▲
    ┌────────┼────────┐         │ ┌──────────────┐
    │        │        │         ├─│  FuelEngine  │
┌───────┐┌───────┐┌───────┐     │ └──────────────┘
│BigCar ││TinyCar││BossCar│     │ ┌──────────────┐
└───────┘└───────┘└───────┘     ├─│ElectricEngine│
                                │ └──────────────┘
                                │ ┌──────────────┐
                                └─│ HybridEngine │
                                  └──────────────┘
public abstract class Car {
    protected Engine engine;
    public Car(Engine engine) { this.engine = engine;}
    public abstract void drive();
}

public interface Engine { void start();}

public abstract class RefinedCar extends Car {
    public RefinedCar(Engine engine) { super(engine);}

    public void drive() {
        this.engine.start();
        System.out.println("Drive " + getBrand() + " car...");
    }

    public abstract String getBrand();
}

// 最终的不同品牌继承自RefinedCar
public class BossCar extends RefinedCar {
    public BossCar(Engine engine) { super(engine);}

    public String getBrand() { return "Boss";}
}

// 而针对每一种引擎，继承自Engine，例如HybridEngine
public class HybridEngine implements Engine {
    public void start() { System.out.println("Start Hybrid Engine...");}
}

//客户端通过自己选择一个品牌，再配合一种引擎，得到最终的Car
RefinedCar car = new BossCar(new HybridEngine());
car.drive();
```



## Composite (组合模式)

又叫部分整体模式，是用于把一组相似的对象，依据树形结构来组合一个对象，用来表示部分以及整体层次

```java
// 例子：在XML中，从根节点开始，每个节点都可能包含任意个其他节点，这些层层嵌套的节点就构成了一颗树
public interface Node {
    // 添加一个节点为子节点:
    Node add(Node node);
    // 获取子节点:
    List<Node> children();
    // 输出为XML:
    String toXml();
}

// 对于一个<abc>这样的节点，我们称之为ElementNode，它可以作为容器包含多个子节点
public class ElementNode implements Node {
    private String name;
    private List<Node> list = new ArrayList<>();

    public ElementNode(String name) { this.name = name;}

    public Node add(Node node) {
        list.add(node);
        return this;
    }

    public List<Node> children() {return list; }

    public String toXml() {
        String start = "<" + name + ">\n";
        String end = "</" + name + ">\n";
        StringJoiner sj = new StringJoiner("", start, end);
        list.forEach(node -> {
            sj.add(node.toXml() + "\n");
        });
        return sj.toString();
    }
}

// 对于普通文本，我们把它看作TextNode，它没有子节点
public class TextNode implements Node {
	private String text;

	public TextNode(String text) {
		this.text = text;
	}

	public Node add(Node node) {
		throw new UnsupportedOperationException();
	}

	public List<Node> children() {
		return List.of();
	}

	public String toXml() {
		return text;
	}
}

// 解释节点
public class CommentNode implements Node {
	private String text;

	public CommentNode(String text) {
		this.text = text;
	}

	public Node add(Node node) {
		throw new UnsupportedOperationException();
	}

	public List<Node> children() {
		return List.of();
	}

	public String toXml() {
		return "<!-- " + text + " -->";
	}
}

// 通过ElementNode、TextNode和CommentNode，可以构造出一颗树
Node root = new ElementNode("school");
root.add(new ElementNode("classA")
        .add(new TextNode("Tom"))
        .add(new TextNode("Alice")));
root.add(new ElementNode("classB")
        .add(new TextNode("Bob"))
        .add(new TextNode("Grace"))
        .add(new CommentNode("comment...")));
System.out.println(root.toXml());

<school>
<classA>
Tom
Alice
</classA>
<classB>
Bob
Grace
<!-- comment... -->
</classB>
</school>
```



## Decorator (装饰器模式)

在运行期动态，向一个现有的对象添加新的功能，同时又不改变其结构



Java的IO标准库提供的`InputStream`根据来源可以包括：

- `FileInputStream`：从文件读取数据，是最终数据源
- `ServletInputStream`：从HTTP请求读取数据，是最终数据源
- `Socket.getInputStream()`：从TCP连接读取数据，是最终数据源

如果要给`FileInputStream`添加缓冲功能，则可以从`FileInputStream`派生一个类：

```java
class BufferedFileInputStream extends FileInputStream {}
```

如果要给`FileInputStream`添加加密/解密功能，也可以从`FileInputStream`派生一个类：

```java
CipherFileInputStream extends FileInputStream {}
```

这样容易出现子类爆炸



引入Decorator模式：

把一个一个的附加功能，用Decorator的方式给一层一层地累加到原始数据源上，最终，通过组合获得我们想要的功能

```java
// 给FileInputStream增加缓冲和解压缩功能
// 创建原始的数据源:
InputStream fis = new FileInputStream("test.gz");
// 增加缓冲功能:
InputStream bis = new BufferedInputStream(fis);
// 增加解压缩功能:
InputStream gis = new GZIPInputStream(bis);

// 或者一次性写成
InputStream input = new GZIPInputStream( // 第二层装饰
                        new BufferedInputStream( // 第一层装饰
                            new FileInputStream("test.gz") // 核心功能
                        ));
```



- 有一个类A需要添加新功能B，从A抽象出顶级接口C，定义核心功能的规范，原类A实现C；

- 然后编写抽象装饰器D也实现C，并且构造函数接收顶级接口C，默认实现接口 C 的所有方法，方法内部直接调用被装饰组件（持有的 C 实例）的对应方法。
  - 这一步的目的是：让具体装饰器E只需要重写需要增强的方法，无需重写接口的所有方法，减少代码冗余

- 接下来编写实现具体新功能的类E，继承抽象装饰器D，构造器接收顶级接口C，调用父类 D 的构造器（super(c)），并实现具体的新功能 B；
  - 动态增强方法：重写接口 C 的方法，先super.doSomething()，再再添加新功能 B

- 使用：C c = new E(new A());

```java
// 组件接口：定义核心功能
public interface C {
    // 核心业务方法
    void doSomething();
    // 可扩展其他核心方法
    void basicFunc();
}
// 具体组件：原业务类，实现核心接口
public class A implements C {
    @Override
    public void doSomething() {
        System.out.println("执行类A的核心功能：处理业务逻辑");
    }

    @Override
    public void basicFunc() {
        System.out.println("执行类A的基础功能");
    }
}
// 抽象装饰器：实现组件接口，持有组件实例，默认转发方法
public abstract class D implements C {
    // 持有被装饰的组件（核心：装饰器的“被装饰对象”引用）
    protected C component;

    // 构造函数：接收被装饰的组件
    public D(C component) {
        this.component = component;
    }

    // 关键：默认实现接口的所有方法，方法内部调用被装饰组件的对应方法
    // 具体装饰器可选择性重写这些方法
    @Override
    public void doSomething() {
        component.doSomething(); // 转发给被装饰组件
    }

    @Override
    public void basicFunc() {
        component.basicFunc(); // 转发给被装饰组件
    }
}
// 具体装饰器：继承抽象装饰器，添加新功能B
public class E extends D {
    // 构造器：调用父类构造器，传递被装饰组件（无需重新定义component）
    public E(C component) {
        super(component); // 关键：调用父类的构造器
    }

    // 重写需要增强的方法，添加新功能B
    @Override
    public void doSomething() {
        // 1. 先调用原组件的核心方法（可选择前置/后置/环绕增强）
        super.doSomething(); // 等价于 component.doSomething()
        // 2. 添加新功能B（装饰器的核心：动态增强）
        System.out.println("添加新功能B：执行日志记录、权限校验等扩展逻辑");
    }

    // 若需要增强其他方法，可重写（比如basicFunc），否则使用父类的默认实现
}

// 可扩展更多具体装饰器（比如F，添加新功能C）
public class F extends D {...}
// 使用
public class DecoratorTest {
    public static void main(String[] args) {
        // 1. 原始对象：仅执行A的功能
        C c1 = new A();
        c1.doSomething();
        /* 输出：
        执行类A的核心功能：处理业务逻辑
        */

        // 2. 单层装饰：A + 新功能B
        C c2 = new E(new A());
        c2.doSomething();
        /* 输出：
        执行类A的核心功能：处理业务逻辑
        添加新功能B：执行日志记录、权限校验等扩展逻辑
        */

        // 3. 多层装饰：A + 新功能B + 新功能C（装饰器模式的优势：动态叠加功能）
        C c3 = new F(new E(new A()));
        c3.doSomething(); // 执行E增强后的doSomething
        c3.basicFunc();   // 执行F增强后的basicFunc
        /* 输出：
        执行类A的核心功能：处理业务逻辑
        添加新功能B：执行日志记录、权限校验等扩展逻辑
        添加新功能C：前置增强——参数校验
        执行类A的基础功能
        添加新功能C：后置增强——结果缓存
        */
    }
}
```



## Facade (外观模式)

为现有的子系统添加一个高层接口，来隐藏系统的复杂性，使子系统更加容易使用

举个栗子：注册公司需要三步：

1. 向工商局申请公司营业执照

   ```java
   public class AdminOfIndustry {
       public Company register(String name) { ...}
   }
   ```

2. 在银行开设账户

   ```java
   public class Bank {
       public String openAccount(String companyId) { ...}
   }
   ```

3. 在税务局开设纳税号

   ```java
   public class Taxation {
       public String applyTaxCode(String companyId) { ...}
   }
   ```



全部委托给中介：

```java
public class Facade {
    public Company openCompany(String name) {
        Company c = this.admin.register(name);
        String bankAccount = this.bank.openAccount(c.getId());
        c.setBankAccount(bankAccount);
        String taxCode = this.taxation.applyTaxCode(c.getId());
        c.setTaxCode(taxCode);
        return c;
    }
}
// 这样，客户端只跟Facade打交道，一次完成公司注册的所有繁琐流程：
Company c = facade.openCompany("Facade Software Ltd.");
```



很多Web程序，内部有多个子系统提供服务，经常使用一个统一的Facade入口，例如一个`RestApiController`，使得外部用户调用的时候，只关心Facade提供的接口，不用管内部到底是哪个子系统处理的

更复杂的Web程序，会有多个Web服务，这个时候，经常会使用一个统一的网关入口来自动转发到不同的Web服务，这种提供统一入口的网关就是Gateway，它本质上也是一个Facade，但可以附加一些用户认证、限流限速的额外服务



## Flyweight (享元模式)

尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象

减少创建对象的数量，以减少内存占用和提高性能，提高运行速度



享元模式在Java标准库中有很多应用。我们知道，包装类型如`Byte`、`Integer`都是不变类，因此，反复创建同一个值相同的包装类型是没有必要的。以`Integer`为例，如果我们通过`Integer.valueOf()`这个静态工厂方法创建`Integer`实例，当传入的`int`范围在`-128`~`+127`之间时，会直接返回缓存的`Integer`实例

```java
Integer n1 = Integer.valueOf(100);
Integer n2 = Integer.valueOf(100);
System.out.println(n1 == n2); // true
```

对于`Byte`来说，因为它一共只有256个状态，所以，通过`Byte.valueOf()`创建的`Byte`实例，全部都是缓存对象

因此，享元模式就是通过工厂方法创建对象，在工厂方法内部，很可能返回缓存的实例，而不是新创建实例，从而实现不可变实例的复用



在实际应用中，享元模式主要应用于缓存，即客户端如果重复请求某些对象，不必每次查询数据库或者读取文件，而是直接返回内存中缓存的数据

```java
// 以Student为例，设计一个静态工厂方法，它在内部可以返回缓存的对象
public class Student {
    // 持有缓存:
    private static final Map<String, Student> cache = new HashMap<>();

    // 静态工厂方法:
    public static Student create(int id, String name) {
        String key = id + "\n" + name;
        // 先查找缓存:
        Student std = cache.get(key);
        if (std == null) {
            // 未找到,创建新对象:
            System.out.println(String.format("create new Student(%s, %s)", id, name));
            std = new Student(id, name);
            // 放入缓存:
            cache.put(key, std);
        } else {
            // 缓存中存在:
            System.out.println(String.format("return cached Student(%s, %s)", std.id, std.name));
        }
        return std;
    }

    private final int id;
    private final String name;

    public Student(int id, String name) { ...}
}
// 在实际应用中，可以使用成熟的缓存库，例如Guava的Cache，它提供了最大缓存数量限制、定时过期等实用功能
```



## Proxy (代理模式)

引入一个代理对象来控制对原对象的访问，代理对象在客户端和目标对象之间充当中介，负责将客户端的请求转发给目标对象，同时可以在转发请求前后进行额外的处理

```java
public class AProxy implements A {
    private A a;
    public AProxy(A a) { this.a = a;}
    public void a() {
        // this.a.a();
        // 可以添加额外的处理
        // 比如权限检查
        if (getCurrentUser().isRoot()) {
            this.a.a();
        } else {
            throw new SecurityException("Forbidden");
        }
    }
}
```

其他应用：

1. 远程代理 Remote Proxy：本地的调用者持有的接口实际上是一个代理，这个代理负责把对接口的方法访问转换成远程调用，然后返回结果。
   - Java内置的RMI机制就是一个完整的远程代理模式

2. 虚代理 Virtual Proxy：调用者先持有一个代理对象，但真正的对象尚未创建。这个真正的对象直到客户端需要真的必须调用时，才创建。
   - JDBC的连接池返回的JDBC连接（Connection对象）就可以是一个虚代理，即获取连接时根本没有任何实际的数据库连接，直到第一次执行JDBC查询或更新操作时，才真正创建实际的JDBC连接

3. 保护代理 Protection Proxy：它用代理对象控制对原始对象的访问，常用于鉴权
4. 智能引用 Smart Reference：如果有很多客户端对代理对象进行访问，通过内部的计数器可以在外部调用者都不使用后自动释放它