
# [原创] JNDI 注入漏洞的前世今生

前两天的 `log4j` 漏洞引起了安全圈的震动，虽然是二进制选手，但为了融入大家的过年氛围，还是决定打破舒适圈来研究一下 JNDI 注入漏洞。

[toc]

## JNDI 101

首先第一个问题，什么是 JNDI，它的作用是什么？

根据官方文档，JNDI 全称为 **Java Naming and Directory Interface**，即 Java 名称与目录接口。虽然有点抽象，但我们至少知道它是一个接口；下一个问题是，Naming 和 Directory 是什么意思？很多相关资料都对其语焉不详，但其实官方对其有详细解释。

### Naming

直译来说就是“名称”，但更多情况下是与 **Naming Service** 一起使用。所谓**名称服务**，简单来说就是**通过名称查找实际对象的服务**。这是个抽象概念，正如数学中的理论所言：普适的代价就是抽象。名称服务普遍存在于计算机系统中，比如：

- DNS：通过域名查找实际的 IP 地址；
- 文件系统：通过文件名定位到具体的文件；
- 微信：通过一个微信 ID 找到背后的实际用户（并进行对话）；
- ……

通常我们根据名称系统（naming system）定义的命名规则去查找具体的对象，比如在 UNIX 文件系统中，名称（路径）规则就是以根目录为起点，并以 `/` 号分隔逐级查找子目录；DNS 名称系统中则是要求名称（域名）从右到左进行逐级定义，并以点号 `.` 进行分隔。

其中另一个值得一提的名称服务为 LDAP，全称为 [Lightweight Directory Access Protocol][ldap]，即轻量级目录访问协议，其名称也是从右到左进行逐级定义，各级以逗号分隔，每级为一个 name/value 对，以等号分隔。比如一个 LDAP 名称如下：

```sh
cn=John, o=Sun, c=US
```

即表示在 `c=US` 的子域中查找 `o=Sun` 的子域，再在结果中查找 `cn=John` 的对象。关于 LDAP 的详细介绍见后文。

在名称系统中，有几个重要的概念。

**Bindings**：表示一个名称和对应对象的绑定关系，比如在文件系统中文件名绑定到对应的文件，在 DNS 中域名绑定到对应的 IP。

**Context**：上下文，一个上下文中对应着一组名称到对象的绑定关系，我们可以在指定上下文中查找名称对应的对象。比如在文件系统中，一个目录就是一个上下文，可以在该目录中查找文件，其中子目录也可以称为子上下文（subcontext）。

**References**：在一个实际的名称服务中，有些对象可能无法直接存储在系统内，这时它们便以引用的形式进行存储，可以理解为 C/C++ 中的指针。引用中包含了获取实际对象所需的信息，甚至对象的实际状态。比如文件系统中实际根据名称打开的文件是一个整数 `fd`（file descriptor），这就是一个引用，内核根据这个引用值去找到磁盘中的对应位置和读写偏移。

### Directory

名称服务还算比较好理解，那目录服务又是什么呢？简单来说，目录服务是名称服务的一种拓展，除了名称服务中已有的名称到对象的关联信息外，还允许对象拥有属性（attributes）信息。由此，我们不仅可以根据名称去查找（lookup）对象（并获取其对应属性），还可以根据属性值去搜索（search）对象。

以打印机服务为例，我们可以在命名服务中根据打印机名称去获取打印机对象（引用），然后进行打印操作；同时打印机拥有速率、分辨率、颜色等**属性**，作为目录服务，用户可以根据打印机的分辨率去搜索对应的打印机对象。

目录服务（Directory Service）提供了对目录中对象（directory objects）的属性进行增删改查的操作。一些典型的目录服务有：

- NIS：Network Information Service，Solaris 系统中用于查找系统相关信息的目录服务；
- [Active Directory](elink@802K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6m8j5%4c8A6N6X3g2Q4y4h3k6p5K9i4u0W2j5%4c8G2M7Y4V1)：为 Windows 域网络设计，包含多个目录服务，比如域名服务、证书服务等；
- 其他基于 LDAP 协议实现的目录服务。

总而言之，目录服务也是一种特殊的名称服务，关键区别是在目录服务中通常使用搜索（search）操作去定位对象，而不是简单的根据名称查找（lookup）去定位。

在下文中如果没有特殊指明，都会将名称服务与目录服务统称为目录服务。

