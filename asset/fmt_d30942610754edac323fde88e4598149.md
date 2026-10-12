
# 起名正宗 (Version: 1.3)

> 原文：[`www.pediy.com/kssd/pediy03/forum961.htm`](https://www.pediy.com/kssd/pediy03/forum961.htm)

软件名称：起名正宗 (Version: 1.3) `Qmzz.EXE` 1,029KB

下载地点：`http://www.gregorybraun.com`

发信人: 井风

时间: 2000-12-25

破解工具：`Trw20001.22`

解题难度：\[专业\] [学士\] [硕士] [博士]

\*\*\*\*\*\*\*\*

前 言：

此软件加密水平还算不错，使用用户名和本机某个参数作为计算源。程序有一处陷井，因此，有些地方要引起注意。

过 程:

1. 注册窗口填入：姓名 `cccc` 注册码 `123456789`；
2. 用“井风跟踪”法找到出错的 `CALL`，详细过程参见 `WINZIP8.0` 破解教学篇；
3. 分析代码：

```
:004A6A66 8B8304030000            mov eax, dword ptr [ebx+00000304]
:004A6A6C E80FB3F8FF             call 00431D80
:004A6A71 8B45F0                 mov eax, dword ptr [ebp-10]
:004A6A74 8D55F4                 lea edx, dword ptr [ebp-0C]
:004A6A77 E88426F6FF             call 00409100
:004A6A7C 837DF400               cmp dword ptr [ebp-0C], 00000000  <---比较有否输入用户名
:004A6A80 7525                   jne 004A6AA7
:004A6A82 6A00                   push 00000000
:004A6A84 668B0D746B4A00         mov cx, word ptr [004A6B74]
:004A6A8B B202                   mov dl, 02
.
.
.
:004A6AD4 8B8B14030000           mov ecx, dword ptr [ebx+00000314]
:004A6ADA 8B9318030000           mov edx, dword ptr [ebx+00000318]
:004A6AE0 8B830C030000           mov eax, dword ptr [ebx+0000030C]
:004A6AE6 E859F7FFFF             call 004A6244       <---深入跟踪此 CALL
:004A6AEB 84C0                   test al, al
:004A6AED 7420                   je 004A6B0F        <---跳则执行[A]
:004A6AEF 6A00                   push 00000000
:004A6AF1 668B0D746B4A00         mov cx, word ptr [004A6B74]
:004A6AF8 B202                   mov dl, 02
:004A6AFA B8AC6B4A00             mov eax, 004A6BAC
:004A6AFF E8C80BFBFF             call 004576CC
:004A6B04 B201                   mov dl, 01
:004A6B06 8BC3                   mov eax, ebx
:004A6B08 E8CFFCFFFF             call 004A67DC
:004A6B0D EB15                   jmp 004A6B24
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A6AED(C)
|
:004A6B0F 6A00                   push 00000000
:004A6B11 668B0D746B4A00         mov cx, word ptr [004A6B74]
:004A6B18 33D2                   xor edx, edx
:004A6B1A B8C86B4A00             mov eax, 004A6BC8
:004A6B1F E8A80BFBFF             call 004576CC    <---执行此行出错，记为[A]
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004A6AA5(U), :004A6B0D(U)
|
:004A6B24 33C0                   xor eax, eax
:004A6B26 5A                     pop edx
:004A6B27 59                     pop ecx
:004A6B28 59                     pop ecx
:004A6B29 648910                 mov dword ptr fs:[eax], edx
:004A6B2C 686E6B4A00             push 004A6B6E
```

以下为追入 `004A6B1F` `call` `004576CC` 代码：

```
:004A6244 55                     push ebp
:004A6245 8BEC                   mov ebp, esp
:004A6247 81C460FDFFFF           add esp, FFFF0D60
:004A624D 53                     push ebx
:004A624E 56                     push esi
:004A624F 57                     push edi
:004A6250 33DB                   xor ebx, ebx
:004A6252 899D60FEFFFF           mov dword ptr [ebp+FFFFFE60], ebx
:004A6258 895DF8                 mov dword ptr [ebp-08], ebx
:004A625B 894DFC                 mov dword ptr [ebp-04], ecx
:004A625E 8BDA                   mov ebx, edx
:004A6260 8BF8                   mov edi, eax
:004A6262 8B4508                 mov eax, dword ptr [ebp+08]
:004A6265 E816DEF5FF             call 00404080
:004A626A 33C0                   xor eax, eax
:004A626C 55                     push ebp
:004A626D 68ED634A00             push 004A63ED
:004A6272 64FF30                 push dword ptr fs:[eax]
:004A6275 648920                 mov dword ptr fs:[eax], esp
:004A6278 8D9560FEFFFF           lea edx, dword ptr [ebp+FFFFFE60]
:004A627E 8B4508                 mov eax, dword ptr [ebp+08]
:004A6281 E87A2EF6FF             call 00409100
:004A6286 83BD60FEFFFF00         cmp dword ptr [ebp+FFFFFE60], 00000000
:004A628D 7507                   jne 004A6296
:004A628F 33DB                   xor ebx, ebx
:004A6291 E92E010000             jmp 004A63C4
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A628D(C)
|
:004A6296 8B4D08                 mov ecx, dword ptr [ebp+08]
:004A6299 8BD3                   mov edx, ebx
:004A629B 8BC7                   mov eax, edi
:004A629D E872FAFFFF             call 004A5D14
:004A62A2 8BF0                   mov esi, eax
:004A62A4 3B75FC                 cmp esi, dword ptr [ebp-04]  <---此处可见正确注册码 ? esi
                                                         显示：HEW C117DB6EE
                                                    注意是无符号数 16 进制，需转为 10 进制
                                                        转化为 10 进制=3246241518
:004A62A7 0F8515010000           jne 004A63C2         <---正确注册码此处不跳
:004A62AD E8F243F6FF             call 0040A6A4
:004A62B2 DD5DB0                 fstp qword ptr [ebp-50]
:004A62B5 9B                     wait
:004A62B6 E8E943F6FF             call 0040A6A4
:004A62BB DD5DB8                 fstp qword ptr [ebp-48]
:004A62BE 9B                     wait
:004A62BF C645C001               mov [ebp-40], 01
:004A62C3 C645C100               mov [ebp-3F], 00
.
.
.
```

小 结：

验证注册码方法是比较两个寄存器中无符号数值，所以要将寄存器中的十六进制数转化为十进制数（用 98 系统附件中的科学型计算器就很方便）。

注册成功后生成文件 `\_\_\!19925.\_Si`，里面放的就是你的注册码，它的位置在机器的 `windows` 目录下。

我的注册码：

```
机器码    3245818142        3245818142
用  户    abcd              cccc
注册码    3246232782        3246241518
```

后 记：

有疑问请与我联系：`hz.cy@163.net`
