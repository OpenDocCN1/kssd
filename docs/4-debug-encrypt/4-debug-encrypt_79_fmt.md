
# CyberArticle V4.361

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-246.htm`](https://www.pediy.com/kssd/pediy08/pediy8-246.htm)

**【文章标题】**: `CyberArticle V4.361`

**【文章作者】**: `rdsnow[BCG][PYG][D.4s]`

**【作者邮箱】**: `rdsnow@163.com`

**【作者主页】**: `http://rdsnow.ys168.com`

**【作者 QQ 号】**: `83757177`

**【下载地址】**: 主页：`http://www.wizissoft.com`

**【使用工具】**: `OllyICE`

**【软件介绍】**: `CyberArticle V4.361`

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

**【文章简介】**

这个程序有两种注册方式，但是均比较简单：

1. 在程序目录下建立 `"oem.ini"` 文件，输入以下内容则可变成 `OEM` 版

   ```ini
   [Common]
   Name=成都铁路局基层工会资料管理系统
   ```

   这也算个最简单的 keyfile 吧！

2. 使用简单算法得到符合条件的注册码注册成个人版：

   注册码的形式为：`XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`

   下面的注释中给各段取名：`szRegCode1`-`szRegCode2`-`szRegCode3`-`szRegCode4`-`szRegCode5`

   (其实中间不一定用 `"-"`，任何字符都可以通过，只是验证时没有用到这几位，所以用 `"-"` 替代了)

   算法中使用到了修改了加法常数的 `MD5` 运算。

---

**【破解过程】**

程序在输入注册码后会关闭程序，在重启后验证注册码，输入假码：

```
98765-56789-54321-12345-ABCDE
```

监视程序保存注册码，发现假码保存在文件 `C:\Documents and Settings\Administrator\Application Data\CyberArticle\CyberArticle.ini` 中：

内容如下：

```ini
[UserInfo]
SerialNo=BFIXU-LMFJV-NUENK-GTZYA-PMRSW
Name=rdsnow[BCG][PYG][D.4s]
```

典型的 `ini` 的格式。可以通过字符串 `"UserInfo"` 查找关键点下断：

断到这里：

```assembly
004F366C  |.  51             push ecx
004F366D  |.  BA 1E586600    mov edx,Cyber.0066581E          ;  ASCII "UserInfo"
004F3672  |.  8D45 FC        lea eax,dword ptr ss:[ebp-4]
004F3675  |.  E8 8A611000    call Cyber.005F9804
```

`F8` 单步跟代码，`return` 后来到：

```assembly
0040761A  |.  FF45 EC        inc dword ptr ss:[ebp-14]
0040761D  |.  E8 FEBF0E00    call Cyber.WaGetSN               ;  读取注册码
00407622  |.  8D55 F8        lea edx,dword ptr ss:[ebp-8]
00407625  |.  52             push edx
00407626  |.  BA 350D6000    mov edx,Cyber.00600D35           ;  ASCII ""
0040762B  |.  8D45 FC        lea eax,dword ptr ss:[ebp-4]
0040762E  |.  E8 D1211F00    call Cyber.005F9804
00407633  |.  FF45 EC        inc dword ptr ss:[ebp-14]
00407636  |.  5A             pop edx
00407637  |.  59             pop ecx
00407638  |.  E8 2F241F00    call Cyber.005F9A6C
0040763D  |.  8D45 F4        lea eax,dword ptr ss:[ebp-C]
00407640  |.  8B00           mov eax,dword ptr ds:[eax]
00407642  |.  E8 ADC00E00    call Cyber.WaDownloadImageFile    ;  关键 Call
00407647  |.  50             push eax                           ;  返回值压栈
………………
（中间有许多 Call，用于销毁局部变量，所以就省了）
………………
00407678  |.  59             pop ecx                            ;  弹出返回值
00407679  |.  84C9           test cl,cl
0040767B  |.  74 11          je short Cyber.0040768E            ;  若返回 1 则注册成功
0040767D  |.  33C0           xor eax,eax
0040767F  |.  8B55 D0        mov edx,dword ptr ss:[ebp-30]
00407682  |.  64:8915 00000> mov dword ptr fs:[0],edx
00407689  |.  E9 C4000000    jmp Cyber.00407752
```

