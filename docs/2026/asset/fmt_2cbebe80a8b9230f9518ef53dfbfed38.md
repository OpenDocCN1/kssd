
# [原创]emu8086注册算法分析及KeyGen实现

【文章标题】: emu8086注册算法分析及KeyGen实现

【文章作者】: NoAir

【软件名称】: emu8086 v4.08

【下载地址】: 见附件

【保护方式】: 有点戏剧性

【编写语言】: Visual Basic

【软件介绍】: 一款优秀的8086汇编IDE,支持可视化调试，内置FASM

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

仅以此文纪念一下emu8086,现在官网不能打开了,软件也不更新了,用emu8086编写16位汇编程序挺方便的,图形化的操作模式，支持调试，还有一些模板以及虚拟设备供操作。我把emu8086 v4.08和KeyGen及src都上传到了本地。

写完KeyGen之后才发现以前有位朋友发过一篇关于emu8086破解的文章，但是这不影响本文的分析，本文主要分析emu8086另外一种注册算法，有兴趣的朋友就当温习一下VB的逆向，走马观花一下吧。

如何找关键点,这个不用说了,直接搜索字符串,然后回溯就看得见了,这个可以参考一下gudujsk的文章：

http://bbs.pediy.com/showthread.php?t=119741

下面步入正题，为什么说是戏剧性呢，看看下面的代码就知道：

```
005DA83A   .  test edx,edx
005DA83C   .  je short emu8086.005DA850                ; 戏剧性的je,如果前3位是112
005DA83E   .  mov dword ptr ss:[ebp-4],4
005DA845   .  mov word ptr ss:[ebp-24],0FFFF           ; 返回值为-1
005DA84B   .  jmp emu8086.005DAAA5                     ; 注册成功
```

分析完戏剧性的代码后，下面开始分析另一种注册算法：

```
005DAA12   .  mov dword ptr ss:[ebp-4],10
005DAA19   .  mov ecx,dword ptr ss:[ebp+C]
005DAA1C   .  mov edx,dword ptr ds:[ecx]
005DAA1E   .  mov dword ptr ss:[ebp-64],edx
005DAA21   .  mov dword ptr ss:[ebp-6C],8
005DAA28   .  mov eax,dword ptr ss:[ebp+10]
005DAA2B   .  push eax
005DAA2C   .  mov ecx,dword ptr ss:[ebp+8]            ;  用户名
005DAA2F   .  mov edx,dword ptr ds:[ecx]
005DAA31   .  push edx
005DAA32   .  lea eax,dword ptr ss:[ebp-3C]
005DAA35   .  push eax
005DAA36   .  call emu8086.005DAAF0                   ;  注册码计算
```

上面有一行 `mov dword ptr ss:[ebp-4],10`

`Dim state as integer`

`state=10`

作者代码中频频出现类似的赋值语句，估计是用于判断语句执行到哪一行了吧，类似的语句可以全部忽略了。

进入`5DAAF0`，

