
### 汇编代码分析

```
004AF82D  add edx,edx                         ;  n1+n1  
004AF82F  imul edx,ebx  
004AF832  imul edx,ebx                        ;  s2=(n1+n1)*n1*n1  
004AF835  add eax,edx                         ;  s1+s2  
004AF837  add eax,edi                         ;  s1+s2+n1*n1  
004AF839  mov dword ptr ss:[ebp-2C],eax       ;  s3=(n1*(n1+1))²  
004AF83C  fild dword ptr ss:[ebp-2C]  
004AF83F  fdiv dword ptr ds:[4AF994]          ;  s3/4  
004AF845  fadd dword ptr ds:[4AF998]          ;  s0=s3/4+3  
004AF84B  faddp st(1),st                      ;  s0+t0  
004AF84D  fstp tbyte ptr ss:[ebp-38]  
004AF850  wait  
004AF851  add esi,3                           ;  n2+3  
004AF854  mov dword ptr ss:[ebp-3C],esi  
004AF857  fild dword ptr ss:[ebp-3C]  
004AF85A  fsqrt                               ;  t1=sqr(n2+3)  
004AF85C  fmul dword ptr ds:[4AF99C]          ;  r0=t1*12  
004AF862  fld tbyte ptr ss:[ebp-38]  
004AF865  fsubrp st(1),st                     ;  s0+t0-r0  
004AF867  fcomp dword ptr ds:[4AF998]         ;  s0+50-r0=3?  
004AF86D  fstsw ax  
004AF86F  sahf  
004AF870  je short dump_.004AF87F  
```

……下面还对第 3、4 位进行了计算，算法与第 1、2 位相同，略过

### 算法总结

这个算法是一个方程式，还用到了浮点数，算法总结为：

设：（`n1`,`n2`,`n3`,`n4` 分别代表 `sn2/3` 的各位数字）

`t1 = sqrt(n2+3)`, `t2 = n1*(n1+1)`

求：

`t1*12 = t2*t2/4 + t1*t1 + t1*t2`

方程式的正整数 0－9 的解。

注意到 `sqrt(n2+3)` 为整数，可得知 `n2` 值必须为 1 或 6，进而计算出唯一解：`2626`。

这里又得到了注册码 7－10 位必须为：`2626*3＝7878`

### 关键 CALL 分析

关键 `CALL④` 只是对用户名进行了简单的字符替换，然后判断是否与注册码第三节 `sn3` 相同。字符替换表为：（只统计了数字和字母）

```
0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
I2M4K6N89:BCDEFGHIJKLMNOPQRSTUVWXYZ[bcdefghijklmnopqrstuvwxyz{
```

算法是大部分字符都是 `ASC` 码＋1，只有数字中的偶数有个小变换。

关键 `CALL⑤` 则是程序预留的后门，判断注册码第三节 `sn3` 是否与 `61686902` 字符替换后的 `N2N9N:IM` 相同。从这里可以得到程序的通用注册码：`4792597878N2N9N:IM`

### 三、程序用到的配置文件

1. 与程序同目录下的 `setting.ini`：保存程序设置的隐藏分区信息。
2. `C:\recycle\set.ini`：保存程序的配置。
3. `C:\recycle\mpan.dat`：程序的运行密码经过加密处理后保存在这里。
4. `C:\recycle\mp.dat`：保存未注册时的密盘打开次数。
5. `C:\WINDOWS\system32\driveset.dll`：保存程序的运行次数（未注册时）以及注册的用户名和注册码。
6. `X:\recycle\S-1-5-21-1060284298-811497611-11778920086-500\com1.{21ec2020-3aea-1069-a2dd-08002b30309d}\data\panfile2.dll`：保存当前分区中的密盘信息，以及经过加密处理的密盘打开密码。

### 四、程序的运行密码和密盘打开密码

调试程序可以发现程序的运行密码及密盘打开密码都只是简单的字符替换处理，于是设置一个所有字符的密码后就能得到对应的字符替换表。

运行密码替换表为：

```
0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
89:;<=>?@AIJKLMNOPQRSTUVWXYZ[\]^_\`abijklmnopqrstuvwxyz{|}~
```

调试程序发现加密算法只是将各位 `ASC` 码加 8 后保存。

密盘打开密码替换表与上相同。

### 五、建立特殊文件夹