程序有意将关键 `Call` 和关键 `JUMP` 拉远距离，不多说了，赶快进去看看吧：

```assembly
004F3753  |.  50             push eax
004F3754  |.  FF45 8C        inc dword ptr ss:[ebp-74]
004F3757  |.  BA 30586600    mov edx,Cyber.00665830           ;  ASCII "oem.ini"
004F375C  |.  8D45 F0        lea eax,dword ptr ss:[ebp-10]
004F375F  |.  E8 A0601000    call Cyber.005F9804
004F3764  |.  FF45 8C        inc dword ptr ss:[ebp-74]
004F3767  |.  8D55 F0        lea edx,dword ptr ss:[ebp-10]
004F376A  |.  52             push edx
004F376B  |.  8D45 F4        lea eax,dword ptr ss:[ebp-C]
004F376E  |.  E8 ED3CF1FF    call Cyber.00407460
004F3773  |.  FF45 8C        inc dword ptr ss:[ebp-74]
004F3776  |.  E8 F9C5FFFF    call Cyber.WaGetAppPath          ;  获取程序路径
004F377B  |.  8D45 F4        lea eax,dword ptr ss:[ebp-C]
004F377E  |.  5A             pop edx
004F377F  |.  59             pop ecx
004F3780  |.  E8 E7621000    call Cyber.005F9A6C              ;  连接程序路径和"oem.ini"
004F3785  |.  8D45 EC        lea eax,dword ptr ss:[ebp-14]
004F3788  |.  50             push eax
004F3789  |.  BA 3F586600    mov edx,Cyber.0066583F          ;  ASCII "Name"
004F378E  |.  8D45 E4        lea eax,dword ptr ss:[ebp-1C]
004F3791  |.  E8 6E601000    call Cyber.005F9804
004F3796  |.  FF45 8C        inc dword ptr ss:[ebp-74]
004F3799  |.  8D55 E4        lea edx,dword ptr ss:[ebp-1C]
004F379C  |.  52             push edx
004F379D  |.  BA 38586600    mov edx,Cyber.00665838          ;  ASCII "Common"
004F37A2  |.  8D45 E8        lea eax,dword ptr ss:[ebp-18]
004F37A5  |.  E8 5A601000    call Cyber.005F9804
004F37AA  |.  FF45 8C        inc dword ptr ss:[ebp-74]        ; |
004F37AD  |.  8D55 E8        lea edx,dword ptr ss:[ebp-18]    ; |
004F37B0  |.  59             pop ecx                           ; |
004F37B1  |.  58             pop eax                           ; |
004F37B2  |.  E8 A9A4FBFF    call Cyber.WizReadStrFromConfig   ;  读取 oem.ini 内的信息
004F37B7  |.  8D45 DC        lea eax,dword ptr ss:[ebp-24]
004F37BA  |.  50             push eax
004F37BB  |.  BA 45586600    mov edx,Cyber.00665845
004F37C0  |.  8D45 D8        lea eax,dword ptr ss:[ebp-28]
004F37C3  |.  E8 3C601000    call Cyber.005F9804
004F37C8  |.  FF45 8C        inc dword ptr ss:[ebp-74]
004F37CB  |.  8D55 D8        lea edx,dword ptr ss:[ebp-28]
004F37CE  |.  58             pop eax
004F37CF  |.  E8 24631000    call Cyber.005F9AF8              ;  判断是不是 OEM 版
```

看下程序所在目录根本没有 `oem.ini` 文件，伪造一个吧，`ini` 格式的文件是最容易伪造的了，根据上面代码文件内容应是：

```ini
[Common]
Name=XXXXXXXXXXXXXXXXXXXXXXX
```

`oem.ini` 造好后，跟进判断是不是 `OEM` 版的 `Call`：

```assembly
005F9AF8  /$  55             push ebp
005F9AF9  |.  8BEC           mov ebp,esp
005F9AFB  |.  53             push ebx
005F9AF8  /$  55             push ebp
005F9AF9  |.  8BEC           mov ebp,esp
005F9AFB  |.  53             push ebx
005F9AFC  |.  8B00           mov eax,dword ptr ds:[eax]       ;  ASCII "XXXXXXXXXXXXXXXXXXXXXXX"
005F9AFE  |.  8B12           mov edx,dword ptr ds:[edx]       ;  ASCII "成都铁路局基层工会资料管理系统"
005F9B00  |.  E8 3F71FCFF    call Cyber.005C0C44              ;  这个当然是比较字符串了
005F9B05  |.  0F94C0         sete al
005F9B08  |.  83E0 01        and eax,1
005F9B0B  |.  5B             pop ebx
005F9B0C  |.  5D             pop ebp
005F9B0D  \\.  C3             retn
```

