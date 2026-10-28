
```java
import java.io.File;
import java.io.IOException;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        System.out.println("Hello World!");

        FileEx fileEx = new FileEx();
        File file = fileEx.CreatFile("test.txt");
        fileEx.WriteFile(file, "liuxingzuo");
        String str = fileEx.ReadFile(file);
        System.out.println(str);
        fileEx.DeleteFile(file);
    }
}
```

## 输入输出流

**Java常用输入输出流体系分类**

| 分类 | 字节输入流 | 字节输出流 | 字符输入流 | 字符输出流 |
| --- | --- | --- | --- | --- |
| 抽象基类 | `InputStream` | `OutputStream` | `Reader` | `Writer` |
| 访问文件 | `FileInputStream` | `FileOutputStream` | `FileReader` | `FileWriter` |
| 访问数组 | `ByteArrayInputStream` | `ByteArrayOutputStream` | `CharArrayReader` | `CharArrayWriter` |
| 访问管道 | `PipedInputStream` | `PipedOutputStream` | `PipedReader` | `PipedWriter` |
| 访问字符串 |  |  | `StringReader` | `StringWriter` |
| 缓冲流 | `BufferedInputStream` | `BufferedOutputStream` | `BufferedReader` | `BufferedWriter` |
| 转换流 |  |  | `InputStreamReader` | `OutputStreamWriter` |
| 对象流 | `ObjectInputStream` | `ObjectOutputStream` |  |  |
| 抽象基类 | `FilterInputStream` | `FilterOutputStream` | `FilterReader` | `FilterWriter` |
| 打印流 |  | `PrintStream` |  | `PrintWriter` |
| 推回输入流 | `PushbackInputStream` |  | `PushbackReader` |  |
| 特殊流 | `DataInputStream` | `DataOutputStream` |  |  |

> 粗体字标出的类代表节点流，必须直接与指定的物理节点关联；斜体字标出的类代表抽象基类，无法直接创建实例。

## 比较器

如果现在要想为一组对象进行排序，那么必须有一个可以区分出对象大小的关系操作，而这个操作在Java之中就是利用比较器完成的。

常用比较器：`Comparable`，如果要为对象指定比较规则，那么对象所在的类必须实现`Comparable`接口。

### `Comparable`

`Person.java`

```java
class Person implements Comparable<Person>{
    private String name;
    private int age;
    public Person(String name, int age)
    {
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString()
    {
        return "Person [name=" + name + ", age=" + age + "]\n";
    }
    @Override
    public int compareTo(Person o) {
        if(this.age > o.age)
        {
            return 1;
        }else if(this.age < o.age){
            return -1;
        }
        return 0;
    }
}
```

`Main.java`

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args){
        System.out.println("Hello World!");
        Person[] persons = new Person[]{
            new Person("张三", 6),
            new Person("李四", 4),
            new Person("王五", 5),
        };
        Arrays.sort(persons);
        System.out.println(Arrays.toString(persons));
    }
}
```

### `Comparator`

类在设计之初并没有考虑排序的要求，但后来又需要进行排序，但类无法修改，所以在Java里面又提供了另外一种比较器接口，不过它需要为`Comparator`单独定义一个类，所以比较麻烦。

`PersonComparator.java`

```java
import java.util.Comparator;

public class PersonComparator implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        if(o1.getAge() > o2.getAge())
        {
            return 1;
        }
        else if(o1.getAge() < o2.getAge())
        {
            return -1;
        }
        return 0;
    }
}
```

`person.java`

```java
class Person{
    private String name;
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Person(String name, int age)
    {
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString()
    {
        return "Person [name=" + name + ", age=" + age + "]\n";
    }
}
```

`Main.java`

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args){
        System.out.println("Hello World!");
        Person[] persons = new Person[]{
            new Person("张三", 6),
            new Person("李四", 4),
            new Person("王五", 5),
        };
        Arrays.sort(persons, new PersonComparator());
        System.out.println(Arrays.toString(persons));
    }
}
```

# Android

## 架构

Android 架构分为五层。

- Linux 内核层

  Android 的核心系统服务基于 Linux 内核，在此基础上添加了部分 Android 专用的驱动。系统的安全性、内存管理、进程管理、网络协议栈和驱动模型等都依赖于该内核。

- 硬件抽象层

  硬件抽象层是位于操作系统内核与硬件电路之间的接口层，其目的在于将硬件抽象化，为了保护硬件厂商的知识产权，它隐藏了特定平台的硬件接口细节，为操作系统提供虚拟硬件平台，使其具有硬件无关性，可在多种平台上进行移植。从软硬件测试的角度来看，软硬件的测试工作都可分别基于硬件抽象层来完成，使得软硬件测试工作的并行进行成为可能。通俗来讲，就是将控制硬件的动作放在硬件抽象层中。

- 系统运行库层

  系统运行库层分为两部分，分别是 C/C++ 程序库和 Android 运行时库。

  - C/C++程序库

