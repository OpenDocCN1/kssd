
# [Asprotect]-Aston 1.92

程序用`Asprotect`加密,有人提供了key,于是试了下有key与无key的脱壳,受益菲浅.

经过进一步的功能分析,发现功能部分还有暗桩:一个`CRC`验证,一个`key`解码及1处`Stolen code`.

软件名称： `Aston 1.92`

下载页面： `09cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6*7K9X3c8Q4x3X3g2Z5j5h3&6*7K9h3k6&6i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0%4L8X3k6A6L8r3g2K6i4K6u0r3b7i4y4@1L8$3&6Q4y4h3j5I4z5e0u0Q4x3X3g2*7K9i4l9`.

软件大小： 3.53 MB

软件语言： 英文

软件类别： 国外软件 / 特别版 / 系统模拟

应用平台： Win2003, WinXp, Win2000, Nt, WinMe, Win9x

推荐等级： \*\*\*\*\*

软件介绍： `Aston` 是一个容易使用、执行速度快的Windows外壳，他不需要高速的电脑硬件，执行起来也非常稳定，对于内存及系统资源的需求较少，让电脑系统能够有更好的性能表现。安装完后无需重新启动，只要注销一次即可更换你的桌面。

压缩包内附带 `serial` 信息。

Version: `ASProtect 1.35 build 04.25 Release [Extract]`

好几个程序都加了`Asp`壳,挑其中一个试试:`A-Master.exe`.

`A-Master.exe`:

## 1.未注册进行脱壳:

### 1.1 Volx脚本运行到OEP:

```
00433154     55                push ebp
00433155     8BEC              mov ebp,esp
00433157     83C4 F4           add esp,-0C
0043315A     B8 24304300       mov eax,A-Master.00433024
0043315F     E8 ACDFFCFF       call A-Master.00401110
00433164     E8 27CAFDFF       call A-Master.0040FB90
00433169     E8 9EDEFCFF       call A-Master.0040100C
0043316E     8BC0              mov eax,eax
00433170     0000              add byte ptr ds:[eax],al
00433172     0000              add byte ptr ds:[eax],al
00433174     0000              add byte ptr ds:[eax],al
```

对比脚本发现:

`Aspr2.XX_IATfixer_v1.02.osc`          // F9后程序退出

`Asprotect 2.xx SKE OEP finder.txt`    // F9后程序正常启动

### 1.2 CRC验证

从`OEP`往下,第二个`call`进去后不久

```
00432D34     56                push esi
00432D35     53                push ebx
00432D36     E8 72000000       call A-Master.00432DAD
00432D3B     BE A64E4200       mov esi,A-Master.00424EA6
00432D40     037424 38         add esi,dword ptr ss:[esp+38]
00432D44     8D7424 04         lea esi,dword ptr ss:[esp+4]
00432D48     8D7426 F8         lea esi,dword ptr ds:[esi-8]
00432D4C     8B36              mov esi,dword ptr ds:[esi]
00432D4E     8DB40E 55E4FCFF   lea esi,dword ptr ds:[esi+ecx+FFFCE455]
00432D55     2BF1              sub esi,ecx
00432D57     8D5C11 17         lea ebx,dword ptr ds:[ecx+edx+17]
00432D5B     2BDA              sub ebx,edx
00432D5D     8B1E              mov ebx,dword ptr ds:[esi]
00432D5F     81E3 FF000000     and ebx,0FF
00432D65     BE 46504300       mov esi,A-Master.00435046
00432D6A     037424 38         add esi,dword ptr ss:[esp+38]
00432D6E     68 F2402834       push 342840F2
00432D73     8D7447 29         lea esi,dword ptr ds:[edi+eax*2+29]
00432D77     8D740E D7         lea esi,dword ptr ds:[esi+ecx-29]
00432D7B     2BF1              sub esi,ecx
00432D7D     5E                pop esi
00432D7E     8D9C0B 26BED7CB   lea ebx,dword ptr ds:[ebx+ecx+CBD7BE26]
00432D85     2BD9              sub ebx,ecx
00432D87     03DE              add ebx,esi
00432D89     0BDB              or ebx,ebx
00432D8B    /0F84 24000000     je A-Master.00432DB5                   ; CRC,jmp 00432DB5
00432D91     8D6C75 F3         lea ebp,dword ptr ss:[ebp+esi*2-D]
00432D95     8D740B 5A         lea esi,dword ptr ds:[ebx+ecx+5A]
00432D99     2BF1              sub esi,ecx
00432D9B     C1CF B7           ror edi,0B7
00432D9E     C1CF D3           ror edi,0D3
00432DA1     64:FF35 00000000  push dword ptr fs:[0]
00432DA8     E9 08000000       jmp A-Master.00432DB5
00432DAD     337424 08         xor esi,dword ptr ss:[esp+8]
00432DB1     C1D6 6D           rcl esi,6D
00432DB4     C3                retn
00432DB5     5B                pop ebx
00432DB6     5E                pop esi
00432DB7     E9 88000000       jmp A-Master.00432E44
```

### 1.3 异常

步过`00432E6F`    `call A-Master.004328D8` 发生异常,地址`00000000`不易读取:

跟进后来到:

