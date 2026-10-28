
# 移动安全学习笔记（一）

## 写在前面

这是一份笔者学习移动安全的笔记，虽然都是一些基础的东西但也算是能为入门小白指引一些学习路线，本篇的主要内容是 JAVA 和 Android 基础内容，是后续学习的基石，笔记如有缺漏敬请斧正。

## JAVA

### 安装JDK

安装`jdk-8u66-windows-x64.exe`。

### 设置环境变量

- 添加系统环境变量`JAVA_HOME`：`C:\Program Files\Java\jdk1.8.0_66`
- 添加系统环境变量`JRE_HOME`：`C:\Program Files\Java\jre1.8.0_66`
- 添加系统环境变量`CLASSPATH`：`%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;%JRE_HOME%\lib\rt.jar`
- 系统环境变量`path`中添加变量：`%JAVA_HOME%\bin;%JRE_HOME%\bin;`
- 测试`java`环境

```cmd
C:\Users\Administrator>java -version
java version "1.8.0_66"
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.66-b17, mixed mode)
C:\Users\Administrator>javac -version
javac 1.8.0_66
```

### First程序

1. 使用快捷键`win+R`，打开运行工具
2. 输入`notepad`，打开记事本
3. 编写代码保存为`Hello.java`

```java
public class Hello{
public static void main(String[] args){
System.out.println("Hello World");
}
}
```

4. 在命令行中编译代码，生成`class`文件

```cmd
javac Hello.java
```

5. 在命令行中解释执行代码

```cmd
java Hello
```

### 安装集成环境

- 安装 IntelliJ IDEA Community Edition 2018.2.4 x64
- 取消默认的打开最后一个工程的设置  
  Settings — Appearance & Behavior — System Settings — Reopen last project on startup — 取消选中
- 设置鼠标滚轮改变字体大小  
  Settings — Editor — General — Change font size with Ctrl+Mouse Wheel — 取消选中
- 关闭第一字母补全代码  
  Settings — Editor — General — Code Completion — Match case — 取消选中
- 开启自动导入  
  Settings — Editor — General — Auto Import — Insert import on paste — Ask
- 设置函数跟踪和返回的快捷键（默认快捷键有冲突）  
  Settings — Keymap — Main menu — Navigate — Back — Alt + 向左箭头  
  Settings — Keymap — Main menu — Navigate — Forward — Alt + 向右箭头
- 设置自定义代码块快捷键  
  Settings — Editor — Live Templates — Other — +号 — Line Template

![image-20230117110619687](upload/attach/202302/950902\_X8Z85MAFFHVTB7A.jpg)

#### 常用快捷键

- Ctrl + Q 函数参数提示
- Alt + Insert 呼出菜单
- Constructor 生成构造方法
- Getter and Setter 为成员变量生成读写函数（常用于私有变量读写）
- Alt + Enter 自动排除错误
- Ctrl + O 呼出重写菜单

### Second 程序

```java
import java.util.Scanner;
public class Main {
public static void main(String[] args) {
System.out.print("请输入数据:");
Scanner scanner = new Scanner(System.in);
String buff = scanner.next();
System.out.println("您输入的数据:"+buff);
}
}
```

### 集合

#### `List`和`ArrayList`

`Main.java`

```java
import java.util.List;
public class Main {
public static void main(String[] args) {
System.out.println("Hello World!");
Zoo zoo = new Zoo();
zoo.add(new Dog("Dog", 3));
zoo.add(new Pig("Pig", 5));
zoo.show();
List<Animal> animals = zoo.search("Dog");
for (int i = 0; i < animals.size(); i++) {
System.out.println(animals.get(i).getName());
}
zoo.clear();
zoo.show();
}
}
```

`Zoo.java`

```java
import java.util.ArrayList;
import java.util.List;
class Zoo {
private List<Animal> animal = new ArrayList<Animal>();
public void add(Animal animal){
this.animal.add(animal);
}
public void clear(){
for (int i = animal.size() - 1; i >= 0; i--) {
this.animal.remove(animal.get(i));
}
}
public void show(){
for (int i = 0; i < animal.size(); i++) {
System.out.println(animal.get(i).getName());
}
}
public List<Animal> search (String name)
{
List<Animal> result = new ArrayList<>();
for (int i = 0; i < animal.size(); i++) {
if(animal.get(i).getName().contains(name))
{
result.add(animal.get(i));
}
}
return result;
}
}
```

`Animal.java`

```java
public interface Animal {
public String getName();
public int getAge();
}
```

`Dog.java`

