
# PDF Password Remover v3.0 注册算法简单分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-208.htm`](https://www.pediy.com/kssd/pediy09/pediy09-208.htm)

> 【文章标题】: PDF Password Remover v3.0 注册算法简单分析  

> 【文章作者】: DarkBoxer 暗夜拳师  

> 【软件名称】: PDF Password Remover v3.0  

> 【下载地址】: `http://www.verypdf.com/pwdremover/pwdremover.exe`  

> 【使用工具】: `OD`  

> 【操作平台】: `WindowsXPsp2`  

> 【软件介绍】: 本软件可以破解 Adobe Acrobat PDF 文件高达 128-bit 加密的“所有者密码”。未破解“所有者密码”的 PDF 文件是不能被编辑和打印的。该程序也可以破解用 `FileOpen` 插件加密的文件。破解可以立即完成。破解后的文件可以用各种 PDF 浏览器阅读而无任何限制。  

> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

-----------------------------------------------------------------------------------------

【详细过程】

## 背景

今天要拷贝一个 PDF 文件里面的代码，不过有限制，所以就去某网站下载了一个 PDF Password Remover v3.0，轻松搞定 PDF。提供下载的网站给出了 2 个注册码，都可用，嘻嘻~反正今天还有点闲，索性分析下此软件的注册算法，结果发现是个软柿子~~^\_^

## 脱壳

软件是 `UPX` 加壳,秒之~

测试试炼码: `12345678901darkboxer`

## 分析

```
0040539D            68 10B54B00      push    004BB510                          ; ASCII "12345678901darkboxer"
004053A2            E8 59F8FFFF      call    00404C00                          ; //这里就是验证试炼码是否正确
004053A7            83C4 04          add     esp, 4
004053AA            85C0             test    eax, eax
004053AC            74 44           je      short 004053F2                    ; //呵呵，爆破点
004053AE            6A 40           push    40
004053B0            68 946A4900      push    00496A94                          ; ASCII "Thank you."
004053B5            68 5C6A4900      push    00496A5C                          ; ASCII "Thanks for purchasing the PDF Password Remover v3.0."
004053BA            56              push    esi
004053BB            FF15 08844800   call    dword ptr [<&user32.MessageBoxA>] ; USER32.MessageBoxA
004053C1            51              push    ecx
004053C2            8BCC             mov     ecx, esp
004053C4            896424 10       mov     dword ptr [esp+10], esp
004053C8            68 10B54B00      push    004BB510                          ; ASCII "12345678901darkboxer"
004053CD            E8 686B0700      call    0047BF3A
004053D2            E8 F9FBFFFF      call    00404FD0
004053D7            83C4 04          add     esp, 4
004053DA            C705 D8B54B00 0>mov     dword ptr [4BB5D8], 1
004053E4            6A 01           push    1
004053E6            56              push    esi
004053E7            FF15 04844800   call    dword ptr [<&user32.EndDialog>]   ; USER32.EndDialog
004053ED            E9 86020000      jmp     00405678
004053F2            6A 10           push    10
004053F4            6A 00           push    0
004053F6            68 146A4900      push    00496A14                          ; ASCII "Your registration key is wrong, please double check it and try again."
004053FB            56              push    esi
004053FC            FF15 08844800   call    dword ptr [<&user32.MessageBoxA>] ; USER32.MessageBoxA
```

;跟进 404c00

