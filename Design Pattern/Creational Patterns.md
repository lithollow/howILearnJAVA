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

public interface AbstractProduct {
	void doSomething();
}

public class productA implements AbstractProduct {
	@Override
	public void doSomething() {
		// sout
	}
}

public class Test {
	public static void main(String[] args) {
        AbstractFactory factory = new FactoryA();
        AbstractProduct product = factory.create();
        product.doSomething();
    }
}
```

