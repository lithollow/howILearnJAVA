# 1 基础

把计算机网络接入互联网，就必须使用TCP/IP协议

TCP/IP协议泛指互联网协议，其中最重要的两个协议是TCP协议和IP协议

## 1.1 IP 地址

在互联网中，一个IP地址用于唯一标识一个网络接口（Network Interface）

一台联入互联网的计算机肯定有一个IP地址，但也可能有多个IP地址

如果一台计算机只有一个网卡，并且接入了网络，那么，它有一个本机地址`127.0.0.1`，还有一个IP地址，例如`101.202.99.12`，可以通过这个IP地址接入网络



IP地址分为

1. IPv4：32位地址，类似`101.202.99.12`；实际上是一个32位整数，总共有大约42亿个，目前已耗尽

2. IPv6：128位地址，类似`2001:0DA8:100A:0000:0000:1020:F2F3:1428`

   a. 公网IP地址：可以直接被访问，

   b. 内网IP地址：只能在内网访问，形如`192.168.x.x`, `10.x.x.x`



如果两台计算机位于同一个网络，那么他们之间可以直接通信，因为他们的IP地址前段是相同的，也就是网络号是相同的

IP地址通过子网掩码过滤得到网络号

```
IP = 101.202.99.2
Mask = 255.255.255.0
Network = IP & Mask = 101.202.99.0
```

如果两台计算机计算出的网络号不同，那么两台计算机不在同一个网络，不能直接通信，它们之间必须通过路由器或者交换机这样的网络设备间接通信，我们把这种设备称为**网关**

网关的作用就是连接多个网络，负责把来自一个网络的数据包发到另一个网络，这个过程叫**路由**

## 1.2 域名

记忆IP地址是很难的，所以我们通常使用域名访问某个特定的服务

**DNS 域名解析服务器 **负责把域名翻译成对应的IP，客户端再根据IP地址访问服务器

本机域名`localhost`，对应本机地址`127.0.0.1`

## 1.3 网络模型

OSI（Open System Interconnect）网络模型：ISO组织定义的一个计算机互联的标准模型，7层

- 应用层，提供应用程序之间的通信；
- 表示层：处理数据格式，加解密等等；
- 会话层：负责建立和维护会话；
- 传输层：负责提供端到端的可靠传输；
- 网络层：负责根据目标地址选择路由来传输数据；
- 链路层和物理层负责把数据进行分片并且真正通过物理网络传输，例如，无线网、光纤等。



互联网实际使用的TCP/IP模型：大致对应OSI的5层模型

- 应用层 - 应用、表示、会话

- 传输层 - 传输
- IP层 - 网络
- 网络接口层 - 链路、物理



- IP协议是一个分组交换，只负责发数据包，不保证顺序和正确性
- TCP协议是传输控制协议，它是面向连接的协议，支持可靠传输和双向通信
  - TCP协议建立在IP协议之上，在传输数据之前需要先建立连接，然后传输数据，传输完后还需要断开连接
- UDP协议数据报文协议，无连接协议，不保证可靠传输
  - 因为UDP协议在通信前不需要建立连接，因此它的传输效率比TCP高
  - UDP协议时，传输的数据通常是能容忍丢失的



---



# 2 TCP 编程

### Socket

个应用程序通过一个Socket来建立一个远程连接，而Socket内部通过TCP/IP协议把数据传输到网络

```ascii
┌───────────┐                                 ┌───────────┐
│Application│                                 │Application│
├───────────┤                                 ├───────────┤
│  Socket   │                                 │  Socket   │
├───────────┤                                 ├───────────┤
│    TCP    │                                 │    TCP    │
├───────────┤      ┌──────┐      ┌──────┐     ├───────────┤
│    IP     │◀───▶│Router│◀───▶│Router│◀───▶│    IP     │
└───────────┘      └──────┘      └──────┘     └───────────┘
```

当操作系统接收到一个数据包的时候，如果只有IP地址，它没法判断应该发给哪个应用程序，所以，操作系统抽象出Socket接口，每个应用程序需要各自对应到不同的Socket，数据包才能根据Socket正确地发到对应的应用程序



一个Socket就是由IP地址和端口号（范围是0～65535）组成，端口号总是由操作系统分配

