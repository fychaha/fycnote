# Java 网络编程

## <font color="#0088dd">Socket 基本概念</font>

Socket 允许程序员将网络看作是一个可以读写的字节流。Socket 对程序员隐藏了网络的底层细节。

Socket 是两台主机之间的的一个连接，它可以完成 7 个基本操作：

- 连接远程机器。
- 发送数据。
- 接收数据。
- 关闭连接。
- 绑定端口。
- 监听连接请求。
- 在绑定端口上接受来自远程机器的连接。

Java 的 Socket 提供了前 4 个操作的方法，它的子类 `ServerSocket` 在它基础上提供了后 3 个操作的方法。

## IP 地址和端口号

Java 使用 `InetAddress` 类（及其子类 `Inet4Address`、`Inet6Address`）封装了IP地址的概念。`InetAddress` 类没有对外可用的构造方法，要实例化 `InetAddress`（及其子类）对象需要使用其下某个静态工厂方法。常见的有：

```
public static InetAddress getLocalHost()
public static InetAddress getByName(String host)
```

`SocketAddress` 类（及其子类 `InetSocketAddress`）封装了 **IP地址** 和 **端口号** 的概念。其常见构造方法有：

```
public InetSocketAddress(String hostname, int port)
```

## 客户端 Socket

#### 构造和连接 Socket

`java.net.Socket` 类是 Java 完成客户端 Socket 操作的基础类。其他任何可以建立 TCP 连接的类最终都是在调用这个类。

每个 Socket 的构造方法指定要连接的主机（host）和端口（port）。主机可以指定为 InetAddress 或 String。远程端口指定为 1 到 65535 之间的 int 值（其中 1024 以下端口基本以被占用完）。

```
public Socket(String host, int port)
public Socket(InetAddress address, int port)
```

host 参数是 String 类型，其内容可以是一个主机的域名，或主机的 ip 地址。

以上两个构造方法不仅是创建 Socket 对象，还会尝试连接远程主机的 Socket 。所以可以用这个对象确定是否与某个端口建立连接。

上述两个方法并未明确指明本机使用端口号，此时，将由系统随机分配一个可用端口号。以下两个方法在实现相同功能的同时，要求明确指明要使用的本地端口号。

```
public Socket(String host, int port, InetAddress localAddr, int localPort)
public Socket(InetAddress address, int port, InetAddress localAddr, int localPort)
```

前两个参数指明远程主机地址和端口，后两个参数指明本地主机地址和端口。

如果本地端口参数传0，则表明由系统自动分配可用端口。

上述四个方法在创建Socket的同时，都会尝试建立一个与远程主机的网络连接。有时你可能想分解这两个操作。

```java
    public Socket() 
    public void connect(SocketAddress endpoint)  
    public void connect(SocketAddress endpoint, int timeout) throws IOException 
```

timeout 参数表示请求建立连接的最长等待时间，单位为毫秒。0表示无限等待。

分解创建-连接这两个操作的好处在于：*在创建之后，连接之前可以对Socket进行设置* （后续讲解）。

#### 收发数据（读写数据）

Socket对底层网络操作的封装体现在，你可以像读写文件一样来实现收发数据。关键在于通过 Socket 对象获得输入输出流。

```java
public InputStream getInputStream()
public OutputStream getOutputStream()
```

`getInputStream()` 方法和 `getOutputStream()` 方法返回的是字节流，如果有需要，可以将字节流包装成字符流，这样就可以***以字符为单位*** 读写数据信息。

```java
public InputStreamReader(InputStream in)
public OutputStreamWriter(OutputStream out)

public InputStreamReader(InputStream in, String charsetName)
public OutputStreamWriter(OutputStream out, String charsetName)
```

`补充`，如果为了考虑进一步提高读写效率，可以将字符流再一次包装成缓冲流。

## 使用 ServerSocket

1. 使用一个 `ServerSocket()` 构造函数在一个特定端口创建一个新的 `ServerSocket` 。
2. `ServerSocket` 使用其 `accept()` 方法监听这个端口，等待客户端的连接。`accept()` 会一直阻塞，直到一个客户端尝试建立连接，此时 `accept()` 将返回一个连接客户端和服务器的 `Socket` 对象。
3. 根据服务器的类型，会调用 `Socket` 的 `getInputStream()` 方法或 `getOutputStream()` 方法，或者这两个方法都调用，以获得与客户端通信的输入和输出流。
4. 服务端与客户端持续通信，直到关闭连接。



## UDP 编程

```java
public void udpClient() {
    DatagramSocket socket = null;

    try {
        socket = new DatagramSocket(0);
        InetAddress serverAddress = InetAddress.getByName("127.0.0.1");
        DatagramPacket request = new DatagramPacket(new byte[1], 1, serverAddress, 9527);
        DatagramPacket response = new DatagramPacket(new byte[1024], 1024);
        socket.send(request);
        socket.receive(response);

        String str = new String(response.getData(), 0, response.getLength());
        System.out.println(str);
    } catch (Exception e) {
        e.printStackTrace();
    } 
    finally {
        if (socket != null && !socket.isClosed())
            socket.close();
    }
}
```

```java
public void udpServer() {

    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String dateStr = dateFormat.format(new Date());

    try {
        DatagramSocket socket = new DatagramSocket(9527);
        DatagramPacket request = new DatagramPacket(new byte[1024], 1024);
        socket.receive(request);

        byte[] data = dateStr.getBytes();
        DatagramPacket response 
            = new DatagramPacket(data, data.length, request.getAddress(),
        request.getPort());
        socket.send(response);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```