赶快将 `oem.ini` 文件中的 `XXX` 换成 `"成都…………"`，重启程序，果然变成 `"成都铁路局基层工会(100 套授权)"`

不要满足，程序并没有处理我们输入的注册码，关键 `Call` 直接返回 `1` 了，删掉 `oem.ini`，继续跟：

下面同样有很多 `Call` 用于销毁局部变量，走了好远，来到：

```assembly
004F38B6  |.  8D45 FC        lea eax,dword ptr ss:[ebp-4]
004F38B9  |.  E8 1E4EF1FF    call Cyber.004086DC               ;  取得注册码的长度
004F38BE  |.  83F8 1D        cmp eax,1D                         ;  if(strlen(szRegCode)>29) bRegLog=1
004F38C1  |.  0F9FC2         setg dl
004F38C4  |.  83E2 01        and edx,1
004F38C7  |.  8895 6FFFFFFF  mov byte ptr ss:[ebp-91],dl       ;  保存 bRegLog
004F38CD  |.  66:C745 80 08> mov word ptr ss:[ebp-80],8
004F38D3  |.  66:C745 80 2C> mov word ptr ss:[ebp-80],2C
004F38D9  |.  8D55 FC        lea edx,dword ptr ss:[ebp-4]
004F38DC  |.  8D45 F8        lea eax,dword ptr ss:[ebp-8]
004F38DF  |.  E8 585F1000    call Cyber.005F983C
004F38E4  |.  FF45 8C        inc dword ptr ss:[ebp-74]
004F38E7  |.  66:C745 80 08> mov word ptr ss:[ebp-80],8
004F38ED  |.  80BD 6FFFFFFF> cmp byte ptr ss:[ebp-91],0
004F38F4  |.  0F84 9E050000  je Cyber.004F3E98                  ;  if( bRegLog==0 ) 跳下去继续验证
```

`bRegLog=1` 时，程序没有直接跳向返回 `0`，作者跟你开了个玩笑，大概过程是这样的：

```c
if(szRegCode==szStr1){
    if(szRegCode==szStr2){
        if(szRegCode==szStr3){
            if(szRegCode==szStr4){
                if(szRegCode==szStr5){
                    if(szRegCode==szStr6){
                        if(szRegCode==szStr7)
                            CheckCode( );
                    }
                }
            }
        }
    }
}

return false;
```

如果你跟下去，你就会发现上当了：注册码怎么可能同时等于 `7` 个不同的字符串呢？即使相等，最后还是去验证注册码。

```assembly
004F3E98  |>  \8D45 F8       lea eax,dword ptr ss:[ebp-8]
004F3E9B  |.  E8 283CF1FF    call Cyber.00407AC8
```



```
004F3EA0  |.  50             push eax                     ; /Arg1  
004F3EA1  |.  E8 6E000000   call Cyber.004F3F14          ; \CheckCode( )  
004F3EA6  |.  59            pop ecx  
004F3EA7  |.  84C0          test al,al  
004F3EA9  |.  75 33         jnz short Cyber.004F3EDE     ;  if(CheckCode( )==true) return true;  
004F3EAB  |.  33C0          xor eax,eax                  ;  return false  
………………  
004F3EDC  |.  EB 31         jmp short Cyber.004F3F0F     ;  JUMP TO：return false  
004F3EDE  |>  B0 01         mov al,1                     ;  return true  
………………  
004F3F0F  |>  8BE5          mov esp,ebp  
004F3F11  |.  5D            pop ebp  
004F3F12  \.  C3            retn  
```

`CheckCode( )` 当然是关键了，继续跟进这个关键：

