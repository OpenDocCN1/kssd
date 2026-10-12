
# EximiousSoft GIF Creator 3.15

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-213.htm`](https://www.pediy.com/kssd/pediy08/pediy8-213.htm)

【破解软件】EximiousSoft GIF Creator 3.15

【下载地址】`http://www.onlinedown.net/soft/44428.htm`

【运行环境】Win9x/Me/NT/2000/XP/2003

【软件类别】国外软件/共享版/动画制作

【保护方式】注册码

【作者声明】初学 Crack，只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教。

【调试环境】Winxp、`OllyDBD`、`PEiD`

【软件信息】一款 GIF 动画制作软件。提供很多强大的功能帮助你快速制作精彩的动画。支持从多种格式的图像中载入动画的一帧或多帧，支持载入的格式包括：`GIF`, `AVI`, `BMP`, `PCX`, `JPG`, `TIFF`, `PNG`, `TGA`, `PSD`, `ICO`, `CUR`, `ANI` 等等。

【破解过程】世界杯比赛如火如荼，才三天已分不清白天黑夜了。忙里偷闲，破解了个验证部分超级简单的软件，算法部分是浮点运算好象还有 `DES`，要看世界杯，算法部分就跳过了。

`PEiD` 检查：Microsoft Visual C++ 6.0，输入注册码：`1234567891`，提示“`Invalid serial number`”，`OD` 载入程序查找字串参考，未找到“`Invalid serial number`”直接下断：`bp MessgeBoxA`

```
77D5050B  >   8BFF             mov     edi, edi                                  ; 断下
77D5050D    55               push    ebp
77D5050E    8BEC             mov     ebp, esp
77D50510    833D 1C04D777 0> cmp     dword ptr [77D7041C], 0
77D50517    74 24            je      short 77D5053D
```

取消断点，`Alt+F9` 返回，在 `0047225E` 处下断。

```
0047225E   .   51              push    ecx                                       ; 断下
0047225F   .   68 C8040000     push    4C8
00472264   .   8BCE            mov     ecx, esi
00472266   .   E8 53820200     call    <jmp.&MFC42.#3092_CWnd::GetDlgItem>
0047226B   .   8BC8            mov     ecx, eax
0047226D   .   E8 B4830200     call    <jmp.&MFC42.#3874_CWnd::GetWindowTextA> ;  取注册码
00472272   .   68 58F00000     push    0F058
00472277   .   8D4C24 10       lea     ecx, [esp+10]
0047227B   .   E8 9A830200     call    <jmp.&MFC42.#4160_CString::LoadStringA>
00472280   .   8B5424 10       mov     edx, [esp+10]                              ;  [esp+10]注册码地址
00472284   .   8B42 F8         mov     eax, [edx-8]                               ;  [edx-8]注册码位数地址
00472287   .   83F8 0A         cmp     eax, 0A                                    ;  注册码是否等于 10 位
0047228A   .   0F85 8D000000   jnz     0047231D
00472290   .   8D4424 14       lea     eax, [esp+14]
00472294   .   33C9            xor     ecx, ecx
00472296   .   2BD0            sub     edx, eax
00472298   >   8D040A          lea     eax, [edx+ecx]
0047229B   .   8A4404 14       mov     al, [esp+eax+14]                           ;  顺取注册码
0047229F   .   3C 30           cmp     al, 30
004722A1   .   7C 10           jl      short 004722B3                             ;  小于转移
004722A3   .   3C 39           cmp     al, 39
004722A5   .   7F 0C           jg      short 004722B3                             ;  大于转移
004722A7   .   2C 30           sub     al, 30
004722A9   .   88440C 14       mov     [esp+ecx+14], al                           ;  保存 al=01 02…09 01
004722AD   .   41              inc     ecx                                        ;  计数器
004722AE   .   83F9 0A         cmp     ecx, 0A
004722B1   .^  7C E5           jl      short 00472298                             ;  ecx < A 循环
004722B3   >   83F9 0A         cmp     ecx, 0A
004722B6   .   75 65           jnz     short 0047231D
004722B8   .   51              push    ecx                                       
004722B9   .   8D4C24 18       lea     ecx, [esp+18]                              
004722BD   .   51              push    ecx                                       
004722BE   .   8B0D 287E4C00   mov     ecx, [4C7E28]                             
004722C4   .   E8 272CFBFF     call    00424EF0                                   ;  F7 进入   
004722C9   .   85C0            test    eax, eax                                   ;  ex=1 注册成功
004722CB   .   74 50           je      short 0047231D                             ;  跳注册失败
004722CD   .   8B4424 0C       mov     eax, [esp+C]
004722D1   .   6A 01           push    1
004722D3   .   8D5424 28       lea     edx, [esp+28]
004722D7   .   8B48 F8         mov     ecx, [eax-8]
004722DA   .   51              push    ecx
004722DB   .   50              push    eax                    ;  eax=00E10FF0, (ASCII "Successful")
004722DC   .   6A 20           push    20
004722DE   .   68 D4534C00     push    004C53D4
004722E3   .   52              push    edx
004722E4   .   E8 C7BE0100     call    0048E1B0
```

