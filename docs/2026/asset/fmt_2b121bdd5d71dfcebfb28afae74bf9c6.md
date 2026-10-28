
+ 使用`OD`继续往下跟就是`RegQueryValueExA`函数，它的功能是检索与打开的注册表项关联的指定值名的类型和数据；
+ 同样跟上面的函数一样，如果失败则返回非`0`错误代码；
+ 所以因为获取成功，所以返回了`0`；
+ 再往下跳转到了`loc_401309`处，依旧是我们喜闻乐见的字符串操作，直接跳过看代码；

```c++
v12 = &Data;
strcpy(a1, (const char *)&Data);
v12 += strlen(a1) + 1;
strcpy(a3, (const char *)v12);
v12 += strlen(a3) + 1;
strcpy(a5, (const char *)v12);
v12 += strlen(a5) + 1;
strcpy(a7, (const char *)v12);
v12 += strlen(a7) + 1;
```

我们可以判断，就是把注册表中的键值依次取出；

`sub_402020`：

+ 首先遇见的是`sub_401E60`处函数，进去瞅瞅；
+ 跳过第一个`__alloca_probe`函数就看到`sub_401420`，`F7`进去看一下；
+ 我们就看到一个熟悉的`sub_401280`函数，我们知道它是获取注册表键值的；
+ 往下跳转到`loc_401468`，直至跳出；
+ 跳转到`loc_401E9B`；
+ 行至`sub_401470`函数，`F7`进去看一下；
+ 又是一个熟悉的`sub_401280`，我们知道它是获取注册表键值的；
+ 往下跳转到`loc_4014C0`，直至跳出；
+ 跳转至`loc_401EB8`；
+ 行至`sub_401D80`函数，`F7`进去看一下；
+ 先是`_time`函数获取系统时间，因为下面用到了`rand`函数，所以这个应该是获取随机数种子；
+ `sub_4030E0`函数是给`0x0040C180`处赋值`0x6077BBD4`；
+ 跳转至`loc_401DB1`；行至`sub_401D10`函数，`F7`进去看一下；
+ 看得出是一个循环`F5`看一下反汇编；

```c++
char sub_401D10()
{
char v1; // [sp+0h] [bp-4h]@1
do
{
v1 = rand();
if ( v1 > 127 )
v1 -= 127;
}
while ( (v1 < 48 || v1 > 57) && (v1 < 97 || v1 > 122) && (v1 < 65 || v1 > 90) );
return v1;
}
```

很显然，判断条件就是随机出来的`v1`（**不是数字**&&**不是小写字母**&&**不是大写字母**）；

因为是随机的，所以看看后面是干啥用的，`F8`直至跳出；

+ 跳转至`loc_401DA8`，这是一个循环；
+ 所以我们仍然使用`F5`看一眼反汇编；

```c++
int __cdecl sub_401D80(char *a1)
{
time_t v1; // eax@1
signed int i; // [sp+0h] [bp-8h]@1
signed int j; // [sp+0h] [bp-8h]@4
signed int k; // [sp+0h] [bp-8h]@7
char *v6; // [sp+4h] [bp-4h]@1
char *v7; // [sp+4h] [bp-4h]@4
char *v8; // [sp+4h] [bp-4h]@7
v1 = time(0);
sub_4030E0(v1);
v6 = a1;
for ( i = 0; i < 4; ++i )
*v6++ = sub_401D10();
*v6 = 47;
v7 = v6 + 1;
for ( j = 0; j < 4; ++j )
*v7++ = sub_401D10();
*v7 = 46;
v8 = v7 + 1;
for ( k = 0; k < 3; ++k )
*v8++ = sub_401D10();
*v8 = 0;
return 0;
}
```

这里我们发现，这就是一个无聊的延时函数，所以我们直接使用`F8`直至跳出；

+ 跳转至`loc_401ED7`；
+ 行至`sub_401AF0`函数，`F7`跳入；
+ 步进到`sub_401640`，`F7`跳入；

我们发现这是一个网络连接功能函数；

```c++
WSAStartup(...);
gethostbyname(...);
WSACleanup(...);
socket(...);
htons(...);
connect(...);
closesocket(...);
```

既然知道函数功能了，我们就直接`F8`跳出`sub_401640`；

*这里我们要说的是一下，因为我们电脑是断网环境，所以网络连接是失败的，下一步应该是直接跳出函数了*；

**此处，我们假设联网成功，使用静态分析**；

+ 跳转`loc_401B35`，可以看到是一个`Get`头；
+ 然后协议是`HTTP/1.0\r\n\r\n`；
+ ![image-20210415125247157](upload/attach/202104/856790_FEK2FXNKNFBN6QR.jpg)