```
004F3F14  /$  55            push ebp  
004F3F15  |.  8BEC          mov ebp,esp  
004F3F17  |.  83C4 C4       add esp,-3C  
004F3F1A  |.  C645 DE 00    mov byte ptr ss:[ebp-22],0  
004F3F1E  |.  6A 05         push 5                       ; /Arg3 = 00000005  
004F3F20  |.  FF75 08       push dword ptr ss:[ebp+8]     ; |Arg2  
004F3F23  |.  8D45 D4       lea eax,dword ptr ss:[ebp-2C] ; |  
004F3F26  |.  50            push eax                     ; |Arg1  
004F3F27  |.  E8 F00C0D00   call Cyber.005C4C1C          ; \取 szRegCode 的第 1 段 szRegCode1  
004F3F2C  |.  83C4 0C       add esp,0C  
004F3F2F  |.  6A 05         push 5                       ; /Arg3 = 00000005  
004F3F31  |.  8B55 08       mov edx,dword ptr ss:[ebp+8] ; |  
004F3F34  |.  83C2 06       add edx,6                   ; |  
004F3F37  |.  52            push edx                     ; |Arg2  
004F3F38  |.  8D4D D9       lea ecx,dword ptr ss:[ebp-27] ; |  
004F3F3B  |.  51            push ecx                     ; |Arg1  
004F3F3C  |.  E8 DB0C0D00   call Cyber.005C4C1C          ; \取 szRegCode 的第 2 段 szRegCode2  
004F3F41  |.  83C4 0C       add esp,0C  
004F3F44  |.  8D45 C4       lea eax,dword ptr ss:[ebp-3C]  
004F3F47  |.  50            push eax                     ; /Arg3  
004F3F48  |.  6A 0A         push 0A                      ; |Arg2 = 0000000A  
004F3F4A  |.  8D55 D4       lea edx,dword ptr ss:[ebp-2C] ; |  
004F3F4D  |.  52            push edx                     ; |Arg1  
004F3F4E  |.  E8 812A0000   call Cyber.004F69D4          ; \用 szRegCode1 和 szRegCode2 生成 bMd5Result[16]  
004F3F53  |.  83C4 0C       add esp,0C  
004F3F56  |.  33C9          xor ecx,ecx  
004F3F58  |.  894D FC       mov dword ptr ss:[ebp-4],ecx  
004F3F5B  |>  8B45 FC       /mov eax,dword ptr ss:[ebp-4]  
004F3F5E  |.  33D2          |xor edx,edx  
004F3F60  |.  8A5405 C4     |mov dl,byte ptr ss:[ebp+eax-3C]   ;  bMd5Result[i]  
004F3F64  |.  8955 F8       |mov dword ptr ss:[ebp-8],edx  
004F3F67  |.  8B4D 08       |mov ecx,dword ptr ss:[ebp+8]  
004F3F6A  |.  8B45 FC       |mov eax,dword ptr ss:[ebp-4]  
004F3F6D  |.  8A5401 0C     |mov dl,byte ptr ds:[ecx+eax+C]    ;  szRegCode3[i]  
004F3F71  |.  8855 F7       |mov byte ptr ss:[ebp-9],dl  
004F3F74  |.  8B45 F8       |mov eax,dword ptr ss:[ebp-8]  
004F3F77  |.  B9 1A000000   |mov ecx,1A                    ;  bMd5Result[i] % 26  
004F3F7C  |.  33D2          |xor edx,edx  
004F3F7E  |.  F7F1          |div ecx  
004F3F80  |.  80C2 41       |add dl,41                      ;  result = bMd5Result[i] % 26 + 0x41  
004F3F83  |.  3A55 F7       |cmp dl,byte ptr ss:[ebp-9]  
004F3F86  |.  74 07         |je short Cyber.004F3F8F       ;  if (szRegCode3[i]!=result) retrun false  
004F3F88  |.  33C0          |xor eax,eax  
004F3F8A  |.  E9 8D000000   |jmp Cyber.004F401C  
004F3F8F  |>  FF45 FC       |inc dword ptr ss:[ebp-4]  
004F3F92  |.  837D FC 05    |cmp dword ptr ss:[ebp-4],5  
004F3F96  |.^ 7C C3         \jl short Cyber.004F3F5B  
004F3F98  |.  C745 F0 05000>mov dword ptr ss:[ebp-10],5  
004F3F9F  |>  8B55 F0       /mov edx,dword ptr ss:[ebp-10]  
004F3FA2  |.  33C9          |xor ecx,ecx  
004F3FA4  |.  8A4C15 C4     |mov cl,byte ptr ss:[ebp+edx-3C] ;  bMd5Result[i+5]  
004F3FA8  |.  894D EC       |mov dword ptr ss:[ebp-14],ecx  
004F3FAB  |.  8B45 08       |mov eax,dword ptr ss:[ebp+8]  
004F3FAE  |.  8B55 F0       |mov edx,dword ptr ss:[ebp-10]  
004F3FB1  |.  8A4C10 0D     |mov cl,byte ptr ds:[eax+edx+D]  ;  szRegCode4[i]  
004F3FB5  |.  884D EB       |mov byte ptr ss:[ebp-15],cl  
004F3FB8  |.  8B45 EC       |mov eax,dword ptr ss:[ebp-14]  
004F3FBB  |.  B9 1A000000   |mov ecx,1A  
004F3FC0  |.  33D2          |xor edx,edx  
004F3FC2  |.  F7F1          |div ecx                       ;  bMd5Result[i+5] % 26  
004F3FC4  |.  80C2 41       |add dl,41                     ;  result = bMd5Result[i] % 26 + 0x41  
004F3FC7  |.  3A55 EB       |cmp dl,byte ptr ss:[ebp-15]  
004F3FCA  |.  74 04         |je short Cyber.004F3FD0       ;  if (szRegCode4[0]!=result) retrun false  
004F3FCC  |.  33C0          |xor eax,eax  
004F3FCE  |.  EB 4C         |jmp short Cyber.004F401C  
004F3FD0  |>  FF45 F0       |inc dword ptr ss:[ebp-10]  
004F3FD3  |.  837D F0 0A    |cmp dword ptr ss:[ebp-10],0A  
004F3FD7  |.^ 7C C6         \jl short Cyber.004F3F9F  
004F3FD9  |.  C745 E4 0A000>mov dword ptr ss:[ebp-1C],0A  
004F3FE0  |>  8B55 E4       /mov edx,dword ptr ss:[ebp-1C]  
004F3FE3  |.  33C9          |xor ecx,ecx  
004F3FE5  |.  8A4C15 C4     |mov cl,byte ptr ss:[ebp+edx-3C] ;  bMd5Result[i+10]  
004F3FE9  |.  894D E0       |mov dword ptr ss:[ebp-20],ecx  
004F3FEC  |.  8B45 08       |mov eax,dword ptr ss:[ebp+8]  
004F3FEF  |.  8B55 E4       |mov edx,dword ptr ss:[ebp-1C]  
004F3FF2  |.  8A4C10 0E     |mov cl,byte ptr ds:[eax+edx+E]  ;  szRegCode5[i]  
004F3FF6  |.  884D DF       |mov byte ptr ss:[ebp-21],cl  
004F3FF9  |.  8B45 E0       |mov eax,dword ptr ss:[ebp-20]  
004F3FFC  |.  B9 1A000000   |mov ecx,1A  
004F4001  |.  33D2          |xor edx,edx  
004F4003  |.  F7F1          |div ecx                       ;  bMd5Result[i+10] % 26  
004F4005  |.  80C2 41       |add dl,41                     ;  result = bMd5Result[i+10] % 26 + 0x41  
004F4008  |.  3A55 DF       |cmp dl,byte ptr ss:[ebp-21]  
004F400B  |.  74 04         |je short Cyber.004F4011       ;  if (szRegCode5[0]!=result) retrun false  
004F400D  |.  33C0          |xor eax,eax  
004F400F  |.  EB 0B         |jmp short Cyber.004F401C  
004F4011  |>  FF45 E4       |inc dword ptr ss:[ebp-1C]  
004F4014  |.  837D E4 0F    |cmp dword ptr ss:[ebp-1C],0F  
004F4018  |.^ 7C C6         \jl short Cyber.004F3FE0  
004F401A  |.  B0 01         mov al,1  
004F401C  |>  8BE5          mov esp,ebp  
004F401E  |.  5D            pop ebp  
004F401F  \.  C3            retn  
```