小于1024的端口属于*特权端口*，需要管理员权限，大于1024的端口可以由任意用户的应用程序打开



使用Socket进行网络编程时，本质上就是两个进程之间的网络通信

其中一个进程必须充当服务器端，它会主动监听某个指定的端口

​	服务器端的Socket是指定的IP地址和指定的端口号

另一个进程必须充当客户端，它必须主动连接服务器的IP地址和指定端口

​	客户端的Socket是它所在计算机的IP地址和一个由操作系统分配的随机端口号

如果连接成功，服务器端和客户端就成功地建立了一个TCP连接，双方后续就可以随时发送和接收数据



```java
public class Server {
    public static void main(String[] args) throws IOException {
    	// 监听指定端口 没有指定IP地址，表示在计算机的所有网络接口上进行监听
        ServerSocket ss = new ServerSocket(6666);
        System.out.println("server is running...");
        // 使用一个无限循环来处理客户端的连接
        for (;;) {
        	// 每当有新的客户端连接进来后，就返回一个Socket实例 用来和刚连接的客户端进行通信
            Socket sock = ss.accept();
            System.out.println("connected from " + sock.getRemoteSocketAddress());
            // 由于客户端很多，要为每个新的Socket创建一个新线程来实现并发处理
            // 主线程的作用就是接收新的连接，每当收到新连接后，就创建一个新线程进行处理
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

    @Override
    public void run() {
        try (InputStream input = this.sock.getInputStream()) {
            try (OutputStream output = this.sock.getOutputStream()) {
                handle(input, output);
            }
        } catch (Exception e) {
            try {
                this.sock.close();
            } catch (IOException ioe) {}
            System.out.println("client disconnected.");
        }
    }

    private void handle(InputStream input, OutputStream output) throws IOException {
        var writer = new BufferedWriter(new OutputStreamWriter(output, StandardCharsets.UTF_8));
        var reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
        writer.write("hello\n");
        writer.flush();
        for (;;) {
            String s = reader.readLine();
            if (s.equals("bye")) {
                writer.write("bye\n");
                writer.flush();
                break;
            }
            writer.write("ok: " + s + "\n");
            writer.flush();
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) throws IOException {
    	// 连接指定服务器 IP地址和端口，连接成功则返回一个Socket实例
        Socket sock = new Socket("localhost", 6666);
        // TCP是一种基于流的协议 Java标准库使用InputStream和OutputStream来封装Socket的数据流
        // 读取网络数据
        try (InputStream input = sock.getInputStream()) {
        	// 写入网络数据
            try (OutputStream output = sock.getOutputStream()) {
                handle(input, output);
            }
        }
        sock.close();
        System.out.println("disconnected.");
    }

    private static void handle(InputStream input, OutputStream output) throws IOException {
        var writer = new BufferedWriter(new OutputStreamWriter(output, StandardCharsets.UTF_8));
        var reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
        Scanner scanner = new Scanner(System.in);
        System.out.println("[server] " + reader.readLine());
        for (;;) {
            System.out.print(">>> "); // 打印提示
            String s = scanner.nextLine(); // 读取一行输入
            writer.write(s);
            writer.newLine();
            // 强制发送 缓冲区数据
            writer.flush();
            String resp = reader.readLine();
            System.out.println("<<< " + resp);
            if (resp.equals("bye")) {
                break;
            }
        }
    }
}
```



### Socket流

当Socket连接创建成功后，无论是服务器端，还是客户端，我们都使用`Socket`实例进行网络通信

因为TCP是一种基于流的协议，因此，Java标准库使用`InputStream`和`OutputStream`来封装Socket的数据流

使用Socket的流，和普通IO流类似

`flush()`方法：强制发送缓冲区数据；如果不用，客户端和服务器都收不到数据：为了提高传输效率，以流的形式写入数据的时候，并不是一写入就立刻发送到网络，而是先写入内存缓冲区，直到缓冲区满了以后，才会一次性真正发送到网络



---



# 3 UDP 编程

UDP没有创建连接，数据包一次收发一个，所以没有流的概念

