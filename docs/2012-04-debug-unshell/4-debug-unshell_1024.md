
# 最新魔兽皇冠外挂 SVKP 1.3x - Pavol Cerven 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-624.htm`](https://www.pediy.com/kssd/pediy07/pediy7-624.htm)

**【破解作者】** `stasi[DCM][BCG][DFCG][FCG][OCN][CZG][D.4s]`

**【作者邮箱】** `stasi@163.com`

**【作者主页】** `www.icehack.org/stasi`

**【使用工具】** `od`

**【破解平台】** `Win9x/NT/2000/XP`

**【软件名称】** `魔兽皇冠 5.0.0805 免费版`

**【加壳方式】** `SVKP 1.3x -> Pavol Cerven`

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 【破解内容】

```
0040C000 w>   60               pushad                       入口
0040C001     E8 00000000      call wowcrown.0040C006
0040C006     5D               pop ebp
0040C007     81ED 06000000    sub ebp,6
0040C00D     EB 05            jmp short wowcrown.0040C014
0040C00F     B8 49DC0F06      mov eax,60FDC49
0040C014     64:A0 23000000   mov al,byte ptr fs:[23]
0040C01A     EB 03            jmp short wowcrown.0040C01F
0040C01C     C784E8 84C0EB03> mov dword ptr ds:[eax+ebp*8+3EBC084]>
0040C027     67:B9 49000000   mov ecx,49
0040C02D     8DB5 C5020000    lea esi,dword ptr ss:[ebp+2C5]
0012E3B6     6285 1E220000    bound eax,qword ptr ss:[ebp+221E]    标志异常
0012E3BC     EB 02            jmp short 0012E3C0
0012E3BE     0FE88B D1EB02CD  psubsb mm1,qword ptr ds:[ebx+CD02EBD>
0012E3C5     208B C2EB02CD    and byte ptr ds:[ebx+CD02EBC2],cl
0012E3CB     208B 8A401600    and byte ptr ds:[ebx+16408A],cl
0012E3D1     008B 89740100    add byte ptr ds:[ebx+17489],cl
0012E3D7     00BA 61110000    add byte ptr ds:[edx+1161],bh
0012E3DD     68 00400000      push 4000
```

区段名被清空，下断 `PE header`，只要是 `IAT` 在的段就行

内存映射，项目 `11`

```
地址=00400000
大小=0002D000 (184320.)
宿主=wowcrown 00400000
区段=
包含=PE header
类型=Imag 01001002
访问=R
初始访问=RWE
```

下断点 `bp GetModuleHandleA+5`（`od` 下这个断点在 `2000` 下是不是有问题？）

`shift+f9` 断下，取消断点，`ctrl+f9` 返回主程序。

`ctrl+f` 搜索特征码

```
cmp dword ptr ds:[ebx],251097CC
```

```
06055778     813B CC971025   cmp dword ptr ds:[ebx],251097CC      冲这来
0605577E     0F84 41170000   je 06056EC5 这里的处理方法很多 allnop 也行，jmp 060557E4 也行
06055784     813B C5B1662D   cmp dword ptr ds:[ebx],2D66B1C5
0605578A     0F84 62180000   je 06056FF2
06055790     813B 9404B2D9   cmp dword ptr ds:[ebx],D9B20494
06055796     0F84 AA1C0000   je 06057446
0605579C     813B A41A86D0   cmp dword ptr ds:[ebx],D0861AA4
060557A2     0F84 58210000   je 06057900
060557A8     813B 706586B1   cmp dword ptr ds:[ebx],B1866570
060557AE     0F84 C1240000   je 06057C75
060557B4     813B 0E46769B   cmp dword ptr ds:[ebx],9B76460E
060557BA     0F84 36280000   je 06057FF6
060557C0     813B DB0793E6   cmp dword ptr ds:[ebx],E69307DB
060557C6     0F84 76280000   je 06058042
060557CC     813B 627B6CA5   cmp dword ptr ds:[ebx],A56C7B62
060557D2     0F84 BA280000   je 06058092
060557D8     813B 664E96BB   cmp dword ptr ds:[ebx],BB964E66
060557DE     0F84 00290000   je 060580E4
060557E4     813B 4506D75B   cmp dword ptr ds:[ebx],5BD70645 点名批评这三个不良处理：）
060557EA     0F84 43290000   je 06058133
060557F0     813B 0DE0FC1D   cmp dword ptr ds:[ebx],1DFCE00D
060557F6     0F84 83290000   je 0605817F
060557FC     813B 31DD0F00   cmp dword ptr ds:[ebx],0FDD31
06055802     0F84 C6290000   je 060581CE
06055808     813B 95B75126   cmp dword ptr ds:[ebx],2651B795
0605580E     0F84 132A0000   je 06058227                      jmp 06055850
06055814     813B B482F64B   cmp dword ptr ds:[ebx],4BF682B4
0605581A     0F84 582A0000   je 06058278
```