将 `bMd5Result[16]` 中前 15 个 `byte` 对 26 取余，再加 0x41 就得到组成 `szRegCode3`、`4`、`5` 的 15 个字符

程序将 `szRegCode1="98765"` 和 `szRegCode2="56789"` 连接得到 `"9876556789"`

`bMd5Result[16]`，正是对 `"9876556789"` 后的字符串 MD5 运算的结果，跟进

引用:
```
004F3F4E  |.  E8 812A0000   call Cyber.004F69D4          ; \用 szRegCode1 和 szRegCode2 生成 bMd5Result[16] 
```

看看：

```
004F69D4  /$  55            push ebp  
004F69D5  |.  8BEC          mov ebp,esp  
004F69D7  |.  83C4 A8       add esp,-58  
004F69DA  |.  8D45 A8       lea eax,dword ptr ss:[ebp-58]  
004F69DD  |.  50            push eax                     ; /Arg1  
004F69DE  |.  E8 45020000   call Cyber.004F6C28          ; \MD5\_Init( )  
004F69E3  |.  59            pop ecx  
004F69E4  |.  FF75 0C       push dword ptr ss:[ebp+C]    ; /Arg3  
004F69E7  |.  FF75 08       push dword ptr ss:[ebp+8]    ; |Arg2  
004F69EA  |.  8D55 A8       lea edx,dword ptr ss:[ebp-58] ; |  
004F69ED  |.  52            push edx                     ; |Arg1  
004F69EE  |.  E8 71020000   call Cyber.004F6C64          ; \MD5\_Update( )  
```



