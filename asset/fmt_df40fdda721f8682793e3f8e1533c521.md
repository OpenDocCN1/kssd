
# 五子棋 3.22

> 原文：[`www.pediy.com/kssd/pediy03/forum295.htm`](https://www.pediy.com/kssd/pediy03/forum295.htm)

最近有事，有一段时间不能上网，现发布如下三个软件破解，供初学们练手。

难度等级：一般

## 一。五子棋 3.22

下载：`http://www.3q1.com/zip/Five.zip`

一个可用注册码：

```
name:backboy

code:cZTJC4
```

注意事项：假码要为 6 位，程序有判断。

程序比较核心代码如下：

```
0137:004759ff lea eax,[ebp-012e]  d eax 可见真注册码
0137:00475a05 lea edx,[ebp-00c9]  d edx 可见假注册码
0137:00475a0b mov bl,[eax]
0137:00475a0d cmp bl,[edx]  比较注册码第一位
0137:00475a0f jz 00475a1a
0137:00475a11 mov byte ptr[0047d348],00  不等，flag 为 0
0137:00475a18 jmp 00475a1f  跳出比较
0137:00475a1a inc edx      取假码下一位
0137:00475a1b inc eax      取真码下一位
0137:00475a1c dec ecx      计数器减一
0137:00475a1d jnz 00475a0b  未比完返回继续比较
```

## 二。推箱子 1.54

下载：`http://www.3q1.com/zip/Cargador.zip`

一个可用的注册码：

```
name:backboy

code:e\VLE6
```

注意事项：假码要为 6 位

程序比较核心代码如下：比较方式与上相同

```
0137:0040177f lea eax,[ebp-012e]  D EAX 可见真注册码
0137:00401785 lea edx,[ebp-00c9]
0137:0040178b mov bl,[eax]
0137:0040178d cmp bl,[edx]
0137:0040178f jz 0040179a
0137:00401791 mov byte ptr[0047493c],00
0137:00401798 jmp 0040179f
0137:0040179a inc edx
0137:0040179b inc eax
0137:0040179c dec ecx
0137:0040179d jnz 0040178b
```

另一个斗地主论坛已发过贴子，未看到的话可在 `0137：00403c3d` `push edi` 处设断

`d eax` 可见假注册码

`d edx` 可见真注册码

程序下载地址同上。 `4dd466d0e1e0cc123afbe504da2462ca`
