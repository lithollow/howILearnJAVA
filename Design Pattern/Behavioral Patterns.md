# Behavioral Patterns (行为型模式)

关注对象之间的通信和交互，旨在解决对象之间的责任分配和算法的封装

通过使用对象组合，行为型模式可以描述一组对象应该如何协作来完成一个整体任务



## Strategy (策略模式)

定义一系列算法或策略，并将每个算法封装在独立的类中，使得它们可以互相替换

使用策略模式，可以在运行时根据需要选择不同的算法，而不需要修改客户端代码



举个栗子：购物车结算为例：假设网站针对普通会员、Prime会员有不同的折扣

```java
public interface DiscountStrategy {
    // 计算折扣额度:
    BigDecimal getDiscount(BigDecimal total);
}

public class UserDiscountStrategy implements DiscountStrategy {
    public BigDecimal getDiscount(BigDecimal total) {
        // 普通会员打九折:
        return total.multiply(new BigDecimal("0.1")).setScale(2, RoundingMode.DOWN);
    }
}

public class OverDiscountStrategy implements DiscountStrategy {
    public BigDecimal getDiscount(BigDecimal total) {
        // 满100减20优惠:
        return total.compareTo(BigDecimal.valueOf(100)) >= 0 ? BigDecimal.valueOf(20) : BigDecimal.ZERO;
    }
}

// 应用策略，我们需要一个DiscountContext
// Context类的价值：
// 1. 封装复杂算法逻辑：如果算法逻辑变得更复杂，这时直接调用策略，这些复杂逻辑会散落在客户端代码各处
// 2. 提供统一的接口 提供默认策略
// 3. 统一管理和配置 便于扩展和维护
public class DiscountContext {
    // 持有某个策略:
    private DiscountStrategy strategy = new UserDiscountStrategy();

    // 允许客户端设置新策略:
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }

    public BigDecimal calculatePrice(BigDecimal total) {
        return total.subtract(this.strategy.getDiscount(total)).setScale(2);
    }
}

// 调用
DiscountContext ctx = new DiscountContext();

// 默认使用普通会员折扣:
BigDecimal pay1 = ctx.calculatePrice(BigDecimal.valueOf(105));
System.out.println(pay1);

// 使用满减折扣:
ctx.setStrategy(new OverDiscountStrategy());
BigDecimal pay2 = ctx.calculatePrice(BigDecimal.valueOf(105));
System.out.println(pay2);

// 简单场景下 不使用 Context类 功能上也能计算出正确结果
DiscountStrategy d = new UserDiscountStrategy();
BigDecimal total = BigDecimal.valueOf(105);
BigDecimal pay = total.subtract(d.getDiscount(total)).setScale(2);
```



## Template Method (模板方法模式)

定义一个操作的一系列步骤，对于某些暂时确定不下来的步骤，就留给子类去实现好了，这样不同的子类就可以定义出不同的步骤

```java
public abstract class Game {
   abstract void initialize();
   abstract void startPlay();
   abstract void endPlay();
 
   //模板方法被设置为 final
   public final void play(){
      initialize();
      startPlay();
      endPlay();
   }
}
public class Football extends Game {
   @Override
   void endPlay() {
      System.out.println("Football Game Finished!");
   }
   @Override
   void initialize() {
      System.out.println("Football Game Initialized! Start playing.");
   }
   @Override
   void startPlay() {
      System.out.println("Football Game Started. Enjoy the game!");
   }
}
public class TemplatePatternDemo {
   public static void main(String[] args) {
      game = new Football();
      game.play();      
   }
}
```



## Observer (观察者模式)

对象间存在一对多关系，当一个对象的状态发生改变时，其所有依赖者都会收到通知并自动更新

```java
interface Observer {
    // 更新方法
    void update(String message);
}
// 被观察者：维护观察者列表，负责通知
class UnderObserve {
    // 存储所有注册的观察者
    private List<Observer> observers = new ArrayList<>();
    // 主题的状态
    private String state;

    public String getState() { return state;}

    // 设置主题状态，状态变化时通知所有观察者
    public void setState(String state) {
        this.state = state;
        notifyObservers(); // 状态变更，触发通知
    }

    // 注册观察者
    public void registerObserver(Observer observer) { observers.add(observer);}

    // 移除观察者
    public void removeObserver(Observer observer) { observers.remove(observer);}

    // 通知所有注册的观察者
    private void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(state); // 调用观察者的更新方法
        }
    }
}
// 具体观察者C
class C implements Observer {
    @Override
    public void update(String message) {
        System.out.println("观察者C收到通知：" + message);
    }
}
```



