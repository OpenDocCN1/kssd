
``` 
0048933A   .  E8 A12AFBFF   call 11.0043BDE0                         ;  获取磁盘序列号函数
```

再跟进此函数可知程序时根据此函数 `kernel32.GetVolumeInformationA` 来获取 c 盘的序列号。不再跟进。

``` 
0044E21D   .  8BC8          mov ecx,eax
0044E21F   .  FF15 04114000 call dword ptr ds:[<&MSVBVM60.__vbaI4Abs>;  求绝对值，磁盘序列号变正值
0044E225   .  50            push eax
0044E226   .  FF15 3C104000 call dword ptr ds:[<&MSVBVM60.__vbaStrI4>;  转换成10进制字符串
0044E22C   .  8BD0          mov edx,eax                              ;  得到机器码
0044E22E   .  8D4D C4       lea ecx,dword ptr ss:[ebp-3C]               将机器码存入ss:[0012FAD8]
.......................省略部分代码
0044E39D   > \C785 40FEFFFF>mov dword ptr ss:[ebp-1C0],0               ;
0044E3A7   >  8B4D C4       mov ecx,dword ptr ss:[ebp-3C]              ;  从0012FAD8拿回机器码
0044E3AA   .  51            push ecx                                      压栈
0044E3AB   .  FF15 84124000 call dword ptr ds:[<&MSVBVM60.__vbaR8Str>] ;  转为双精度浮点型
0044E3B1   .  D9E1          fabs                                       ;  求绝对值
0044E3B3   .  DFE0          fstsw ax                                   ;  把值送给通用寄存器AX
0044E3B5   .  A8 0D         test al,0D                                 ;  测试al中余数是否为空
0044E3B7   .  0F85 D44C0000 jnz 11.00453091                            ;  为空就玩完，跳走了
0044E3BD   .  83EC 08       sub esp,8                                  ;  把0012F77C的值减8，及esp=0012F774
0044E3C0   .  DD1C24        fstp qword ptr ss:[esp]                    ;  弹栈指令，将st0弹出
0044E3C3   .  FF15 FC124000 call dword ptr ds:[<&MSVBVM60.#614>]       ;  求平方根
0044E3C9   .  DD9D D0FEFFFF fstp qword ptr ss:[ebp-130]                ;  弹栈指令，将st0弹出
0044E3CF   .  DD05 30184000 fld qword ptr ds:[401830]                  ;  载入浮点数4141
0044E3D5   .  DC8D D0FEFFFF fmul qword ptr ss:[ebp-130]                ;  乘以4141
0044E3DB   .  DFE0          fstsw ax                                   ;  把值送给通用寄存器AX
0044E3DD   .  A8 0D         test al,0D                                 ;  测试al中余数是否为空
0044E3DF   .  0F85 AC4C0000 jnz 11.00453091                            ;  为空就玩完，跳走了
0044E3E5   .  FF15 3C134000 call dword ptr ds:[<&MSVBVM60.__vbaR8IntI4>;  取整数，得出注册码
0044E3EB   .  8945 D4       mov dword ptr ss:[ebp-2C],eax              ;  存放注册码，算法结束，和下面对应
0044E3EE   .  8D4D C4       lea ecx,dword ptr ss:[ebp-3C]              ;  下面释放内存
0044E3F1   .  FF15 68134000 call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>;  MSVBVM60.__vbaFreeStr
0044E3F7   .  8D4D B8       lea ecx,dword ptr ss:[ebp-48]
0044E3FA   .  FF15 64134000 call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>;  MSVBVM60.__vbaFreeObj
.......................省略部分代码
0044E66C   .  894A 0C       mov dword ptr ds:[edx+C],ecx             ;  访问注册表
0044E66F   .  68 40C64300   push 11.0043C640                         ; /flag
0044E674   .  68 34C64300   push 11.0043C634                         ; |REG
0044E679   .  8B55 C4       mov edx,dword ptr ss:[ebp-3C]            ; |
0044E67C   .  52            push edx                                 ; |AppName
0044E67D   .  FF15 D4124000 call dword ptr ds:[<&MSVBVM60.#689>]     ; \rtcGetSetting
0044E683   .  8945 A8       mov dword ptr ss:[ebp-58],eax
0044E686   .  C745 A0 08000>mov dword ptr ss:[ebp-60],8
0044E68D   .  8D45 A0       lea eax,dword ptr ss:[ebp-60]
0044E690   .  50            push eax
0044E691   .  8D4D 90       lea ecx,dword ptr ss:[ebp-70]
0044E694   .  51            push ecx
```

检查注册表中 `reg` 是否标记为 `true`

``` 
.......................省略部分代码
0044E809   .  8950 0C       mov dword ptr ds:[eax+C],edx             ;  访问注册表，检查是否注册
0044E80C   .  68 D0C64300   push 11.0043C6D0                         ; /Info
0044E811   .  68 E0C64300   push 11.0043C6E0                         ; |UnlockCode
0044E816   .  8B45 C4       mov eax,dword ptr ss:[ebp-3C]            ; |
0044E819   .  50            push eax                                 ; |AppName
0044E81A   .  FF15 D4124000 call dword ptr ds:[<&MSVBVM60.#689>]     ; \rtcGetSetting
0044E820   .  8945 A8       mov dword ptr ss:[ebp-58],eax
0044E823   .  C745 A0 08000>mov dword ptr ss:[ebp-60],8
0044E82A   .  8D4D A0       lea ecx,dword ptr ss:[ebp-60]
0044E82D   .  51            push ecx
0044E82E   .  8D55 90       lea edx,dword ptr ss:[ebp-70]           和上面雷同
0044E831   .  52            push edx
```

