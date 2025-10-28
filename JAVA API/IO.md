## 一、文件

文件可以认为是相关记录存放在一起的数据的集合，一般存放在磁盘上



```
文件和目录路径名的抽象表示形式
File不能操作(读写)文件内容，只能对文件或目录的属性进行操作，如：文件名、最后修改日期、文件大小...
Class File{
	public File(String pathname)	// 根据文件路径创建文件对象
	// 刚创建的File对象仅仅是一个路径的抽象，此时如果磁盘上不存在该路径，isFile和isDirectory都为false
	// 调用createNewFile()或mkdirs()方法在磁盘上创建文件或目录后，相应的isFile()或isDirectory()才会返回true
	public File(String parent, String child)	// 根据父路径名字符串和子路径名字符串创建文件对象（拼接）
	public File(File parent, String child)	// 根据父路径对应文件对象和子路径名字符串创建文件对象（拼接）
	
	boolean exists()	// 文件是否存在
	boolean isFile()	// 是否为文件
	boolean isDirectory()	// 是否为目录
	boolean createNewFile()	// 创建新文件，可能抛出IOException
	boolean delete()	// 删除文件，成功true
	// 移动文件，实际上是重命名
	File file = new File("d:/text/a.txt");
	file.createNewFile();
	file.renameTo(new File("d:/a.txt"));
}
```

```
File dir = new File("c:/ClashforWIndows");
System.out.println("---------查询文件名---------");
String[] files = dir.list();
// 数组转集合：Arrays.asList，方便使用forEach
// 不能区分文件和目录
Arrays.asList(files).forEach(System.out::println);
System.out.println("---------查询文件信息---------");
File[] fObjs = dir.listFiles();
Arrays.asList(fObjs)
      .forEach(f->System.out.printf(
              "%s: %s \n",
              f.isFile()?"文件":"目录",
              f.getName()
              ));
System.out.println("---------按文件名过滤---------");
files = dir.list(new FilenameFilter() {
    @Override
    public boolean accept(File dir, String name) {
        // 返回值表示 该文件被过滤出来，存入结果数组
        return name.startsWith("v");
    }
});
Arrays.asList(files).forEach(System.out::println);
System.out.println("---------按文件大大小过滤---------");
fObjs = dir.listFiles(fObj -> fObj.isFile() && fObj.length() <= 1024);
Arrays.asList(fObjs).forEach(f->System.out.printf(
            "%s: %s \n",
            f.isFile()?"文件":"目录",
            f.getName(),
            f.length()
        ));
}
```



---



## 二、流

流是指一连串流动的数据信号，是以**先进先出**的方式发送和接收数据的通道

硬盘中的一切文件数据在存储时，都是二进制数字的形式，所以，**字节流可以传输任意文件数据**

在操作流的时候，无论使用什么样的流对象，**底层传输的始终为二进制数据(1,0)**



基类流：

- InputStream: 所有字节输入流的父类
- OutputStream: 所有字节输出流的父类
- Reader: 所有字符输入流的父类
- Writer: 所有字符输出流的父类



常用类：

- 文件流(资源): 文件输入输出流是以 File 开头的流, 实现文件读写操作
- 数据流(处理): 文件输入输出流是以 Data 开头的流, 实现对基本数据类型数据的读写操作
- 缓冲流(处理): 文件输入输出流是以 Buffered 开头的流, 可以缓存数据, BufferedReader可以按行读取
- 对象流(处理): 文件输入输出流是以 Object 开头的流, 实现对Java对象的读写操作, 需要序列化支持



```
// 文件字节输出流
try(FileOutputStream fos = new FileOutputStream("d:/output.txt")){
    fos.write("hello world".getBytes());
}

// 字符流,更简单
// true 追加模式
try(FileWriter fw = new FileWriter("d:/output.txt",true);
    PrintWriter bw = new PrintWriter(fw)){}

// 读入文本文件
try(FileReader fr = new FileReader("d:/output.txt");
    BufferedReader br = new BufferedReader(fr)){
    String line = null;
    while((line = br.readLine()) != null) {
        System.out.println(line);
    }
}

// 文件复制
// 如果目标文件不存在，fos会自动创建
try(FileInputStream fis = new FileInputStream("d:/output.txt");
    FileOutputStream fos = new FileOutputStream("c:/output.txt")){
    // 每次读入 1024 字节
    byte[] buffer = new byte[1024];
    // 记录读入数据的量
    int count;
    while((count = fis.read(buffer)) > -1) {
        // 将buffer中从第0个字节到第count个字节的数据通过fos写出
        fos.write(buffer, 0, count);
    }
}
```

