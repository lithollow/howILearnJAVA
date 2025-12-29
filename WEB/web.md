# 1 入门

## 1.1 C/S 架构与 B/S 架构

**Client/Server**：客户端运行程序，负责处理业务逻辑与用户的交互任务；服务器负责管理数据

**Browser/Server**：程序完全部署在服务器，用户通过浏览器访问应用程序完成用户交互；服务器即负责 管理数据 又负责处理 业务逻辑

**对比**：

- B/S开发更标准，开发成本低，因为C/S需要在不同的系统上执行，B/S只需要在浏览器上执行

- C/S性能高，很多程序已经安装到本地，只需要联网获取数据，但因此软件升级维护成本高
- C/S 一般建立在专用的网络上, B/S 建立在 广域网 (互联网)之上的

- B/S更为安全，由于程序部署在 服务器端，没有感染病毒的风险

## 1.2 HTTP  协议

HTTP是在网络上传输HTML的协议，用于浏览器和服务器的通信

- 在Web应用中，浏览器请求一个URL，服务器就把生成的HTML网页发送给浏览器，而浏览器和服务器之间的传输协议是HTTP



HTTP协议是一个基于TCP协议之上的请求-响应协议，对于Browser来说，请求页面的流程如下：

1. 与服务器建立TCP连接
2. 发送HTTP请求
3. 收取HTTP响应，然后把网页在浏览器中显示出来



编写HTTP Server：

```java
// HTTP Server本质上是一个TCP服务器 使用TCP编程的多线程实现的服务器端框架
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(8080); // 监听指定端口
        System.out.println("server is running...");
        for (;;) {
            Socket sock = ss.accept();
            System.out.println("connected from " + sock.getRemoteSocketAddress());
            Thread t = new Handler(sock);
            t.start();
        }
    }
}

class Handler extends Thread {
    Socket sock;

    public Handler(Socket sock) {
        this.sock = sock;
    }

    public void run() {
        try (InputStream input = this.sock.getInputStream()) {
            try (OutputStream output = this.sock.getOutputStream()) {
                handle(input, output);
            }
        } catch (Exception e) {
        } finally {
            try {
                this.sock.close();
            } catch (IOException ioe) {
            }
            System.out.println("client disconnected.");
        }
    }

    // 用Reader读取HTTP请求，用Writer发送HTTP响应
    private void handle(InputStream input, OutputStream output) throws IOException {
        System.out.println("Process new http request...");
        var reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
        var writer = new BufferedWriter(new OutputStreamWriter(output, StandardCharsets.UTF_8));
        // 读取HTTP请求:
        boolean requestOk = false;
        String first = reader.readLine();
        if (first.startsWith("GET / HTTP/1.")) {	// 只处理GET /的请求
            requestOk = true;
        }
        for (;;) {
            String header = reader.readLine();
            if (header.isEmpty()) { // 读取到空行时, HTTP Header读取完毕 请求结束，可以发送响应
                break;
            }
            System.out.println(header);
        }
        System.out.println(requestOk ? "Response OK" : "Response Error");
        if (!requestOk) {
            // 发送错误响应:
            writer.write("HTTP/1.0 404 Not Found\r\n");
            writer.write("Content-Length: 0\r\n");
            writer.write("\r\n");
            writer.flush();
        } else {
            // 发送成功响应:
            String data = "<html><body><h1>Hello, world!</h1></body></html>";
            int length = data.getBytes(StandardCharsets.UTF_8).length;
            writer.write("HTTP/1.0 200 OK\r\n");
            writer.write("Connection: close\r\n");
            writer.write("Content-Type: text/html\r\n");
            writer.write("Content-Length: " + length + "\r\n");
            writer.write("\r\n"); // 空行标识Header和Body的分隔
            writer.write(data);
            writer.flush();
        }
    }
}

```



HTTP 1.0 浏览器每次建立TCP连接后，只发送一个HTTP请求并接收一个HTTP响应，然后就关闭TCP连接

HTTP 1.1允许浏览器和服务器在同一个TCP连接上反复发送、接收多个HTTP请求和响应

HTTP 2.0可以支持浏览器同时发出多个请求，但每个请求需要唯一标识，服务器可以不按请求的顺序返回多个响应，由浏览器自己把收到的响应和请求对应起来

HTTP 3.0为了进一步提高速度，抛弃TCP协议，改为使用无需创建连接的UDP协议(实验阶段)



---



# 2 Servlet

## 2.1 简介