- 在Java中使用UDP编程，仍然需要使用Socket，因为应用程序在使用UDP时必须指定网络接口（IP）和端口号
  - UDP端口和TCP端口相互独立，一个应用程序用TCP占用了端口1234，不影响另一个应用程序用UDP占用端口1234



```java
// 服务器
DatagramSocket ds = new DatagramSocket(6666); // 监听指定端口
for (;;) { // 监听成功 无限循环 处理收到的UDP数据包
    // 数据缓冲区:
    byte[] buffer = new byte[1024];
    // 通过DatagramPacket实现接收
    DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
    ds.receive(packet); // 收取一个UDP数据包 假设收到一个String
    // 收取到的数据存储在buffer中，由packet.getOffset(), packet.getLength()指定起始位置和长度
    // 将其按UTF-8编码转换为String:
    String s = new String(packet.getData(), packet.getOffset(), packet.getLength(), StandardCharsets.UTF_8);
    // 发送数据:
    byte[] data = "ACK".getBytes(StandardCharsets.UTF_8);
    packet.setData(data);
    ds.send(packet);
}
```

```java
// 客户端
// 打开一个DatagramSocket 不需要指定端口，由操作系统自动指定一个当前未使用的端口
DatagramSocket ds = new DatagramSocket();
// 超时1秒
ds.setSoTimeout(1000);
// 在客户端的DatagramSocket实例中保存服务器端的IP和端口号，确保这个DatagramSocket实例只能往指定的地址和端口发送UDP包，不能往其他地址和端口发送
// 若不用connect() var packet1 = new DatagramPacket(data1, data1.length, InetAddress.getByName("localhost"), 6666);
ds.connect(InetAddress.getByName("localhost"), 6666); // 连接指定服务器和端口
// 发送:
byte[] data = "Hello".getBytes();
DatagramPacket packet = new DatagramPacket(data, data.length);
ds.send(packet);
// 接收:
byte[] buffer = new byte[1024];
packet = new DatagramPacket(buffer, buffer.length);
ds.receive(packet);
String resp = new String(packet.getData(), packet.getOffset(), packet.getLength());
// 清除客户端DatagramSocket实例记录的远程服务器地址和端口号
ds.disconnect();
// 关闭: 
ds.close();
```



---



# 4 发送 Email

用户电脑的邮箱软件 MUA：Mail User Agent 给用户服务的邮件代理

​		↓

邮件服务器 MTA：Mail Transfer Agent 邮件中转的代理

​		↓

对方邮件服务器：MDA：Mail Delivery Agent 意思是邮件到达的代理



电子邮件一旦到达MDA，就不再动了；电子邮件通常就存储在MDA服务器的硬盘上，然后等收件人通过软件或者登陆浏览器查看邮件



MTA和MDA这样的服务器软件通常是现成的，不需要关心这些服务器的运行

要发送邮件，我们关心的是如何编写一个MUA的软件，把邮件发送到MTA上

MUA到MTA发送邮件的协议就是SMTP(Simple Mail Transport Protocol)协议，标准端口`25`、加密端口`465`或`587`

SMTP协议是一个建立在TCP之上的协议，任何程序发送邮件都必须遵守SMTP协议

- QQ邮箱：SMTP服务器是`smtp.qq.com`，端口是465/587
- 163邮箱：SMTP服务器是`smtp.163.com`，端口是465
- Gmail邮箱：SMTP服务器是`smtp.gmail.com`，端口是465/587



使用JavaMail发送邮件

1. maven 引入依赖

- jakarta.mail:jakarta.mail-api:2.0.1
- com.sun.mail:jakarta.mail:2.0.1