搜索特征码

```
mov dword ptr ds:[edi],eax
popad
```

改成：

```
popad
mov dword ptr ds:[edi],eax  这方法真聪明，以后看能不能在其他地方用上：）
```

`hr 0012FFB0`

```
0012FCFB     E8 00000000      call 0012FD00
0012FD00     5D               pop ebp
0012FD01     E8 02000000      call 0012FD08
0012FD06     CD20 83042408    vxdcall 8240483
0012FD0C     C3               retn
```

模拟的干 `tc ebp==12ffc0`

```
060DE7DB     55               push ebp
060DE7DC     E9 E0080000      jmp 060DF0C1           不断的乱序处理
060DF0C1     50               push eax
060DF0C2     B8 23E36501      mov eax,165E323
060DF0C7     294424 04        sub dword ptr ss:[esp+4],eax
060DF0CB     58               pop eax
060DF0CC     E9 21050000      jmp 060DF5F2           乘乱还要偷东西：（
```

下午在 `dfcg` 里聊天的时候，`Dream`:正在搞 `DVDFab` 的 `IAT`，其实方法人人都会，就缺耐心了

`stolen code` 的寻找其实就是比耐心了

没有其他更好的方法了，只能一行一行的找：（

```
push ebp
MOV EBP,ESP
PUSH -1
PUSH 0165E323
PUSH 060DE159
MOV EAX,DWORD PTR FS:[0]
PUSH EAX
mov dword ptr fs:[0],esp
sub esp,68
POP EBX
PUSH ESI
PUSH EDI
mov dword ptr ss:[ebp-18],esp
sub ebx,ebx
mov dword ptr ss:[ebp-4],ebx
PUSH 0012FFE0
call dword ptr ds:[4063C8]
pop ecx
call dword ptr ds:[4063CC]
mov ecx,dword ptr ds:[40998C]
mov dword ptr ds:[eax],ecx
call dword ptr ds:[4063E0]
mov ecx,dword ptr ds:[409988]
mov dword ptr ds:[eax],ecx
mov eax,dword ptr ds:[4063E4]
mov eax,dword ptr ds:[eax]
mov dword ptr ds:[409994],eax
```

在空的地方补全 `stolen code`，最后跳到伪 `oep` 就行了