```
004945FE  push dump_.00494A60                   ;  ASCII "recycle"
00494603  lea eax,dword ptr ss:[ebp-C]
00494606  mov edx,3
0049460B  call dump_.0040444C
00494610  mov eax,dword ptr ds:[4B6114]
00494615  mov eax,dword ptr ds:[eax]
00494617  mov edx,dword ptr ss:[ebp-C]
0049461A  call dump_.004AEE5C                   ;  建立 recycle
0049461F  push dword ptr ss:[ebp-20]
00494622  push dump_.00494A54
00494627  push dump_.00494A60                   ;  ASCII "recycle"
0049462C  push dump_.00494A70
00494631  push dump_.00494AAC                   ;  ASCII "com1.{21ec2020-3aea-1069-a2dd-08002b30309d}"
00494636  lea eax,dword ptr ss:[ebp-18]
00494639  mov edx,5
0049463E  call dump_.0040444C
00494643  lea edx,dword ptr ss:[ebp-44]
00494646  mov eax,dword ptr ss:[ebp-18]
00494649  call dump_.00408DE4
0049464E  mov eax,dword ptr ss:[ebp-44]
00494651  call dump_.00408AF0                   ;  建立 S-1-5-21-1060284298-811497611-11778920086-500
00494656  xor eax,eax
00494658  push ebp
00494659  push dump_.00494705
0049465E  push dword ptr fs:[eax]
00494661  mov dword ptr fs:[eax],esp
00494664  lea eax,dword ptr ss:[ebp-48]
00494667  mov ecx,dump_.00494A54
0049466C  mov edx,dword ptr ss:[ebp-18]
0049466F  call dump_.004043D8
00494674  mov eax,dword ptr ss:[ebp-48]
00494677  call dump_.00408ACC
0049467C  test al,al
0049467E  jnz short dump_.0049469D              ;  判断 com1.是否存在
00494680  lea eax,dword ptr ss:[ebp-4C]
00494683  mov ecx,dump_.00494A54
00494688  mov edx,dword ptr ss:[ebp-18]
0049468B  call dump_.004043D8
00494690  mov eax,dword ptr ss:[ebp-4C]
00494693  call <dump_.CreateSpecialDir>         ;  建立 com1.{21ec2020-3aea-1069-a2dd-08002b30309d}
00494698  call dump_.0040284C
0049469D  push dword ptr ss:[ebp-20]
004946A0  push dump_.00494A54
004946A5  push dump_.00494A60                   ;  ASCII "recycle"
004946AA  push dump_.00494AE0
004946AF  lea eax,dword ptr ss:[ebp-50]
004946B2  mov edx,4
004946B7  call dump_.0040444C
004946BC  mov eax,dword ptr ss:[ebp-50]
004946BF  call dump_.00408ACC
004946C4  test al,al
004946C6  jnz short dump_.004946F4              ;  判断 mp...\ 是否存在
004946C8  push dword ptr ss:[ebp-20]
004946CB  push dump_.00494A54
004946D0  push dump_.00494A60                   ;  ASCII "recycle"
004946D5  push dump_.00494AE0
004946DA  lea eax,dword ptr ss:[ebp-54]
004946DD  mov edx,4
004946E2  call dump_.0040444C
004946E7  mov eax,dword ptr ss:[ebp-54]
004946EA  call <dump_.CreateSpecialDir>         ;  建立 mp...\
004946EF  call dump_.0040284C
004946F4  push 64
004946F6  call <jmp.&kernel32.Sleep>
004946FB  xor eax,eax
004946FD  pop edx
004946FE  pop ecx
004946FF  pop ecx
00494700  mov dword ptr fs:[eax],edx
00494703  jmp short dump_.00494728
00494705  jmp dump_.0040381C
0049470A  mov eax,dump_.00494B20
0049470F  call dump_.004312DC
00494714  call dump_.00403B84
00494719  call dump_.00403BB4
0049471E  jmp dump_.00494929
00494723  call dump_.00403B84
00494728  lea eax,dword ptr ss:[ebp-14]
0049472B  mov ecx,dump_.00494B4C               ;  ASCII "\data"
00494730  mov edx,dword ptr ss:[ebp-18]
00494733  call dump_.004043D8
00494738  mov eax,dword ptr ss:[ebp-14]
0049473B  call dump_.00408ACC
00494740  test al,al
00494742  jnz short dump_.0049474C              ;  判断 data 是否存在
00494744  mov eax,dword ptr ss:[ebp-14]
00494747  call <dump_.CreateDir>                ;  建立 data
0049474C  push dword ptr ss:[ebp-14]
0049474F  push dump_.00494A54
00494754  lea edx,dword ptr ss:[ebp-60]
00494757  mov eax,dword ptr ss:[ebp-4]
0049475A  mov eax,dword ptr ds:[eax+32C]
00494760  call dump_.004573B4
00494765  mov eax,dword ptr ss:[ebp-60]
00494768  mov dl,byte ptr ds:[eax]
0049476A  lea eax,dword ptr ss:[ebp-5C]
0049476D  call dump_.004042B4
00494772  push dword ptr ss:[ebp-5C]
00494775  lea eax,dword ptr ss:[ebp-58]
00494778  mov edx,3
0049477D  call dump_.0040444C
00494782  mov eax,dword ptr ss:[ebp-58]
00494785  call dump_.00408ACC
0049478A  test al,al
0049478C  jnz short dump_.004947CC              ;  判断盘符文件夹是否存在
0049478E  push dword ptr ss:[ebp-14]
00494791  push dump_.00494A54
00494796  lea edx,dword ptr ss:[ebp-6C]
00494799  mov eax,dword ptr ss:[ebp-4]
0049479C  mov eax,dword ptr ds:[eax+32C]
004947A2  call dump_.004573B4
004947A7  mov eax,dword ptr ss:[ebp-6C]
004947AA  mov dl,byte ptr ds:[eax]
004947AC  lea eax,dword ptr ss:[ebp-68]
004947AF  call dump_.004042B4
004947B4  push dword ptr ss:[ebp-68]
004947B7  lea eax,dword ptr ss:[ebp-64]
004947BA  mov edx,3
004947BF  call dump_.0040444C
```