2. ```java
   // 通过JavaMail API连接到SMTP服务器
   // 服务器地址:
   String smtp = "smtp.163.com";
   // 登录用户名:
   String username = "sky_xiaomi_li@163.com";
   // 登录口令:(并非密码 登陆邮箱-设置-STMP-授权码)
   String password = "CZmJt3Brx7DAKqr4";
   // 连接到SMTP服务器587端口:
   Properties props = new Properties();
   // 填入相关信息
   props.put("mail.smtp.host", smtp); // SMTP主机名
   props.put("mail.smtp.port", "587"); // 主机端口号
   props.put("mail.smtp.auth", "true"); // 是否需要用户认证
   props.put("mail.smtp.ssl.enable", "true"); // 启用SSL
   props.put("mail.smtp.starttls.enable", "true"); // 启用TLS加密
   props.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
   props.put("mail.smtp.socketFactory.port", "465");
   // 获取Session实例:
   Session session = Session.getInstance(props, new Authenticator() {
       protected PasswordAuthentication getPasswordAuthentication() {
           return new PasswordAuthentication(username, password);
       }
   });
   // 设置debug模式便于调试:
   session.setDebug(true);
   
   // 发送邮件
   MimeMessage message = new MimeMessage(session);
   // 设置发送方地址:
   message.setFrom(new InternetAddress("sky_xiaomi_li@163.com"));
   // 设置接收方地址:
   message.setRecipient(Message.RecipientType.TO, new InternetAddress("lithollow@foxmail.com"));
   // 设置邮件主题:
   message.setSubject("Hello", "UTF-8");
   // 设置邮件正文:
   message.setText("Hi from sky_xiaomi_li...", "UTF-8");
   // 发送:
   Transport.send(message);
   ```

3. 发送 html

   ```java
   message.setText(body, "UTF-8", "html");
   
   传入的body是类似<h1>Hello</h1><p>Hi, xxx</p>这样的HTML字符串即可
   ```

4. 发送附件

   ```java
   Multipart multipart = new MimeMultipart();
   // 一个Multipart对象可以添加若干个BodyPart
   // 第一个BodyPart是文本，即邮件正文
   BodyPart textpart = new MimeBodyPart();
   textpart.setContent(body, "text/html;charset=utf-8");
   multipart.addBodyPart(textpart);
   // 后面的BodyPart是附件
   BodyPart imagepart = new MimeBodyPart();
   imagepart.setFileName(fileName);
   imagepart.setDataHandler(new DataHandler(new ByteArrayDataSource(input, "application/octet-stream")));
   multipart.addBodyPart(imagepart);
   // 设置邮件内容为multipart:
   message.setContent(multipart);
   ```

5. 内嵌图片的HTML邮件

   ```java
   //内嵌图片实际上也是一个附件，即邮件本身也是Multipart
   Multipart multipart = new MimeMultipart();
   // 添加text:
   BodyPart textpart = new MimeBodyPart();
   textpart.setContent("<h1>Hello</h1><p><img src=\"cid:img01\"></p>", "text/html;charset=utf-8");
   multipart.addBodyPart(textpart);
   // 添加image:
   BodyPart imagepart = new MimeBodyPart();
   imagepart.setFileName(fileName);
   imagepart.setDataHandler(new DataHandler(new ByteArrayDataSource(input, "image/jpeg")));
   // 与HTML的<img src="cid:img01">关联:
   imagepart.setHeader("Content-ID", "<img01>");
   multipart.addBodyPart(imagepart);
   ```



---



# 5 接收 Email

接收邮件是收件人用自己的客户端把邮件从MDA服务器上抓取到本地的过程

接收邮件使用的协议：

- POP3（最广泛）：Post Office Protocol version 3，它也是一个建立在TCP连接之上的协议；标准端口是`110`，加密端口`995`
- IMAP：Internet Mail Access Protocol，它使用标准端口`143`和加密端口`993`
- 区别：IMAP协议在本地的所有操作都会自动同步到服务器上，并且，IMAP可以允许用户在邮件服务器的收件箱中创建文件夹