```
00432AAB     B8 482B4300       mov eax,A-Master.00432B48              ; ASCII "ColorClass"
00432AB0     8946 24           mov dword ptr ds:[esi+24],eax
00432AB3     56                push esi
00432AB4     E8 87EAFCFF       call <jmp.&user32.RegisterClassA>
00432AB9     66:85C0           test ax,ax
00432ABC     74 40             je short A-Master.00432AFE
00432ABE     E9 10000000       jmp A-Master.00432AD3                  ; jmp 00432AFC
00432AC3   ^ 7C 97             jl short A-Master.00432A5C
00432AC5     E4 08             in al,8
00432AC7     33EE              xor ebp,esi
00432AC9     2E:0921           or dword ptr cs:[ecx],esp
00432ACC   - E9 A1B36FAA       jmp AAB2DE72
00432AD1     90                nop
00432AD2     90                nop
00432AD3     A1 68604300       mov eax,dword ptr ds:[436068]
00432AD8     8B00              mov eax,dword ptr ds:[eax]
00432ADA     50                push eax
00432ADB     A1 545A4300       mov eax,dword ptr ds:[435A54]
00432AE0     8B00              mov eax,dword ptr ds:[eax]
00432AE2     50                push eax
00432AE3     E8 30E7FCFF       call <jmp.&kernel32.GetProcAddress>
00432AE8     FFD0              call eax                               ; 这里eax=0,异常
00432AEA     8B15 E05F4300     mov edx,dword ptr ds:[435FE0]          ; A-Master.004484D0
00432AF0     8902              mov dword ptr ds:[edx],eax
00432AF2     A1 E05F4300       mov eax,dword ptr ds:[435FE0]
00432AF7     8338 00           cmp dword ptr ds:[eax],0
00432AFA     74 02             je short A-Master.00432AFE
00432AFC     B3 01             mov bl,1
00432AFE     8BC3              mov eax,ebx
00432B00     5F                pop edi
00432B01     5E                pop esi
00432B02     5B                pop ebx
00432B03     C3                retn
```

跟踪发现原程序从`00432AD3`到`00432AFA`,堆栈没有变化,寄存器只有`eax`与`edx`有变化,就是`ds:[435FE0]`保存的地址=`004484D0`;整个过程对`ds:[004484D0]`进行赋值并比较是否为0,原程序在`00432AFA`跳转未实现.

而`eax`在`00432AFE`马上被`ebx`填充,跟踪`edx`,发现在下面这行代码处被枪毙:

```
00432F8B     E8 F0A0FEFF       call A-Master.0041D080                ; 跟进
0041D08B     33D2              xor edx,edx                           ; A-Master.004484D0
```

因此可以说,这一段从`00432AD3`到`00432AFA`无用处,于是修改`00432ABE`为:

`00432ABE     E9 10000000       jmp A-Master.00432AD3                 ; jmp 00432AFC`

或者可以修改`00432AD3`为:

```
00432AD3     B8 01000000       mov eax,1D                            ; 原程序为1D,保留
00432AD8     EB 10             jmp short A-Master.00432AEA
```

### 1.4 Stolen code 1:

修改了上面2次,`F9`,异常,`0148042B`不易读取:

```
0043009B     E8 3014FDFF       call <jmp.&user32.InvalidateRect>
004300A0   - E9 86030501       jmp 0148042B
```

跟踪原程序得到`code`:

```
0043009B     E8 3014FDFF       call <jmp.&user32.InvalidateRect>
004300A0     8BC3              mov eax,ebx
004300A2     5B                pop ebx
004300A3     8BE5              mov esp,ebp
004300A5     5D                pop ebp
004300A6     C2 1000           retn 10
```

### 1.5 程序退出:自校验?

`F9`,程序退出,从`00432E6F`往下继续跟踪:

```
00432F9A     A1 105D4300       mov eax,dword ptr ds:[435D10]
00432F9F     8038 00           cmp byte ptr ds:[eax],0
00432FA2     75 16             jnz short A-Master.00432FBA           ; 跳转实现,over
```

对`ds:[435D10]`下硬件访问断点,重新加载:

第一次写入0:

```
00432C75     A1 105D4300       mov eax,dword ptr ds:[435D10]
00432C7A     C600 00           mov byte ptr ds:[eax],0
```

第二次写入1:

```
00432E44     E8 BFF9FCFF       call A-Master.00402808                ; 需要修改赋值
00432E49     84C0              test al,al
00432E4B     74 07             je short A-Master.00432E54            ; 或者修改这里
00432E4D     E8 F645FDFF       call A-Master.00407448
00432E52     EB 11             jmp short A-Master.00432E65           ; 要在这里跳转
00432E54     833E 00           cmp dword ptr ds:[esi],0
00432E57     74 04             je short A-Master.00432E5D
00432E59     33C0              xor eax,eax
00432E5B     8906              mov dword ptr ds:[esi],eax
00432E5D     A1 105D4300       mov eax,dword ptr ds:[435D10]
00432E62     C600 01           mov byte ptr ds:[eax],1
00432E65     E8 1A3BFDFF       call A-Master.00406984
00432E6A     E8 919FFEFF       call A-Master.0041CE00
00432E6F     E8 64FAFFFF       call A-Master.004328D8
00432E74     84C0              test al,al
00432E76     0F84 48010000     je A-Master.00432FC4
```

若要避开1的赋值,溯源,需要修改`00432E4B`的跳转,或者修改`00432E44`的`call`里面的值.

跟进:`00432E44     E8 BFF9FCFF       call A-Master.00402808`

...
