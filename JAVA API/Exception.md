## 一、概念

异常是程序在执行过程中出现的非正常情况，会**中断正常的执行流** 



| 特性     | 受检异常(Checked Exceptions)          | 非受检异常(Unchecked Exceptions)          |
| -------- | :------------------------------------ | ----------------------------------------- |
| 检查时机 | 编译时检查                            | 编译时不检查，运行时抛出                  |
| 处理要求 | 必须显式地捕获或声明抛出              | 可以不处理                                |
| 继承自   | `Exception`（除`RuntimeException`外） | `RuntimeException`和`Error`               |
| 常见例子 | `IOException`，`SQLException`         | `NullPointerException`,`OutOfMemoryError` |
| 使用场景 | 外部因素，可恢复的异常                | 编程错误或系统错误，通常不可恢复          |



```
Throwable
    ├── Error (不可处理)
    │     ├── OutOfMemoryError
    │     ├── StackOverflowError
    │     └── VirtualMachineError
    │
    └── Exception (可处理)
          ├── RuntimeException (非受检异常)
          │     ├── NullPointerException
          │     ├── IllegalArgumentException
          │     ├── IndexOutOfBoundsException
          │     └── ArithmeticException
          │
          └── 其他Exception (受检异常)
                ├── IOException
                ├── SQLException
                └── FileNotFoundException
```



---



## 二、处理

##### 1. try-catch-finally 结构

```
try{
	// 可能出现异常的代码
}Catch(Exception e){
	// TODO: 处理异常，若catch没有捕获到则不执行此块
}finally{
	// finally块总是执行
	// close()关闭连接、资源释放等
	// 避免在finally中返回，会覆盖try中的返回值
}
```

```
try{
	Object obj = null;
	obj.toString();
}Catch(NullPointerException e){
	// 若要针对不同异常做不同处理，则不用Exception
	// 避免空 catch 块，至少记录异常
	System.out.println("空指针异常: " + e.getMessage());
}catch (ClassCastException | IllegalArgumentException e) {
	// 多重捕获(java 7+)
    System.out.println("捕获异常: " + e.getClass().getSimpleName());
}Catch(Exception e){
	// 具体异常优先，最后处理通用异常
	logger.error("异常", e);
}...
```



##### 2. try-with-resources (Java 7+)

**传统资源管理：**使用 try-catch-finally 结构，最终在 `finally{}` 显式调用 `close()`

```
FileOutputStream fos = null;
try{
	fos = new FileOutputStream("d:/output.txt");
	// TODO 向文件输出内容
}catch(FileNotFoundException e){
	System.out.println("文件没找到: " + e.getMessage());
}finally{
	try{
		fos.close();
	}catch(IOException e){
		System.out.println("文件关闭错误: " + e.getMessage());
	}
}
```

**try-with-resources 方式：**

- 自动调用`close()`方法，关闭 `try()` 内定义的所有资源对象
  - 先 1 再 2 顺序创建，先 2 再 1 逆序关闭
- 只有实现了 `AutoCloseable` 接口的类的对象才能被当成资源，在 `try()` 里创建对象，最后关闭

```
try(FileOutputStream fos1 = new FileOutputStream("d:/output.txt");
	FileOutputStream fos2 = new FileOutputStream("d:/output.txt")){
	// TODO 向文件输出内容
}catch(Exception e){
	// 如果方法声明了Exception，可以省略catch
    System.out.println("捕获异常: " + e.getMessage());
}
```



##### 3. throws

- 主动抛出异常。并没有处理，而是把异常处理责任**传递给调用者**
  - `throw`语句可以在方法体内的任何地方使用，但一旦执行到`throw`语句，方法会**立即终止**，并将异常抛给调用者

- 抛出**非受检异常(`RuntimeException`)**的方法不需要声明该类异常(~~`throws RuntimeException`~~)
  - 非受检异常不要求强制捕获

```
static void methodA() {
	// NullPointerException 是 RuntimeException
    throw new NullPointerException();
}
```

- **受检异常(编译器异常)**，可以1. 现场处理(`try-catch`)

  ​						 2. `throw`抛出（受检异常所在的方法必须声明(`throws IOException`)

  - 处理了异常的调用者不再`throws`声明异常

```
// IOException 和 SQLException 都是 受检异常
static void methodB() throws IOException,SQLException{
    int i = 1;
    if(i == 0) { throw new IOException();}
    if(i == 1) { throw new SQLException();}
    else { throw new NullPointerException();}	// 空指针异常是非受检异常，无需声明
}
--------------------------------------------------------
public static void methodC() throws IOException {
    throw new IOException("底层IO异常");
}
public static void methodD() throws IOException {}

public static void main(String[] args) {
    try {
    	// 两个方法在声明上都声明了异常，它们的调用者都必须进行处理(or继续throw)
        methodD(); // or methodC();
    } catch (IOException e) {
    	// D没有抛出，调用者实际上永远不会收到这个异常
        // 这里的代码不会执行
    }
}
```



---



## 三、自定义异常

Java提供的异常都有特定的含义，开发中有更具体更实际的异常(service层throw，controller层catch)  

- 非受检异常 `extends RuntimeException` 
- 受检异常 `extends Exception` 