```
005DAAF0   $  push ebp
005DAAF1   .  mov ebp,esp
005DAAF3   .  sub esp,18
005DAAF6   .  push <jmp.&MSVBVM60.\_\_vbaExceptHandler>      ;  SE 处理程序安装
............................
005DAB35   .  mov edx,dword ptr ss:[ebp+C]
005DAB38   .  lea ecx,dword ptr ss:[ebp-38]
005DAB3B   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCopy>] ;  初始化name=用户名
............................
005DAB57   .  mov edx,emu8086.00450CA4                     ;  abcdefghijklmnopqrstuvwxyz
005DAB5C   .  lea ecx,dword ptr ss:[ebp-5C]
005DAB5F   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCopy>] ;  初始化str1
005DAB6C   .  mov edx,emu8086.00450CE0                     ;  qw10pasdfghjklzxcvbnmertyu
005DAB71   .  lea ecx,dword ptr ss:[ebp-28]
005DAB74   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCopy>] ;  初始化str2
005DAB81   .  mov edx,emu8086.00450D1C                     ;  z9rtasdf01823asjfsd1234346gfhplmasdr613412qwerx
005DAB86   .  lea ecx,dword ptr ss:[ebp-2C]
005DAB89   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCopy>] ;  初始化str3
005DAB96   .  mov edx,emu8086.00450D80                     ;  jkaserkkn837c3frtqzx
005DAB9B   .  lea ecx,dword ptr ss:[ebp-50]
005DAB9E   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCopy>] ;  初始化str4
005DABAB   .  mov edx,emu8086.004376EC
005DABB0   .  lea ecx,dword ptr ss:[ebp-4C]
005DABB3   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCopy>] ;  初始化key为空字符串
...........................
005DABD9   .  push ecx
005DABDA   .  lea edx,dword ptr ss:[ebp-6C]
005DABDD   .  push edx
005DABDE   .  call dword ptr ds:[<&MSVBVM60.rtcUpperCaseVar>]  ;  用户名转换成大写，保存到[ebp-58]
...........................
005DABF0   .  lea ecx,dword ptr ss:[ebp-58]
005DABF3   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrMove>] ;  MSVBVM60.\_\_vbaStrMove
005DAC09   .  lea ecx,dword ptr ss:[ebp-58]                ; &nbsp
005DAC0C   .  push ecx
005DAC0D   .  call emu8086.00548830                        ;  用户名翻转
{
    00548875   .  push eax                                     ; 用户名
    00548876   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaLenBstr>] ; 得到用户名长度,用户名是用unicode编码的
    0054887C   .  mov ecx,eax
    0054887E   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaI2I4>]    ; 用户名长度转换成long型
    00548884   .  mov esi,eax
    00548886   >  mov eax,1
    0054888B   .  cmp si,ax
    0054888E   .  jl short emu8086.005488F6
    .......................................
    005488B4   .  call dword ptr ds:[<&MSVBVM60.rtcMidCharVar>];截取最后第i字符，i初始为用户名长度，循环自减
    ;esi--
    005488F4   .  jmp short emu8086.00548886
}
005DAC24   .  mov word ptr ss:[ebp-24],1                   ;  i=1
005DAC31   .  mov word ptr ss:[ebp-34],1                   ;  j=1
005DAC3E   .  mov word ptr ss:[ebp-54],0                   ;  flag 初始化 0
005DAC4B   .  movsx esi,word ptr ss:[ebp-24]               ;  i 循环体
005DAC4F   .  mov edx,dword ptr ss:[ebp-58]
005DAC52   .  push edx                                     ;  用户名name
005DAC53   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaLenBstr>] ;  \_\_vbaLenBstr
005DAC59   .  xor ebx,ebx
005DAC5B   .  cmp esi,eax
005DAC5D   .  setg bl
005DAC60   .  movsx esi,word ptr ss:[ebp-34]
005DAC64   .  mov eax,dword ptr ss:[ebp-50]
005DAC67   .  push eax                                     ; str4
005DAC68   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaLenBstr>] ; \_\_vbaLenBstr
005DAC6E   .  xor ecx,ecx
005DAC70   .  cmp esi,eax
005DAC72   .  setg cl
005DAC75   .  or ebx,ecx
005DAC77   .  test ebx,ebx
005DAC79   .  jnz emu8086.005DAF65                         ; for (i=1;i<=user\_len && j<=0x14;i++)
{
005DACCE   .  call dword ptr ds:[<&MSVBVM60.#632>]           ;  rtcMidCharVar截取name中第i个字符
005DACEA   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaInStrVar>]  ;  查找str1中索引位置u\_index
005DACF0   .  push eax
005DACF1   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaI2Var>]     ;  MSVBVM60.\_\_vbaI2Var
005DAD21   .  jle emu8086.005DAF02                           ;  u\_index小于0，说明没搜索到 因此不是字母
005DAD2E   .  cmp word ptr ss:[ebp-54],1                     ;  判断flag是否为1
005DAD33   .  jnz emu8086.005DADE4
;if flag==0
{
005DAD88   .  call dword ptr ds:[<&MSVBVM60.#632>]           ; rtcMidCharVar截取str2中第u\_index个字符
;接下来保存字符到key中，并修改flag为1
005DAE58   .  mov edx,eax                                    ;  第u\_index个字符
005DAE5A   .  lea ecx,dword ptr ss:[ebp-4C]
005DAE5D   .  call dword ptr ds:[<&MSVBVM60.\_\_vbaStrMove>]   ;  保存key中
005DAE84   .  mov word ptr ss:[ebp-54],1                     ;  flag 赋值为1
}
;else
{
........   .  call dword ptr ds:[<&MSVBVM60.#632>]           ; rtcMidCharVar截取str3中第u\_index个字符
;接下来保存字符到key中，并修改flag为0
}
005DAE91   .  mov dx,word ptr ss:[ebp-34]                    ;  j++
005DAE95   .  add dx,1
005DAE9F   .  mov word ptr ss:[ebp-34],dx
005DAEAA   .  lea eax,dword ptr ss:[ebp-28]
005DAEAD   .  push eax
005DAEAE   .  call <emu8086.move>                            ;  move(str2),move函数的功能是把字符串尾字符移动到首位
005DAEC5   .  lea ecx,dword ptr ss:[ebp-2C]
005DAEC8   .  push ecx
005DAEC9   .  call <emu8086.move>                            ;  move(str3)
005DAEE0   .  cmp word ptr ss:[ebp-54],1                     ;  flag为1
005DAEE5   .  jnz short emu8086.005DAF02
{
005DAEEE   .  lea edx,dword ptr ss:[ebp-28]
005DAEF1   .  push edx
005DAEF2   .  call <emu8086.move>                            ;  move(str2)
}
005DAF09   .  lea eax,dword ptr ss:[ebp-50]
005DAF0C   .  push eax
005DAF0D   .  call <emu8086.move>                            ;  move(str4)
005DAF24   .  movsx ecx,word ptr ss:[ebp-54]
005DAF28   .  test ecx,ecx
005DAF2A   .  jnz short emu8086.005DAF47                     ;  flag为0
{
005DAF33   .  lea edx,dword ptr ss:[ebp-50]
005DAF36   .  push edx
005DAF37   .  call <emu8086.move>                            ;  move(str4)
}
005DAF4E   .  mov ax,word ptr ss:[ebp-24]                    ;  i ++
005DAF52   .  add ax,1
005DAF5C   .  mov word ptr ss:[ebp-24],ax
005DAF60   .^ jmp emu8086.005DAC44
;move函数(char\* str)
{
005DB234   .  call dword ptr ds:[<&MSVBVM60.#632>]           ;  rtcMidCharVar截取str最后一个字符
005DB244   .  mov edx,eax
```