```java
// 这里使用 POP3 演示
// 首先连接到 Store 对象
// 准备登录信息:
String host = "pop3.example.com";
int port = 995;
String username = "bob@example.com";
String password = "password";

Properties props = new Properties();
props.setProperty("mail.store.protocol", "pop3"); // 协议名称
props.setProperty("mail.pop3.host", host);// POP3主机名
props.setProperty("mail.pop3.port", String.valueOf(port)); // 端口号
// 启动SSL:
props.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
props.put("mail.smtp.socketFactory.port", String.valueOf(port));

// 连接到Store:
URLName url = new URLName("pop3", host, post, "", username, password);
Session session = Session.getInstance(props, null);
session.setDebug(true); // 显示调试信息
Store store = new POP3SSLStore(session, url);
store.connect();

// 收取邮件
// 获取收件箱:
Folder folder = store.getFolder("INBOX");
// 以读写方式打开:
folder.open(Folder.READ_WRITE);
// 打印邮件总数/新邮件数量/未读数量/已删除数量:
System.out.println("Total messages: " + folder.getMessageCount());
System.out.println("New messages: " + folder.getNewMessageCount());
System.out.println("Unread messages: " + folder.getUnreadMessageCount());
System.out.println("Deleted messages: " + folder.getDeletedMessageCount());
// 获取每一封邮件:
Message[] messages = folder.getMessages();
for (Message message : messages) {
    // 打印每一封邮件:
    printMessage((MimeMessage) message);
}

// 格式化打印信息
void printMessage(MimeMessage msg) throws IOException, MessagingException {
    // 邮件主题:
    System.out.println("Subject: " + MimeUtility.decodeText(msg.getSubject()));
    // 发件人:
    Address[] froms = msg.getFrom();
    InternetAddress address = (InternetAddress) froms[0];
    String personal = address.getPersonal();
    String from = personal == null ? address.getAddress() : (MimeUtility.decodeText(personal) + " <" + address.getAddress() + ">");
    System.out.println("From: " + from);
    // 继续打印收件人:
    ...
}

// 递归地解析出完整的正文
String getBody(Part part) throws MessagingException, IOException {
    if (part.isMimeType("text/*")) {
        // Part是文本:
        return part.getContent().toString();
    }
    if (part.isMimeType("multipart/*")) {
        // Part是一个Multipart对象:
        Multipart multipart = (Multipart) part.getContent();
        // 循环解析每个子Part:
        for (int i = 0; i < multipart.getCount(); i++) {
            BodyPart bodyPart = multipart.getBodyPart(i);
            String body = getBody(bodyPart);
            if (!body.isEmpty()) {
                return body;
            }
        }
    }
    return "";
}

// 最后关闭 Folder 和 Store
folder.close(true); // 传入true表示删除操作会同步到服务器上（即删除服务器收件箱的邮件）
store.close();
```



---



# 6 HTTP 编程

HTTP(HyperText Transfer Protocol, 超文本传输协议)是目前使用最广泛的Web应用程序使用的基础协议

​	浏览器访问网站，手机App访问后台服务器，都是通过HTTP协议实现的



浏览器访问某个网站：

- 浏览器和网站服务器之间首先建立TCP连接（服务器总是使用`80`端口和加密端口`443`
- 浏览器向服务器发送一个HTTP请求
  - HTTP请求的格式是固定的：HTTP Header和HTTP Body
  - 第一行总是`请求方法 路径 HTTP版本`：GET / HTTP/1.1
  - 后续的每一行都是固定的`Header: Value`
    - Host：表示请求的域名
    - User-Agent：表示客户端自身标识信息，不同的浏览器有不同的标识
    - Accept：表示客户端能处理的HTTP响应格式：`*/*`表示任意格式，`text/*`表示任意文本，`image/png`表示PNG格式的图片
    - Accept-Language：表示客户端接收的语言，多种语言按优先级排序
  - 如果是`POST`请求，那么该HTTP请求带有Body
    - `Content-Type`表示Body的类型：application/json
    - `Content-Length`表示Body的长度
    - Content
- 服务器收到请求后，返回一个HTTP响应
  - 由Header和Body两部分组成
  - 第一行：`HTTP版本 响应代码 响应说明`：HTTP/1.1 200 OK
  - `Content-Type`，`Content-Length`，Content



对于最早期的HTTP/1.0协议，每次发送一个HTTP请求，客户端都需要先创建一个新的TCP连接，服务器响应后关闭连接

HTTP/1.1协议允许在一个TCP连接中反复发送-响应，客户端在发送了一个HTTP请求后，必须等待服务器响应后，才能发送下一个请求

HTTP/2.0允许客户端在没有收到响应的时候，发送多个HTTP请求，服务器返回响应的时候，不一定按顺序返回



早期Java标准库通过`HttpURLConnection`访问HTTP，代码编写繁琐且需要手动处理`InputStream`，略

Java 11引入的`HttpClient`，使用链式调用的API，能大大简化HTTP的处理