然后是一个发送指令，根据上面参数的传递，是`[ebp+buf]`，这个就是函数传入的参数，也就是我们之前获取的注册表配置信息；

如果发送成功，返回的应该是发送的字节数，所以失败就退出，否则跳转`loc_401C18`；

+ 在`loc_401C18`函数中，出现了`recv`函数，应该是从网络上获取了数据；

![image-20210415200524233](upload/attach/202104/856790_DVWUXJ8725XYY7M.jpg)

在这里，有一个判断回收长度是否小于等于`0`，如果是，则跳转`loc_401CAD`；

话说两头，左右我们都得看一下，我们直接使用`F5`看机器翻译的代码；

```c
do{
v10 = recv(s, &v6, 512, 0);
if (v10 <= 0){
if (v10){
sub_401740(&s);
return 1;
}
continue;
}
if ((unsigned int)(v10 + v8) > *(_DWORD *)a5){
sub_401740(&s);
return 1;
}
qmemcpy(&a4[v8], &v6, v10);
v8 += v10;
if (strstr(a4, asc_40C068))
break;
} while (v10 > 0);
if (sub_401740(&s)){
result = 1;
}
else{
*(_DWORD *)a5 = v8;
result = 0;
}
```

这里我们可以看见，重点在于`sub_401740`函数是做什么的，找到它，进去看一下；

```c++
signed int __cdecl sub_401740(SOCKET *a1)
{
signed int result; // eax@2
if ( shutdown(*a1, 1) == -1 )
{
closesocket(*a1);
WSACleanup();
result = 1;
}
else
{
closesocket(*a1);
WSACleanup();
result = 0;
}
return result;
}
```

很显然哦，`shutdown`函数功能是关闭功能禁止在套接字上发送或接收；

`F8`跳出`sub_401740`；

+ 行至跳转`loc_401CFC`跳出`sub_401AF0`函数；
+ 假设获得了相应字符串，跳转`loc_401F10`；

![image-20210416132713921](upload/attach/202104/856790_SCF3C7MQQ8XUCWV.jpg)

![image-20210416135503642](upload/attach/202104/856790_Q8CP4F69DYJWUB2.jpg)

我们可以看见这是一个字符串匹配函数`strstr`，以及即将跳转的`loc_401F3D`函数；

再往下，就是`sub_401E60`的结束了；

纵观`sub_401E60`函数，我们可以得到以下这么个结构：

```c++
int _cdecl sub_401E60(char *a1,signed int a2){
signed int result;
if(sub_401420(...)){ // 获取注册表键并截取
result = 1;
}
else if(sub_401470(...)){ // 获取注册表键
result = 1;
}
else if(sub_401D80(...)){ // 无聊的延时
result = 1;
}
else if(sub_401AF0(...)){ // 网络Get和recv
result = 1;
}
else{
if(a = strstr(...)){ // 字符串匹配
if(b = strstr(...)){
if(b - a + 1 <= a2){
memcpy(a1 , ...); // 复制
a1[...] = 0;
result = 0;
}
else{
result = 1;
}
}
else{
result = 1;
}
}
else{
result = 1;
}
}
return result;
}
```

以上就是`sub_401E60`函数的功能，最主要的是从网络获取字符串；

+ 如果`sub_401E60`网络获取字符串成功，那么就进入`loc_40204C`；

一路往下总揽一下，发现又是一个`if - else if`结构，直接使用结构代码看情况；

```c++
if(!strncmp(...,"SLEEP",...)){
...;
}
else if(!strncmp(...,"UPLOAD",...)){
...;
}
else if(!strncmp(...,"DOWNLOAD",...)){
...;
}
else if(!strncmp(...,"CMD",...)){
...;
}
else{
strncmp(...,"NOTHING",...);
}
```

很显然，这就是一个后门功能；

### Lab9-1分析总结

到此，应该算是大体理清楚了，看了一眼书上答案，有点迷，感觉自己还是漏了点细节；

还有就是关于我们查看字符串，我们可以使用`xref`或者`x`键进行交叉引用查询，然后一路向上反向操作，这样可以有目的性的查看网络或者其它功能；

#### 问题：

> 1、如何让这个恶意代码安装自身；

`-in`选项；

> 2、这个恶意代码的命令行选项是什么？它要求的密码是什么？

有四个命令选项，一个密码；密码是`abcd`，四个选项分别是`-in`、`-re`、`-c`、`-cc`；

> 3、如何利用`OllyDbg`永久修补这个恶意代码，使其不需要指定的命令行密码？

方法很多，重点是查看密码检查方式；

> 4、这个恶意代码基于系统的特征是什么？

使用了注册表项，以及安装服务；