要编写一个完善的HTTP服务器，以HTTP/1.1为例，需要考虑的包括：

- 识别正确和错误的HTTP请求；
- 识别正确和错误的HTTP头；
- 复用TCP连接；
- 复用线程；
- IO异常处理；
- ...

这样无疑是低效的



现在，把处理TCP连接，解析HTTP协议这些底层工作交给现成的Web服务器去做，开发者只需要把自己的应用程序跑在Web服务器上

使用 Java 提供的 Servlet API，编写自己的Servlet来处理HTTP请求，Web服务器实现Servlet API接口



一个最简单的Servlet：

```java
// WebServlet注解表示这是一个Servlet，并映射到地址/:
// Servlet总是继承自HttpServlet，然后覆写doGet()或doPost()方法
@WebServlet(urlPatterns = "/")
public class HelloServlet extends HttpServlet {
    // doGet 方法传入的两个对象分别代表HTTP请求和响应
    // 使用Servlet API时，并不直接与底层TCP交互，也不需要解析HTTP协议，因为HttpServletRequest和HttpServletResponse就已经封装好了请求和响应
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // 设置响应类型:
        resp.setContentType("text/html");
        // 获取输出流:
        PrintWriter pw = resp.getWriter();
        // 写入响应:
        pw.write("<h1>Hello, world!</h1>");
        // 最后不要忘记flush强制输出:
        pw.flush();
    }
}
```



#### HttpServletRequest

`HttpServletRequest`封装了一个HTTP请求，从`ServletRequest`继承而来

通过`HttpServletRequest`提供的接口方法可以拿到HTTP请求的几乎全部信息

- getMethod()：返回请求方法，例如，`"GET"`，`"POST"`
- getRequestURI()：返回请求路径，但不包括请求参数，例如，`"/hello"`
- getQueryString()：返回请求参数，例如，`"name=Bob&a=1&b=2"`
- getParameter(name)：返回请求参数，GET请求从URL读取参数，POST请求从Body中读取参数
- getContentType()：获取请求Body的类型，例如，`"application/x-www-form-urlencoded"`
- getContextPath()：获取当前Webapp挂载的路径，对于ROOT来说，总是返回空字符串`""`
- getCookies()：返回请求携带的所有Cookie
- getHeader(name)：获取指定的Header，对Header名称不区分大小写
- getHeaderNames()：返回所有Header名称
- getInputStream()：如果该请求带有HTTP Body，该方法将打开一个输入流用于读取Body
- getReader()：和getInputStream()类似，但打开的是Reader
- getRemoteAddr()：返回客户端的IP地址
- getScheme()：返回协议类型，例如，`"http"`，`"https"`

- `setAttribute()`和`getAttribute()`: 给当前`HttpServletRequest`对象附加多个Key-Value，相当于把`HttpServletRequest`当作一个`Map<String, Object>`使用



#### HttpServletResponse

封装了一个HTTP响应

由于HTTP响应必须先发送Header，再发送Body，所以，操作`HttpServletResponse`对象时，必须先调用设置Header的方法，最后调用发送Body的方法



常用设置Header的方法有：

- setStatus(sc)：设置响应代码，默认是`200`
- setContentType(type)：设置Body的类型，例如，`"text/html"`
- setCharacterEncoding(charset)：设置字符编码，例如，`"UTF-8"`
- setHeader(name, value)：设置一个Header的值
- addCookie(cookie)：给响应添加一个Cookie
- addHeader(name, value)：给响应添加一个Header，因为HTTP协议允许有多个相同的Header

1. 写入响应前，无需设置`setContentLength()`，因为底层服务器会根据写入的字节数自动设置，如果写入的数据量很小，实际上会先写入缓冲区，如果写入的数据量很大，服务器会自动采用Chunked编码让浏览器能识别数据结束符而不需要设置Content-Length头

2. 写入响应时，需要通过`getOutputStream()`获取写入流，或者通过`getWriter()`获取字符流，二者只能获取其中一个

3. 写入完毕后必须调用`flush()`，否则将导致缓冲区的内容无法及时发送到客户端

​	写入完毕后千万不要调用`close()`，如果关闭写入流，将关闭TCP连接，使得Web服务器无法复用此TCP连接



## 2.2 Servlet 入门

1. 通过maven引入jar包

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>web-servlet-hello</artifactId>
    <!-- 打包类型是war，表示Java Web Application Archive，而非普通普通Java程序的 jar -->
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>5.0.0</version>
            <!-- provided 编译时使用 不打包到.war文件 运行期Web服务器本身已经提供了Servlet API相关的jar包 -->
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>hello</finalName>
    </build>