```java
// 创建一个全局HttpClient实例，因为HttpClient内部使用线程池优化多个HTTP连接，可以复用
static HttpClient httpClient = HttpClient.newBuilder().build();

// 使用GET请求获取文本内容:
String url = "https://www.sina.com.cn/";
HttpRequest request = HttpRequest.newBuilder(new URI(url))
    // 设置Header:
    .header("User-Agent", "Java HttpClient").header("Accept", "*/*")
    // 设置超时:
    .timeout(Duration.ofSeconds(5))
    // 设置版本:
    .version(Version.HTTP_2).build();
HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
// HTTP允许重复的Header，因此一个Header可对应多个Value:
Map<String, List<String>> headers = response.headers().map();
for (String header : headers.keySet()) {
    System.out.println(header + ": " + headers.get(header).get(0));
}
System.out.println(response.body().substring(0, 1024) + "...");

// POST
String url = "http://www.example.com/login";
String body = "username=bob&password=123456";
HttpRequest request = HttpRequest.newBuilder(new URI(url))
    // 设置Header:
    .header("Accept", "*/*")
    .header("Content-Type", "application/x-www-form-urlencoded")
    // 设置超时:
    .timeout(Duration.ofSeconds(5))
    // 设置版本:
    .version(Version.HTTP_2)
    // 使用POST并设置Body:
    .POST(BodyPublishers.ofString(body, StandardCharsets.UTF_8)).build();
HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
String s = response.body();
```



---



# 7 RMI 远程调用

RMI(Remote Method Invocation)：一个JVM中的代码可以通过网络实现远程调用另一个JVM的某个方法

提供服务方为服务器，调用远程方位客户端



实现一个最简单的RMI：服务器提供一个`WorldClock`服务，允许客户端获取指定时区的时间

```java
// 实现RMI，服务器和客户端必须共享同一个接口
// 此接口必须派生自java.rmi.Remote，并在每个方法声明抛出RemoteException
public interface WorldClock extends Remote {
    LocalDateTime getLocalDateTime(String zoneId) throws RemoteException;
}

// 服务器实现类
public class WorldClockService implements WorldClock {
    @Override
    public LocalDateTime getLocalDateTime(String zoneId) throws RemoteException {
        return LocalDateTime.now(ZoneId.of(zoneId)).withNano(0);
    }
}
// 通过Java RMI提供的一系列底层支持接口，把上面编写的服务以RMI的形式暴露在网络上，客户端才能调用
public class Server {
    public static void main(String[] args) throws RemoteException {
        System.out.println("create World clock remote service...");
        // 实例化一个WorldClock:
        WorldClock worldClock = new WorldClockService();
        // 将此服务转换为远程服务接口:
        WorldClock skeleton = (WorldClock) UnicastRemoteObject.exportObject(worldClock, 0);
        // 将RMI服务注册到1099端口:
        Registry registry = LocateRegistry.createRegistry(1099);
        // 注册此服务，服务名为"WorldClock":
        registry.rebind("WorldClock", skeleton);
    }
}

// 客户端 有同样的WorldClock接口
public class Client {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        // 连接到服务器localhost，端口1099:
        Registry registry = LocateRegistry.getRegistry("localhost", 1099);
        // 查找名称为"WorldClock"的服务并强制转型为WorldClock接口:
        WorldClock worldClock = (WorldClock) registry.lookup("WorldClock");
        // 正常调用接口方法:
        // 客户端实际上只有接口，并没有实现类的方法
        // 实现类由Registry内部动态生成，并负责把方法调用通过网络传递到服务器端
        // 服务器端接收网络调用的服务并不是我们自己编写的WorldClockService，而也是Registry自动生成的代码
        LocalDateTime now = worldClock.getLocalDateTime("Asia/Shanghai");
        // 打印调用结果:
        System.out.println(now);
    }
}

```

Java的RMI严重依赖序列化和反序列化，而这种情况下可能会造成严重的安全漏洞，因为Java的序列化和反序列化不但涉及到数据，还涉及到二进制的字节码

使用RMI时，双方必须是内网互相信任的机器，不要把1099端口暴露在公网上作为对外服务



如果要使用不同语言进行RPC调用，可以选择更通用的协议，例如[gRPC](https://grpc.io/)