```
00404C00            83EC 18         sub     esp, 18
00404C03            83C9 FF        or      ecx, FFFFFFFF
00404C06            33C0             xor     eax, eax
00404C08            53              push    ebx
00404C09            56              push    esi
00404C0A            8B7424 24       mov     esi, dword ptr [esp+24]
00404C0E            57              push    edi
00404C0F            8BFE             mov     edi, esi
00404C11            F2:AE          repne   scas byte ptr es:[edi]
00404C13            F7D1             not     ecx
00404C15            49              dec     ecx
00404C16            83F9 14        cmp     ecx, 14                             ; 注册码长度必须为十 20(10 进制)位
00404C19            74 07           je      short 00404C22                    ; 不满足就拜拜
00404C1B            5F              pop     edi
00404C1C            5E              pop     esi
00404C1D            5B              pop     ebx
00404C1E            83C4 18        add     esp, 18
00404C21            C3              retn
00404C22            8A06             mov     al, byte ptr [esi]               ; 注册码第一位,这里是‘1’
00404C24            8A4E 01        mov     cl, byte ptr [esi+1]             ; 注册码第二位,这里是‘2’
00404C27            8D5424 0C      lea     edx, dword ptr [esp+C]           ; [edx]就是注册码 12A45678903darkboxer
00404C2B            32DB             xor     bl, bl                           ; bl 清零
00404C2D            52              push    edx
00404C2E            884424 1C      mov     byte ptr [esp+1C], al
00404C32            885C24 1D      mov     byte ptr [esp+1D], bl
00404C36            884C24 10      mov     byte ptr [esp+10], cl
00404C3A            885C24 11      mov     byte ptr [esp+11], bl
00404C3E            E8 AD090600    call    004655F0                          ; //关键点,多次被调用
00404C43            8BF8             mov     edi, eax                          ; //由第 2 位得出的结果->edi
00404C45            8D4424 1C      lea     eax, dword ptr [esp+1C]
00404C49            50              push    eax
00404C4A            E8 A1090600    call    004655F0
00404C4F            03F8             add     edi, eax                          ; //第 2 位得出的结果和第 2 位得出的结果相加
00404C51            83C4 08        add     esp, 8
00404C54            83FF 0A        cmp     edi, 0A                             ; //是否等于 0Ah
00404C57            74 09           je      short 00404C62
00404C59            5F              pop     edi
00404C5A            5E              pop     esi
00404C5B            33C0             xor     eax, eax
00404C5D            5B              pop     ebx
00404C5E            83C4 18        add     esp, 18
00404C61            C3              retn
00404C62            8A4E 12        mov     cl, byte ptr [esi+12]             ; //注册码第 19 位
00404C65            8A56 13        mov     dl, byte ptr [esi+13]             ; //注册码第 20 位
00404C68            8D4424 0C      lea     eax, dword ptr [esp+C]
00404C6C            884C24 18      mov     byte ptr [esp+18], cl
00404C70            50              push    eax
00404C71            885C24 1D      mov     byte ptr [esp+1D], bl
00404C75            885424 10      mov     byte ptr [esp+10], dl
00404C79            885C24 11      mov     byte ptr [esp+11], bl
00404C7D            E8 6E090600    call    004655F0
00404C82            8D4C24 1C      lea     ecx, dword ptr [esp+1C]
00404C86            8BF8             mov     edi, eax                          ; //第 20 位得出的结果->edi
00404C88            51              push    ecx
00404C89            E8 62090600    call    004655F0
00404C8E            03F8             add     edi, eax                          ; //第 19 位得出的结果和第 20 位得出的结果相加
00404C90            83C4 08        add     esp, 8
00404C93            83FF 0C        cmp     edi, 0C                             ; //是否等于 0Ch
00404C96            74 09           je      short 00404CA1
00404C98            5F              pop     edi
00404C99            5E              pop     esi
00404C9A            33C0             xor     eax, eax
00404C9C            5B              pop     ebx
00404C9D            83C4 18        add     esp, 18
00404CA0            C3              retn
00404CA1            8A56 05        mov     dl, byte ptr [esi+5]              ; //注册码第 6 位
00404CA4            8A46 0D        mov     al, byte ptr [esi+D]              ; //注册码第 14 位
00404CA7            8D4C24 0C      lea     ecx, dword ptr [esp+C]
00404CAB            885424 18      mov     byte ptr [esp+18], dl
00404CAF            51              push    ecx
```



### 汇编代码片段

```asm
00404CB0    885C24 1D    mov     byte ptr [esp+1D], bl  
00404CB4    884424 10    mov     byte ptr [esp+10], al  
00404CB8    885C24 11    mov     byte ptr [esp+11], bl  
00404CBC    E8 2F090600  call    004655F0  
00404CC1    8D5424 1C    lea     edx, dword ptr [esp+1C]  
00404CC5    8BF8         mov     edi, eax             ; //第 14 位得出的结果->edi  
00404CC7    52           push    edx  
00404CC8    E8 23090600  call    004655F0  
00404CCD    03F8         add     edi, eax             ; //第 6 位得出的结果和第 14 位得出的结果相加  
00404CCF    83C4 08     add     esp, 8  
00404CD2    83FF 0B     cmp     edi, 0B              ; //是否等于 0Bh  
00404CD5    74 09      je      short 00404CE0  
00404CD7    5F          pop     edi  
00404CD8    5E          pop     esi  
00404CD9    33C0       xor     eax, eax  
00404CDB    5B          pop     ebx  
00404CDC    83C4 18     add     esp, 18  
00404CDF    C3          retn  
00404CE0    807E 0C 52  cmp     byte ptr [esi+C], 52      ; //第 13 位是否等于 52h,即‘R’  
00404CE4    74 09      je      short 00404CEF  
00404CE6    5F          pop     edi  
00404CE7    5E          pop     esi  
00404CE8    33C0       xor     eax, eax  
00404CEA    5B          pop     ebx  
00404CEB    83C4 18     add     esp, 18  
00404CEE    C3          retn  
00404CEF    807E 0E 33  cmp     byte ptr [esi+E], 33      ; //第 15 位是否等于 33h，即‘3’  
00404CF3    74 09      je      short 00404CFE  
00404CF5    5F          pop     edi  
00404CF6    5E          pop     esi  
00404CF7    33C0       xor     eax, eax  
00404CF9    5B          pop     ebx  
00404CFA    83C4 18     add     esp, 18  
00404CFD    C3          retn  
00404CFE    8A4E 0F     mov     cl, byte ptr [esi+F]      ; //第 16 位->cl  
00404D01    33C0       xor     eax, eax  
00404D03    80F9 30     cmp     cl, 30                     ; //第 16 位是否等于 30h,即 0  
00404D06    5F          pop     edi  
00404D07    5E          pop     esi  
00404D08    5B          pop     ebx  
00404D09    0F94C0     sete    al                         ; //等于则置 al 为 1  
00404D0C    83C4 18     add     esp, 18  
00404D0F    C3          retn  
```