</project>
```

2. 运行`war`文件：

Web应用程序有所不同，无法直接运行`war`文件

必须先启动Web服务器，再由Web服务器加载我们编写的`HelloServlet`

这样就可以让`HelloServlet`处理浏览器发送的请求



常用的支持Servlet API的Web服务器

- [Tomcat](https://tomcat.apache.org/)：由Apache开发的开源免费服务器
- [Jetty](https://www.eclipse.org/jetty/)：由Eclipse开发的开源免费服务器
- [GlassFish](https://javaee.github.io/glassfish/)：一个开源的全功能JavaEE服务器



运行hello.war：

- 下载Tomcat服务器后，把`hello.war`复制到Tomcat的`webapps`目录下，然后切换到`bin`目录，执行`startup.sh`或`startup.bat`启动Tomcat服务器

- 浏览器输入`http://localhost:8080/hello/`即可看到`HelloServlet`的输出



实际上，类似Tomcat这样的服务器也是Java编写的
启动Tomcat服务器实际上是启动Java虚拟机，执行Tomcat的`main()`方法，然后由Tomcat负责加载我们的`.war`文件，并创建一个`HelloServlet`实例，最后以多线程的模式来处理HTTP请求
如果Tomcat服务器收到的请求路径是`/`（假定部署文件为ROOT.war），就转发到`HelloServlet`并传入`HttpServletRequest`和`HttpServletResponse`两个对象

因为我们编写的Servlet并不是直接运行，而是由Web服务器加载后创建实例运行，所以，类似Tomcat这样的Web服务器也称为Servlet容器



## 2.3 Servlet 开发

一个完整的Web应用程序的开发流程如下：

1. 编写Servlet
2. 打包为war文件
3. 复制到Tomcat的webapps目录下
4. 启动Tomcat

显然太繁琐



直接在IDE中启动并调试webapp：

1. 新的`web-servlet-embedded`工程，编写`pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>web-servlet-embedded</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <java.version>17</java.version>
        <tomcat.version>10.1.1</tomcat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>${tomcat.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>${tomcat.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

2. 编写Servlet，同上HelloServlet 
3. 编写`main()`方法，启动Tomcat服务器

```java
// 直接运行main()方法，即可启动嵌入式Tomcat服务器
// 通过预设的tomcat.addWebapp("", new File("src/main/webapp")，Tomcat会自动加载当前工程作为根webapp，可直接在浏览器访问http://localhost:8080/
public class Main {
    public static void main(String[] args) throws Exception {
        // 启动Tomcat:
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(Integer.getInteger("port", 8080));
        tomcat.getConnector();
        // 创建webapp:
        Context ctx = tomcat.addWebapp("", new File("src/main/webapp").getAbsolutePath());
        WebResourceRoot resources = new StandardRoot(ctx);
        resources.addPreResources(
                new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));
        ctx.setResources(resources);
        tomcat.start();
        tomcat.getServer().await();
    }
}
```

4. 生成可执行war包：略



## 2.4 Servlet 进阶

#### dispatch

一个Web App就是由一个或多个Servlet组成的，每个Servlet通过注解说明自己能处理的路径

```java
@WebServlet(urlPatterns = "/hello")
public class HelloServlet extends HttpServlet { ...}

@WebServlet(urlPatterns = "/signin")
public class SignInServlet extends HttpServlet { ...}

@WebServlet(urlPatterns = "/")
public class IndexServlet extends HttpServlet { ...}

// 处理GET请求 就覆写doGet()方法
// 处理POST请求 就覆写doPost()方法
// 这种根据路径转发的功能我们一般称为dispatch
```

浏览器发出的HTTP请求总是由Web Server先接收，然后，根据Servlet配置的映射，不同的路径转发到不同的Servlet

```ascii art
               ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐

               │            /hello    ┌───────────────┐│
                          ┌──────────▶│ HelloServlet  │
               │          │           └───────────────┘│