`004722C4` 处 `F7` 来到：

```
00424EF0  /$   83EC 0C         sub     esp, 0C
00424EF3  |.   33C0            xor     eax, eax
00424EF5  |.   8D5424 00       lea     edx, [esp]
00424EF9  |.   894424 01       mov     [esp+1], eax
00424EFD  |.   6A 0A           push    0A          
00424EFF  |.   894424 09       mov     [esp+9], eax           
00424F03  |.   52              push    edx                 
00424F04  |.   C64424 08 00    mov     byte ptr [esp+8], 0       
00424F09  |.   884424 11       mov     [esp+11], al           
00424F0D  |.   E8 CEFDFFFF     call    00424CE0         ;  算法 Call
00424F12  |.   85C0            test    eax, eax
00424F14  |.   74 3B           je      short 00424F51
00424F16  |.   8B4424 10       mov     eax, [esp+10]
00424F1A  |.   8A5424 02       mov     dl, [esp+2]                 ;  [esp+2]=00
00424F1E  |.   8A48 02         mov     cl, [eax+2]                 ;  第 3 位注册码入 cl
00424F21  |.   3ACA            cmp     cl, dl
00424F23  |.   75 2C           jnz     short 00424F51
00424F25  |.   8A50 05         mov     dl, [eax+5]                 ;  第 6 位注册码入 cl
00424F28  |.   8A4C24 05       mov     cl, [esp+5]                 ;  [esp+5]=08
00424F2C  |.   3AD1            cmp     dl, cl
00424F2E  |.   75 21           jnz     short 00424F51
00424F30  |.   8A48 06         mov     cl, [eax+6]                 ;  第 7 位注册码入 cl
00424F33  |.   8A5424 06       mov     dl, [esp+6]                 ;  [esp+6]=00
00424F37  |.   3ACA            cmp     cl, dl
00424F39  |.   75 16           jnz     short 00424F51
00424F3B  |.   8A50 08         mov     dl, [eax+8]                 ;  第 9 位注册码入 cl
00424F3E  |.   8A4424 08       mov     al, [esp+8]                 ;  [esp+8]=07
00424F42  |.   3AD0            cmp     dl, al
00424F44  |.   75 0B           jnz     short 00424F51
00424F46  |.   B8 01000000     mov     eax, 1
00424F4B  |.   83C4 0C         add     esp, 0C
00424F4E  |.   C2 0800         retn    8                ;  4 个数字都相等在此处返回
00424F51  |>   33C0            xor     eax, eax
00424F53  |.   83C4 0C         add     esp, 0C
00424F56  \\.   C2 0800         retn    8                ;  不相等此处返回
```

注册码：`**0**80*7*` 第 3、6、7、9 位是固定数 `0`、`8`、`0`、`7` （`*`为 `0-9` 之间任意数）

注册信息保存在：`HKEY_CURRENT_USER\Software\EximiousSoft\EximiousSoft GIF Creator\Default\KeyCode`

呵呵有意思。。贴个注册机

`delphi` 的

```
procedure TForm1.Button1Click(Sender: TObject);

begin
edit1.Text:=inttostr(random(10))+inttostr(random(10))+'0'+inttostr(random(10))+inttostr(random(10))+'80'+inttostr(random(10))+'7'+inttostr(random(10));

end;

end.
```

`4dd466d0e1e0cc123afbe504da2462ca`