### 分析过程

在分析的过程中，发现被程序检测的第 19、20、14 位填的字母，在调用 `4655f0` 后得出的结果都是 0。反正在吃粉丝,嘻嘻，不好意思，分析的时候肚子饿了，干脆就悠哉悠哉的多测试了几组数据，发现注册码 `0~9` 之间，函数返回的就是 `0~9`，而输入字母，则统统为 0，嘻嘻，颇有点选择明文攻击的味道。加上看到原先下载的网站给出的注册码是通用的时候，我就已经怀疑机器 ID 是唬人的，再加上，注册算法中确实没看到机器 ID 的出现,莫非我眼花了。于是乎我就猜测它的注册算法：

1. 长度必须等于 20(十进制)位
2. 第 1 位和第 2 位的和必须等于 `A`
3. 第 19 位和第 20 位的和必须等于 `C`
4. 第 6 位和第 14 位必须等于 `B`
5. 第 13 位必须等于 `52h`，即`'R'`
6. 第 15 位必须等于 `33h`，即`'3'`
7. 第 16 位必须等于 `30h`，即`'0'`

`OK!`胡乱测试一组试炼码：`55@@@5@@@@@@R630@@66`

`my god;`差点喷面，居然成功了....GS 运~~~嘻嘻~

### 深入分析

现在方面的粉丝的份量真的太少了，没几下就被我解决了-\_-!自己其实也想看看 `4655f0` 函数是咋样的，所以索性接着看了下

#### `4655f0` 函数

```asm
; 4655f0
004655F0    FF7424 04   push    dword ptr [esp+4]       ; //需要被计算的那位注册码,这里是第 2 位‘2’  
004655F4    E8 6CFFFFFF call    00465565                ; //继续跟进看了  
004655F9    59          pop     ecx  
004655FA    C3          retn  
```

#### `465565` 函数

```asm
; 465565
00465565    53          push    ebx  
00465566    55          push    ebp  
00465567    56          push    esi  
00465568    57          push    edi  
00465569    8B7C24 14   mov     edi, dword ptr [esp+14]           ; //edi 存放被压入的那位注册码'2'  
0046556D    833D DCA84B00 0>cmp     dword ptr [4BA8DC], 1  
00465574    7E 0F       jle     short 00465585  
00465576    0FB607      movzx   eax, byte ptr [edi]  
00465579    6A 08       push    8  
0046557B    50          push    eax  
0046557C    E8 A35E0000 call    0046B424  
00465581    59          pop     ecx  
00465582    59          pop     ecx  
00465583    EB 0F       jmp     short 00465594  
00465585    0FB607      movzx   eax, byte ptr [edi]  
00465588    8B0D D0A64B00   mov     ecx, dword ptr [4BA6D0]           ; //ds:[004BA6D0]=004BA6DA  
0046558E    8A0441      mov     al, byte ptr [ecx+eax*2]             ; //al<-84  
00465591    83E0 08     and     eax, 8                               ; //和 8 与  
00465594    85C0        test    eax, eax                             ; //是否为 0  
00465596    74 03       je      short 0046559B  
00465598    47          inc     edi  
00465599      ^ EB D2     jmp     short 0046556D  
0046559B    0FB637      movzx   esi, byte ptr [edi]                 ; //esi<-'2'  
0046559E    47          inc     edi                                   ; //[edi]为 0，参考前面地址 404c32  
0046559F    83FE 2D     cmp     esi, 2D                               ; //是否等于‘-’  
004655A2    8BEE        mov     ebp, esi  
004655A4    74 05       je      short 004655AB  
004655A6    83FE 2B     cmp     esi, 2B                               ; //是否等于‘+’  
004655A9    75 04       jnz     short 004655AF  
004655AB    0FB637      movzx   esi, byte ptr [edi]  
004655AE    47          inc     edi  
004655AF    33DB        xor     ebx, ebx  
004655B1    833D DCA84B00 0>cmp     dword ptr [4BA8DC], 1  
004655B8    7E 0C       jle     short 004655C6  
004655BA    6A 04       push    4  
004655BC    56          push    esi  
004655BD    E8 625E0000 call    0046B424  
004655C2    59          pop     ecx  
004655C3    59          pop     ecx  
004655C4    EB 0B       jmp     short 004655D1                        ; //下面才是关键  
004655C6    A1 D0A64B00 mov     eax, dword ptr [4BA6D0]               ; //ds:[004BA6D0]=004BA6DA  
004655CB    8A0470      mov     al, byte ptr [eax+esi*2]             ; //al<-84，esi 是注册码 32h  
004655CE    83E0 04     and     eax, 4                               ; //和 4 相与  
004655D1    85C0        test    eax, eax                             ; //是否为 0  
004655D3    74 0D       je      short 004655E2  
004655D5    8D049B      lea     eax, dword ptr [ebx+ebx*4]  
004655D8    8D5C46 D0   lea     ebx, dword ptr [esi+eax*2-30]        ; //ebx 指向 esi+eax*2-30 的地方,而这个地方的值是:该位注册码+eax*2-30，而 eax 为 0，所以为：注册码-30  
004655DC    0FB637      movzx   esi, byte ptr [edi]  
004655DF    47          inc     edi  
004655E0      ^ EB CF     jmp     short 004655B1  
004655E2    83FD 2D     cmp     ebp, 2D  
004655E5    8BC3        mov     eax, ebx  
004655E7    75 02       jnz     short 004655EB  
004655E9    F7D8        neg     eax  
004655EB    5F          pop     edi  
004655EC    5E          pop     esi  
004655ED    5D          pop     ebp  
004655EE    5B          pop     ebx  
```