### API

根据上面的介绍，我们知道目录服务是中心化网络应用的一个重要组件。使用目录服务可以简化应用中服务管理验证逻辑，集中存储共享信息。在 Java 应用中除了以常规方式使用名称服务（比如使用 DNS 解析域名），另一个常见的用法是使用目录服务作为对象存储的系统，即用目录服务来存储和获取 Java 对象。

比如对于打印机服务，我们可以通过在目录服务中查找打印机，并获得一个打印机对象，基于这个 Java 对象进行实际的打印操作。

为此，就有了 JNDI，即 Java 的名称与目录服务接口，应用通过该接口与具体的目录服务进行交互。从设计上，JNDI 独立于具体的目录服务实现，因此可以针对不同的目录服务提供统一的操作接口。

JNDI 架构上主要包含两个部分，即 Java 的应用层接口和 SPI，如下图所示：

![jndi][img-jndi]

SPI 全称为 Service Provider Interface，即服务供应接口，主要作用是为底层的具体目录服务提供统一接口，从而实现目录服务的可插拔式安装。在 JDK 中包含了下述内置的目录服务：

- RMI：Java Remote Method Invocation，Java 远程方法调用；
- LDAP：轻量级目录访问协议；
- CORBA：Common Object Request Broker Architecture，通用对象请求代理架构，用于 COS 名称服务（Common Object Services）。

除此之外，用户还可以在 Java 官网下载其他目录服务实现。由于 SPI 的统一接口，厂商也可以提供自己的私有目录服务实现，用户可无需重复修改代码。

JNDI 接口主要分为下述 5 个包：

- [javax.naming](elink@88bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3K9X3&6V1K9g2)9J5c8Y4c8#2N6r3!0J5K9h3q4D9i4K6u0r3k6$3g2@1f1%4c8S2M7Y4c8W2k6q4)9J5c8X3!0$3k6i4u0$3K9h3g2%4i4K6u0r3L8X3q4E0K9h3&6Y4i4K6u0W2K9s2c8E0L8l9`.`.)
- [javax.naming.directory](elink@fbdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3K9X3&6V1K9g2)9J5c8Y4c8#2N6r3!0J5K9h3q4D9i4K6u0r3k6$3g2@1f1%4c8S2M7Y4c8W2k6q4)9J5c8X3!0$3k6i4u0$3K9h3g2%4i4K6u0r3k6r3W2J5k6h3y4@1L8%4u0&6i4K6u0W2K9s2c8E0L8l9`.`.)
- [javax.naming.event](elink@52aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3K9X3&6V1K9g2)9J5c8Y4c8#2N6r3!0J5K9h3q4D9i4K6u0r3k6$3g2@1f1%4c8S2M7Y4c8W2k6q4)9J5c8X3!0$3k6i4u0$3K9h3g2%4i4K6u0r3k6i4k6W2L8Y4c8Q4x3X3g2Z5N6r3#2D9)
- [javax.naming.ldap](elink@467K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3K9X3&6V1K9g2)9J5c8Y4c8#2N6r3!0J5K9h3q4D9i4K6u0r3k6$3g2@1f1%4c8S2M7Y4c8W2k6q4)9J5c8X3!0$3k6i4u0$3K9h3g2%4i4K6u0r3L8r3c8S2M7q4)9J5k6h3S2@1L8h3H3`. 
- [javax.naming.spi](elink@717K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3K9X3&6V1K9g2)9J5c8Y4c8#2N6r3!0J5K9h3q4D9i4K6u0r3k6$3g2@1f1%4c8S2M7Y4c8W2k6q4)9J5c8X3!0$3k6i4u0$3K9h3g2%4i4K6u0r3M7s2u0G2N6X3W2V1k6i4u0Q4x3X3g2Z5N6r3#2D9)

其中最重要的是 `javax.naming` 包，包含了访问目录服务所需的类和接口，比如 `Context`、`Bindings`、`References`、`lookup` 等。以上述打印机服务为例，通过 JNDI 接口，用户可以透明地调用远程打印服务，伪代码如下所示：

```java
Context ctx = new InitialContext(env);
Printer printer = (Printer)ctx.lookup("myprinter");
printer.print(report);
```