## Iterator (迭代器模式)

提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示



以一个自定义的集合为例，通过Iterator模式实现倒序遍历

```java
public class ReverseArrayCollection<T> implements Iterable<T> {
    private T[] array;

    public ReverseArrayCollection(T... objs) {
        this.array = Arrays.copyOfRange(objs, 0, objs.length);
    }

    // 返回一个Iterator对象，该对象知道集合的内部结构，因为它可以实现倒序遍历
    public Iterator<T> iterator() {
        return new ReverseIterator();
    }

    // 使用内部类的好处是内部类隐含地持有一个它所在对象的this引用，可以通过ReverseArrayCollection.this引用到它所在的集合
    class ReverseIterator implements Iterator<T> {
        // 索引位置:
        int index;

        public ReverseIterator() {
            // 创建Iterator时,索引在数组末尾:
            this.index = ReverseArrayCollection.this.array.length;
        }

        public boolean hasNext() {
            // 如果索引大于0,那么可以移动到下一个元素(倒序往前移动):
            return index > 0;
        }

        public T next() {
            // 将索引移动到下一个元素并返回(倒序往前移动):
            index--;
            return array[index];
        }
    }
}
```



## Visitor (访问者模式)

一个作用于某对象结构中的各元素的操作，它的目的是不改变对象的定义，但允许新增不同的访问者，来定义新的操作

核心思想是为了访问比较复杂的数据结构，不去改变数据结构，而是把对数据的操作抽象出来，在“访问”的过程中以回调形式在访问者中处理操作逻辑

- 如果要新增一组操作，那么只需要增加一个新的访问者

```ascii art
   ┌─────────┐       ┌───────────────────────┐
   │ Client  │─ ─ ─ ▶│        Visitor        │
   └─────────┘       ├───────────────────────┤
        │            │visitElementA(ElementA)│
                     │visitElementB(ElementB)│
        │            └───────────────────────┘
                                 ▲
        │                ┌───────┴───────┐
                         │               │
        │         ┌─────────────┐ ┌─────────────┐
                  │  VisitorA   │ │  VisitorB   │
        │         └─────────────┘ └─────────────┘
        ▼
┌───────────────┐        ┌───────────────┐
│ObjectStructure│─ ─ ─ ─▶│    Element    │
├───────────────┤        ├───────────────┤
│handle(Visitor)│        │accept(Visitor)│
└───────────────┘        └───────────────┘
                                 ▲
                        ┌────────┴────────┐
                        │                 │
                ┌───────────────┐ ┌───────────────┐
                │   ElementA    │ │   ElementB    │
                ├───────────────┤ ├───────────────┤
                │accept(Visitor)│ │accept(Visitor)│
                │doA()          │ │doB()          │
                └───────────────┘ └───────────────┘
```

此模式因为实现了”双重分派“，即设计了一个回调再回调的机制，大大增加了代码的复杂性



这里介绍简化的访问者模式，现在我们要递归遍历某个文件夹的所有子文件夹和文件，找出`.java`文件

递归做法：问题在于扫描目录的逻辑和处理.java文件的逻辑混合，扩展性查

```java
void scan(File dir, List<File> collector) {
    for (File file : dir.listFiles()) {
        if (file.isFile() && file.getName().endsWith(".java")) {
            collector.add(file);
        } else if (file.isDir()) {
            // 递归调用:
            scan(file, collector);
        }
    }
}
```

引入访问者模式：

​	把数据结构（这里是文件夹和文件构成的树型结构）和对其的操作（查找文件）分离

```java
// 访问者接口
public interface Visitor {
    void visitDir(File dir);	 // 访问文件夹
    void visitFile(File file);	// 访问文件
}

// 持有文件夹和文件的数据结构
public class FileStructure {
    private File path;	// 根目录
    public FileStructure(File path) { this.path = path;}
    // handle 方法 传入一个访问者
    public void handle(Visitor visitor) {
		scan(this.path, visitor);
	}

	private void scan(File file, Visitor visitor) {
		if (file.isDirectory()) {
            // 让访问者处理文件夹:
			visitor.visitDir(file);
			for (File sub : file.listFiles()) {
                // 递归处理子文件夹:
				scan(sub, visitor);
			}
		} else if (file.isFile()) {
            // 让访问者处理文件:
			visitor.visitFile(file);
		}
	}
}

// 这样访问者的行为就抽象出来了，要实现功能，如查找.java文件，只需实现一个 Visitor
```



