
# Roaring Falls Screensaver

> 原文：[`www.pediy.com/kssd/pediy05/pediy50646.htm`](https://www.pediy.com/kssd/pediy05/pediy50646.htm)

简单算法——`Roaring Falls Screensaver`  

下载页面：`http://www.pjsoft.com/`  

软件大小：1.28M

**【软件简介】**：瀑布屏保。

**【软件限制】**：30 Day Trial

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm` 9.0 白金版

---

**【过程】**：

呵呵，不知道这个东东是否还可以下载。晕，在我 60M 的 `Geforce` 显卡上居然无法流畅运行。

`Roaring Falls.exe` 无壳。`Borland Delphi` 编写。

`User Name`：`fly`  

试炼码：`13572468`  

---

```
:00460ECD E87EE0FFFF           call 0045EF50  
:00460ED2 8BB3F0020000         mov esi, dword ptr [ebx+000002F0]  
:00460ED8 837E3800             cmp dword ptr [esi+38], 00000000  
                               ==== > 没填名字？
:00460EDC 0F8454010000         je 00461036  
                               ==== > 跳则 OVER！
:00460EE2 837E3C00             cmp dword ptr [esi+3C], 00000000  
                               ==== > 没有注册码？
:00460EE6 0F844A010000         je 00461036  
                               ==== > 跳则 OVER！
:00460EEC 6A01                 push 00000001  
:00460EEE 8D45FC               lea eax, dword ptr [ebp-04]  
:00460EF1 50                   push eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00460E8B(C)  
|
* Possible StringData Ref from Code Obj -> "235"  
                               |  
:00460EF2 B860104600           mov eax, 00461060  
                               ==== > EAX=235
:00460EF7 E84C6CFAFF           call 00407B48  
                               ==== > 取 235 的 16 进制值
:00460EFC 8BC8                 mov ecx, eax  
                               ==== > ECX=EAX=EB(H)=235(D)  运算参数
:00460EFE 8B83F0020000         mov eax, dword ptr [ebx+000002F0]  
:00460F04 8B4038               mov eax, dword ptr [eax+38]  
                               ==== > EAX=fly
* Possible StringData Ref from Code Obj -> "RfaLLs1"  
                               |  
:00460F07 BA6C104600           mov edx, 0046106C  
                               ==== > EDX=RfaLLs1           运算参数
:00460F0C E80BE5FFFF           call 0045F41C  
                               ==== > 算法 CALL！进入！
:00460F11 8B55FC               mov edx, dword ptr [ebp-04]  
                               ==== > EDX=EB000AE2          注册码
:00460F14 A1B4394600           mov eax, dword ptr [004639B4]  
:00460F19 E82E2AFAFF           call 0040394C  
:00460F1E A1B4394600           mov eax, dword ptr [004639B4]  
:00460F23 8B00                 mov eax, dword ptr [eax]  
:00460F25 8B93F0020000         mov edx, dword ptr [ebx+000002F0]  
:00460F2B 8B523C               mov edx, dword ptr [edx+3C]  
                               ==== > EDX=13572468          试炼码
:00460F2E E8512DFAFF           call 00403C84  
                               ==== > 比较 CALL！  
:00460F33 743C                 je 00460F71  
:00460F35 8B83F0020000         mov eax, dword ptr [ebx+000002F0]  
:00460F3B FF703C               push [eax+3C]
* Possible StringData Ref from Code Obj -> " is not the correct password!"  
                               ==== > BAD BOY！  
:00460F3E 687C104600           push 0046107C  
:00460F43 68A4104600           push 004610A4
* Possible StringData Ref from Code Obj -> " Please contact the author at:"  
                               |  
:00460F48 68B0104600           push 004610B0  
:00460F4D 68A4104600           push 004610A4
* Possible StringData Ref from Code Obj -> "pgj@ix.netcom.com"  
                               |  
:00460F52 68D8104600           push 004610D8  
:00460F57 8D45FC               lea eax, dword ptr [ebp-04]  
:00460F5A BA06000000           mov edx, 00000006  
:00460F5F E8D02CFAFF           call 00403C34  
:00460F64 8B45FC               mov eax, dword ptr [ebp-04]  
:00460F67 E8CC55FEFF           call 00446538  
:00460F6C E9C5000000           jmp 00461036
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00460F33(C)  
|  
:00460F71 A1303A4600           mov eax, dword ptr [00463A30]  
:00460F76 C60001               mov byte ptr [eax], 01  
:00460F79 33D2                 xor edx, edx  
:00460F7B 8B83E4020000         mov eax, dword ptr [ebx+000002E4]  
:00460F81 E8CE53FCFF           call 00426354
* Possible StringData Ref from Code Obj -> "Roaring Waterfalls - Registered"  
                               |  
:00460F86 BAF4104600           mov edx, 004610F4  
:00460F8B 8BC3                 mov eax, ebx  
:00460F8D E8DA54FCFF           call 0042646C  
:00460F92 B201                 mov dl, 01  
:00460F94 A158A04500           mov eax, dword ptr [0045A058]  
:00460F99 E8FA91FFFF           call 0045A198  
:00460F9E 8BF0                 mov esi, eax  
:00460FA0 B101                 mov cl, 01
                               ==== > 下面是保存注册信息  
* Possible StringData Ref from Code Obj -> "##@)\Roar Falls"  
                               |  
:00460FA2 BA1C114600           mov edx, 0046111C  
:00460FA7 8BC6                 mov eax, esi  
:00460FA9 E8DA93FFFF           call 0045A388  
:00460FAE 6A01                 push 00000001  
:00460FB0 8D45FC               lea eax, dword ptr [ebp-04]  
:00460FB3 50                   push eax
* Possible StringData Ref from Code Obj -> "126"  
                               |  
:00460FB4 B838114600           mov eax, 00461138  
:00460FB9 E88A6BFAFF           call 00407B48  
:00460FBE 8BC8                 mov ecx, eax  
:00460FC0 8B83F0020000         mov eax, dword ptr [ebx+000002F0]  
:00460FC6 8B4038               mov eax, dword ptr [eax+38]
* Possible StringData Ref from Code Obj -> "RfaLLs1UsEr"  
                               |  
:00460FC9 BA44114600           mov edx, 00461144  
:00460FCE E849E4FFFF           call 0045F41C  
:00460FD3 8B55FC               mov edx, dword ptr [ebp-04]  
:00460FD6 A104394600           mov eax, dword ptr [00463904]  
:00460FDB E86C29FAFF           call 0040394C  
:00460FE0 8B0D04394600         mov ecx, dword ptr [00463904]  
:00460FE6 8B09                 mov ecx, dword ptr [ecx]
* Possible StringData Ref from Code Obj -> "User Name"  
                               |  
:00460FE8 BA58114600           mov edx, 00461158  
:00460FED 8BC6                 mov eax, esi  
:00460FEF E83095FFFF           call 0045A524  
:00460FF4 8B83F0020000         mov eax, dword ptr [ebx+000002F0]  
:00460FFA 8B483C               mov ecx, dword ptr [eax+3C]
* Possible StringData Ref from Code Obj -> "Registration"  
                               |  
:00460FFD BA6C114600           mov edx, 0046116C  
:00461002 8BC6                 mov eax, esi  
:00461004 E81B95FFFF           call 0045A524  
:00461009 8B83F0020000         mov eax, dword ptr [ebx+000002F0]  
:0046100F FF7038               push [eax+38]  
:00461012 6884114600           push 00461184  
:00461017 68A4104600           push 004610A4
* Possible StringData Ref from Code Obj -> "Thank you for registering and "  
                               -> "supporting shareware!"  
                               ==== > 呵呵，胜利女神！
:0046101C 6890114600           push 00461190  
:00461021 8D45FC               lea eax, dword ptr [ebp-04]  
:00461024 BA04000000           mov edx, 00000004  
:00461029 E8062CFAFF           call 00403C34  
:0046102E 8B45FC               mov eax, dword ptr [ebp-04]  
:00461031 E80255FEFF           call 00446538
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00460EDC(C), :00460EE6(C), :00460F6C(U)  
|  
:00461036 33C0                 xor eax, eax  
```



### 00461038-0046104B

```
:00461038 5A                   pop edx
:00461039 59                   pop ecx
:0046103A 59                   pop ecx
:0046103B 648910               mov dword ptr fs:[eax], edx
:0046103E 6853104600           push 00461053

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00461051(U)
|

:00461043 8D45FC               lea eax, dword ptr [ebp-04]
:00461046 E8AD28FAFF           call 004038F8
:0046104B C3                   ret
```

---

进入算法 `CALL`：`460F0C` `call 0045F41C`

### 0045F41C 入口函数

```
* Referenced by a CALL at Addresses:
|:0045F6EF, :0045F72E, :0045F7BD, :0045F80E, :0045F87B
|:0045F8E6, :00460F0C, :00460FCE
|

:0045F41C 55                   push ebp
:0045F41D 8BEC                 mov ebp, esp
:0045F41F 83C4D4               add esp, FFFFFFD4
:0045F422 53                   push ebx
:0045F423 56                   push esi
:0045F424 57                   push edi
:0045F425 33DB                 xor ebx, ebx
:0045F427 895DD8               mov dword ptr [ebp-28], ebx
:0045F42A 895DD4               mov dword ptr [ebp-2C], ebx
:0045F42D 895DF0               mov dword ptr [ebp-10], ebx
:0045F430 8BD9                 mov ebx, ecx
:0045F432 8955F8               mov dword ptr [ebp-08], edx
:0045F435 8945FC               mov dword ptr [ebp-04], eax
:0045F438 8B45FC               mov eax, dword ptr [ebp-04]
:0045F43B E8E848FAFF           call 00403D28
:0045F440 8B45F8               mov eax, dword ptr [ebp-08]
:0045F443 E8E048FAFF           call 00403D28
:0045F448 33C0                 xor eax, eax
:0045F44A 55                   push ebp
:0045F44B 6816F64500           push 0045F616
:0045F450 64FF30               push dword ptr fs:[eax]
:0045F453 648920               mov dword ptr fs:[eax], esp
:0045F456 837DF400             cmp dword ptr [ebp-0C], 00000000
:0045F45A 750D                 jne 0045F469
:0045F45C 8D45F8               lea eax, dword ptr [ebp-08]

* Possible StringData Ref from Code Obj -> "umbra"
|

:0045F45F BA30F64500           mov edx, 0045F630
:0045F464 E82745FAFF           call 00403990
```

### 循环校验逻辑

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045F45A(C)
|

:0045F469 8B45F8               mov eax, dword ptr [ebp-08]
                               =====> EAX=RfaLLs1
:0045F46C E80347FAFF           call 00403B74
                               =====> 取 RfaLLs1 的位数
:0045F471 8945F4               mov dword ptr [ebp-0C], eax
                               =====> [ebp-0C]=7
:0045F474 33F6                 xor esi, esi
:0045F476 807D0C00             cmp byte ptr [ebp+0C], 00
:0045F47A 0F8498000000         je 0045F518
:0045F480 8BFB                 mov edi, ebx
:0045F482 8D45F0               lea eax, dword ptr [ebp-10]
:0045F485 50                   push eax
:0045F486 897DDC               mov dword ptr [ebp-24], edi
:0045F489 C645E000             mov [ebp-20], 00
:0045F48D 8D55DC               lea edx, dword ptr [ebp-24]
:0045F490 33C9                 xor ecx, ecx

* Possible StringData Ref from Code Obj -> "%1.2x"
|

:0045F492 B840F64500           mov eax, 0045F640
:0045F497 E84891FAFF           call 004085E4
:0045F49C 8B45FC               mov eax, dword ptr [ebp-04]
                               =====> EAX=fly
:0045F49F E8D046FAFF           call 00403B74
                               =====> 取 fly 的位数
:0045F4A4 85C0                 test eax, eax
                               =====> EAX=3
:0045F4A6 0F8E2F010000         jle 0045F5DB
:0045F4AC 8945E4               mov dword ptr [ebp-1C], eax
:0045F4AF C745EC01000000       mov [ebp-14], 00000001
```

### 主循环体

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045F511(C)
|

:0045F4B6 8B45FC               mov eax, dword ptr [ebp-04]
                               =====> EAX=fly
:0045F4B9 8B55EC               mov edx, dword ptr [ebp-14]
:0045F4BC 0FB64410FF           movzx eax, byte ptr [eax+edx-01]
                               =====> 依次取 fly 字符的 HEX 值
                               1、 =====> EAX=66
                               2、 =====> EAX=6C
                               3、 =====> EAX=79
:0045F4C1 03C7                 add eax, edi
                               1、 =====> EAX=66 + EB=151
                               2、 =====> EAX=6C + 00=6C
                               3、 =====> EAX=79 + 0A=83
:0045F4C3 B9FF000000           mov ecx, 000000FF
                               =====> ECX=000000FF
:0045F4C8 99                   cdq
:0045F4C9 F7F9                 idiv ecx
                               1、 =====> EDX=151 % FF=52
                               2、 =====> EDX=6C  % FF=6C
                               3、 =====> EDX=83  % FF=83
:0045F4CB 8BDA                 mov ebx, edx
                               =====> EBX=EDX
:0045F4CD 3B75F4               cmp esi, dword ptr [ebp-0C]
:0045F4D0 7D03                 jge 0045F4D5
:0045F4D2 46                   inc esi
:0045F4D3 EB05                 jmp 0045F4DA
```

### 条件分支与字符串处理

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045F4D0(C)
|

:0045F4D5 BE01000000           mov esi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045F4D3(U)
|

:0045F4DA 8B45F8               mov eax, dword ptr [ebp-08]
                               =====> EAX=RfaLLs1
:0045F4DD 0FB64430FF           movzx eax, byte ptr [eax+esi-01]
                               =====> 依次取 RfaLLs1 字符的 HEX 值 共取与用户名相同位数
                               1、 =====> EAX=52
                               2、 =====> EAX=66
                               3、 =====> EAX=61
:0045F4E2 33D8                 xor ebx, eax
                               1、 =====> EBX=52 XOR 52=00
                               2、 =====> EBX=6C XOR 66=0A
                               3、 =====> EBX=61 XOR 83=E2
:0045F4E4 8D45D8               lea eax, dword ptr [ebp-28]
:0045F4E7 50                   push eax
:0045F4E8 895DDC               mov dword ptr [ebp-24], ebx
:0045F4EB C645E000             mov [ebp-20], 00
:0045F4EF 8D55DC               lea edx, dword ptr [ebp-24]
:0045F4F2 33C9                 xor ecx, ecx

* Possible StringData Ref from Code Obj -> "%1.2x"
|

:0045F4F4 B840F64500           mov eax, 0045F640
:0045F4F9 E8E690FAFF           call 004085E4
                               =====> 将以上所得直接转成字符
:0045F4FE 8B55D8               mov edx, dword ptr [ebp-28]
                               1、 =====> EDX=00
                               2、 =====> EDX=0A
                               3、 =====> EDX=E2
:0045F501 8D45F0               lea eax, dword ptr [ebp-10]
:0045F504 E87346FAFF           call 00403B7C
                               =====> 将以上所得字符依次连接在 EB 后面
:0045F509 8BFB                 mov edi, ebx
:0045F50B FF45EC               inc [ebp-14]
:0045F50E FF4DE4               dec [ebp-1C]
:0045F511 75A3                 jne 0045F4B6
                               =====> 循环用户名位数次
:0045F513 E9C3000000           jmp 0045F5DB
                               ...... ...... 省 略 ...... ......
```

### 结尾与退出

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0045F4A6(C), :0045F513(U)
|

:0045F5DB 8B4508               mov eax, dword ptr [ebp+08]
:0045F5DE 8B55F0               mov edx, dword ptr [ebp-10]
```



`====>EDX=EB000AE2` 这就是注册码了！

```
:0045F5E1 E8AA43FAFF         call 00403990
:0045F5E6 33C0               xor eax, eax
:0045F5E8 5A                 pop edx
:0045F5E9 59                 pop ecx
:0045F5EA 59                 pop ecx
:0045F5EB 648910             mov dword ptr fs:[eax], edx
:0045F5EE 681DF64500         push 0045F61D
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
* |:0045F61B(U)
* |

```
:0045F5F3 8D45D4             lea eax, dword ptr [ebp-2C]
:0045F5F6 BA02000000         mov edx, 00000002
:0045F5FB E81C43FAFF         call 0040391C
:0045F600 8D45F0             lea eax, dword ptr [ebp-10]
:0045F603 E8F042FAFF         call 004038F8
:0045F608 8D45F8             lea eax, dword ptr [ebp-08]
:0045F60B BA02000000         mov edx, 00000002
:0045F610 E80743FAFF         call 0040391C
:0045F615 C3                 ret
```

—————————————————————————————————

### 注册信息保存

REGEDIT4

`[HKEY_CURRENT_USER\##@)(\Roar Falls]`

`"User Name"="7EB645DF"`
`"Registered"=dword:00000000`
`"Registration"="EB000AE2"`

—————————————————————————————————

### 整 理

`User Name`：`fly`
`Password`：`EB000AE2`

—————————————————————————————————

```
        ,     \_/    
   /| \_.-~/         \\_\     ,      
 ( /~   /             \\~-.\_ |\
 \`\\ \_/              \   ~\ )    忍把浮名
 _-~~~-.)  )\_\_/;;,.     \\_  //'
  '/',\  --~  \ ~~~-  ,;;\_\_\_(  (.-~~~-.    换了破解轻狂
 \`~ \_( ,\_..--\ (     ,;'' /   ~--   /.\_\`\\
 /~~//'  /' \`~\          ) /--..\_, )\_  \`~
 "  \`~"  "      `"      /~'\`\\   \`\\~~\
 
        "     "   "~'  ""
```

Cracked By 巢水工作坊——fly `[OCN][FCG]`
2003-04-25 1:44 `4dd466d0e1e0cc123afbe504da2462ca`