```
0040553C     55               push ebp                  这里新建 oep
0040553D     8BEC             mov ebp,esp
0040553F     6A FF            push -1
00405541     68 23E36501      push 165E323
00405546     68 59E10D06      push 60DE159
0040554B     64:A1 00000000   mov eax,dword ptr fs:[0]
00405551     50               push eax
00405552     64:8925 0000000> mov dword ptr fs:[0],esp
00405559     83EC 68          sub esp,68
0040555C     5B               pop ebx
0040555D     56               push esi
0040555E     57               push edi
0040555F     8965 E8          mov dword ptr ss:[ebp-18],esp
00405562     2BDB             sub ebx,ebx
00405564     895D FC          mov dword ptr ss:[ebp-4],ebx
00405567     68 E0FF1200      push 12FFE0
0040556C     FF15 C8634000    call dword ptr ds:[4063C8]           ; msvcrt.__set_app_type
00405572     59               pop ecx
00405573     FF15 CC634000    call dword ptr ds:[4063CC]           ; msvcrt.__p__fmode
00405579     8B0D 8C994000    mov ecx,dword ptr ds:[40998C]
0040557F     8908             mov dword ptr ds:[eax],ecx
00405581     FF15 E0634000    call dword ptr ds:[4063E0]           ; msvcrt.__p__commode
00405587     8B0D 88994000    mov ecx,dword ptr ds:[409988]
0040558D     8908             mov dword ptr ds:[eax],ecx
0040558F     A1 E4634000      mov eax,dword ptr ds:[4063E4]
00405594     8B00             mov eax,dword ptr ds:[eax]
00405596     A3 94994000      mov dword ptr ds:[409994],eax
004055A6     E8 1C010000      call wowcrown.004056C7               伪 oep
004055AB     391D 68984000    cmp dword ptr ds:[409868],ebx
004055B1     75 0C            jnz short wowcrown.004055BF
004055B3     68 C4564000      push wowcrown.004056C4
004055B8     FF15 E8634000    call dword ptr ds:[4063E8]           ; msvcrt.__setusermatherr
004055BE     59               pop ecx
004055BF     E8 EE000000      call wowcrown.004056B2
004055C4     68 1C904000      push wowcrown.0040901C
004055C9     68 18904000      push wowcrown.00409018
```

`IAT` 全面修复，因为跳过了输入表的加密和特殊函数的处理，`IAT` 就能全部 dump 出来，建立新的 `IAT`

```
; Syntax for each function in a thunk (the separator is a TAB)
; ------------------------------------------------------------
; Flag    RVA    ModuleName   Ordinal   Name
;
; Details for  parameter:
; ------------------------------
; Flag:  0 = valid: no  -> - Name contains the address of the redirected API (you can set
;                            it to zero if you edit it).
;                        - Ordinal is not considered but you should let '0000' as value.
;                        - ModuleName is not considered but you should let '?' as value.
;
;       1 = valid: yes -> All next parameters on the line will be considered.
```



