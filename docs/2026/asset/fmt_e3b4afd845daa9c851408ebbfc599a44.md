
# Uninstall Manager 4.30破解+加密字串汉化

- 【破解作者】 CCDebuger
- 【使用工具】 OllyDBG 1.10自己汉化的版本,DeDe 3.50,PeID 0.94
- 【破解平台】 Win9x/NT/2000/XP
- 【软件名称】 Uninstall Manager
- 【下载地址】 `a6dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4S2L8i4y4#2L8Y4y4W2k6$3#2S2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4g2E0i4K6u0r3N6h3#2K6k6i4c8#2M7q4)9J5k6i4A6A6M7l9`.`.
- 【软件简介】 这是一个简单好用的反安装软件, 具有对系统安装检测功能，能记录你硬盘上添加的文件和改变的设置。当你需要时，能彻底地安全地删除你安装的应用程序，使系统恢复安装前的状态。
- 【加壳方式】 无
- 【破解声明】 只是感兴趣，不当之处还请各位大侠指教：）

---

## 破解内容

这个软件是汉化新世纪论坛上的余飞雨兄弟因有部分字串无法汉化，请求协助时我下载下来看的。原来这个软件把注册及未注册的相关英文字串都加了密，这可能也是一种防静态反汇编查关键字串的一种方法吧，呵呵。但这个软件的加密实在不敢恭维，只要找好位置，属可秒杀的那种。当时做的时候记录了一下，整理一下也当是给刚入门的兄弟们一个参考吧。因这篇文章主要是想写给刚入门的朋友看的，我尽量写的详细点，高手就不要看了。不说废话了，下面进入正题：

### 一、软件破解

先用 PeID 检测，无壳，Delphi 程序。用 OllyDBG 载入，F9运行，出来一个对话框，对话框的下面显示你的软件试用期还有多少天（这里就是加密过的字串）。点对话框上面的 Register 按钮，出来一个可以输姓名和注册码的地方，姓名输入CCDebuger，注册码输入78787878（积习难改，当年都是受了八爪鱼的毒害啊，呵呵），在OllyDBG命令行中下`bp GetDlgItemTextA`，点被调试程序的OK按钮，跳出一个错误框。没拦住！没关系，我们再来。取消刚才的断点，在OllyDBG中CTR+N，右击`GetWindowTextA`函数，选择在每个参考上设置断点，再在被调试的程序中输入姓名和试炼码，点OK按钮，又没拦住。看看错误对话框上有这么一行字：“Sorry not a correct reg key”，在OllyDBG中查找ASCII字串参考，没有。那我们再用ApiBreak插件设个万能断点吧，很遗憾，还是不行。到此我就不再试了，Delphi程序当然应该用DeDe来对付它一下。用DeDe来反编译一下（关于用DeDe来寻找关键点的方法可以看一下我上传的那个演示动画），很快我们就找到了点OK按钮后的相关处理代码：

```
0049CA90   55                     push    ebp
0049CA91   8BEC                   mov     ebp, esp
0049CA93   B907000000             mov     ecx, $00000007
0049CA98   6A00                   push    $00
0049CA9A   6A00                   push    $00
0049CA9C   49                     dec     ecx
0049CA9D   75F9                   jnz     0049CA98
0049CA9F   51                     push    ecx
```

现在我们在OllyDBG中按CTR+G，输入地址`0049CA90`按确定，在地址`0049CA90`这个位置设个断点，再次输入姓名和试炼码，点OK按钮，被断下：

```
0049CA90   .  55            PUSH EBP
0049CA91   .  8BEC          MOV EBP,ESP
0049CA93   .  B9 07000000   MOV ECX,7
0049CA98   >  6A 00         PUSH 0
0049CA9A   .  6A 00         PUSH 0
0049CA9C   .  49            DEC ECX
0049CA9D   .^ 75 F9         JNZ SHORT Uninsman.0049CA98
```

现在我们开始分析：

```
0049CACC   .  E8 3F0BFDFF   CALL Uninsman.0046D610
0049CAD1   .  837D F4 00    CMP DWORD PTR SS:[EBP-C],0               ;  检查是否输入了用户名
0049CAD5   .  74 17         JE SHORT Uninsman.0049CAEE
0049CAD7   .  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
0049CADA   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0049CADD   .  8B80 14030000 MOV EAX,DWORD PTR DS:[EAX+314]
0049CAE3   .  E8 280BFDFF   CALL Uninsman.0046D610
0049CAE8   .  837D F0 00    CMP DWORD PTR SS:[EBP-10],0              ;  检查是否输入了注册码
......
0049CAD1   .  837D F4 00    CMP DWORD PTR SS:[EBP-C],0               ;  检查是否输入了用户名
0049CAD5   .  74 17         JE SHORT Uninsman.0049CAEE
0049CAD7   .  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
0049CADA   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0049CADD   .  8B80 14030000 MOV EAX,DWORD PTR DS:[EAX+314]
0049CAE3   .  E8 280BFDFF   CALL Uninsman.0046D610
0049CAE8   .  837D F0 00    CMP DWORD PTR SS:[EBP-10],0              ;  检查是否输入了注册码
0049CAEC   .  75 41         JNZ SHORT Uninsman.0049CB2F
0049CAEE   >  6A 00         PUSH 0
0049CAF0   .  66:8B0D 50CE4>MOV CX,WORD PTR DS:[49CE50]
0049CAF7   .  B2 02         MOV DL,2
0049CAF9   .  B8 5CCE4900   MOV EAX,Uninsman.0049CE5C
0049CAFE   .  E8 A978F9FF   CALL Uninsman.004343AC
0049CB03   .  48            DEC EAX
0049CB04   .  75 17         JNZ SHORT Uninsman.0049CB1D
0049CB06   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0049CB09   .  8B80 00030000 MOV EAX,DWORD PTR DS:[EAX+300]
0049CB0F   .  66:BE EBFF    MOV SI,0FFEB
0049CB13   .  E8 BC74F6FF   CALL Uninsman.00403FD4
0049CB18   .  E9 E7020000   JMP Uninsman.0049CE04
0049CB1D   >  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0049CB20   .  C780 4C020000>MOV DWORD PTR DS:[EAX+24C],2
0049CB2A   .  E9 D5020000   JMP Uninsman.0049CE04
0049CB2F   >  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0049CB32   .  E8 C9F8FFFF   CALL Uninsman.0049C400                   ;  关键CALL，跟进去
0049CB37   .  3C 01         CMP AL,1                                 ;  注册标准位
0049CB39   .  0F85 74020000 JNZ Uninsman.0049CDB3                    ;  不等则完蛋
```

走到`0049CB32`那个CALL时我们按F7跟进去，停在`0049C400`处：

```
0049C400  /$  55            PUSH EBP
0049C401  |.  8BEC          MOV EBP,ESP
0049C403  |.  33C9          XOR ECX,ECX
```

#### 用户名计算部分：

```
0049C42C  |.  837D F4 00    CMP DWORD PTR SS:[EBP-C],0               ;  检查是否输入了用户名
0049C430  |.  74 7B         JE SHORT Uninsman.0049C4AD
0049C432  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
0049C435  |.  8B86 10030000 MOV EAX,DWORD PTR DS:[ESI+310]
0049C43B  |.  E8 D011FDFF   CALL Uninsman.0046D610
0049C440  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0049C443  |.  E8 E089F6FF   CALL Uninsman.00404E28
0049C448  |.  8BF8          MOV EDI,EAX                              ;  用户名长度
0049C44A  |.  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
0049C44D  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]             ;  用户名
0049C450  |.  E8 03CAF6FF   CALL Uninsman.00408E58                   ;  这个CALL的作用是把用户名中的大写字母转换为小写
0049C455  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]
0049C458  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
0049C45B  |.  E8 A087F6FF   CALL Uninsman.00404C00
0049C460  |.  33DB          XOR EBX,EBX
0049C462  |.  85FF          TEST EDI,EDI
0049C464  |.  7E 1D         JLE SHORT Uninsman.0049C483
0049C466  |.  B8 01000000   MOV EAX,1
0049C46B  |>  8B55 FC       /MOV EDX,DWORD PTR SS:[EBP-4]            ;  取用户名
0049C46E  |.  8A5402 FF     |MOV DL,BYTE PTR DS:[EDX+EAX-1]          ;  取用户名的每位ASCII值
0049C472  |.  80FA 20       |CMP DL,20
0049C475  |.  74 08         |JE SHORT Uninsman.0049C47F
0049C477  |.  81E2 FF000000 |AND EDX,0FF                             ;  保留后两位（低8位）。如EDX原来是10041E64，执行这条指令后就是00000064。
0049C47D  |.  03DA          |ADD EBX,EDX                             ;  把用户名的每位ASCII值相加后放在EBX
0049C47F  |>  40            |INC EAX
0049C480  |.  4F            |DEC EDI
0049C481  |.^ 75 E8         \JNZ SHORT Uninsman.0049C46B             ;  判断是否计算完
0049C483  |>  81F3 89000000 XOR EBX,89                               ;  用户名计算后值与89H异或
0049C489  |.  83F3 33       XOR EBX,33                               ;  再与33H异或
0049C48C  |.  43            INC EBX                                  ;  加1
0049C48D  |.  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
0049C490  |.  8B86 14030000 MOV EAX,DWORD PTR DS:[ESI+314]
0049C496  |.  E8 7511FDFF   CALL Uninsman.0046D610
0049C49B  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]             ;  我们输入的注册码，跟进下面的CALL
0049C49E  |.  E8 6DCFF6FF   CALL Uninsman.00409410                   ;  注册码计算部分
0049C4A3  |.  3BD8          CMP EBX,EAX                              ;  用户名与注册码计算后的值比较
0049C4A5  |.  75 04         JNZ SHORT Uninsman.0049C4AB              ;  不等则完蛋
```

在`0049C49E`地址处按F7跟进那个CALL，来到这里：

```
00409410  /$  53            PUSH EBX
00409411  |.  56            PUSH ESI
00409412  |.  83C4 F4       ADD ESP,-0C
00409415  |.  8BD8          MOV EBX,EAX
00409417  |.  8BD4          MOV EDX,ESP
00409419  |.  8BC3          MOV EAX,EBX                              ;  我们输的注册码
0040941B  |.  E8 84A2FFFF   CALL Uninsman.004036A4                   ;  跟进去
```

进入`0040941B`的CALL，来到注册码计算部分：

```
004036A4  /$  53            PUSH EBX                                 ;  来到这里
004036A5  |.  56            PUSH ESI
004036A6  |.  57            PUSH EDI
004036A7  |.  89C6          MOV ESI,EAX
004036A9  |.  50            PUSH EAX
004036AA  |.  85C0          TEST EAX,EAX
004036AC  |.  74 6C         JE SHORT Uninsman.0040371A
004036AE  |.  31C0          XOR EAX,EAX
004036B0  |.  31DB          XOR EBX,EBX
004036B2  |.  BF CCCCCC0C   MOV EDI,Uninsman.0CCCCCCC
004036B7  |>  8A1E          /MOV BL,BYTE PTR DS:[ESI]                ;  依次取注册码的每个值
004036B9  |.  46            |INC ESI
004036BA  |.  80FB 20       |CMP BL,20                               ;  判断是否是空格
004036BD  |.^ 74 F8         \JE SHORT Uninsman.004036B7
004036BF  |.  B5 00         MOV CH,0
```