```
004F69F3  .|  83C4 0C      add esp,0C  
004F69F6  .|  8D4D A8      lea ecx,dword ptr ss:[ebp-58]  
004F69F9  .|  51         push ecx                    ; /Arg2  
004F69FA  .|  FF75 10     push dword ptr ss:[ebp+10] ; |Arg1  
004F69FD  .|  E8 3A030000 call Cyber.004F6D3C       ; \MD5_Final( )  
004F6A02  .|  83C4 08     add esp,8  
004F6A05  .|  8BE5        mov esp,ebp  
004F6A07  .|  5D         pop ebp  
004F6A08  .\  C3         retn  
```

引用:
```
004F69DE  .|  E8 45020000 call Cyber.004F6C28       ; \MD5_Init( ) 
```

```
004F6C28  /$  55         push ebp  
004F6C29  .|  8BEC        mov ebp,esp  
004F6C2B  .|  8B45 08     mov eax,dword ptr ss:[ebp+8]  
004F6C2E  .|  33D2        xor edx,edx  
004F6C30  .|  8950 14     mov dword ptr ds:[eax+14],edx  
004F6C33  .|  8B45 08     mov eax,dword ptr ss:[ebp+8]  
004F6C36  .|  8950 10     mov dword ptr ds:[eax+10],edx  
004F6C39  .|  8B4D 08     mov ecx,dword ptr ss:[ebp+8]  
004F6C3C  .|  C701 01234567 mov dword ptr ds:[ecx],67452301  
004F6C42  .|  8B45 08     mov eax,dword ptr ss:[ebp+8]  
004F6C45  .|  C740 04 89ABC mov dword ptr ds:[eax+4],EFCDAB89  
004F6C4C  .|  8B55 08     mov edx,dword ptr ss:[ebp+8]  
004F6C4F  .|  C742 08 FEDCB mov dword ptr ds:[edx+8],98BADCFE  
004F6C56  .|  8B4D 08     mov ecx,dword ptr ss:[ebp+8]  
004F6C59  .|  C741 0C 76543 mov dword ptr ds:[ecx+C],10325476  
004F6C60  .|  5D         pop ebp  
004F6C61  .\  C3         retn  
```

看来常数没有变形，不过可以确认 `bMd5Result[16]` 正是 MD5 的结果了。

但是用 HASH 计算器计算结果却不一样，看来 MD5 变形了。

MD5 运算首先要填充数据，当填充完成后，看看内容数据确认，数据填充有没有变形：

引用:
```
0012FC8C  39 38 37 36 35 35 36 37 38 39 80 00 00 00 00 00  9876556789€.....  
0012FC9C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
0012FCAC  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
0012FCBC  00 00 00 00 00 00 00 00 50 00 00 00 00 00 00 00  ........P....... 
```

数据填充也没有变形，那么应该是函数变形或加法常数变形了。

