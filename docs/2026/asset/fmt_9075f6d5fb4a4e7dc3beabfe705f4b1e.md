
# [VC]Advanced Installer 3.81 注册文件算法分析

【标题】【VC】Advanced Installer 3.81 注册文件算法分析

【作者】forever[RCT]

【语言】VC7.1

【保护】无壳，注册文件

【难度】中等

【工具】ollydbg,ida,peid

【简介】一个能制作CAB安装包的安装制作软件。

## 正文

这个软件用`peid`查不出是什么语言的，直接看到连接版本是7.1。不过在软件的安装目录下有`msvcr71.dll`、`msvcp71.dll`这两个VC的运行时文件，可以猜测是用VC写的。

这个软件的注册方式也特别，是要你先输入一个32字符的序列号，然后到他的网站上验证，通过后会下载一个协议文件来。网站验证地址是：

`347K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4V1N6X3q4F1j5$3g2V1K9h3&6K6N6r3q4D9L8r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2Y4K9i4y4@1k6i4u0Q4x3V1k6U0L8$3&6X3K9i4u0E0i4K6u0W2M7r3S2H3i4K6y4r3K9h3c8Q4x3@1b7H3x3o6l9H3x3o6l9H3x3o6l9H3x3o6l9H3x3o6l9H3x3o6l9H3x3o6l9H3x3o6l9H3x3o6l9H3x3o6l9H3x3#2)9J5y4Y4k6W2M7W2)9K6c8o6y4Q4x3X3f1^5i4K6u0W2x3g2)9J5y4Y4m8D9j5i4c8X3L8%4u0E0i4K6y4p5g2$3W2F1h3q4l9`。

那个`id`字段就是要你输入的序列号。弄协议文件我是不行啦。随便写个协议文件又异常，看样子是没法分析啦。

这里要感谢`LeNgHost`，帮助找了一个2.X版本的注册机。一般软件的编写都有继承性，即使变化也很少从根本上变化。注册算法也是这样。先生成一个老的注册文件，然后慢慢跟踪分析。

下面这段是`ida`里反汇编出来的，结合`ollydbg`，可以容易分析软件怎么验证注册文件。

用`ollydbg`载入程序，下断点`bp CreateFileA`，会来到下面位置：

![](http://bbs.pediy.com/upload/2006/4/image/a1.jpg)

逐层返回后来到下面：

```
004342C6             lea     eax, [ebp-8Ch]

004342CC
004342CC loc_4342CC:
004342CC             push    esi
004342CD             push    edi
004342CE             push    1B6h
004342D3             push    1
004342D5             push    eax             ; C:\Program Files\Caphyon\Advanced Installer\.license
004342D6             lea     ecx, [ebp-278h]
004342DC             call    ds:std::basic_ifstream<wchar_t,std::char_traits<wchar_t>>::open(char const *,int,int)
004342E2             mov     eax, [ebp-278h]
004342E8             mov     eax, [eax+4]
004342EB             mov     eax, [ebp+eax-270h]
004342F2             test    al, 6
004342F4             jnz     loc_4348AF
004342FA             mov     edi, [ebp-54h]  ; 取第一行，即用户名
004342FD             push    ebx
004342FE             push    edi
004342FF             lea     ecx, [ebp-278h]
00434305             call    ds:std::basic_istream<wchar_t,std::char_traits<wchar_t>>::getline(wchar_t *,int)
0043430B             push    edi
0043430C             lea     ecx, [ebp-48h]  ; 连接上第一行字符串
0043430F             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::operator+=(wchar_t const *)
00434315             push    offset a347134d7 ; "347134d7"
0043431A             lea     ecx, [ebp-48h]  ; 连接上"347134d7"
0043431D             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::operator+=(wchar_t const *)
00434323             push    edi
00434324             lea     ecx, [ebp-2Ch]
00434327             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(wchar_t const *)
0043432D             mov     byte ptr [ebp-4], 5
00434331             lea     eax, [ebp-2Ch]
00434334             push    eax
00434335             lea     eax, [ebp-74h]
00434338             push    eax
00434339             call    sub_434B8E      ; 取第一个空格后面的字符串
0043433E             pop     ecx
0043433F             pop     ecx
00434340             mov     byte ptr [ebp-4], 6
00434344             push    eax
00434345             mov     ecx, offset username_73CFF4 ; 这里保存用户名
0043434A             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::operator=(std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>> const &)
00434350             mov     byte ptr [ebp-4], 5
00434354             lea     ecx, [ebp-74h]
00434357             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::~basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(void)
0043435D             mov     byte ptr [ebp-4], 4
00434361             lea     ecx, [ebp-2Ch]
00434364             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::~basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(void)
0043436A             push    ebx
0043436B             push    edi
0043436C             lea     ecx, [ebp-278h]
00434372             call    ds:std::basic_istream<wchar_t,std::char_traits<wchar_t>>::getline(wchar_t *,int)
00434378             push    edi             ; 取第二行，即email
00434379             lea     ecx, [ebp-48h]  ; 连接上第二行字符串
0043437C             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::operator+=(wchar_t const *)
00434382             push    offset a29fbcb9c ; "29fbcb9c"
00434387             lea     ecx, [ebp-48h]  ; 连接上字符串"29fbcb9c"
0043438A             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::operator+=(wchar_t const *)
00434390             push    ebx
00434391             push    edi
00434392             lea     ecx, [ebp-278h]
00434398             call    ds:std::basic_istream<wchar_t,std::char_traits<wchar_t>>::getline(wchar_t *,int)
0043439E             push    edi             ; 取第三行，即注册组织
0043439F             lea     ecx, [ebp-48h]  ; 连接上第三行字符串
004343A2             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::operator+=(wchar_t const *)
004343A8             push    offset aDb481813 ; "db481813"
004343AD             lea     ecx, [ebp-48h]  ; 连接上字符串"db481813"
004343B0             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::operator+=(wchar_t const *)
004343B6             push    edi
004343B7             lea     ecx, [ebp-2Ch]
004343BA             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(wchar_t const *)
004343C0             mov     byte ptr [ebp-4], 7
004343C4             lea     eax, [ebp-2Ch]
004343C7             push    eax
004343C8             lea     eax, [ebp-100h] ; 第三行
004343CE             push    eax
004343CF             call    sub_434BD6      ; 取第一个空格前面的字符串
004343D4             pop     ecx
004343D5             pop     ecx
004343D6             mov     byte ptr [ebp-4], 9
004343DA             lea     ecx, [ebp-2Ch]
004343DD             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::~basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(void)
004343E3             lea     eax, [ebp-100h]
004343E9             push    offset unk_73D0DC ; "IP:"
004343EE             push    eax
004343EF             call    ds:std::operator!=<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>> const &,std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>> const &)
004343F5             test    al, al          ; 比较是否等于"IP:"
004343F7             pop     ecx
004343F8             pop     ecx
004343F9             jz      short loc_434417
004343FB             mov     byte_73CF79, 1  ; 不等则校验失败
00434402             mov     byte ptr [ebp-4], 4
00434406             lea     ecx, [ebp-100h]
0043440C             call    ds:std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::~basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(void)
00434412             jmp     loc_4348BB

00434417
00434417 loc_434417:
00434417             push    ebx
00434418             push    edi
00434419             lea     ecx, [ebp-278h]
0043441F             call    ds:std::basic_istream<wchar_t,std::char_traits<wchar_t>>::getline(wchar_t *,int)
00434425             push    edi             ; 取第四行，即协议类型
```