`;
;                                                                                                  Function imported by ordinal must have no name (the 4th TAB must  

;                                                                                                  be there though).  

;  
;        2 = 和 0 相同，但它是为了这个载入器。          
  
;  
;        3 = 和 1 相同，但它是为了这个载入器。          
  
;  
; 并且最后，当编辑这个文件之时你同样也有危险！ :-)  

目标: `C:\\Documents and Settings\\shaojiajun\\桌面\\wowcrown.exe`  

`OEP: 0000553C` `IATRVA: 00006000` `IATSize: 00000444`  

`FThunk: 00006000` `NbFunc: 00000003`  

`1` `00006000` `advapi32.dll` `018C` `RegCloseKey`  

`1` `00006004` `advapi32.dll` `0193` `RegDeleteKeyA`  

`1` `00006008` `advapi32.dll` `0190` `RegCreateKeyExA`  

`FThunk: 00006010` `NbFunc: 00000006`  

`1` `00006010` `kernel32.dll` `0175` `GetSystemDirectoryA`  

`1` `00006014` `kernel32.dll` `002C` `CopyFileA`  

`1` `00006018` `kernel32.dll` `016B` `GetStartupInfoA`  

`1` `0000601C` `kernel32.dll` `013F` `GetModuleHandleA`  

`1` `00006020` `kernel32.dll` `0295` `SetEvent`  

`1` `00006024` `kernel32.dll` `0158` `GetProcAddress`  

`FThunk: 0000602C` `NbFunc: 000000CD`  

`1` `0000602C` `mfc42.dll` `12F5`  

`1` `00006030` `mfc42.dll` `0A18`  

`1` `00006034` `mfc42.dll` `09D2`  

`1` `00006038` `mfc42.dll` `17A4`  

`1` `0000603C` `mfc42.dll` `0EF1`  

`1` `00006040` `mfc42.dll` `06EF`  

`1` `00006044` `mfc42.dll` `1137`  

`1` `00006048` `mfc42.dll` `1479`  

`1` `0000604C` `mfc42.dll` `0951`  

`1` `00006050` `mfc42.dll` `142B`  

`1` `00006054` `mfc42.dll` `18E6`  

`1` `00006058` `mfc42.dll` `1101`  

`1` `0000605C` `mfc42.dll` `14A0`  

`1` `00006060` `mfc42.dll` `0ED6`  

`1` `00006064` `mfc42.dll` `12E5`  

`1` `00006068` `mfc42.dll` `1159`  

`1` `0000606C` `mfc42.dll` `0A58`  

`1` `00006070` `mfc42.dll` `0807`  

`1` `00006074` `mfc42.dll` `18E8`  

`1` `00006078` `mfc42.dll` `0BA6`  

`1` `0000607C` `mfc42.dll` `13C9`  

`1` `00006080` `mfc42.dll` `06BF`  

`1` `00006084` `mfc42.dll` `148D`  

`1` `00006088` `mfc42.dll` `098E`  

`1` `0000608C` `mfc42.dll` `084C`  

`1` `00006090` `mfc42.dll` `1479`  

`1` `00006094` `mfc42.dll` `0BA6`  

`1` `00006098` `mfc42.dll` `0BA6`  

`1` `0000609C` `mfc42.dll` `0BA6`  

`1` `000060A0` `mfc42.dll` `06F0`  

`1` `000060A4` `mfc42.dll` `0C40`  

`1` `000060A8` `mfc42.dll` `0CBE`  

`1` `000060AC` `mfc42.dll` `0BA9`  

`1` `000060B0` `mfc42.dll` `0C09`  

`1` `000060B4` `mfc42.dll` `0BA0`  

`1` `000060B8` `mfc42.dll` `0EF6`  

`1` `000060BC` `mfc42.dll` `0EF1`  

`1` `000060C0` `mfc42.dll` `0EF1`  

`1` `000060C4` `mfc42.dll` `0BA6`  

`1` `000060C8` `mfc42.dll` `0FF0`  

`1` `000060CC` `mfc42.dll` `1213`  

`1` `000060D0` `mfc42.dll` `1149`  

`1` `000060D4` `mfc42.dll` `0E0D`  

`1` `000060D8` `mfc42.dll` `0290`  

`1` `000060DC` `mfc42.dll` `0281`  

`1` `000060E0` `mfc42.dll` `0237`  

`1` `000060E4` `mfc42.dll` `0144`  

`1` `000060E8` `mfc42.dll` `0339`  

`1` `000060EC` `mfc42.dll` `0261`  

`1` `000060F0` `mfc42.dll` `08FE`  

`1` `000060F4` `mfc42.dll` `108A`  

`1` `000060F8` `mfc42.dll` `1266`  

`1` `000060FC` `mfc42.dll` `0A55`  

`1` `00006100` `mfc42.dll` `19C5`  

`1` `00006104` `mfc42.dll` `1A90`  

`1` `00006108` `mfc42.dll` `0E89`  

`1` `0000610C` `mfc42.dll` `1AE0`  

`1` `00006110` `mfc42.dll` `031B`  

`1` `00006114` `mfc42.dll` `1861`  

`1` `00006118` `mfc42.dll` `1935`  

`1` `0000611C` `mfc42.dll` `094B`  

`1` `00006120` `mfc42.dll` `0320`  

`1` `00006124` `mfc42.dll` `035C`  

`1` `00006128` `mfc42.dll` `021C`  

`1` `0000612C` `mfc42.dll` `08F1`  

`1` `00006130` `mfc42.dll` `0942`  

`1` `00006134` `mfc42.dll` `1241`  

`1` `00006138` `mfc42.dll` `10B2`  

`1` `0000613C` `mfc42.dll` `18E7`  

`1` `00006140` `mfc42.dll` `1186`  

`1` `00006144` `mfc42.dll` `09FA`  

`1` `00006148` `mfc42.dll` `09D0`  

`1` `0000614C` `mfc42.dll` `1663`  

`1` `00006150` `mfc42.dll` `0F52`  

`1` `00006154` `mfc42.dll` `0441`  

`1` `00006158` `mfc42.dll` `144F`  

`1` `0000615C` `mfc42.dll` `095C`  

`1` `00006160` `mfc42.dll` `0D12`  

`1` `00006164` `mfc42.dll` `14B4`  

`1` `00006168` `mfc42.dll` `14B6`  

`1` `0000616C` `mfc42.dll` `0AA5`  

`1` `00006170` `mfc42.dll` `0FEF`  

`1` `00006174` `mfc42.dll` `125A`  

`1` `00006178` `mfc42.dll` `14BB`  

`1` `0000617C` `mfc42.dll` `14A9`  

`1` `00006180` `mfc42.dll` `1652`  

`1` `00006184` `mfc42.dll` `120E`  

`1` `00006188` `mfc42.dll` `0E9A`  

`1` `0000618C` `mfc42.dll` `0231`  

`1` `00006190` `mfc42.dll` `032F`  

`1` `00006194` `mfc42.dll` `0A3D`  

`1` `00006198` `mfc42.dll` `046E`  

`1` `0000619C` `mfc42.dll` `04AF`  

`1` `000061A0` `mfc42.dll` `04DF`  

`1` `000061A4` `mfc42.dll` `19FA`  

`1` `000061A8` `mfc42.dll` `19BF`  

`1` `000061AC` `mfc42.dll` `1118`  

`1` `000061B0` `mfc42.dll` `068F`  

`1` `000061B4` `mfc42.dll` `068F`  

`1` `000061B8` `mfc42.dll` `068F`  

`1` `000061BC` `mfc42.dll` `068F`  

`1` `000061C0` `mfc42.dll` `068F`  

`1` `000061C4` `mfc42.dll` `068F`  

`1` `000061C8` `mfc42.dll` `1251`  

`1` `000061CC` `mfc42.dll` `068F`  

`1` `000061D0` `mfc42.dll` `1282`  

`1` `000061D4` `mfc42.dll` `068F`  

`1` `000061D8` `mfc42.dll` `068F`  

`1` `000061DC` `mfc42.dll` `1251`  

`1` `000061E0` `mfc42.dll` `1251`  

`1` `000061E4` `mfc42.dll` `1282`  

`1` `000061E8` `mfc42.dll` `1282`  

`1` `000061EC` `mfc42.dll` `1AC8`  

`1` `000061F0` `mfc42.dll` `1A98`  

`1` `000061F4` `mfc42.dll` `068F`  

`1` `000061F8` `mfc42.dll` `11ED`  

`1` `000061FC` `mfc42.dll` `10F5`  

`1` `00006200` `mfc42.dll` `1323`  

`1` `00006204` `mfc42.dll` `10F5`  

`1` `00006208` `mfc42.dll` `131C`  

`1` `0000620C` `mfc42.dll` `068F`  

`1` `00006210` `mfc42.dll` `13D4`  

`1` `00006214` `mfc42.dll` `10F4`  

`1` `00006218` `mfc42.dll` `10FB`  

`1` `0000621C` `mfc42.dll` `1270`  

`1` `00006220` `mfc42.dll` `1319`  

`1` `00006224` `mfc42.dll` `11B3`  

`1` `00006228` `mfc42.dll` `11C1`  

`1` `0000622C` `mfc42.dll` `11AC`  

`1` `00006230` `mfc42.dll` `095F`  

`1` `00006234` `mfc42.dll` `11AC`  

`1` `00006238` `mfc42.dll` `11AC`  

`1` `0000623C` `mfc42.dll` `1363`  

`1` `00006240` `mfc42.dll` `1360`  

`1` `00006244` `mfc42.dll` `100C`  

`1` `00006248` `mfc42.dll` `17A6`  

`1` `0000624C` `mfc42.dll` `14A1`  

`1` `00006250` `mfc42.dll` `0EA4`  

`1` `00006254` `mfc42.dll` `06BD`  

`1` `00006258` `mfc42.dll` `148C`  

`1` `0000625C` `mfc42.dll` `19D6`  

`1` `00006260` `mfc42.dll` `1A23`  

`1` `00006264` `mfc42.dll` `1150`  

`1` `00006268` `mfc42.dll` `0297`  

`1` `0000626C` `mfc42.dll` `047A`  

`1` `00006270` `mfc42.dll` `0490`  

`1` `00006274` `mfc42.dll` `015C`  

`1` `00006278` `mfc42.dll` `194E`  

`1` `0000627C` `mfc42.dll` `1972`  

`1` `00006280` `mfc42.dll` `047F`  

`1` `00006284` `mfc42.dll` `029C`  

`1` `00006288` `mfc42.dll` `0AD2`  

`1` `0000628C` `mfc42.dll` `039C`  

`1` `00006290` `mfc42.dll` `0164`  

`1` `00006294` `mfc42.dll` `1837`  

`1` `00006298` `mfc42.dll` `04A9`  

`1` `0000629C` `mfc42.dll` `04B0`  

`1` `000062A0` `mfc42.dll` `0219`  

`1` `000062A4` `mfc42.dll` `1A95`  

`1` `000062A8` `mfc42.dll` `0A52`  

`1` `000062AC` `mfc42.dll` `0B02`  

`1` `000062B0` `mfc42.dll` `0B04`  

`1` `000062B4` `mfc42.dll` `02F3`  

`1` `000062B8` `mfc42.dll` `01D6`  

`1` `000062BC` `mfc42.dll` `0451`  

`1` `000062C0` `mfc42.dll` `1847`  

`1` `000062C4` `mfc42.dll` `18BE`  

`1` `000062C8` `mfc42.dll` `03BD`  

`1` `000062CC` `mfc42.dll` `0959`  

`1` `000062D0` `mfc42.dll` `0318`  

`1` `000062D4` `mfc42.dll` `025B`  

`1` `000062D8` `mfc42.dll` `1540`  

`1` `000062DC` `mfc42.dll` `15C4`  

`1` `000062E0` `mfc42.dll` `0AF1`  

`1` `000062E4` `mfc42.dll` `18EF`  

`1` `000062E8` `mfc42.dll` `0B63`  

`1` `000062EC` `mfc42.dll` `03AB`  

`1` `000062F0` `mfc42.dll` `03AC`  

`1` `000062F4` `mfc42.dll` `035B`  

`1` `000062F8` `mfc42.dll` `0337`  

`1` `000062FC` `mfc42.dll` `0111`  

`1` `00006300` `mfc42.dll` `020C`  

`1` `00006304` `mfc42.dll` `0317`  

`1` `00006308` `mfc42.dll` `081D`  

`1` `0000630C` `mfc42.dll` `020B`  

`1` `00006310` `mfc42.dll` `184F`  

`1` `00006314` `mfc42.dll` `106C`  

`1` `00006318` `mfc42.dll` `188B`  

`1` `0000631C` `mfc42.dll` `035A`  

`1` `00006320` `mfc42.dll` `02C6`  

`1` `00006324` `mfc42.dll` `1045`  

`1` `00006328` `mfc42.dll` `019C`  

`1` `0000632C` `mfc42.dll` `10B3`  

`1` `00006330` `mfc42.dll` `1479`  

`1` `00006334` `mfc42.dll` `0DF6`  

`1` `00006338` `mfc42.dll` `1148`  

`1` `0000633C` `mfc42.dll` `0D4A`  

`1` `00006340` `mfc42.dll` `14AA`  

`1` `00006344` `mfc42.dll` `112C`  

`1` `00006348` `mfc42.dll` `06F0`  

`1` `0000634C` `mfc42.dll` `0BA6`  

`1` `00006350` `mfc42.dll` `096B`  

`1` `00006354` `mfc42.dll` `1A97`  

`1` `00006358` `mfc42.dll` `0E1A`  

`1` `0000635C` `mfc42.dll` `0628`  



## FThunk: `00006364` NbFunc: `00000011`

1 `00006364` `msvcp60.dll` `0339`

1 `00006368` `msvcp60.dll` `0407`

1 `0000636C` `msvcp60.dll` `03F3`

1 `00006370` `msvcp60.dll` `034B`

1 `00006374` `msvcp60.dll` `032E`

1 `00006378` `msvcp60.dll` `0393`

1 `0000637C` `msvcp60.dll` `03F9`

1 `00006380` `msvcp60.dll` `0662`

1 `00006384` `msvcp60.dll` `041D`

1 `00006388` `msvcp60.dll` `00EA`

1 `0000638C` `msvcp60.dll` `0048`

1 `00006390` `msvcp60.dll` `0421`

1 `00006394` `msvcp60.dll` `0216`

1 `00006398` `msvcp60.dll` `0218`

1 `0000639C` `msvcp60.dll` `052A`

1 `000063A0` `msvcp60.dll` `0219`

1 `000063A4` `msvcp60.dll` `0406`

## FThunk: `000063AC` NbFunc: `00000018`

1 `000063AC` `msvcrt.dll` `0044` `_CxxThrowException`

1 `000063B0` `msvcrt.dll` `01B8` `_setmbcp`

1 `000063B4` `msvcrt.dll` `01A0` `_purecall`

1 `000063B8` `msvcrt.dll` `0058` `__dllonexit`

1 `000063BC` `msvcrt.dll` `00BA` `_controlfp`

1 `000063C0` `msvcrt.dll` `0193` `_onexit`

1 `000063C4` `msvcrt.dll` `00CE` `_except_handler3`

1 `000063C8` `msvcrt.dll` `0084` `__set_app_type`

1 `000063CC` `msvcrt.dll` `0072` `__p__fmode`

1 `000063D0` `msvcrt.dll` `0009` `??0exception@@QAE@XZ`

1 `000063D4` `msvcrt.dll` `000A` `??1__non_rtti_object@@UAE@XZ`

1 `000063D8` `msvcrt.dll` `0008` `??0exception@@QAE@ABV0@@Z`

1 `000063DC` `msvcrt.dll` `004C` `__CxxFrameHandler`

1 `000063E0` `msvcrt.dll` `006D` `__p__commode`

1 `000063E4` `msvcrt.dll` `00A0` `_adjust_fdiv`

1 `000063E8` `msvcrt.dll` `0086` `__setusermatherr`

1 `000063EC` `msvcrt.dll` `011A` `_initterm`

1 `000063F0` `msvcrt.dll` `005B` `__getmainargs`

1 `000063F4` `msvcrt.dll` `0092` `_acmdln`

1 `000063F8` `msvcrt.dll` `025F` `exit`

1 `000063FC` `msvcrt.dll` `004B` `_XcptFilter`

1 `00006400` `msvcrt.dll` `00D7` `_exit`

1 `00006404` `msvcrt.dll` `000E` `??1type_info@@UAE@XZ`

1 `00006408` `msvcrt.dll` `02AE` `memmove`

## FThunk: `00006410` NbFunc: `0000000C`

1 `00006410` `user32.dll` `01E4` `PostMessageA`

1 `00006414` `user32.dll` `0191` `IsIconic`

1 `00006418` `user32.dll` `014A` `GetSystemMetrics`

1 `0000641C` `user32.dll` `00AC` `DrawIcon`

1 `00006420` `user32.dll` `019A` `KillTimer`

1 `00006424` `user32.dll` `0219` `SendMessageA`

1 `00006428` `user32.dll` `0258` `SetTimer`

1 `0000642C` `user32.dll` `01E6` `PostQuitMessage`

1 `00006430` `user32.dll` `01A3` `LoadIconA`

1 `00006434` `user32.dll` `0161` `GetWindowRect`

1 `00006438` `user32.dll` `00BA` `EnableWindow`

1 `0000643C` `user32.dll` `00F4` `GetClientRect`

## 完整输入表

`004063AC` `780070D4` `msvcrt._CxxThrowException`

`004063B0` `7800162D` `msvcrt._setmbcp`

`004063B4` `7800BC73` `msvcrt._purecall`

`004063B8` `7801E504` `msvcrt.__dllonexit`

`004063BC` `78001EC9` `msvcrt._controlfp`

`004063C0` `7801E417` `msvcrt._onexit`

`004063C4` `7800BD6A` `msvcrt._except_handler3`

`004063C8` `7800776E` `msvcrt.__set_app_type`

`004063CC` `78007FD7` `msvcrt.__p__fmode`

`004063D0` `780092E9` `msvcrt.exception::exception`

`004063D4` `780074BC` `msvcrt.exception::~exception`

`004063D8` `78009337` `msvcrt.exception::exception`

`004063DC` `78007191` `msvcrt.__CxxFrameHandler`

`004063E0` `78007FB9` `msvcrt.__p__commode`

`004063E4` `7803A670` `offset` `msvcrt._adjust_fdiv`

`004063E8` `78007778` `msvcrt.__setusermatherr`

`004063EC` `7800119B` `msvcrt._initterm`

`004063F0` `78007EDA` `msvcrt.__getmainargs`

`004063F4` `7803A020` `offset` `msvcrt._acmdln`

`004063F8` `78007C53` `msvcrt.exit`

`004063FC` `7800C03E` `msvcrt._XcptFilter`

`00406400` `78007CDA` `msvcrt._exit`

`00406404` `7800756F` `msvcrt.type_info::~type_info`

`00406408` `7800FFB4` `msvcrt.memmove`

`0040640C` `00000000`

`00406410` `77DF58B4` `USER32.PostMessageA`

`00406414` `77DF4C88` `USER32.IsIconic`

`00406418` `77DF5B67` `USER32.GetSystemMetrics`

`0040641C` `77DF6BE4` `USER32.DrawIcon`

`00406420` `77DF382E` `USER32.KillTimer`

`00406424` `77DF5366` `USER32.SendMessageA`

`00406428` `77DF356A` `USER32.SetTimer`

`0040642C` `77E13B6A` `USER32.PostQuitMessage`

`00406430` `77E070DA` `USER32.LoadIconA`

`00406434` `77DF5AAE` `USER32.GetWindowRect`

`00406438` `77DF54CA` `USER32.EnableWindow`

`0040643C` `77DF36EB` `USER32.GetClientRect`

## 内存映射

地址 大小 ? 宿主 区段 包含 类型 访问 初始访问

`00400000` `00001000` `dumped_` `PE header` `Imag 01001002` `R` `RWE`

`00401000` `00005000` `dumped_` `code` `Imag 01001002` `R` `RWE`

`00406000` `00003000` `dumped_` `data` `Imag 01001002` `R` `RWE`

`00409000` `00001000` `dumped_` `Imag 01001002` `R` `RWE`

`0040A000` `00002000` `dumped_` `resources` `Imag 01001002` `R` `RWE`

`0040C000` `00021000` `dumped_` `relocations` `Imag 01001002` `R` `RWE`

`0042D000` `00001000` `dumped_` `.mackt` `imports` `Imag 01001002` `R` `RWE`

没什么能减肥的了：）

`peid: Microsoft Visual C++`

`MISSION ALL OVER！`

## 破解总结

终于看到外挂的恐怖了，写的比破的快多了，8 月 5 号出了一版，8 月 6 号又出了一版，24 小时后又出了一版。。。。。。

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`2005-8-21`

`4dd466d0e1e0cc123afbe504da2462ca`