```java
import javax.naming.Name;
public class Dog implements Animal{
private String name;
private int age;
public Dog(String name, int age) {
this.name = name;
this.age = age;
}
@Override
public String getName() {
return name;
}
@Override
public int getAge() {
return age;
}
@Override
public boolean equals(Object obj){
if(this == obj)
{
return true;
}
if(this == null)
{
return false;
}
if(!(obj instanceof Dog))
{
return false;
}
if(this.name.equals(((Dog)obj).name))
{
return true;
}
return false;
}
}
```

`Pig.java`

```java
public class Pig implements Animal{
private String name;
private int age;
public Pig(String name, int age) {
this.name = name;
this.age = age;
}
@Override
public String getName() {
return name;
}
@Override
public int getAge() {
return age;
}
@Override
public boolean equals(Object obj){
if(this == obj)
{
return true;
}
if(this == null)
{
return false;
}
if(!(obj instanceof Pig))
{
return false;
}
if(this.name.equals(((Pig)obj).name))
{
return true;
}
return false;
}
}
```

#### `Set`和`HashSet`

基本实现

```java
import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;
public class Main {
public static void main(String[] args) {
System.out.println("Hello World!");
Set<String> all = new HashSet<>();
all.add("Hello");
all.add("Hello"); //重复添加无效
all.add("World");
System.out.println(all);
Iterator<String> iter = all.iterator();
while(iter.hasNext()){
System.out.println(iter.next());
}
}
}
```

#### `Map`和`HashMap`

`Map`提供了一种映射关系，其中的元素是以键值对（`key`-`value`）的形式存储的，能够实现根据`key`快速查找`value`。`Map`中的键值对以`Entry`类型的对象实例形式存在，键（`key`值）不可重复，`value`值可以。

`Map`支持泛型，如`Map<K,V>`。

`Map`中常用方法：`put(key,value)`、`putAll`添加元素，`get(key)`获取元素，`remove(key)`删除。

```java
import java.util.*;
public class Main {
public static void main(String[] args) {
System.out.println("Hello World!");
Map<Integer, String> map = new HashMap<>();
map.put(3,"张三");
map.put(null,"佚名");
map.put(5,"王五");
map.put(0,"赵零");
map.put(3,"张3"); //key值重复，value覆盖为新内容
//keySet() 方法返回映射中所有 key 组成的 Set 视图。
Set<Integer> set = map.keySet();
Iterator<Integer> iter = set.iterator();
while(iter.hasNext()){
Integer key = iter.next();
System.out.println(key + "：" + map.get(key));
}
}
}
```

### 文件

#### `File` 类

**常见方法**

| 方法名 | 说明 |
| --- | --- |
| `File(String pathName)` | 通过将给定路径的字符串来创建一个表示对应文件的`File`实例 |
| `exists()` | 判断此`File`的实例对应的文件是否存在 |
| `isFile()` | 判断此`File`的实例是否是一个标准文件 |
| `isDirectory()` | 判断此`File`的实例是否为一个文件夹 |
| `getName()` | 获取此`File`实例对应的文件或文件夹的名称 |
| `getAbsolutePath()` | 获取`File`实例对应的文件或文件夹的绝对路径 |
| `length()` | 返回此`File`实例的长度，单位为字节 |
| `createNewFile()` | 文件的创建，返回值为布尔型 |
| `delete()` | 删除文件 |

#### `FileInputStream`

- 从文件系统中的某个文件中获取输入字节
- 用于读取文件、图像等数据的原始字节流
- 如果返回-1，则表示到达文件末尾

| 方法名 | 描述 |
| --- | --- |
| `public int read()` | 从输入流中读取一个数据字节 |
| `public int read(byte[] b)` | 从输入流中将最多`b.length`个字节的数据读入一个`byte`数组中 |
| `public int read(byte[] b, int off, int len)` | 从输入流中将最多`len`个字节的数据读入`byte`数组中 |
| `public void close()` | 关闭此文件输入流并释放与此流有关的所有系统资源 |

#### `FileOutputStream`

**常用方法**

| 方法名 | 描述 |
| --- | --- |
| `public void write(int b)` | 将指定字节写入此文件输出流 |
| `public void write(byte[] b)` | 将`b.length`个字节从指定`byte`数组写入此文件输出流中 |
| `public void write(byte[] b, int off, int len)` | 将指定`byte`数组中从偏移量`off`开始的`len`个字节写入此文件输出流 |
| `public void close()` | 关闭此文件输出流并释放与此流有关的所有系统资源 |

`FileEx.java`
