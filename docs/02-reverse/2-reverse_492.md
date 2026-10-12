
# IDA 分析结构体是如何按值传递的，顺便求一个邀请码

> 原文：[`www.pediy.com/kssd/pediy11/116093.html`](https://www.pediy.com/kssd/pediy11/116093.html)

最近又拿起了黑客反汇编揭秘翻了翻，其中有一个例子引起了我的关注，我们知道数值类型和指针类型的参数一般是通过寄存器 `push(stdcall、cdecl 约定)` 来传递的，那么一个结构体的大小肯定不止 `4` 字节(`32` 位系统)，那么它是如何进行传递的呢？难道像对象一样通过指针？其实不是，是通过堆栈指针直接传递，希望能给初学者带来一点帮助。（本人也是初学者，大牛们嘴下留情，有错误请指出。）

`vs2008`，`Max Speed (/02)` 编译源代码如下：

```
#include <stdio.h>
#include <string.h>

struct zzz
{
  char s0[16];
  int a;
  float f;
};

void func(zzz y)
{
  printf("%s  %x  %f\n", &y.s0[0], y.a, y.f);
}

int main()
{
  zzz y;
  strcpy(&y.s0[0], "Hello");
  y.a=0x666;
  y.f=6.6;
  func(y);
}
```

代码:

```
.text:00401070 ; int __cdecl main(int argc, const char **argv, const char **envp)
.text:00401070 _main:                     ; CODE XREF: ___tmainCRTStartup+10Ap
.text:00401070             sub     esp, 1Ch
.text:00401073             mov     eax, dword ptr stru_403000.s0 ; 将结构体中 s0 的地址放入 eax
.text:00401078             xor     eax, esp        ; 这里不明白，为何要和 esp 进行异或
.text:0040107A             mov     [esp+18h], eax  ; s0 地址存入放入 esp+0x18
.text:0040107E             mov     ax, word ptr ds:aHello+4 ; 此处将"Hello"的第四个字符 Ascii 放入 ax
.text:00401084             fld     ds:flt_402108   ; 6.6 压入浮点寄存器
.text:0040108A             mov     ecx, dword ptr ds:aHello ; "Hello"
.text:00401090             fstp    dword ptr [esp+14h] ; 弹出 6.6 到 esp+0x14
.text:00401094             mov     [esp+4], ax     ; 转存 esp+4，不明白为何要保存 ax?
.text:00401099             sub     esp, 18h        ; 分配 sizeof(zzz)=0x18 字节
.text:0040109C             mov     eax, esp        ; eax 指向栈顶
.text:0040109E             mov     [eax], ecx      ; 字符串拷贝 1，因为 s0 有 16 字节，所以分 4 句
.text:004010A0             mov     ecx, [esp+1Ch]
.text:004010A4             mov     [eax+4], ecx    ; 字符串拷贝 2
.text:004010A7             mov     ecx, [esp+20h]
.text:004010AB             mov     [eax+8], ecx    ; 字符串拷贝 3
.text:004010AE             mov     ecx, [esp+24h]
.text:004010B2             mov     edx, 666h
.text:004010B7             mov     [eax+0Ch], ecx  ; 字符串拷贝 4
.text:004010BA             mov     [eax+10h], edx  ; 666
.text:004010BD             mov     edx, [esp+2Ch]
.text:004010C1             mov     [eax+14h], edx  ; 6.6
.text:004010C4             call    func            ; 此时 eax 即 esp 开始的 0x18 字节均为 zzz 结构体了
.text:004010C9             mov     ecx, [esp+30h]
.text:004010CD             add     esp, 18h
.text:004010D0             xor     ecx, esp
.text:004010D2             xor     eax, eax
.text:004010D4             call    sub_4010DD
.text:004010D9             add     esp, 1Ch
.text:004010DC             retn
```

下面看看结构体是如何被取出的

代码:

```
.text:00401000 func           proc near                ; CODE XREF: .text:004010C4p
.text:00401000
.text:00401000 var_24         = qword ptr -24h
.text:00401000 var_1C         = dword ptr -1Ch
.text:00401000 var_18         = dword ptr -18h
.text:00401000 var_14         = dword ptr -14h
.text:00401000 var_10         = dword ptr -10h
.text:00401000 var_C          = dword ptr -0Ch
.text:00401000 var_8          = dword ptr -8
.text:00401000 var_4          = dword ptr -4
.text:00401000 arg_0          = zzz ptr   4
.text:00401000
.text:00401000             sub     esp, 1Ch
.text:00401003             mov     eax, dword ptr stru_403000.s0 ; 取出 s0 的地址
.text:00401008             xor     eax, esp        ; 又进行了异或，为何？
.text:0040100A             mov     [esp+1Ch+var_4], eax
.text:0040100E             mov     ecx, dword ptr [esp+1Ch+arg_0.s0+4] ; 取出"Hello"第二部分，由此可见取出字符串并不是按照顺序来的
.text:00401012             mov     eax, dword ptr [esp+1Ch+arg_0.s0] ; 取出"Hello"第一部分
.text:00401016             mov     edx, dword ptr [esp+1Ch+arg_0.s0+8] ; 取出"Hello"第三部分
.text:0040101A             mov     [esp+1Ch+var_1C], eax ; 保存"Hello"第一部分
.text:0040101D             mov     eax, dword ptr [esp+1Ch+arg_0.s0+0Ch] ; 取出"Hello"第四部分
.text:00401021             mov     [esp+1Ch+var_18], ecx ; 保存"Hello"第二部分
.text:00401025             mov     ecx, [esp+1Ch+arg_0.f] ; 取出 6.6
.text:00401029             sub     esp, 8
.text:0040102C             mov     [esp+24h+var_8], ecx
.text:00401030             fld     [esp+24h+var_8]
.text:00401034             mov     [esp+24h+var_10], eax ; 保存"Hello"第四部分
.text:00401038             fstp    [esp+24h+var_24] ; 相当于 push 6.6
.text:0040103B             mov     eax, [esp+24h+arg_0.a] ; 取出 666
.text:0040103F             mov     [esp+24h+var_14], edx ; 保存"Hello"第三部分
.text:00401043             push    eax             ; push 666
.text:00401044             lea     edx, [esp+28h+var_1C]
.text:00401048             push    edx             ; push "Hello"
.text:00401049             push    offset Format   ; "%s\t%x\t%f\n"
.text:0040104E             mov     [esp+30h+var_C], eax
.text:00401052             call    ds:printf
.text:00401058             mov     ecx, [esp+30h+var_4]
.text:0040105C             add     esp, 14h
.text:0040105F             xor     ecx, esp
.text:00401061             call    sub_4010DD
.text:00401066             add     esp, 1Ch
.text:00401069             retn
.text:00401069 func           endp
```

为了分析方便，结构体 `zzz` 在 `IDA` 中定义一下

代码:

```
00000000 zzz             struc ; (sizeof=0x18)
00000000 s0              db 16 dup(?)
00000010 a               dd ?
00000014 f               dd ?
00000018 zzz             ends
```

编译好的 `exe` 和 `idb` 文件都附上。

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `Release.rar` |
