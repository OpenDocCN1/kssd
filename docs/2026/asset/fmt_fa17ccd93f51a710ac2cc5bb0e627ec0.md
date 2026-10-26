
# [原创]WinImage 8.10注册算法简单分析

- 【文章标题】: WinImage 8.10注册算法分析
- 【文章作者】: herx
- 【作者邮箱】: heuix1422@163.com
- 【作者QQ号】: 369136816
- 【软件名称】: WinImage（英文版）
- 【下载地址】: `30dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6A6L8X3W2E0j5h3N6W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2F1K9h3#2S2k6$3g2Q4x3X3g2Z5N6r3@1`.
- 【加壳方式】: 无壳
- 【编写语言】: VC8
- 【使用工具】: `W32DASM`，`OLLYDBG`，`PEID`
- 【操作平台】: xp sp2
- 【软件介绍】: 制作.img,.iso,等映像文件
- 【作者声明】: 只是感兴趣，作为学习，如果需要请购买正版，由于水平有限错误之处请高手指出。。。。

---

## 详细过程

由于刚开始进行算法分析，希望大虾们给提点宝贵意见。。  
用`PEID`查看是`VC8 -> Microsoft Corporation [Overlay] \*`  
软件未注册有30天试用  
不输入或输入假的注册码都提示 "Registration information is invalid"  
用`W32DASM`或下断点`bp GetDlgItemTextA`很容易找到关键地方

```assembly
00466474    > \8B7424 0C   mov esi,dword ptr ss:[esp+C]        ;
00466478    .  8B3D 587549>mov edi,dword ptr ds:[<&USER32.GetD>;  USER32.GetDlgItemTextA
0046647E    .  68 01010000 push 101                            ;
00466483    .  68 F83A4B00 push winimage.004B3AF8              ; |Buffer = 004B3AF8 获得name后放在这
00466488    .  68 16080000 push 816                            ;
0046648D    .  56          push esi                            ;
0046648E    .  FFD7        call edi                            ; GetDlgItemTextA
00466490    .  6A 7F       push 7F                             ; 下断点后在注册框输入假注册码，返回后停在这
00466492    .  68 683E4B00 push winimage.004B3E68              ; |Buffer = 004B3E68  输入假注册码放在这
00466497    .  68 17080000 push 817                            ;
0046649C    .  56          push esi                            ;
0046649D    .  FFD7        call edi                            ; GetDlgItemTextA
0046649F    .  68 F83A4B00 push winimage.004B3AF8              ; ASCII "heruixi"       输入的name
004664A4    .  B8 683E4B00 mov eax,winimage.004B3E68           ; ASCII "9876543210"    eax<=假注册码
004664A9    .  E8 42C4FAFF call winimage.004128F0              ；进行计算的CALL ，跟进，
004664AE    .  8B15 B0374B>mov edx,dword ptr ds:[4B37B0]
004664B4    .  33C9        xor ecx,ecx
004664B6    .  83C4 04     add esp,4
004664B9    .  3BC1        cmp eax,ecx                         ；关键比较eax是不是零，注册码正确eax=1，不正确为0
004664BB    .  A3 1C424B00 mov dword ptr ds:[4B421C],eax
004664C0    .  74 06       je short winimage.004664C8
004664C2    .  8915 20344B>mov dword ptr ds:[4B3420],edx
004664C8    >  390D 20344B>cmp dword ptr ds:[4B3420],ecx
004664CE    .  8915 CC3A4B>mov dword ptr ds:[4B3ACC],edx
004664D4    .  75 05       jnz short winimage.004664DB
004664D6    .  A3 CC3A4B00 mov dword ptr ds:[4B3ACC],eax
004664DB    >  3BC1        cmp eax,ecx                            ； 关键比较
004664DD    .  75 4B       jnz short winimage.0046652A             ；eax不为零跳到注册成功
004664DF    .  68 10200000 push 2010                              ；显示注册码无效
004664E4    .  68 2D040000 push 42D
004664E9    .  68 2B040000 push 42B
004664EE    .  56          push esi
004664EF    .  C705 203F4B>mov dword ptr ds:[4B3F20],1
004664F9    .  C705 C43A4B>mov dword ptr ds:[4B3AC4],1
00466503    .  880D 683E4B>mov byte ptr ds:[4B3E68],cl
00466509    .  880D F83A4B>mov byte ptr ds:[4B3AF8],cl
0046650F    .  E8 DC80FFFF call winimage.0045E5F0               ;
00466514    .  83C4 10     add esp,10
00466517    .  6A 01       push 1                              ; /Result = 1
00466519    .  56          push esi                            ; |hWnd
0046651A    .  FF15 7C7549>call dword ptr ds:[<&USER32.EndDial>; \EndDialog
00466520    .  5F          pop edi
00466521    .  B8 01000000 mov eax,1
00466526    .  5E          pop esi
00466527    .  C2 1000     retn 10
0046652A    >  68 40200000 push 2040                           ；显示注册成功
0046652F    .  68 2D040000 push 42D
00466534    .  68 2A040000 push 42A
00466539    .  56          push esi
0046653A    .  890D 203F4B>mov dword ptr ds:[4B3F20],ecx
00466540    .  890D C43A4B>mov dword ptr ds:[4B3AC4],ecx
00466546    .  E8 A580FFFF call winimage.0045E5F0
0046654B    .  83C4 10     add esp,10
0046654E    .  6A 01       push 1                              ; /Result = 1
00466550    .  56          push esi                            ; |hWnd
00466551    .  FF15 7C7549>call dword ptr ds:[<&USER32.EndDial>; \EndDialog
00466557    .  5F          pop edi
00466558    .  B8 01000000 mov eax,1
0046655D    .  5E          pop esi
0046655E    .  C2 1000     retn 10
```