    | 名称 | 功能描述 |
    | --- | --- |
    | OpenGL ES | 3D绘图函数库 |
    | Libc | 从BSD继承来的标准C系统函数库，专门为基于嵌入式Linux的设备定制 |
    | Media Framework | 多媒体库，支持多种常用的音频、视频格式录制和回放。 |
    | SQLite | 轻型的关系型数据库引擎 |
    | SGL | 底层的2D图形渲染引擎 |
    | SSL | 安全套接层，是为网络通信提供安全及数据完整性的一种安全协议 |
    | FreeType | 可移植的字体引擎，它提供统一的接口来访问多种字体格式文件 |

  - Android运行时库

    运行时库又分为核心库和 ART（5.0 系统之后，Dalvik 虚拟机被 ART 取代）。核心库提供了 Java 语言核心库的大多数功能，这样开发者可以使用 Java 语言来编写 Android 应用。相较于 JVM，Dalvik 虚拟机是专门为移动设备定制的，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个 Dalvik 应用作为一个独立的 Linux 进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。而替代 Dalvik 虚拟机的 ART 的机制与 Dalvik 不同。在 Dalvik 下，应用每次运行的时候，字节码都需要通过即时编译器转换为机器码，这会拖慢应用的运行效率，而在 ART 环境中，应用在第一次安装的时候，字节码就会预先编译成机器码，使其成为真正的本地应用。

- 应用框架层

  应用框架层为开发人员提供了开发应用程序所需要的 API，我们平常开发应用程序都是调用这一层所提供的 API，当然也包括系统应用。这一层是由 Java 代码编写的，可以称为 Java Framework。下面来看这一层所提供的主要组件：

  | 名称 | 功能描述 |
  | --- | --- |
  | Activity Manager（活动管理器） | 管理各个应用程序生命周期，以及常用的导航回退功能 |
  | Location Manager（位置管理器） | 提供地理位置及定位功能服务 |
  | Package Manager（包管理器） | 管理所有安装在 Android 系统中的应用程序 |
  | Notification Manager（通知管理器） | 使得应用程序可以在状态栏中显示自定义的提示信息 |
  | Resource Manager（资源管理器） | 提供应用程序使用的各种非代码资源，如本地化字符串、图片、布局文件、颜色文件等 |
  | Telephony Manager（电话管理器） | 管理所有的移动设备功能 |
  | Window Manager（窗口管理器） | 管理所有开启的窗口程序 |
  | Content Provider（内容提供者） | 使得不同应用程序之间可以共享数据 |
  | View System（视图系统） | 构建应用程序的基本组件 |

- 应用层

  系统内置的应用程序以及非系统级的应用程序都属于应用层，负责与用户进行直接交互，通常都是用 Java 进行开发的。

## `adb`

**添加环境变量**

`C:\Users\Administrator\AppData\Local\Android\Sdk\platform-tools;`

**常用命令**

| 命令 | 说明 |
| --- | --- |
| `adb devices` | 列出所有设备 |
| `adb install hello.apk` | 安装应用 `hello.apk` |
| `adb install -s emulator-5554 D:/hello.apk` | 安装应用到指定模拟器 |
| `adb uninstall hello.apk` | 卸载 `hello.apk` 包 |
| `adb pull` | 获取模拟器中的文件 |
| `adb push` | 向模拟器中写文件 |
| `adb shell` | 进入模拟器的 `shell` 模式 |
| `adb reboot` | 重启设备 |
| `adb logcat` | 查看日志 |
| `emulator -avd advname` | 启动模拟器 |
| `adb kill-server` | 关闭服务，常用于 `adb` 连接失败 |
| `adb start-server` | 开启服务，常用于 `adb` 连接失败 |

## 第三方模拟器

这里推荐使用夜神3.8.3.1版本（因为这个版本的 Android 系统是 4.4 的），这里说一下，我的电脑安装后启动模拟器会蓝屏，安装高版本的夜神再将其卸载，重新安装 3.8.3.1 版本可以解决这个问题。

**替换夜神模拟器的 `adb` 文件**

将 Android Studio 使用的 sdk 文件夹 `C:\Users\Administrator\AppData\Local\Android\Sdk\platform-tools` 中的下列三个文件复制到夜神模拟器的 `C:\Program Files (x86)\Nox\bin` 文件夹中，请注意模拟器的原文件应有必要备份，另外夜神模拟器将 `adb.exe` 更名为 `nox_adb.exe`，所以在文件复制后，也需要将复制过来的 `adb.exe` 重命名为 `nox_adb.exe`。

- `adb.exe`
- `AdbWinApi.dll`
- `AdbWinUsbApi.dll`

替换夜神模拟器的 `adb` 文件后，打开 `cmd`，输入如下代码表示模拟器连接成功：

```text
Microsoft Windows [Version 10.0.19044.1288]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\Administrator>adb devices
List of devices attached
127.0.0.1:62001 device

C:\Users\Administrator>adb -s 127.0.0.1:62001 shell
root@android:/#
```

## 布局

### `LinearLayout`

**新增 `LinearLayout` 布局**