跟进 `call 004F6DFC` 看看：

```
004F6E36  .|  8B45 F8     mov eax,dword ptr ss:[ebp-8]  
004F6E39  .|  2345 F4     and eax,dword ptr ss:[ebp-C]  
004F6E3C  .|  8B55 F8     mov edx,dword ptr ss:[ebp-8]  
004F6E3F  .|  F7D2        not edx  
004F6E41  .|  2355 F0     and edx,dword ptr ss:[ebp-10]  
004F6E44  .|  0BC2        or eax,edx  
004F6E46  .|  0345 B0     add eax,dword ptr ss:[ebp-50]  
004F6E49  .|  05 783234D7 add eax,D7343278             ;  加法常数 1 - 1  
004F6E4E  .|  0145 FC     add dword ptr ss:[ebp-4],eax  
004F6E51  .|  8B4D FC     mov ecx,dword ptr ss:[ebp-4]  
004F6E54  .|  C1E1 07     shl ecx,7  
004F6E57  .|  8B45 FC     mov eax,dword ptr ss:[ebp-4]  
004F6E5A  .|  C1E8 19     shr eax,19  
004F6E5D  .|  0BC8        or ecx,eax  
004F6E5F  .|  894D FC     mov dword ptr ss:[ebp-4],ecx  
004F6E62  .|  8B55 F8     mov edx,dword ptr ss:[ebp-8]  
004F6E65  .|  0155 FC     add dword ptr ss:[ebp-4],edx  
004F6E68  .|  8B4D FC     mov ecx,dword ptr ss:[ebp-4]  
004F6E6B  .|  234D F8     and ecx,dword ptr ss:[ebp-8]  
004F6E6E  .|  8B45 FC     mov eax,dword ptr ss:[ebp-4]  
004F6E71  .|  F7D0        not eax  
004F6E73  .|  2345 F4     and eax,dword ptr ss:[ebp-C]  
004F6E76  .|  0BC8        or ecx,eax  
004F6E78  .|  034D B4     add ecx,dword ptr ss:[ebp-4C]  
004F6E7B  .|  81C1 463FCAE8 add ecx,E8CA3F46            ;  加法常数 1 - 2  
```

………………

从 1 - 1 到 1 - 16

从 2 - 1 到 2 - 16

从 3 - 1 到 3 - 16

从 4 - 1 到 4 - 16

………………

```
004F79CA  .|  0B55 F0     or edx,dword ptr ss:[ebp-10]  
004F79CD  .|  3355 FC     xor edx,dword ptr ss:[ebp-4]  
004F79D0  .|  0355 B8     add edx,dword ptr ss:[ebp-48]  
004F79D3  .|  81C2 3B44D72A add edx,2AD7443B            ;  加法常数 4 - 15  
004F79D9  .|  0155 F4     add dword ptr ss:[ebp-C],edx  
004F79DC  .|  8B4D F4     mov ecx,dword ptr ss:[ebp-C]  
004F79DF  .|  C1E1 0F     shl ecx,0F  
004F79E2  .|  8B45 F4     mov eax,dword ptr ss:[ebp-C]  
004F79E5  .|  C1E8 11     shr eax,11  
004F79E8  .|  0BC8        or ecx,eax  
004F79EA  .|  894D F4     mov dword ptr ss:[ebp-C],ecx  
004F79ED  .|  8B55 F0     mov edx,dword ptr ss:[ebp-10]  
004F79F0  .|  0155 F4     add dword ptr ss:[ebp-C],edx  
004F79F3  .|  8B4D FC     mov ecx,dword ptr ss:[ebp-4]  
004F79F6  .|  F7D1        not ecx  
004F79F8  .|  0B4D F4     or ecx,dword ptr ss:[ebp-C]  
004F79FB  .|  334D F0     xor ecx,dword ptr ss:[ebp-10]  
004F79FE  .|  034D D4     add ecx,dword ptr ss:[ebp-2C]  
004F7A01  .|  81C1 91D386EB add ecx,EB86D391            ;  加法常数 4 - 16  
004F7A07  .|  014D F8     add dword ptr ss:[ebp-8],ecx  
```

仔细比对，函数确实没有变形，而加法常数改变了好多。没有什么太好的办法，一个一个整理出来吧！这确实是一件非常繁琐的事。