## Chain of Responsibility (责任链模式)

为请求创建了一个接收者对象的链，对请求的发送者和接收者进行解耦，请求沿着这条链传递，直到有一个处理器处理该请求为止



```java
// 请求对象 在责任链上传递
public class Request {
    private String name;
    private BigDecimal amount;

    public Request(String name, BigDecimal amount) { this.name = name; this.amount = amount;}
    public String getName() { return name;}
    public BigDecimal getAmount() { return amount;}
}

// 抽象处理器
public interface Handler {
    // 返回Boolean.TRUE = 成功、Boolean.FALSE = 拒绝
    // 返回null = 交下一个处理
	Boolean process(Request request);
}
// 各个 处理器
public class ManagerHandler implements Handler {
    public Boolean process(Request request) {
        // 如果超过1000元，处理不了，交下一个处理:
        if (request.getAmount().compareTo(BigDecimal.valueOf(1000)) > 0) {
            return null;
        }
        // 对Bob有偏见:
        return !request.getName().equalsIgnoreCase("bob");
    }
}
...	// DirectorHandler、CEOHandler
// 组合 Handler 为链，通过一个统一入口处理
public class HandlerChain {
    // 持有所有Handler:
    private List<Handler> handlers = new ArrayList<>();

    public void addHandler(Handler handler) { this.handlers.add(handler);}

    public boolean process(Request request) {
        // 依次调用每个Handler:
        for (Handler handler : handlers) {
            Boolean r = handler.process(request);
            if (r != null) {
                // 如果返回TRUE或FALSE，处理结束:
                System.out.println(request + " " + (r ? "Approved by " : "Denied by ") + handler.getClass().getSimpleName());
                return r;
            }
        }
        throw new RuntimeException("Could not handle request: " + request);
    }
}
// 客户端使用
// 构造责任链:
HandlerChain chain = new HandlerChain();
chain.addHandler(new ManagerHandler());
chain.addHandler(new DirectorHandler());
chain.addHandler(new CEOHandler());
// 处理请求:
chain.process(new Request("Bob", new BigDecimal("123.45")));
chain.process(new Request("Alice", new BigDecimal("1234.56")));
chain.process(new Request("Bill", new BigDecimal("12345.67")));
chain.process(new Request("John", new BigDecimal("123456.78")));
```



还有一些责任链模式，每个`Handler`都有机会处理`Request`，通常这种责任链被称为拦截器（Interceptor）或者过滤器（Filter），它的目的不是找到某个`Handler`处理掉`Request`，而是每个`Handler`都做一些工作，比如：

- 记录日志
- 检查权限
- 准备相关资源
- ...



## Command (命令模式)

将一个请求封装为一个对象，从而使你可以用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作



现在有一个文本编辑器

```java
public class TextEditor {
    private StringBuilder buffer = new StringBuilder();

    public void copy() {...}
    public void paste() {...}
    public void add(String s) {...}
    public void delete() {...}
    public String getState() {...}
}

// 普通调用 调用方需要了解TextEditor的所有接口信息
TextEditor editor = new TextEditor();
editor.add("Command pattern in text editor.\n");
editor.copy();
editor.paste();
System.out.println(editor.getState());
// 这样事实上已经足够简单，但是如果TextEditor复杂到一定程度，并且需要支持Undo、Redo的功能时，就需要使用命令模式
```

```java
public interface Command {
    void execute();
}
public class CopyCommand implements Command {
    // 持有执行者对象:
    private TextEditor receiver;

    public CopyCommand(TextEditor receiver) { this.receiver = receiver;}
    public void execute() { receiver.copy();}
}

public class PasteCommand implements Command {...}

// 客户端 确实增加了系统的复杂度
TextEditor editor = new TextEditor();
editor.add("Command pattern in text editor.\n");
// 执行一个CopyCommand:
Command copy = new CopyCommand(editor);
copy.execute();
editor.add("----\n");
// 执行一个PasteCommand:
Command paste = new PasteCommand(editor);
paste.execute();
System.out.println(editor.getState());
```