### 跟进计算的CALL：

```assembly
004128F0    $  81EC 000200>sub esp,200
004128F6    .  56          push esi
004128F7    .  8D7424 04   lea esi,dword ptr ss:[esp+4]
004128FB    .  C705 B0374B>mov dword ptr ds:[4B37B0],0
00412905    .  E8 C6210000 call winimage.00414AD0          ；取得注册码，并转为大写存入[esp+4]
0041290A    .  8B8424 0802>mov eax,dword ptr ss:[esp+208]  ；取得用户名——》eax
00412911    .  E8 0A210000 call winimage.00414A20          ；对用户名进行计算call，然后跟进
00412916    .  3D 26DDDCB8 cmp eax,B8DCDD26                ；如果没有输入用户名
0041291B    .  0F84 3DA102>je winimage.0043CA5E            ；计算结果为B8DCDD26
                                                                       跳到0043CA5E  xor eax,eax，然后跳到
                                                                    004129F4    >  5E          pop esi
                                                                    004129F5    .  81C4 000200>add esp,200
                                                                    004129FB    .  C3          retn
00412921    .  8BF0        mov esi,eax                  ；esi指向第一次计算出的结果
00412923    .  57          push edi
00412924    .  8D4424 08   lea eax,dword ptr ss:[esp+8]  ；eax指向假注册码
00412928    .  50          push eax
00412929    .  8BC6        mov eax,esi
0041292B    .  8DBC24 0C01>lea edi,dword ptr ss:[esp+10C]
00412932    .  E8 69FFFFFF call winimage.004128A0           ；计算出最后的真注册码
00412937    .  50          push eax
00412938    .  E8 33870000 call winimage.0041B070            ；与假注册码每一位进行比较call
0041293D    .  83C4 08     add esp,8
00412940    .  85C0        test eax,eax
00412942    .  0F84 E9A102>je winimage.0043CB31             ；注册码正确跳转到0043CB31  mov eax，1
                                                                                jmp   004129F3返回

     .

     .

     .

     .

004129C8    .  8D5424 08   lea edx,dword ptr ss:[esp+8]
004129CC    .  52          push edx
004129CD    .  8D86 971906>lea eax,dword ptr ds:[esi+2061997]
004129D3    .  E8 C8FEFFFF call winimage.004128A0
004129D8    .  50          push eax
004129D9    .  E8 92860000 call winimage.0041B070
004129DE    .  83C4 08     add esp,8
004129E1    .  85C0        test eax,eax
004129E3    .  0F85 7CA002>jnz winimage.0043CA65
004129E9    >  B8 01000000 mov eax,1
004129EE    .  A3 B0374B00 mov dword ptr ds:[4B37B0],eax
004129F3    >  5F          pop edi
004129F4    >  5E          pop esi
004129F5    .  81C4 000200>add esp,200
004129FB    .  C3          retn
```

### 用户名计算call

```assembly
00414A20 <> $  81EC 04010000      sub esp,104
00414A26    .  56                 push esi
00414A27    .  57                 push edi
00414A28    .  8D7424 0C          lea esi,dword ptr ss:[esp+C]
00414A2C    .  BF 4C694700        mov edi,winimage.0047694C
00414A31    .  E8 9A000000        call <winimage.00414AD0>               ;用户名转为大写
00414A36    .  56                 push esi                              ; esi指向转换后的用户名
00414A37    .  FF15 1C734900      call dword ptr ds:[<&KERNEL32.lstrlen>; \lstrlenA
00414A3D    .  85C0               test eax,eax
00414A3F    .  7E 75              jle short winimage.00414AB6           ; 用户名长度为零?
00414A41    .  53                 push ebx
00414A42    .  55                 push ebp
00414A43    .  8BEE               mov ebp,esi                           ; ebp指向转换后的用户名
00414A45    .  33C9               xor ecx,ecx
00414A47    .  83ED 03            sub ebp,3
```