整理出来是这样的：

```
D7343278 E8CA3F46 242430DB C1B89EEE  
F57C45AF 4743562A A8343513 FD445501  
698598D8 8B24F7AF FF555BB1 895FD7BE  
6B989122 FD980983 A67944EE 49B4AD21  
F6142562 C0405640 26577A51 E9B678AA  
D62F198D 24414FF  BBA1E681 ECD CFBC8  
21ECCDE6 C33707D6 F4D5CD87 455A14ED  
A9E3E905 FCECA3F8 676F02D9 8C2ACC8A  
FF756742 8771F681 6D9D6122 FDE5380C  
A4BEEA44 76DECFA9 F6BB4B60 BEBFBC70  
26577EC6 EAA127FA D4653085 4881D05  
D9D4D039 E89B99E5 1FA27CF8 34AC5665  
F4292244 432AFF97 AB9423A7 FC934039  
655B59C3 8F0CCC92 FFEFF47D 85845DD1  
6FA87E4F FE2CE6E0 A3014314 434811A1  
F7537E82 BD3AF235 2AD7443B EB86D391  
```

将这些常数填入 MD5 的 C++ 类中，就可以写出自己的注册机了。

---

【Keygen】

```cpp
void CKeygenDlg::OnOK()  
{  
  // TODO: Add extra validation here  
  m_Edit1=_T("用户名可以任意输入！注册与此无关！");  
  char szInbuff[16]={0},    //用于存储合并后的 szRegCode1 和 szRegCode2  
    szRegCode1[6],  
    szRegCode2[6],  
    szRegCode3[6],  
    szRegCode4[6],  
    szRegCode5[6];  

//get szRegCode1、szRegCode2  
  for(byte i=0;i<5;i++){  
    szRegCode1[i]=0x41+rand()%26;  
    szRegCode2[i]=0x41+rand()%26;  
  }  
  szRegCode1[5]=0;  
  szRegCode2[5]=0;  
  strcat(szInbuff,szRegCode1);  
  strcat(szInbuff,szRegCode2);  

//MD5_calc  
  MD5_CTX context;  
  MD5Init(&context);  
  MD5Update(&context,(unsigned char *)szInbuff,10);  
  MD5Final(&context);  

//get szRegCode3、szRegCode4、szRegCode5  
  szRegCode3[0] = char(0x41+(context.state[0]&0xFF)%26);  
  szRegCode3[1] = char(0x41+((context.state[0]>>8)&0xFF)%26);  
  szRegCode3[2] = char(0x41+((context.state[0]>>16)&0xFF)%26);  
  szRegCode3[3] = char(0x41+(context.state[0]>>24)%26);  
  szRegCode3[4] = char(0x41+(context.state[1]&0xFF)%26);  
  szRegCode3[5] = 0;  
  szRegCode4[0] = char(0x41+((context.state[1]>>8)&0xFF)%26);  
  szRegCode4[1] = char(0x41+((context.state[1]>>16)&0xFF)%26);  
  szRegCode4[2] = char(0x41+(context.state[1]>>24)%26);  
  szRegCode4[3] = char(0x41+(context.state[2]&0xFF)%26);  
  szRegCode4[4] = char(0x41+((context.state[2]>>8)&0xFF)%26);  
  szRegCode4[5] = 0;  
  szRegCode5[0] = char(0x41+((context.state[2]>>16)&0xFF)%26);  
  szRegCode5[1] = char(0x41+(context.state[2]>>24)%26);  
  szRegCode5[2] = char(0x41+(context.state[3]&0xFF)%26);  
  szRegCode5[3] = char(0x41+((context.state[3]>>8)&0xFF)%26);  
  szRegCode5[4] = char(0x41+((context.state[3]>>16)&0xFF)%26);  
  szRegCode5[5] = 0;  
  m_Edit2 = szRegCode1;  m_Edit2 += '-';  
  m_Edit2 += szRegCode2; m_Edit2 += '-';  
  m_Edit2 += szRegCode3; m_Edit2 += '-';  
  m_Edit2 += szRegCode4; m_Edit2 += '-';  
  m_Edit2 += szRegCode5;  
  UpdateData(false);   
}
```

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

16:45 2006-4-28
4dd466d0e1e0cc123afbe504da2462ca