为了更好理解 JNDI，我们需要了解其背后的服务提供者（Service Provider），这些目录服务本身和 JNDI 有没直接耦合性，但基于 SPI 接口和 JNDI 构建起了重要的联系。

## SPI

本节主要介绍在 JDK 中内置的几个 Service Provider，分别是 RMI、LDAP 和 CORBA。这几个服务本身和 JNDI 没有直接的依赖，而是通过 SPI 接口实现了联系，因此本节先脱离 JNDI 对这些服务进行简单介绍。

### RMI

第一个就是 RMI，即 [Remote Method Invocation](elink@8a8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3y4#2)9J5c8X3c8G2j5%4y4Q4x3V1k6@1k6h3y4Z5L8X3!0@1k6i4y4Q4x3V1k6Y4N6h3W2V1k6i4y4Q4x3V1k6J5L8h3W2Q4x3V1j5`.), Java 的远程方法调用。RMI 为应用提供了远程调用的接口，可以理解为 Java 自带的 RPC 框架。

一个简单的 RMI `hello world` 主要由三部分组成，分别是接口、服务端和客户端。

接口定义：

```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface Hello extends Remote {
    String sayHello() throws RemoteException;
}
```

这里定义一个名为 `Hello` 的接口，其中包含一个方法。

服务端：

```java
import java.rmi.registry.Registry;
import java.rmi.registry.LocateRegistry;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class Server implements Hello {
    public Server() {}
    
    public String sayHello() {
        return "Hello, world!";
    }
    
    public static void main(String args[]) {
        try {
            Server obj = new Server();
            Hello stub = (Hello) UnicastRemoteObject.exportObject(obj, 1098);
            // Bind the remote object's stub in the registry
            Registry registry = LocateRegistry.getRegistry(1099);
            registry.bind("Hello", stub);
            System.err.println("Server ready");
        } catch (Exception e) {
            System.err.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```

服务端有两个作用，一方面是实现 `Hello` 接口，另一方面是通过 RMI Registry 注册当前的实现。其中涉及到两个端口，`1098` 表示当前对象的 stub 端口，可以用 `0` 表示随机选择；另外一个是 `1099` 端口，表示 `rmiregistry` 的监听端口，后面会讲到。

客户端代码如下：

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Client {
    private Client() {}
    
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.getRegistry(1099);
            Hello stub = (Hello) registry.lookup("Hello");
            String response = stub.sayHello();
            System.out.println("response: " + response);
        } catch (Exception e) {
            System.err.println("Client exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```

通过 `registry.lookup` 获取其中的 `Hello` 对象，从而进行远程调用。

编译：

```sh
javac -d out Client.java Hello.java Server.java
```

生成 `out/{Client,Hello,Server}.class` 文件。在启动服务端之前，我们需要先启动 registry：

```sh
$ cd out
$ rmiregistry 1099
```

个人理解 `registry` 类似于服务注册窗口，通过这个窗口 RMI 服务器可以注册自己的服务器到全局注册表中，客户端可以从而查询获取所有已经注册的服务提供商并进行具体的远程调用。启动 registry 后其运行于 `1099` 端口，随后启动 RMI 服务器进行注册并运行：

```sh
# 回到工程所在路径
$ java -cp out Server
Server ready
```

RMI 服务注册并启动后，同时会监听在 `1098` 端口，也就是我们前面绑定的端口，用于客户端调用具体方法（如 `sayHello`）时实际传输数据到服务端。

最后启动客户端进行查询并远程调用：

```sh
$ java -cp out Client
response: Hello, world!
```

需要注意的点：

- `rmiregistry` 程序运行在 `out` 目录下，也就是我们编译的输出路径；
- `rmiregistry` 启动后可能会过一段时间后才真正开始监听端口；
- 如果 Server 绑定后退出，那么绑定信息仍然残留在 `rmiregistry` 中，再次绑定会提示 `java.rmi.AlreadyBoundException`，因此 RMI 服务端退出前应该先解除绑定；
- 远程调用的参数和返回值经过序列化后通过网络传输（marshals/unmarshals）。

拓展阅读：

- [Java™ Remote Method Invocation API](elink@8a8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3y4#2)9J5c8X3c8G2j5%4y4Q4x3V1k6@1k6h3y4Z5L8X3!0@1k6i4y4Q4x3V1k6Y4N6h3W2V1k6i4y4Q4x3V1k6J5L8h3W2Q4x3V1j5`.)
