
# 文字录入速度测试系统 V3.0

> 原文：[`www.pediy.com/kssd/pediy02/forum261.htm`](https://www.pediy.com/kssd/pediy02/forum261.htm)

## 简介

**文字录入速度测试系统 V3.0 网络版**

计算机文字录入练习、考试、比赛工作者的好帮手! 是一个集中英文打字练习、测试，五笔编码、拼音编码查询功能为一体的纯 32 位应用软件，她不局限于某一种输入法，您可以用任何输入法进行测试。她对于中英文各提供了三种测试方式供您选择，她还支持多用户管理，您可以在一台机器上建立若干个用户，分别进行测试，并分别查看各自的成绩；您还可以自由设定测试时间、用户权限、随时暂停测试；您也可以设定文章的载入点，这样您就可以每次都从同一地方开始测试了。

### 网络版功能

除了具有单机版的所有功能外，还增加了一些网络功能。她分为服务器版和工作站版，基于 `TCP/IP` 网络协议，可以通过局域网，甚至是 `Internet` 进行远程考核。工作站版的功能和单机版的功能基本一样，并且，当工作站版不联接服务器时，也可做单机版用。

**下载地址**：`http://go.163.com/~mrxiong/download/stclient3.exe`

## 注册破解过程

该软件有 30 天的试用期。我输入用户编号 `1111-1111-1111` 后，胡乱输入注册号：`7878-7878-7878`。经过长时间的跟踪，得到一个注册号码为：`6119-1911-1121`。急忙把它填入，按注册，哈哈，显示：注册成功。

正当我高兴这时，又重新启动了该软件，什么？？？？它摇身一变又变成了 30 天限制的试用版。我当时就晕倒。。。作者搞了一个假的注册码。那么真的在什么地方呢？我算是功夫不到，败下阵来了。望各位大侠出一篇它的破解实录，帮我报一箭之仇，小弟不胜感激。

### 汇编代码片段 1

```
:004D7981 E8DEC7F2FF          call 00404164
:004D7986 8B45D8              mov eax, dword ptr [ebp-28]
:004D7989 8D4DDC              lea ecx, dword ptr [ebp-24]
:004D798C BA0C000000          mov edx, 0000000C
:004D7991 E8221CFDFF         call 004A95B8
:004D7996 8B55DC              mov edx, dword ptr [ebp-24]————根据用户名计算的注册码。  d edx
:004D7999 8B45F8              mov eax, dword ptr [ebp-08]————你输入的注册码 d eax
:004D799C E813C8F2FF         call 004041B4
:004D79A1 0F85E2000000       jne 004D7A89-------------------不跳成功
:004D79A7 8D55C4              lea edx, dword ptr [ebp-3C]
:004D79AA 8B83F4020000       mov eax, dword ptr [ebx+000002F4]
```

我的注册码是:

用户名：`1111 1111 1111`
注册码：`9211 1156 0161`

注册后，完全成功。

### 汇编代码片段 2

```
:004D7981 E8DEC7F2FF         call 00404164
:004D7986 8B45D8             mov eax, dword ptr [ebp-28]
:004D7989 8D4DDC             lea ecx, dword ptr [ebp-24]
:004D798C BA0C000000         mov edx, 0000000C
:004D7991 E8221CFDFF         call 004A95B8
:004D7996 8B55DC             mov edx, dword ptr [ebp-24] //根据用户名计算的注册码。  d edx
:004D7999 8B45F8             mov eax, dword ptr [ebp-08]//你输入的注册码 d eax
:004D799C E813C8F2FF         call 004041B4
:004D79A1 0F85E2000000       jne 004D7A89//不跳成功（此处才是正确的注册码了）
:004D79A7 8D55C4             lea edx, dword ptr [ebp-3C]
:004D79AA 8B83F4020000       mov eax, dword ptr [ebx+000002F4]
```

### 跳转引用与后续代码

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:004D79A1(C)

:004D7A89 8D55A0             lea edx, dword ptr [ebp-60]
:004D7A8C 8B83F4020000       mov eax, dword ptr [ebx+000002F4]
:004D7A92 E85DB4F5FF         call 00432EF4
:004D7A97 FF75A0             push [ebp-60]
:004D7A9A 8D559C             lea edx, dword ptr [ebp-64]
:004D7A9D 8B83F8020000       mov eax, dword ptr [ebx+000002F8]
:004D7AA3 E84CB4F5FF         call 00432EF4
:004D7AA8 FF759C             push [ebp-64]
:004D7AAB 8D5598             lea edx, dword ptr [ebp-68]
:004D7AAE 8B83FC020000       mov eax, dword ptr [ebx+000002FC]
:004D7AB4 E83BB4F5FF         call 00432EF4
:004D7AB9 FF7598             push [ebp-68]
:004D7ABC 8D45A4             lea eax, dword ptr [ebp-5C]
:004D7ABF BA03000000         mov edx, 00000003
:004D7AC4 E89BC6F2FF         call 00404164
:004D7AC9 8B45A4             mov eax, dword ptr [ebp-5C]
:004D7ACC 8D4DA8             lea ecx, dword ptr [ebp-58]
:004D7ACF BA0C000000         mov edx, 0000000C
:004D7AD4 E85B19FDFF         call 004A9434
:004D7AD9 8B55A8             mov edx, dword ptr [ebp-58]
:004D7ADC 8B45F8             mov eax, dword ptr [ebp-08]//在此 D EAX 看到：6119-1911-1121 陷井！！！
:004D7ADF E8D0C6F2FF         call 004041B4
:004D7AE4 0F8574010000       jne 004D7C5E//不跳则显示注册成功的假象！
```

-------------------------------------------------------------------------------------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:004D7AE4(C)
```

* Possible StringData Ref from Code Obj ->`"ccccccccc"`注册失败提示处！

```
:004D7C5E B8107E4D00         mov eax, 004D7E10
:004D7C63 E8083CF8FF         call 0045B870
:004D7C68 8B8300030000       mov eax, dword ptr [ebx+00000300]
:004D7C6E E80D1FF5FF         call 00429B80
:004D7C73 8B8300030000       mov eax, dword ptr [ebx+00000300]
:004D7C79 8B10               mov edx, dword ptr [eax]
```
`4dd466d0e1e0cc123afbe504da2462ca`