`004655EF` `C3` `retn`

`OK!` 关键的地方其实就在 `004655C6`~`004655D8` 之间，必须保证和 `4` 相与后的 `eax` 不为 `0`，按这条流程下去，才会保证返回值为（注册码-30），否则返回值为 `0`

而 `eax` 的保证来源于内存 `4ba6da`，即如下数组：

```c
char eax_key[] = 
{ 
0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00,  
0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x28, 0x00,  
0x28, 0x00, 0x28, 0x00, 0x28, 0x00, 0x28, 0x00, 0x20, 0x00,  
0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00,  
0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00,  
0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00, 0x20, 0x00,  
0x20, 0x00, 0x20, 0x00, 0x48, 0x00, 0x10, 0x00, 0x10, 0x00,  
0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00,  
0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00,  
0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x84, 0x00, 0x84, 0x00,  
0x84, 0x00, 0x84, 0x00, 0x84, 0x00, 0x84, 0x00, 0x84, 0x00,  
0x84, 0x00, 0x84, 0x00, 0x84, 0x00, 0x10, 0x00, 0x10, 0x00,  
0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00,  
0x81, 0x00, 0x81, 0x00, 0x81, 0x00, 0x81, 0x00, 0x81, 0x00,  
0x81, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00,  
0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00,  
0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00,  
0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00,  
0x01, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00, 0x10, 0x00,  
0x10, 0x00, 0x10, 0x00, 0x82, 0x00, 0x82, 0x00, 0x82, 0x00,  
0x82, 0x00, 0x82, 0x00, 0x82, 0x00, 0x02, 0x00, 0x02, 0x00,  
0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00,  
0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00,  
0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00,  
0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x10, 0x00, 0x10, 0x00,  
0x10, 0x00, 0x10, 0x00, 0x20 
};
```

如 `004655CB` `8A0470` `mov al, byte ptr [eax+esi*2]` ; `//al<-84，esi 是注册码 32h`

`al` 就是 `eax_key[eax+esi*2]`, `eax` 是 `004BA6DA`=数组基址 `0`，`esi` 为该位注册码 `'2'`，即 `32h*2=64h=100`，所以为 `eax_key[100]` = `0x84`； `0x84 & 4` = `4`；

在来一个 `'A'`，16 进制 `41`，`41h*2=82H=130`，所以 `eax_key[130]` = `0x81`； `0x81 & 4`=0

只有为 `0x84`，和 `4` 与才不会为 `0`，而 `0x84` 正好是 10 个，正好是 `0~9` 十个即数组 `96` 开始，因为乘 `2`，正好相隔一个，自己算算吧~~

程序就是这样来判断的。。

**注册机**

呵呵，不用了吧。手动计算貌似还更快，呵呵~今天捡到个软柿子~~运气不错^\_^

--------------------------------------------------------------------------------

【版权声明】: 

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 07 月 22 日 20:17:02 

`4dd466d0e1e0cc123afbe504da2462ca`