检查注册表中是否已经存放好注册码，注册信息在 `HKEY_CURRENT_USER\Software\VB and VBA Program Settings\HistoryCleaner\`

``` 
.......................省略部分代码
0044E85F   .  8D4D 90       lea ecx,dword ptr ss:[ebp-70]
0044E862   .  51            push ecx                        进栈
0044E863   .  8D55 A0       lea edx,dword ptr ss:[ebp-60]
0044E866   .  52            push edx                        进栈，开始对上面的“侦查”结果进行判断了
0044E867   .  6A 02         push 2                          2进栈
0044E869   .  FF15 68104000 call dword ptr ds:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList
0044E86F   .  83C4 0C       add esp,0C
0044E872   .  C745 FC 0C000>mov dword ptr ss:[ebp-4],0C
0044E879   .  66:833D D0424>cmp word ptr ds:[4942D0],0FFFF           ;  进行比较
0044E881      0F85 4C090000 jnz 11.0044F1D3                          ;  关键跳，爆破位置，标志位修改，
0044E887   .  C745 FC 0D000>mov dword ptr ss:[ebp-4],0D
0044E88E   .  8B45 CC       mov eax,dword ptr ss:[ebp-34]
0044E891   .  50            push eax
0044E892   .  8B4D D4       mov ecx,dword ptr ss:[ebp-2C]            ;  还记得上面算法分析时这个位置的值吗？
0044E895   .  51            push ecx
0044E896   .  FF15 3C104000 call dword ptr ds:[<&MSVBVM60.__vbaStrI4>;  MSVBVM60.__vbaStrI4  转换成10进制字符串
0044E89C   .  8BD0          mov edx,eax                              ;  注册码
0044E89E   .  8D4D C4       lea ecx,dword ptr ss:[ebp-3C]
0044E8A1   .  FF15 28134000 call dword ptr ds:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
0044E8A7   .  50            push eax
0044E8A8   .  FF15 80114000 call dword ptr ds:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp
0044E8AE   .  F7D8          neg eax
0044E8B0   .  1BC0          sbb eax,eax
0044E8B2   .  F7D8          neg eax
0044E8B4   .  F7D8          neg eax
0044E8B6   .  66:8985 CCFEF>mov word ptr ss:[ebp-134],ax
0044E8BD   .  8D4D C4       lea ecx,dword ptr ss:[ebp-3C]
0044E8C0   .  FF15 68134000 call dword ptr ds:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr
0044E8C6   .  0FBF95 CCFEFF>movsx edx,word ptr ss:[ebp-134]
0044E8CD   .  85D2          test edx,edx
0044E8CF      0F84 25010000 je 11.0044E9FA                           ;  关键跳，跳向注册成功
```

程序的算法比较简单，但是我们可以从多个方面来分析这个程序，就非常有意思。从程序启动后注册处来下手追码，随便输入假码，会弹出 "invalid unlock code. try again"，就可以从这里下手，从这里追码相对简单这里就不在详述。

这个程序没注册时，标题显示的时“Unregistered Copy”，但是从字符串中试搜索不到的，因为这个是它 `form` 的原始 `caption`，当判断注册后才会修改 `form.caption = "registered Copy"`，这样只能搜索到“registered Copy”这个字串。并且这样追码的难点是程序做了这样的设置，如果已经注册，就在注册表中将 `reg` 键值的属性写为 ture，如果没有就写为 `false`，启动时判断这个值是否为 `true`，如果不是它就不进行下一步的注册码的比较了，直接跳走，那么我们也就追不到名码了。但是启动后从注册处下手，它会对输入的假码进行直接的比较，那么也就很容易追出来。所以如果仅仅为了爆破还是从注册处追码较好。

至此，这个程序的算法就分析完了，用 `vb` 还原一下就是：`注册码 = Int(Sqr(Abs(CDbl(机器码))) * 4141)`

`vb` 注册机源码：

``` 
Option Explicit

Private Declare Function GetVolumeInformation Lib "kernel32" Alias "GetVolumeInformationA" (ByVal lpRootPathName As String, ByVal lpVolumeNameBuffer As String, ByVal nVolumeNameSize As Long, lpVolumeSerialNumber As Long, lpMaximumComponentLength As Long, lpFileSystemFlags As Long, ByVal lpFileSystemNameBuffer As String, ByVal nFileSystemNameSize As Long) As Long

Private Sub Drive_Change()

    Dim Volume As String, SysName As String
    Dim SerialNum As Long, SysFlags As Long, ComponentLength As Long, Res As Long
    Dim DrvName As String

    Volume = String(256, 0)
    SysName = String(256, 0)
    DrvName = "c:\"

    Res = GetVolumeInformation(DrvName, Volume, 255, SerialNum, ComponentLength, SysFlags, SysName, 255)

    If Res = 0 Then

        Text1.Text = "不能得到磁盘信息"

    Else
```