┌───────┐    ┌──────────┐ │ /signin   ┌───────────────┐
│Browser│───▶│Dispatcher│─┼──────────▶│ SignInServlet ││
└───────┘    └──────────┘ │           └───────────────┘
               │          │ /         ┌───────────────┐│
                          └──────────▶│ IndexServlet  │
               │                      └───────────────┘│
                              Web Server
               └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```



#### Servlet多线程模型

一个Servlet类在服务器中只有一个实例，但对于每个HTTP请求，Web服务器会使用多线程执行请求

因此，一个Servlet的`doGet()`、`doPost()`等处理请求的方法是多线程并发执行的

如果Servlet中定义了字段，要注意多线程并发访问的问题

```java
public class HelloServlet extends HttpServlet {
    private Map<String, String> map = new ConcurrentHashMap<>();

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 注意读写map字段是多线程并发的:
        this.map.put(key, value);
    }
}
```

对于每个请求，Web服务器会创建唯一的`HttpServletRequest`和`HttpServletResponse`实例，因此，`HttpServletRequest`和`HttpServletResponse`实例只有在当前处理线程中有效，它们总是局部变量，不存在多线程共享的问题



#### 重定向与转发

**Redirect** 重定向是指当浏览器请求一个URL时，服务器返回一个重定向指令，告诉浏览器地址已经变了，麻烦使用新的URL再重新发送新请求

```java
// 收到的路径为/hi 重定向到/hello
@WebServlet(urlPatterns = "/hi")
public class RedirectServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 构造重定向的路径:
        String name = req.getParameter("name");
        String redirectToUrl = "/hello" + (name == null ? "" : "?name=" + name);
        // resp.setStatus(HttpServletResponse.SC_MOVED_PERMANENTLY); // 301
		// resp.setHeader("Location", "/hello");
        // 发送重定向响应:
        resp.sendRedirect(redirectToUrl);
    }
}
// 浏览器发送GET/hi请求 -> 由RedirectServlet处理 -> RedirectServlet内部发送重定向响应 -> 
// 浏览器收到响应 HTTP/1.1 302 Found Location: /hello ->
// 浏览器根据Location的指示发送 GET/hello请求
// 如果是301永久重定向，则 请求/hi的时候，浏览器直接发送/hello请求
```

重定向的目的是当Web应用升级后，如果请求路径发生了变化，可以将原来的路径重定向到新路径，从而避免浏览器请求原路径找不到资源

**Forward** 内部转发：当一个Servlet处理请求的时候，它可以决定自己不继续处理，而是转发给另一个Servlet处理

```java
@WebServlet(urlPatterns = "/morning")
public class ForwardServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 收到请求后，并不自己发送响应，而是把请求和响应都转发给路径为/hello的Servlet
        req.getRequestDispatcher("/hello").forward(req, resp);
    }
}

```

| Redirect                                             | Forward                                         |
| ---------------------------------------------------- | ----------------------------------------------- |
| 浏览器发出两次请求，第一次的响应只得到了重定向的指示 | 在Web服务器内部完成，浏览器只发出了一个HTTP请求 |



#### Session & Cookie

HTTP协议是一个无状态协议，即Web应用程序无法区分收到的两个HTTP请求是否是同一个浏览器发出的

要跟踪用户状态，服务器可以向浏览器分配一个唯一ID，并以Cookie的形式发送到浏览器，浏览器在后续访问时总是附带此Cookie，这样，服务器就可以识别用户身份



1. **Session**

我们把这种基于唯一ID识别用户身份的机制称为**Session**

每个用户第一次访问服务器后，会自动获得一个Session ID

如果用户在一段时间内没有访问服务器，那么Session会自动失效

```java
@WebServlet(urlPatterns = "/signin")
public class SignInServlet extends HttpServlet {
    // 模拟一个数据库:
    private Map<String, String> users = Map.of("bob", "bob123", "alice", "alice123", "tom", "tomcat");

