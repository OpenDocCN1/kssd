
# 我也来 PEDIY--让 ZeroAdd 永不失败

> 原文：[`www.pediy.com/kssd/pediy10/88633.html`](https://www.pediy.com/kssd/pediy10/88633.html)

**【文章标题】**: 我也来 PEDIY--让 ZeroAdd 永不失败

**【文章作者】**: `jackozoo`

**【作者邮箱】**: `jackozoo@163.com`

**【下载地址】**: 自己搜索下载

**【使用工具】**: `OD`

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

`ZeroAdd` 这款小软件用于为 PE 文件添加区段，虽然是一款很简单的软件，不过大家对它肯定都非常熟悉，同类软件还有 `topo`。虽然现在 `topo` 的功能已经做的很好，不过遗憾的是，和 `ZeroAdd` 一样，当最后一个节表的末尾和第一个节的开始之间的空隙小于 40（即一个节表的大小）时，它们都会提示失败的。

这显然不能令我们满意，尤其在我们欲加多重壳的时候，这种情况更是普遍。

所以今天我们要解决的即是通过修改 `ZeroAdd` 来使之加区永不失败，即使是在空间不足的情况下。(这里的空间即最后一个节表的末尾到第一个节的文件偏移之间的区域)

很巧合的是，通过对 `ZeroAdd` 的分析，我还发现它存在一个 bug，应该是作者粗心的缘故吧。

好了，废话不说了。

`ZeroAdd` 的处理流程是先将待加区的文件复制一份拷贝 `swapit.sca`，然后对该拷贝进行加区，然后直接复制该中间文件去覆盖原文件，再删除 `swapit.sca` 以达到目的。

看关键代码处:

```
// ... 打开文件对话框
// ... 得到程序路径，区段名，要加的字节数。
... ...
00401190    68 08304000     push    zeroadd_.00403008                      ; ASCII "111"
00401195    E8 13030000     call    zeroadd_.004014AD                      ; 将“111”转换为 0x111，即新区字节数。
0040119A    A3 98354000     mov     [403598], eax
0040119F    6A 00           push    0
004011A1    FF35 7C354000   push    dword ptr [40357C]
004011A7    E8 D4030000     call    <jmp.&KERNEL32.GetFileSize>
004011AC    0305 98354000   add     eax, [403598]
004011B2    05 10270000     add     eax, 2710
004011B7    6A 00           push    0
004011B9    50             push    eax
004011BA    6A 00           push    0
004011BC    6A 04           push    4
004011BE    6A 00           push    0
004011C0    FF35 7C354000   push    dword ptr [40357C]
004011C6    E8 A3030000     call    <jmp.&KERNEL32.CreateFileMappingA>
004011CB    85C0           test    eax, eax
004011CD    0F84 E5010000  je      zeroadd_.004013B8
004011D3    A3 80354000     mov     [403580], eax
004011D8    6A 00           push    0
004011DA    6A 00           push    0
004011DC    6A 00           push    0
004011DE    6A 02           push    2
004011E0    50             push    eax
004011E1    E8 A6030000     call    <jmp.&KERNEL32.MapViewOfFile>         ; 映射文件。
004011E6    85C0           test    eax, eax
004011E8    0F84 CA010000  je      zeroadd_.004013B8
004011EE    A3 84354000     mov     [403584], eax
004011F3    66:8138 4D5A    cmp     word ptr [eax], 5A4D                 ; MZ signature。
004011F8    0F85 C6010000  jnz     zeroadd_.004013C4
004011FE    0FB778 3C       movzx   edi, word ptr [eax+3C]
00401202    033D 84354000   add     edi, [403584]
00401208    813F 50450000   cmp     dword ptr [edi], 4550                ; PE signature。
0040120E    0F85 B0010000  jnz     zeroadd_.004013C4
00401214    893D 88354000   mov     [403588], edi
0040121A    0FB74F 06       movzx   ecx, word ptr [edi+6]                ; 区段数。
0040121E    66:037F 14      add     di, [edi+14]                         ; 加上 e_alfnew
00401222    83C7 18         add     edi, 18                              ; 定位至第一个节表开始。
00401225    83C7 28         add     edi, 28
00401228   ^ E2 FB          loopd   short zeroadd_.00401225               ; 定位至最后一个节表。
0040122A    B9 0A000000     mov     ecx, 0A
0040122F    BA 28000000     mov     edx, 28                              ; 这里是 ZeroAdd 的一个 bug,我们将此句改为 xor edx,edx 即可。
00401234    833C3A 00       cmp     dword ptr [edx+edi], 0               ; 看最后一个节表的后面是否有足够的 0 来安置另一个节表。（大小为 0x28）
00401238   - 0F85 C29D0000  jnz     zeroadd_.0040B000
0040123E    83C2 04         add     edx, 4
00401241   ^ E2 F1          loopd   short zeroadd_.00401234
00401243    8B35 88354000   mov     esi, [403588]                        ; NT 头指针。
00401249    66:FF46 06      inc     word ptr [esi+6]                     ; 区块数加一。
0040124D    B9 01000000     mov     ecx, 1
00401252    837E 38 00      cmp     dword ptr [esi+38], 0                ; 块对齐。
00401256    75 08           jnz     short zeroadd_.00401260
00401258    837E 3C 00      cmp     dword ptr [esi+3C], 0                ; 文件对齐。
0040125C    75 02           jnz     short zeroadd_.00401260
0040125E    33C9           xor     ecx, ecx
00401260    83EF 28         sub     edi, 28                              ; 定位至最后一个节表。
00401263    8B47 0C         mov     eax, [edi+C]                         ; 最后区段 RVA
00401266    0347 08         add     eax, [edi+8]                         ; 最后区段虚拟大小（VSize）
00401269    E3 09          jecxz   short zeroadd_.00401274
0040126B    FF76 38         push    dword ptr [esi+38]
0040126E    50             push    eax
0040126F    E8 A6020000     call    <zeroadd_.Align>                     ; 得到一个对齐值。
00401274    A3 90354000     mov     [403590], eax                        ; 新区段的 RVA
00401279    8B47 14         mov     eax, [edi+14]                        ; 最后区段文件中偏移。
0040127C    0347 10         add     eax, [edi+10]                        ; 最后区段文件中大小。
0040127F    E3 09          jecxz   short zeroadd_.0040128A
00401281    FF76 3C         push    dword ptr [esi+3C]
00401284    50             push    eax
00401285    E8 90020000     call    <zeroadd_.Align>                     ; 得到一个对齐值。
0040128A    A3 94354000     mov     [403594], eax                        ; 新区段的文件中偏移。
0040128F    A1 98354000     mov     eax, [403598]                        ; 我们输入的字节数。
00401294    E3 17          jecxz   short zeroadd_.004012AD
00401296    FF76 3C         push    dword ptr [esi+3C]                   ; 文件对齐。
00401299    50             push    eax
0040129A    E8 7B020000     call    <zeroadd_.Align>                     ; 对齐我们输入的字节数。
0040129F    A3 98354000     mov     [403598], eax                        ; 新区段的文件中大小。
004012A4    FF76 38         push    dword ptr [esi+38]
004012A7    50             push    eax
004012A8    E8 6D020000     call    <zeroadd_.Align>
004012AD    0146 50         add     [esi+50], eax                        ; 更新映像大小。
004012B0    83C7 28         add     edi, 28                              ; 定位至新节表。
004012B3    57             push    edi                                    ; 保护。
004012B4    8D35 00304000   lea     esi, [403000]
004012BA    B9 08000000     mov     ecx, 8
004012BF    F3:A4          rep     movs byte ptr es:[edi], byte ptr [esi]; 新区段名称赋值。
004012C1    5F             pop     edi                                    ; 恢复。
004012C2    A1 98354000     mov     eax, [403598]
004012C7    8947 08         mov     [edi+8], eax                          ; 新区段的 VSize。
004012CA    A1 90354000     mov     eax, [403590]
004012CF    8947 0C         mov     [edi+C], eax                          ; 新区段的 RVA
004012D2    A1 98354000     mov     eax, [403598]
004012D7    8947 10         mov     [edi+10], eax                         ; 新区段的 RSize
004012DA    A1 94354000     mov     eax, [403594]
004012DF    8947 14         mov     [edi+14], eax                         ; 新区段的 Raw（文件偏移）
004012E2    C747 24 200000E>mov     dword ptr [edi+24], E0000020         ; 区块属性，可读可写可执行。
004012E9    8B3D 84354000   mov     edi, [403584]
004012EF    033D 94354000   add     edi, [403594]                         ; 定位至新区段开始位置。
004012F5    8B0D 98354000   mov     ecx, [403598]                         ; 新区段的大小为计数。
004012FB    32C0           xor     al, al
004012FD    F3:AA          rep     stos byte ptr es:[edi]                 ; 用 0 填充。
004012FF    2B3D 84354000   sub     edi, [403584]                         ; 减去基址，得到新文件大小。
```



00401305    8D35 29314000     lea     esi, [403129]                 ; 中间文件名称。  
0040130B    6A 00             push    0  
0040130D    68 80000000       push    80  
00401312    6A 02             push    2  
00401314    6A 00             push    0  
00401316    6A 00             push    0  
00401318    68 00000040       push    40000000  
0040131D    56                push    esi  
0040131E    E8 45020000       call    <jmp.&KERNEL32.CreateFileA>     ; 打开中间文件 `swapit.sca`  
00401323    85C0             test    eax, eax  
00401325    0F84 B1000000    je      `zeroadd_.004013DC`  
0040132B    50                push    eax  
0040132C    6A 00             push    0  
0040132E    68 1C304000       push    `zeroadd_.0040301C`  
00401333    57                push    edi                             ; 大小为添加区段后的大小。  
00401334    FF35 84354000     push    dword ptr [403584]              ; 文件映像的基址。  
0040133A    50                push    eax  
0040133B    E8 58020000       call    <jmp.&KERNEL32.WriteFile>       ; 写入至中间文件。  
00401340    85C0             test    eax, eax  
00401342    0F84 94000000    je      `zeroadd_.004013DC`  
00401348    58                pop     eax  
00401349    50                push    eax  
0040134A    E8 0D020000       call    <jmp.&KERNEL32.CloseHandle>     ; 清理资源。  
0040134F    FF35 84354000     push    dword ptr [403584]  
00401355    E8 38020000       call    <jmp.&KERNEL32.UnmapViewOfFile>  
0040135A    FF35 80354000     push    dword ptr [403580]  
00401360    E8 F7010000       call    <jmp.&KERNEL32.CloseHandle>  
00401365    FF35 7C354000     push    dword ptr [40357C]  
0040136B    E8 EC010000       call    <jmp.&KERNEL32.CloseHandle>  
00401370    6A 00             push    0                                 ; 为 `false`，表示覆盖同名文件。  
00401372    68 20304000       push    `zeroadd_.00403020`  
00401377    68 29314000       push    `zeroadd_.00403129`              ; `ASCII "swapit.sca"`  
0040137C    E8 E1010000       call    <jmp.&KERNEL32.CopyFileA>        ; 强行复制。  
00401381    0BC0             or      eax, eax  
00401383    75 0C            jnz     short `zeroadd_.00401391`  
00401385    68 29314000       push    `zeroadd_.00403129`              ; `ASCII "swapit.sca"`  
0040138A    E8 E5010000       call    <jmp.&KERNEL32.DeleteFileA>  
0040138F    EB 4B            jmp     short `zeroadd_.004013DC`  
00401391    68 29314000       push    `zeroadd_.00403129`              ; `ASCII "swapit.sca"`  
00401396    E8 D9010000       call    <jmp.&KERNEL32.DeleteFileA>      ; 删除中间文件。  

```
0040122F 这里的 mov edx，28h 便是 ZeroAdd 的一个 bug 。 我们只需将其改为 xor edx，edx 即可消除此 bug。 若不消此
bug 的话，则当末节表末尾与起始区段起始处之间空隙在 40<= space <80 时，它也会提示空间不够，显然这属于误判。
```

先看看常规的 `pe` 区段添加方法(原 `ZeroAdd` 的思路)：

1. 在最末节表后紧接着写入一个新的节表。并在文件后追加对应的字节。
2. 更新 `NT` 头中的信息， 具体为文件头中的区段数要加一，可选头中的映像大小也要更新。

好了， 现在我们要说下当 `space` 的的确确不足 40 时，我们该如何添加区段。

这种情况， 我们只需要另外加上这些步骤即可：

a. 将从第一区块的起始位置至文件尾这一区域都向后移动“文件对齐”个字节.
(这里不向后移动 40 个字节即节表的大小而是移动一个 `FileAlignment` 的值还是是为了保证文件对齐)

b. 修改各个节表的文件偏移，具体为皆加上“文件对齐”。

c. 如果 `BoundImport` 不为空，则要特别对待之，`BoundImport` 主要是为了消除 `PE` 装载器对 `IAT` 填充所占用的时间，
如果想比较完美地解决 `BoundImport` 的话，可以将其数据移到其他的地方，然后更新一下其目录表。 不过由于一般
`PE` 很少有 `BoundImport`，而且现在的机器都这么好，这么点时间我就不节省了，(![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")，别拍砖哈~~)， 所以我这里
直接将 `BoundImport` 的目录置为 0，将其对应数据也填 0 了事算了. 追求完美的朋友可以继续改造。

这里 c 步骤本来没有的，感谢看雪大哥指点，特用红色补充在此。![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

我们接下来用 3 块补丁代码来分别完成上面的 a；b；c 步骤及另一个处理。 具体分工如下：

补丁 1： 完成步骤 a 和 b， 并设置“是否需要后移区段”标志位。
补丁 2： 根据上述标志位决定是否增加写入新文件字节数。
补丁 3： 完成上述 c 步骤。

下面是 `ZeroAdd` 的一些全局变量的信息：

```
40357c  hFile
403580  hMap
403084  MZ 头指针
403088  PE 头指针
403590  新区段 RVA
403594  新区段文件偏移
403598  新区段的 RSize
... ...
```

下面这句就是判断空间不足的情况下的跳转：

`00401238  - 0F85 C29D0000   jnz     zeroadd_.0040B000`

这句本来是跳去弹错误框的，被我改成了跳到 `0040B000`. 这是我对 `ZeroAdd` 新加的一个区，用于我们写补丁代码。

也即当空间不足的时候，便会跳到我们的处理代码。

显然，只要我们在我们的处理代码中完成上述的 a，b 两个步骤，然后交给 `ZeroAdd`，再适当的做一些小的调整就可以了。

1）、对于步骤 a ：

原 `ZeroAdd` 使用的是先通过 `CreateFileMapping` 创建文件内存映射对象，再 `MapViewOfFile` 来映射至内存的， 其中
`ZeroAdd` 它在创建文件映射内核对象时，参数 `size` 被设置成了文件大小加上 `0x10000`， 这也极大地方便了我们。

实现代码如下：

```
mov edi, dword ptr [403088]   ;PE header pointer.
mov ebx, dword ptr [edi+3c]   ;FileAlignment
add edi, 0f8h                 ;0xf8 = sizeof(IMAGE_NT_HEADERS) .定位至第一节表。
mov edi, dword ptr [edi+14h]  ;Raw
mov edx, edi                  ;第一个区段的 Raw，我们保存一下。
mov esi, dword ptr [403584]   ;MZ header pointer.
add esi, edi                  ;至此得到 RtlMoveMemory 的 src.
mov edi, esi
add edi, ebx                  ;至此得到 RltMoveMemory 的 dst.
push esi
push edi                    ;保护一下 src 和 dst
push edx                    ;保护一下第一区段的 Raw
push 0
push dword ptr [40357C]     ;hFile
call GetFileSize            ;很不幸 ZeroAdd 没有保存 FileSize，所以我们自己 call 一遍.
pop edx
pop edi
pop esi
sub eax, edx                ;这里得到了 RtlMoveMemory 的 Length 参数.
mov ebx, edi
sub ebx, esi                ;保存一份 FileAlignment，下面的一句用的到
push ebx                    ;为 RtlZeroMemory 压栈 Length
push esi                    ;为 RtlZeroMemory 压栈 Destination.
push eax
push esi
push edi
call RtlMoveMemory
call RtlZeroMemory          ;至此已经完成
```

2）、对于步骤 b，就比较简单，如下：

```
mov edi,dword ptr [403588]   ;PE header.
mov ebx,dword ptr [edi+3c]   ;FileAlignment
movzx ecx,word  ptr [edi+6]  ;NumberOfSections
add edi, 0f8h               ;0xf8 = sizeof(IMAGE_NT_HEADERS) .
@@:
add [edi+14h],ebx
add edi, 28h
loop @B
```

3）、对于 步骤 c ：

补丁 3 执行时间应该为：在判断是否有足够 0 字节之前完成。

我们选择这里：

`0040122F  - E9 6C9E0000     jmp     zeroadd_.0040B0A0`

这里原版是 `mov edx，28` 我们需要改为 `xor edx，edx` 消除 bug，这里改成 `jmp` 过去，到了补丁再置 `edx` 为零。

代码如下：

```
0040B0A0    33D2            xor     edx, edx                      ; 补丁 3 开始
0040B0A2    9C              pushfd
0040B0A3    60              pushad
0040B0A4    8B3D 88354000   mov     edi, [403588]                 ; PE header pointer
0040B0AA    81C7 F8000000   add     edi, 0F8
0040B0B0    83EF 28         sub     edi, 28                       ; 减去 8*5，定位至 BoundImport 目录处。
0040B0B3    8B17            mov     edx, [edi]                    ; Raw
0040B0B5    8B5F 04         mov     ebx, [edi+4]                  ; RSize
0040B0B8    FC              cld
0040B0B9    B9 02000000     mov     ecx, 2
0040B0BE    33C0            xor     eax, eax
0040B0C0    F3:AB          rep     stos dword ptr es:[edi]        ; 清除目录中数据。
0040B0C2    90              nop
0040B0C3    8B3D 84354000   mov     edi, [403584]
0040B0C9    03FA            add     edi, edx
0040B0CB    8BCB            mov     ecx, ebx
0040B0CD    33C0            xor     eax, eax
0040B0CF    F3:AA          rep     stos byte ptr es:[edi]         ; 清除 BoundImport 数据。
0040B0D1    90              nop
0040B0D2    90              nop
0040B0D3    61              popad
0040B0D4    9D              popfd
0040B0D5  - E9 5A61FFFF     jmp     zeroadd_.00401234
```

我们在补丁代码中将步骤 a，b 写入进去， 写完了我们直接跳回判断是否有足够字节的下面的代码 `00401243` 处。

这时还没有完， 在 `ZeroAdd` 向 `swapit.sca` 写入即 `WriteFile` 的时候我们要改一下写入的字节数，应该使其在原来基础上
增加一个 `FileAlignment` 的大小。

我选择这个位置：

`0040131E    E8 45020000     call    <jmp.&KERNEL32.CreateFileA>     ; 打开中间文件 swapit.sca`
`00401323    85C0            test    eax, eax`
`00401325    0F84 B1000000   je      zeroadd_.004013DC`

这里的 `test` 和 `je` 我们可以 `nop` 掉然后跳到我们的补丁 2，改写 `edi` 后再调回来，紧接着便 `WriteFile` 了。

当然我们最好还使用一个全局变量保存一个标志，即是否 `space` 不足。 这里我们使用`[403600]`来保存之。

完整的补丁代码如下：

```
0040B000    9C              pushfd                                ;补丁一。
```



```
0040B001    60                   pushad
0040B002    8B3D 88354000      mov     edi, [403588]                     ;步骤 a 开始
0040B008    8B5F 3C            mov     ebx, [edi+3C]
0040B00B    81C7 F8000000      add     edi, 0F8
0040B011    8B7F 14            mov     edi, [edi+14]
0040B014    8BD7               mov     edx, edi
0040B016    8B35 84354000      mov     esi, [403584]
0040B01C    03F7               add     esi, edi
0040B01E    8BFE               mov     edi, esi
0040B020    03FB               add     edi, ebx
0040B022    56                 push    esi
0040B023    57                 push    edi
0040B024    52                 push    edx
0040B025    6A 00              push    0
0040B027    FF35 7C354000      push    dword ptr [40357C]
0040B02D    E8 D55A407C        call    kernel32.GetFileSize
0040B032    5A                 pop     edx
0040B033    5F                 pop     edi
0040B034    5E                 pop     esi
0040B035    2BC2               sub     eax, edx
0040B037    8BDF               mov     ebx, edi
0040B039    2BDE               sub     ebx, esi
0040B03B    53                 push    ebx
0040B03C    56                 push    esi
0040B03D    50                 push    eax
0040B03E    56                 push    esi
0040B03F    57                 push    edi
0040B040    E8 4F7C517C        call    ntdll.RtlMoveMemory
0040B045    E8 1A7C517C        call    ntdll.RtlZeroMemory
0040B04A    8B3D 88354000      mov     edi, [403588]                     ;步骤 b 开始.
0040B050    8B5F 3C            mov     ebx, [edi+3C]
0040B053    0FB74F 06          movzx   ecx, word ptr [edi+6]
0040B057    81C7 F8000000      add     edi, 0F8
0040B05D    015F 14            add     [edi+14], ebx
0040B060    83C7 28            add     edi, 28
0040B063    ^ E2 F8            loopd   short zeroadd_.0040B05D
0040B065    C705 00364000 0>mov    dword ptr [403600], 1
0040B06F    61                 popad
0040B070    9D                 popfd
0040B071    - E9 CD61FFFF      jmp     zeroadd_.00401243
0040B076    90                 nop
0040B077    90                 nop
0040B078    90                 nop                                     ; 补丁二：当 WriteFile 时，会到这里。
0040B079    90                 nop
0040B07A    85C0               test    eax, eax
0040B07C    - 0F84 5A63FFFF    je      zeroadd_.004013DC
0040B082    53                 push    ebx
0040B083    8B1D 88354000      mov     ebx, [403588]
0040B089    8B5B 3C            mov     ebx, [ebx+3C]
0040B08C    833D 00364000 0>cmp    dword ptr [403600], 0
0040B093    74 02              je      short zeroadd_.0040B097
0040B095    03FB               add     edi, ebx
0040B097    5B                 pop     ebx
0040B098    - E9 8E62FFFF      jmp     zeroadd_.0040132B
0040B09D    90                 nop
0040B09E    90                 nop
0040B09F    90                 nop
0040B0A0    33D2               xor     edx, edx                     ; 补丁 3 开始
0040B0A2    9C                 pushfd
0040B0A3    60                 pushad
0040B0A4    8B3D 88354000      mov     edi, [403588]                  ; PE header pointer
0040B0AA    81C7 F8000000      add     edi, 0F8
0040B0B0    83EF 28            sub     edi, 28                        ; 减去 8*5，定位至 BoundImport 目录处。
0040B0B3    8B17               mov     edx, [edi]                     ; Raw
0040B0B5    8B5F 04            mov     ebx, [edi+4]                   ; RSize
0040B0B8    FC                 cld
0040B0B9    B9 02000000        mov     ecx, 2
0040B0BE    33C0               xor     eax, eax
0040B0C0    F3:AB              rep     stos dword ptr es:[edi]        ; 清除目录中数据。
0040B0C2    90                 nop
0040B0C3    8B3D 84354000      mov     edi, [403584]
0040B0C9    03FA               add     edi, edx
0040B0CB    8BCB               mov     ecx, ebx
0040B0CD    33C0               xor     eax, eax
0040B0CF    F3:AA              rep     stos byte ptr es:[edi]         ; 清除 BoundImport 数据。
0040B0D1    90                 nop
0040B0D2    90                 nop
0040B0D3    61                 popad
0040B0D4    9D                 popfd
0040B0D5    - E9 5A61FFFF      jmp     zeroadd_.00401234
```

另外，每次`ZeroAdd`加完区段后，都会自动退出。因此我顺便把后面的`ExitProcess`也擦掉了。

附件为修改后的`ZeroAdd`，以及一个`space`不足的供测试的样品`PE`。

呵呵，没什么技术含量，就是把个`PE`文件搞来搞去。
![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

---

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2009 年 01 月 12 日 PM 11:23:36

上传的附件

| | `test.rar` |
|---|---|
| | `ZeroAdd_modified_sp2.rar` |