## Interpreter (解释器模式)

给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子

这种模式被用在 SQL 解析、符号处理引擎等



```java
// 示例：实现简单的加减法解释器
// 抽象表达式：定义解释方法
public interface Expression {
    // 解释方法，返回表达式的计算结果
    int interpret();
}
// 终结符表达式：数字表达式 对应加减法中的数字，是最小的解释单元
public class NumberExpression implements Expression {
    private int number; // 存储数字值

    public NumberExpression(int number) {
        this.number = number;
    }

    @Override
    public int interpret() {
        // 数字的解释逻辑：直接返回自身值
        return this.number;
    }
}
// 非终结符表达式：加法表达式
public class AddExpression implements Expression {
    private Expression left; // 左表达式
    private Expression right; // 右表达式

    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        // 加法的解释逻辑：左表达式结果 + 右表达式结果
        return left.interpret() + right.interpret();
    }
}
// 非终结符表达式：减法表达式
public class SubtractExpression implements Expression {...}

// 客户端：解析表达式字符串，构建表达式树，并调用解释方法执行计算
public class InterpreterClient {
    public static void main(String[] args) {
        // 计算 1 + 2
        Expression exp1 = new AddExpression(
                new NumberExpression(1),
                new NumberExpression(2)
        );
        System.out.println("1 + 2 = " + exp1.interpret());
    }
}
```



## Mediator (中介者模式)

定义了一个中介对象来封装一系列对象之间的交互
中介者使各对象之间不需要显式地相互引用，从而使其耦合松散，复杂度降低，且可以独立地改变它们之间的交互



举个栗子：聊天室

```java
public class ChatRoom {
   public static void showMessage(User user, String message){
      System.out.println(new Date().toString()
         + " [" + user.getName() +"] : " + message);
   }
}

public class User {
   private String name;
   public User(String name){ this.name  = name;}
   public String getName() ...}
   public void setName(String name) {...}
 
   public void sendMessage(String message){
      ChatRoom.showMessage(this,message);
   }
}
```

MVC框架：控制器作为模型和视图的中介者



## Memento  (备忘录模式)

在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便在适当的时候恢复对象

- Originator（发起人）：负责创建一个备忘录，并记录当前对象的内部状态

- Memento（备忘录）：存储原发器对象的内部状态。备忘录可以包含原发器的一部分或全部状态

- Caretaker（管理者）：负责保存备忘录，但不能对备忘录的内容进行操作或检查



我们使用的几乎所有软件都用到了备忘录模式

最简单的备忘录模式就是保存到文件，打开文件
对于文本编辑器来说，保存就是把`TextEditor`类的字符串存储到文件，打开就是恢复`TextEditor`类的状态

```java
public class Memento {
   private String state;
   public Memento(String state){ this.state = state; }
   public String getState(){ return state; }  
}

public class Originator {
   private String state; 
   public void setState(String state){ this.state = state; }
   public String getState(){ return state; }
 
   public Memento saveStateToMemento(){ return new Memento(state);}
   public void getStateFromMemento(Memento Memento){ state = Memento.getState();}
}

public class CareTaker {
   private List<Memento> mementoList = new ArrayList<Memento>();
   public void add(Memento state){ mementoList.add(state);}
   public Memento get(int index){ return mementoList.get(index);}
}

// 使用
Originator originator = new Originator();
CareTaker careTaker = new CareTaker();
originator.setState("State #1");
originator.setState("State #2");
careTaker.add(originator.saveStateToMemento());
originator.setState("State #3");
careTaker.add(originator.saveStateToMemento());
originator.setState("State #4");

System.out.println("Current State: " + originator.getState());    
originator.getStateFromMemento(careTaker.get(0));
System.out.println("First saved State: " + originator.getState());
originator.getStateFromMemento(careTaker.get(1));
System.out.println("Second saved State: " + originator.getState());
```



## State (状态模式)

创建表示各种状态的对象和一个行为随着状态对象改变而改变的 context 对象
允许对象在内部状态改变时改变其**行为**，使得对象在不同的状态下有不同的行为表现
通过将每个状态封装成独立的类，可以**避免使用大量的条件语句**来实现状态切换