    // GET请求时显示登录页:
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter pw = resp.getWriter();
        pw.write("<h1>Sign In</h1>");
        pw.write("<form action=\"/signin\" method=\"post\">");
        pw.write("<p>Username: <input name=\"username\"></p>");
        pw.write("<p>Password: <input name=\"password\" type=\"password\"></p>");
        pw.write("<p><button type=\"submit\">Sign In</button> <a href=\"/\">Cancel</a></p>");
        pw.write("</form>");
        pw.flush();
    }

    // POST请求时处理用户登录:
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String name = req.getParameter("username");
        String password = req.getParameter("password");
        String expectedPassword = users.get(name.toLowerCase());
        if (expectedPassword != null && expectedPassword.equals(password)) {
            // 登录成功: 把这个用户的名字放入一个HttpSession对象
            req.getSession().setAttribute("user", name);
            resp.sendRedirect("/");
        } else {
            resp.sendError(HttpServletResponse.SC_FORBIDDEN);
        }
    }
}
// 从HttpSession获取当前用户
@WebServlet(urlPatterns = "/")
public class IndexServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 从HttpSession获取当前用户名:
        String user = (String) req.getSession().getAttribute("user");
        resp.setContentType("text/html");
        resp.setCharacterEncoding("UTF-8");
        resp.setHeader("X-Powered-By", "JavaEE Servlet");
        PrintWriter pw = resp.getWriter();
        pw.write("<h1>Welcome, " + (user != null ? user : "Guest") + "</h1>");
        if (user == null) {
            // 未登录，显示登录链接:
            pw.write("<p><a href=\"/signin\">Sign In</a></p>");
        } else {
            // 已登录，显示登出链接:
            pw.write("<p><a href=\"/signout\">Sign Out</a></p>");
        }
        pw.flush();
    }
}
// 登出 从HttpSession中移除用户相关信息
@WebServlet(urlPatterns = "/signout")
public class SignOutServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 从HttpSession移除用户名:
        req.getSession().removeAttribute("user");
        resp.sendRedirect("/");
    }
}
```

Session 原理：

```ascii art
可以认为Web服务器在内存中自动维护了一个ID到HttpSession的映射表
           ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
           │      ┌───────────────┐                │
             ┌───▶│ IndexServlet  │◀──────────┐
           │ │    └───────────────┘           ▼    │
┌───────┐    │    ┌───────────────┐      ┌────────┐
│Browser│──┼─┼───▶│ SignInServlet │◀────▶│Sessions││
└───────┘    │    └───────────────┘      └────────┘
           │ │    ┌───────────────┐           ▲    │
             └───▶│SignOutServlet │◀──────────┘
           │      └───────────────┘                │
           └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
服务器依靠一个名为JSESSIONID的Cookie识别Session
在Servlet中第一次调用req.getSession()时，Servlet容器自动创建一个Session ID，然后通过一个名为JSESSIONID的Cookie发送给浏览器
由于服务器把所有用户的Session都存储在内存中，放入Session的对象要尽量小

粘滞会话（Sticky Session）机制：反向代理在转发请求的时候，总是根据JSESSIONID的值判断，相同的JSESSIONID总是转发到固定的Web Server，（需要反向代理的支持
	避免多台Web Server集群间进行Session复制消耗带宽
```





2. **Cookie**

Servlet提供的`HttpSession`本质上就是通过一个名为`JSESSIONID`的Cookie来跟踪用户会话，除了这个名称外，其他名称的Cookie我们可以任意使用

```java
// 设置一个Cookie 记录用户选择的语言
@WebServlet(urlPatterns = "/pref")
public class LanguageServlet extends HttpServlet {

    private static final Set<String> LANGUAGES = Set.of("en", "zh");

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String lang = req.getParameter("lang");
        if (LANGUAGES.contains(lang)) {
            // 创建一个新的Cookie:
            Cookie cookie = new Cookie("lang", lang);
            // 该Cookie生效的路径范围:
            cookie.setPath("/");
            // 该Cookie有效期:
            cookie.setMaxAge(8640000); // 8640000秒=100天
            // cookie.setSecure(true) // 访问https网页需要
            // 将该Cookie添加到响应:
            resp.addCookie(cookie);
        }
        resp.sendRedirect("/");
    }
}
// 读取Cookie
private String parseLanguageFromCookie(HttpServletRequest req) {
    // 获取请求附带的所有Cookie:
    Cookie[] cookies = req.getCookies();
    // 如果获取到Cookie: 遍历HttpServletRequest附带的所有Cookie
    if (cookies != null) {
        // 循环每个Cookie:
        for (Cookie cookie : cookies) {
            // 如果Cookie名称为lang:
            if (cookie.getName().equals("lang")) {
                // 返回Cookie的值:
                return cookie.getValue();
            }
        }
    }
    // 返回默认值:
    return "en";
}

```



---



# 3 JSP

之前的Servlet发送响应：获取`PrintWriter`，然后输出HTML：

```java
PrintWriter pw = resp.getWriter();
pw.write("<html>");
pw.write("<body>");
pw.write("<h1>Welcome, " + name + "!</h1>");
pw.write("</body>");
pw.write("</html>");
pw.flush();
```

麻烦



使用Java Server Pages (JSP)：

文件必须放到`/src/main/webapp`下，文件名必须以`.jsp`结尾

整个文件与HTML类似，但需要插入变量，或者动态输出的地方，使用特殊指令`<% ... %>`

```jsp
<!-- hello.jsp -->
<%@ page import="java.io.*" %>
<%@ page import="java.util.*" %>
<html>
<head>
    <title>Hello World - JSP</title>