> 5、这个恶意代码通过网络命令执行哪些不同操作？

五个`SLEEP`、`UPLOAD`、`DOWNLOAD`、`CMD`和`NOTHING`；

> 6、这个恶意代码是否有网络特征？

一个网址`<mark class="encrypted">9e3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8J5j5h3y4@1K9h3y4S2L8r3#2S2L8s2N6S2M7X3g2S2L8X3q4D9P5i4y4A6M7#2)9J5k6h3y4G2L8g2)9$3x3q4!0q4y4g2)9&6x3W2)9^5b7#2)9$3x3p5S2f1g2q4m8Q4x3V1j5I4i4K6u0W2x3l9`.`.</mark> GET`请求；

---

### Lab9-2

> 1、用`OllyDbg`分析恶意代码文件`Lab09-02.exe`，回答下列问题；

#### Lab9-2分析

1、首先查壳；

![image-20210416145858116](upload/attach/202104/856790_9BXQEA24UFGGQRH.jpg)

无壳，下一步；

2、检查导入表；

首先导入动态库有两个`WS2_32`和`KERNEL32`，可以看出主要还是网络操作，和一些敏感系统函数；

```c
connect(); // WS2_32 网络连接;
gethostbyname(); // WS2_32 从主机数据库中检索与主机名对应的主机信息;
WaitForSingleObject(); // KERNEL32 多线程函数;
CreateProcessA(); // KERNEL32 创建进程;
loadLibrary(); // KERNEL32 动态库加载;
TerminateProcess(); // KERNEL32 终止所有线程;
GetCurrentProcess(); // KERNEL32 获取当前进程句柄;
WriteFile(); // KERNEL32 写操作；
```

3、检查字符串

![image-20210416151705308](upload/attach/202104/856790_ZC7A6DXPPUHN579.jpg)

貌似暂时没看到什么有意思的敏感字符串；

4、分析行为；

+ 先用`IDA`打开，找到`main()`函数地址是`0x00401128`，用`OD`打开程序，找到相应点，设置软中断，一路跳到相应位置，开始分析；

![image-20210416152121654](upload/attach/202104/856790_PJMUH8BBXV4GQWW.jpg)

![image-20210416152135398](upload/attach/202104/856790_4U64SBW9DNF67AE.jpg)

+ 首先看到一连串的立即数赋值，这个就很奇怪了，怀疑是硬编码，我们可以用`OD`单步，然后找到内存位置，查看是什么；

![image-20210416154342040](upload/attach/202104/856790_V3MR4MCWRZBQP48.jpg)

这是两个字符串，一个是`1qaz2wsx3edc`、一个是`ocl.exe`；

+ 再往下是一个又是一个字符串;

![image-20210416154813536](upload/attach/202104/856790_BQDWN8HGKXA684G.jpg)

+ 使用`IDA`查看;

![image-20210416154743146](upload/attach/202104/856790_83T9Q95BCQDMHGN.jpg)

在这里，我们看见一个比较有意思的事情，这里是一个`ComandLine[4]`的字符串数组，指向的是另外一个字符串组；

而且我们看见了`cmd`这个敏感命令；

+ 继续`OD`往下单步到`GetModuleFileName`函数；

![image-20210416155843047](upload/attach/202104/856790_S3DHKTGY2DW5SX4.jpg)

这是传参，直接跳过；

+ 往下看是两个`__strrchr`和`__strcmp`两个函数；

```c++
strrchr(); // 查找一个字符串在另一个字符串中 末次 出现的位置，并返回从字符串中的这个 //位置起，一直到字符串结束的所有字符；
strcmp(); // 字符串匹配，没啥好说的；
```

按照这样的逻辑，可以得到反汇编框架:

```c++
char *strexe = "olc.exe"
GetModuleFileNameA(...);
char *strchr(...);
if(!strcmp(strexe,strchr)){
while(1){
...
}
}
```

目前知道的是通过文件路径获取文件名，如果是`ocl.exe`，则进入`while(1)`的死循环；

+ 而我们因为断网问题，所以需要在检查跳转处修改判断值；

![image-20210416165352218](upload/attach/202104/856790_CJBXPP3JZPKUA3E.jpg)

![image-20210416165441137](upload/attach/202104/856790_FUF46EMYMQNW89V.jpg)

修改了`strcmp`的返回值`eax`，就可以进入`loc_0040124C`；

![image-20210416165602552](upload/attach/202104/856790_4G6GYEP8939M6HW.jpg)

+ 下面就是依次的调用`WSAStartup`、`WSASocket`，一路往下；

+ 继续往下，又发现一个返回值检查，一个一个的改感觉有点麻烦，先整理一下静态反汇编框架；
