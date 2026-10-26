
# [原创]一个软件的乘除法注册过程---莱鸟破文之四

由于某种原因，这次的分析我不能详加说明，有关软件的信息也从略了，请不要问我。

我个人感觉，这个算法有些意思，就贴上分析结果，大家简单看看行了。![](https://bbs.kanxue.com/view/img/face/1.gif)

这个东东分注册过程和启动验证，都是分段比较的：

后9位和4-6位是在注册事件前已计算,(注册码输入框的按键响应事件);

注册事件中计算成本1-3位(`F(用户名)=注册码前3位`)和1-9位,并验证`F(前9位)=F(后9位)`;

启动时又对4-9位进行了检测;

delphi的东东，入口：

```
004D1B4C u> $  55             push ebp
004D1B4D    .  8BEC           mov ebp,esp
004D1B4F    .  B9 0A000000    mov ecx,0A
004D1B54    >  6A 00          push 0
004D1B56    .  6A 00          push 0
004D1B58    .  49             dec ecx
004D1B59    .^ 75 F9          jnz short XXXXXXXX.004D1B54
```

## 注册过程

### 计算注册码前3位

```
004BCEC4   |.  8B83 18030000  mov eax,[dword ds:ebx+318]
004BCECA   |.  E8 8527F9FF    call XXXXXXXX.0044F654          ;  GetText(TControl)
004BCECF   |.  8B45 F4        mov eax,[dword ss:ebp-C]        ;  取假码
004BCED2   |.  50             push eax
004BCED3   |.  8D55 F0        lea edx,[dword ss:ebp-10]
004BCED6   |.  8B83 14030000  mov eax,[dword ds:ebx+314]      ;  取用户名
004BCEDC   |.  E8 7327F9FF    call XXXXXXXX.0044F654          ;  GetText(TControl)
004BCEE1   |.  8B45 F0        mov eax,[dword ss:ebp-10]       ;  用户名
004BCEE4   |.  5A             pop edx                         ;  00E785F0, 假码
004BCEE5   |.  E8 1AFDFFFF    call XXXXXXXX.004BCC04          ;  运算!进入!
004BCEEA   |.  84C0           test al,al
004BCEEC   |.  0F84 20010000  je XXXXXXXX.004BD012            ;  一定不能跳!
004BCEF2   |.  E8 7108F5FF    call XXXXXXXX.0040D768
```

进入`call`:

```
004BCC53   |.  8B45 FC        mov eax,[dword ss:ebp-4]
004BCC56   |.  E8 7D7FF4FF    call XXXXXXXX.00404BD8          ;  LStrLen(String):Integer;
004BCC5B   |.  8BC8           mov ecx,eax
004BCC5D   |.  83F9 03        cmp ecx,3
004BCC60   |.  0F8C 39010000  jl XXXXXXXX.004BCD9F
004BCC66   |.  83F9 14        cmp ecx,14
004BCC69   |.  0F8F 30010000  jg XXXXXXXX.004BCD9F            ;  用户名在3-20之间
004BCC6F   |.  8B45 FC        mov eax,[dword ss:ebp-4]
004BCC72   |.  BA E0CD4B00    mov edx,XXXXXXXX.004BCDE0       ;  'XXXXXXXX',这好象是个黑名单
004BCC77   |.  E8 A080F4FF    call XXXXXXXX.00404D1C
004BCC7C   |.  0F84 1D010000  je XXXXXXXX.004BCD9F
004BCC82   |.  8B45 F8        mov eax,[dword ss:ebp-8]
004BCC85   |.  E8 4E7FF4FF    call XXXXXXXX.00404BD8
004BCC8A   |.  83F8 12        cmp eax,12                      ;  注册码要18位,$12
004BCC8D   |.  0F85 0C010000  jnz XXXXXXXX.004BCD9F
004BCC93   |.  B9 01000000    mov ecx,1
004BCC98   |>  8B45 F8        /mov eax,[dword ss:ebp-8]       ;  eax=假码
004BCC9B   |.  8A4408 FF      |mov al,[byte ds:eax+ecx-1]     ;  从第一位开始取!
004BCC9F   |.  04 D0          |add al,0D0                     ;  +d0,进位不要
004BCCA1   |.  2C 0A          |sub al,0A                      ;  -a
004BCCA3   |.  72 0A          |jb short XXXXXXXX.004BCCAF     ;  比a小,跳!
004BCCA5   |.  04 D9          |add al,0D9
004BCCA7   |.  2C 1A          |sub al,1A
004BCCA9   |.  0F83 F0000000  |jnb XXXXXXXX.004BCD9F
004BCCAF   |>  41             |inc ecx                        ;  计数器加1
004BCCB0   |.  83F9 13        |cmp ecx,13
004BCCB3   |.^ 75 E3          \jnz short XXXXXXXX.004BCC98    ;  未处理完,继续!
004BCCB5   |.  BE 01000000    mov esi,1                       ;  处理完的结果放在????
004BCCBA   |>  BB 01000000    /mov ebx,1
004BCCBF   |.  8B45 FC        |mov eax,[dword ss:ebp-4]       ;  处理用户名;计算注册码前3位
004BCCC2   |.  E8 117FF4FF    |call XXXXXXXX.00404BD8
004BCCC7   |.  8BF8           |mov edi,eax
004BCCC9   |.  85FF           |test edi,edi
004BCCCB   |.  7E 21          |jle short XXXXXXXX.004BCCEE
004BCCCD   |.  B9 01000000    |mov ecx,1
004BCCD2   |>  8B45 FC        |/mov eax,[dword ss:ebp-4]
004BCCD5   |.  0FB64408 FF    ||movzx eax,[byte ds:eax+ecx-1]
004BCCDA   |.  F7EB           ||imul ebx                      ;  乘法!乘上次余数,第一次是1
004BCCDC   |.  03C6           ||add eax,esi                   ;  +大轮数
004BCCDE   |.  03C1           ||add eax,ecx                   ;  +小轮数
004BCCE0   |.  BB 65010000    ||mov ebx,165
004BCCE5   |.  99             ||cdq                           ;  CDQ  双字扩展
004BCCE6   |.  F7FB           ||idiv ebx                      ;  IDIV 除法,商回送AX,余数回送DX
004BCCE8   |.  8BDA           ||mov ebx,edx                   ;  余数回送
004BCCEA   |.  41             ||inc ecx
004BCCEB   |.  4F             ||dec edi
004BCCEC   |.^ 75 E4          |\jnz short XXXXXXXX.004BCCD2
004BCCEE   |>  8BC3           |mov eax,ebx                    ;  结果
004BCCF0   |.  B9 24000000    |mov ecx,24
004BCCF5   |.  99             |cdq
004BCCF6   |.  F7F9           |idiv ecx                       ;  除24,取余.我的11
004BCCF8   |.  B8 F4CD4B00    |mov eax,XXXXXXXX.004BCDF4      ;  ASCII "0123456789abcdefghijklmnopqrstuvwxyz"
004BCCFD   |.  0FB60410       |movzx eax,[byte ds:eax+edx]    ;  从余数位开始取!
004BCD01   |.  8B55 F8        |mov edx,[dword ss:ebp-8]       ;  注册码!
004BCD04   |.  0FB65432 FF    |movzx edx,[byte ds:edx+esi-1]  ;  注册码从第一位开始取!
004BCD09   |.  2BD0           |sub edx,eax                    ;  注册码-str(余数)
004BCD0B   |.  81C2 AE000000  |add edx,0AE                    ;  +ae
004BCD11   |.  81FA AE000000  |cmp edx,0AE
004BCD17   |.  0F85 82000000  |jnz XXXXXXXX.004BCD9F          ;  不等就跳！一跳就死!
004BCD1D   |.  46             |inc esi
004BCD1E   |.  83FE 04        |cmp esi,4                      ;  处理注册码前3位！
004BCD21   |.^ 75 97          \jnz short XXXXXXXX.004BCCBA
```

### 计算注册码4-6位

```
004BD42A    .  8D45 F4        lea eax,[dword ss:ebp-C]        ;  放结果的变量
004BD42D    .  50             push eax
004BD42E    .  B9 06000000    mov ecx,6
004BD433    .  BA 04000000    mov edx,4
004BD438    .  8B45 FC        mov eax,[dword ss:ebp-4]        ;  假码地址传eax
004BD43B    .  E8 780A0000    call XXXXXXXX.004BDEB8          ;  好象是取eax中的4-6位,结果放在变量中
004BD440    .  8B55 F4        mov edx,[dword ss:ebp-C]
004BD443    .  8D45 FC        lea eax,[dword ss:ebp-4]
004BD446    .  E8 6D75F4FF    call XXXXXXXX.004049B8          ;  LStrLAsg(void;void;void;void)
004BD44B    .  33C0           xor eax,eax
004BD44D    .  55             push ebp
004BD44E    .  68 D5D44B00    push XXXXXXXX.004BD4D5
004BD453    .  64:FF30        push [dword fs:eax]
004BD456    .  64:8920        mov [dword fs:eax],esp
004BD459    .  8D45 F0        lea eax,[dword ss:ebp-10]
004BD45C    .  8B55 FC        mov edx,[dword ss:ebp-4]
004BD45F    .  8A12           mov dl,[byte ds:edx]            ;  取第四位
004BD461    .  E8 9A76F4FF    call XXXXXXXX.00404B00          ;  LStrFromChar(String;String;Char)
004BD466    .  8B45 F0        mov eax,[dword ss:ebp-10]
004BD469    .  E8 E2BFF4FF    call XXXXXXXX.00409450          ;  StrToInt(AnsiString),,?????
004BD46E    .  8BD8           mov ebx,eax
004BD470    .  8D45 EC        lea eax,[dword ss:ebp-14]
004BD473    .  8B55 FC        mov edx,[dword ss:ebp-4]
004BD476    .  8A52 01        mov dl,[byte ds:edx+1]          ;  取第五位
004BD479    .  E8 8276F4FF    call XXXXXXXX.00404B00          ;  LStrFromChar(String;String;Char)
004BD47E    .  8B45 EC        mov eax,[dword ss:ebp-14]
004BD481    .  E8 CABFF4FF    call XXXXXXXX.00409450          ;  StrToInt(AnsiString)
004BD486    .  03C3           add eax,ebx                     ;  第五位+第四位
004BD488    .  F7EB           imul ebx                        ;  *第四位
004BD48A    .  8BD8           mov ebx,eax                     ;  结果暂存于ebx
004BD48C    .  8D45 E8        lea eax,[dword ss:ebp-18]
004BD48F    .  8B55 FC        mov edx,[dword ss:ebp-4]
004BD492    .  8A52 02        mov dl,[byte ds:edx+2]          ;  取第六位
004BD495    .  E8 6676F4FF    call XXXXXXXX.00404B00          ;  LStrFromChar(String;String;Char)
004BD49A    .  8B45 E8        mov eax,[dword ss:ebp-18]
004BD49D    .  E8 AEBFF4FF    call XXXXXXXX.00409450          ;  StrToInt(AnsiString)
004BD4A2    .  03C3           add eax,ebx                     ;  第六位+上面结果
004BD4A4    .  F7EB           imul ebx                        ;  *上面结果
004BD4A6    .  8BD8           mov ebx,eax                     ;  结果暂存于ebx
004BD4A8    .  8D45 E4        lea eax,[dword ss:ebp-1C]
```