</head>
<body>
    <%@ include file="header.jsp"%>
    
    <%-- JSP的注释，被完全忽略 --%>
    <h1>Hello World!</h1>
    <p>
    <%	// java 代码块
         out.println("Your IP address is ");
    %>
    <span style="color:red">
        <%= request.getRemoteAddr() %>
    </span>
    </p>
    
    <%@ include file="footer.jsp"%>
</body>
</html>
```

JSP页面内置了几个变量：

- out：表示HttpServletResponse的PrintWriter
- session：表示当前HttpSession对象
- request：表示HttpServletRequest对象

这几个变量可以直接使用



访问JSP页面时，直接指定完整路径 例如，`http://localhost:8080/hello.jsp`



**JSP本质上就是一个Servlet**，只不过无需配置映射路径，Web Server会根据路径查找对应的`.jsp`文件，如果找到了，就自动编译成Servlet再执行

在服务器运行过程中，如果修改了JSP的内容，那么服务器会自动重新编译

在Tomcat的临时目录下，可以找到一个`hello_jsp.java`的源文件，这个文件就是Tomcat把JSP自动转换成的Servlet源码



---



# 4 MVC 开发

- Servlet适合编写Java代码，实现各种复杂的业务逻辑，但不适合输出复杂的HTML
- JSP适合编写HTML，并在其中插入动态内容，但不适合编写复杂的Java代码

结合：

```java
public class User {
    public long id;
    public String name;
    public School school;
}

public class School {
    public String name;
    public String address;
}
// Servlet
@WebServlet(urlPatterns = "/user")
public class UserServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 假装从数据库读取:
        School school = new School("No.1 Middle School", "101 South Street");
        User user = new User(123, "Bob", school);
        // 放入HttpServletRequest:
        req.setAttribute("user", user);
        // 通过forward传给user.jsp处理
        req.getRequestDispatcher("/WEB-INF/user.jsp").forward(req, resp);
    }
}
// JSP 只负责展示相关JavaBean的信息，不需要编写访问数据库等复杂逻辑
<%@ page import="com.itranswarp.learnjava.bean.*"%>
<% User user = (User) request.getAttribute("user"); %>
<html>
<head>
    <title>Hello World - JSP</title>
</head>
<body>
    <h1>Hello <%= user.name %>!</h1>
    <p>School Name:
        <span style="color:red">
            <%= user.school.name %>
        </span>
    </p>
    <p>School Address:
        <span style="color:red">
            <%= user.school.address %>
        </span>
    </p>
</body>
</html>


```

MVC架构：

- Controller专注于业务处理，它的处理结果就是Model
- Model可以是一个JavaBean，也可以是一个包含多个对象的Map
- View负责对Controller传来的Model进行“渲染”

```ascii art
                   ┌───────────────────────┐
             ┌────▶│Controller: UserServlet│
             │     └───────────────────────┘
             │                 │
┌───────┐    │           ┌─────┴─────┐
│Browser│────┘           │Model: User│
│       │◀───┐           └─────┬─────┘
└───────┘    │                 │
             │                 ▼
             │     ┌───────────────────────┐
             └─────│    View: user.jsp     │
                   └───────────────────────┘
```



---



# 5 Filter

使用 Filter 来对请求进行预处理：

在Web应用中经常需要处理用户上传文件，抽取其中对文件上传完整性的验证逻辑到 Filter，以便复用



使用 Filter 来对响应进行预处理：

抽取可复用的逻辑到Filter：缓存每次返回的响应中的固定内容，以提升Web应用程序的运行效率



---



# 6 Listener

监听器



---



# 7 部署

















1. Java Servlet 是Java编写的服务器端程序，获取并针对Web客户端的请求作出响应

2. Servlet运行于支持Java的应用服务器中。从实现上讲，Servlet可以响应任何类型的请求，**但绝大多数情况下，Servlet只用来扩展基于HTTP协议的Web服务器**

3. Servlet 工作模式：

   ① 客户端发送请求至服务器

   ② 服务器启动并调用Servlet，Servlet根据客户端请求生成响应内容并将其传给服务器

   ③ 服务器将响应返回客户端

## 2.2 Servlet API



