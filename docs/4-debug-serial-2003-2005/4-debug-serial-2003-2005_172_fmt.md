
# Instyler Ex-it! 1.64

> 原文：[`www.pediy.com/kssd/pediy06/pediy6008.htm`](https://www.pediy.com/kssd/pediy06/pediy6008.htm)

Instyler Ex-it! 汉化版 1.64 简单算法分析

软件名称：`Instyler Ex-it! 汉化版 1.64`

软件简介：`instyler ex-it!` 是一个创建自解压缩文件的功能强大的开发工具。

它不同于其他自解压缩程序，`instyler ex-it!` 为你提供多个附加选项，你可以配置每个可视化的对象、使用消息框、设置不同的语言、或者创建单个文件的安装程序。用 `instyler ex-it!` 创建的自解压缩文件支持密码保护、显示许可协议、可执行文件，并包含一个内建的卸载程序。创建的自解压缩文件不显示任何广告，看上去就象是你自己创建的。

下载地址：[`antivirus.pchome.net/utility/pack/10964.html`](http://antivirus.pchome.net/utility/pack/10964.html)

此文目的：学习该软件的注册码生成方法

调试工具：`ollydbg1.09` 中文版、`W32Dasm10`、`language`

调试平台：`Windows XP` (哈哈,`ollydbg` 真好,`XP` 下也能调试了,想想吧,还有 `MP3`)

过程：

1) 运行程序，首先弹出一提示注册窗口，点击输入注册码按钮，输入注册姓名 & 注册码，确定。

弹出 "输入的注册值是无效的。旧数据已经回存。" 对话框。先关闭程序。

2) 使用 `language` 检测主程序 `Exit.exe`，没有壳。

3) 用 `W32dasm10` 反编译 `Exit.exe`，然后查找字符串 "输入的注册值是无效的。旧数据已经回存。" 双击找到的字符串，来到以下地方：

发现到出错信息的跳转：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00458590(C)
```

往上来到 `00458590`，看到上面一行的 `0045858E` 有一个比较，再上面又有一个 `CALL`

于是跟进上面的这个 `CALL`，也就是 `00458589 E8421C0000 call 0045A1D0`

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00458508(C)
|
:00458577 8D55F8              lea edx, dword ptr [ebp-08]
:0045857A 8B83B8010000        mov eax, dword ptr [ebx+000001B8]
:00458580 E8ABAFFBFF          call 00413530
:00458585 8B45F8              mov eax, dword ptr [ebp-08]
:00458588 5A                  pop edx
:00458589 E8421C0000          call 0045A1D0
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045851F(C)
|
:0045858E 3C01                cmp al, 01
:00458590 0F85C3000000        jne 00458659
:00458596 8D55FC              lea edx, dword ptr [ebp-04]
:00458599 8B83B8010000        mov eax, dword ptr [ebx+000001B8]
:0045859F E88CAFFBFF          call 00413530
:004585A4 8B55FC              mov edx, dword ptr [ebp-04]
:004585A7 A1303B4600          mov eax, dword ptr [00463B30]
:004585AC 0574040000          add eax, 00000474
:004585B1 E892B0FAFF          call 00403648
:004585B6 8D55FC              lea edx, dword ptr [ebp-04]
:004585B9 8B83BC010000        mov eax, dword ptr [ebx+000001BC]
:004585BF E86CAFFBFF          call 00413530
:004585C4 8B55FC              mov edx, dword ptr [ebp-04]
:004585C7 A1303B4600          mov eax, dword ptr [00463B30]
:004585CC 0578040000          add eax, 00000478
:004585D1 E872B0FAFF          call 00403648
:004585D6 33C9                xor ecx, ecx
:004585D8 B201                mov dl, 01
:004585DA B860DC4300          mov eax, 0043DC60
:004585DF E8D85CFEFF          call 0043E2BC
:004585E4 8BF0                mov esi, eax
:004585E6 BA01000080          mov edx, 80000001
:004585EB 8BC6                mov eax, esi
:004585ED E8B657FEFF          call 0043DDA8
:004585F2 A1303B4600          mov eax, dword ptr [00463B30]
:004585F7 8B8074040000        mov eax, dword ptr [eax+00000474]
:004585FD 50                  push eax
```

```
* Possible StringData Ref from Code Obj -> "Username"
                    |
:004585FE B99C864500          mov ecx, 0045869C
```

```
* Possible StringData Ref from Code Obj -> "Software\instyler\ex-it!\RegData"
                    |
:00458603 BAB0864500          mov edx, 004586B0
:00458608 8BC6                mov eax, esi
:0045860A E8C15DFEFF          call 0043E3D0
:0045860F A1303B4600          mov eax, dword ptr [00463B30]
:00458614 8B8078040000        mov eax, dword ptr [eax+00000478]
:0045861A 50                  push eax
```

```
* Possible StringData Ref from Code Obj -> "Userkey"
                    |
:0045861B B9DC864500          mov ecx, 004586DC
```

```
* Possible StringData Ref from Code Obj -> "Software\instyler\ex-it!\RegData"
                    |
:00458620 BAB0864500          mov edx, 004586B0
:00458625 8BC6                mov eax, esi
:00458627 E8A45DFEFF          call 0043E3D0
:0045862C 6A40                push 00000040
```

```
* Possible StringData Ref from Code Obj -> "谢谢你注册"
                    |
:0045862E B9E4864500          mov ecx, 004586E4
```

```
* Possible StringData Ref from Code Obj -> "谢谢你注册 instyler ex-it!"
                    |
:00458633 BA00874500          mov edx, 00458700
:00458638 A130364600          mov eax, dword ptr [00463630]
:0045863D E84ACCFCFF          call 0042528C
:00458642 A1143B4600          mov eax, dword ptr [00463B14]
:00458647 80783700            cmp byte ptr [eax+37], 00
:0045864B 7416                je 00458663
:0045864D A1143B4600          mov eax, dword ptr [00463B14]
:00458652 E835B1FCFF          call 0042378C
:00458657 EB0A                jmp 00458663
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00458590(C)
|
```

```
* Possible StringData Ref from Code Obj -> "输入的注册值是无效的。旧数据已经回存。"
                    |
:00458659 B8B0874500          mov eax, 004587B0
:0045865E E8E98BFDFF          call 0043124C
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0045855F(C), :0045864B(C), :00458657(U)
|
:00458663 8BC3                mov eax, ebx
:00458665 E822B1FCFF          call 0042378C
:0045866A 33C0                xor eax, eax
:0045866C 5A                  pop edx
:0045866D 59                  pop ecx
:0045866E 59                  pop ecx
:0045866F 648910              mov dword ptr fs:[eax], edx
:00458672 688C864500          push 0045868C
```

4) 使用 `ollydbg` 载入 `Exit.exe` 在 `0045A1D0` 处设置断点，`F9` 运行程序，输入 `name` and `regcode` 点 `OK` 中断于此：

```
* Referenced by a CALL at Addresses:
|:00458589   , :0045C970   
|
:0045A1D0 55                  push ebp
:0045A1D1 8BEC                mov ebp, esp
:0045A1D3 33C9                xor ecx, ecx
:0045A1D5 51                  push ecx
:0045A1D6 51                  push ecx
:0045A1D7 51                  push ecx
:0045A1D8 51                  push ecx
:0045A1D9 51                  push ecx
:0045A1DA 51                  push ecx
:0045A1DB 51                  push ecx
:0045A1DC 53                  push ebx
:0045A1DD 56                  push esi
:0045A1DE 57                  push edi
:0045A1DF 8955F8              mov dword ptr [ebp-08], edx
:0045A1E2 8945FC              mov dword ptr [ebp-04], eax
:0045A1E5 8B45FC              mov eax, dword ptr [ebp-04]
:0045A1E8 E83397FAFF          call 00403920
:0045A1ED 8B45F8              mov eax, dword ptr [ebp-08]
:0045A1F0 E82B97FAFF          call 00403920
:0045A1F5 33C0                xor eax, eax
:0045A1F7 55                  push ebp
:0045A1F8 68DBA24500          push 0045A2DB
:0045A1FD 64FF30              push dword ptr fs:[eax]
:0045A200 648920              mov dword ptr fs:[eax], esp
:0045A203 33DB                xor ebx, ebx
:0045A205 8B45FC              mov eax, dword ptr [ebp-04]
:0045A208 E85F95FAFF          call 0040376C
:0045A20D 83F804              cmp eax, 00000004   <-----比较注册名的位数是否小于 4
:0045A210 0F8CAA000000        jl 0045A2C0          <-----小于 4 就跳
:0045A216 33F6                xor esi, esi         <-----ESI 清零
:0045A218 33FF                xor edi, edi         <-----EDI 清零
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045A240(C)
|
:0045A21A 8D45F0              lea eax, dword ptr [ebp-10]
```



```asm
:0045A21D 50                 push eax
:0045A21E B901000000         mov ecx, 00000001
:0045A223 8BD7               mov edx, edi            <----第一次 EDX=EDI\=0
:0045A225 8B45FC             mov eax, dword ptr [ebp-04]
:0045A228 E84397FAFF         call 00403970           <----这个 CALL 搞鬼使第一位运算两次,跟进
```

来到此处:哇,好多地方调用呀,里面好多的跳转,没搞懂,哪位大侠愿意,请指教

* Referenced by a CALL at Addresses:

```
:00405FDC   , :00406435   , :0040646E   , :004064B4   , :0040A4DF   
:0040A600   , :0040C68C   , :0040C88F   , :0040C8A2   , :0040FAB8   
:004123D3   , :00412417   , :00430B35   , :0043508A   , :0043541B   
:00436451   , :00436963   , :004380C9   , :00438B5A   , :00438CBB   
:00439047   , :0043906E   , :00439CB1   , :0043B8D5   , :0043F00A   
:0043F040   , :0043FFEF   , :004411D0   , :00441207   , :004412AC   
:004412E3   , :00441569   , :004415A7   , :004415F3   , :0044160B   
:0044162E   , :00441649   , :004425E0   , :004429EC   , :00442A04   
:00442A88   , :00443078   , :004430A1   , :004430FC   , :00443131   
:0044315C   , :00443175   , :00443261   , :004434D2   , :00457ECD   
:004580A1   , :0045A228   , :0045A267   , :0045A28E   , :0045A4C3   
:0045AFD3   , :0045B06D   , :0045B95E   , :0045B984   , :0045CC6F   
:0045D181   , :0045D5BD   , :0045D63F   , :0045F361   , :0045F3D4   
```

```asm
:00403970 53                 push ebx
:00403971 85C0               test eax, eax
:00403973 742D               je 004039A2
:00403975 8B58FC             mov ebx, dword ptr [eax-04]
:00403978 85DB               test ebx, ebx
:0040397A 7426               je 004039A2
:0040397C 4A                 dec edx                <------EDX 值是关键参数
:0040397D 7C1B               jl 0040399A
:0040397F 39DA               cmp edx, ebx
:00403981 7D1F               jge 004039A2
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040399C(U)
```

```asm
:00403983 29D3               sub ebx, edx
:00403985 85C9               test ecx, ecx
:00403987 7C19               jl 004039A2
:00403989 39D9               cmp ecx, ebx
:0040398B 7F11               jg 0040399E
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:004039A0(U)
```

```asm
:0040398D 01C2               add edx, eax
:0040398F 8B442408           mov eax, dword ptr [esp+08]
:00403993 E840FDFFFF         call 004036D8
:00403998 EB11               jmp 004039AB
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040397D(C)
```

```asm
:0040399A 31D2               xor edx, edx
:0040399C EBE5               jmp 00403983
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040398B(C)
```

```asm
:0040399E 89D9               mov ecx, ebx
:004039A0 EBEB               jmp 0040398D
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:00403973(C), :0040397A(C), :00403981(C), :00403987(C)
```

```asm
:004039A2 8B442408           mov eax, dword ptr [esp+08]
:004039A6 E84DFCFFFF         call 004035F8
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00403998(U)
```

```asm
:004039AB 5B                 pop ebx
:004039AC C20400             ret 0004
```

出来继续

```asm
:0045A22D 8B45F0             mov eax, dword ptr [ebp-10]
:0045A230 0FB600             movzx eax, byte ptr [eax]  <----注册名的各位的 ASCII 值入 EAX
                                              此处因为用户名的第一位运算
                                              两次同时又只用运算注册名的
                                              位数次,所以最后一位不参加运算
:0045A233 03F0               add esi, eax           <---EAX 值累加 ESI=ESI+EAX
:0045A235 47                 inc edi                <---计数器加 1
:0045A236 8B45FC             mov eax, dword ptr [ebp-04]
:0045A239 E82E95FAFF         call 0040376C
:0045A23E 3BF8               cmp edi, eax           <-----比较是否达到注册名的长度次
:0045A240 7CD8               jl 0045A21A            <-----没完就回去
```

* Possible StringData Ref from Code Obj ->"1000-"      <-----注册码的前缀

```
```

```asm
:0045A242 68F4A24500         push 0045A2F4
:0045A247 8D55F0             lea edx, dword ptr [ebp-10]
:0045A24A 8BC6               mov eax, esi
:0045A24C E897BEFAFF         call 004060E8          <----上面累加算出的值十六进制转十进制,注册码的第二段
:0045A251 FF75F0             push [ebp-10]
:0045A254 6804A34500         push 0045A304
:0045A259 8D45E8             lea eax, dword ptr [ebp-18]
:0045A25C 50                 push eax
:0045A25D B901000000         mov ecx, 00000001
:0045A262 33D2               xor edx, edx           <-----EDX 清零
:0045A264 8B45FC             mov eax, dword ptr [ebp-04]
:0045A267 E80497FAFF         call 00403970          <----又来了
:0045A26C 8B45E8             mov eax, dword ptr [ebp-18]
:0045A26F 0FB600             movzx eax, byte ptr [eax]  <-----取注册名的第一位的 ASCII
:0045A272 8D55EC             lea edx, dword ptr [ebp-14]
:0045A275 E86EBEFAFF         call 004060E8          <------又调用这个 CALL 进行进制转换,注册码第三段的前部
:0045A27A FF75EC             push [ebp-14]
:0045A27D 8D45E4             lea eax, dword ptr [ebp-1C]
:0045A280 50                 push eax
:0045A281 B901000000         mov ecx, 00000001
:0045A286 BA04000000         mov edx, 00000004      <------ EDX\=4
:0045A28B 8B45FC             mov eax, dword ptr [ebp-04]
:0045A28E E8DD96FAFF         call 00403970          <------还是它
:0045A293 8B45E4             mov eax, dword ptr [ebp-1C]
:0045A296 0FB600             movzx eax, byte ptr [eax]   <-----取注册名的第四位的 ASCII
:0045A299 8D55E8             lea edx, dword ptr [ebp-18]
:0045A29C E847BEFAFF         call 004060E8          <------再调用这个 CALL 进行进制转换,注册码第三段的后部
:0045A2A1 FF75E8             push [ebp-18]
:0045A2A4 8D45F4             lea eax, dword ptr [ebp-0C]
:0045A2A7 BA05000000         mov edx, 00000005
:0045A2AC E87B95FAFF         call 0040382C           <------连接各段注册码
:0045A2B1 8B45F8             mov eax, dword ptr [ebp-08]   <------假码
:0045A2B4 8B55F4             mov edx, dword ptr [ebp-0C]   <------真码
:0045A2B7 E8C095FAFF         call 0040387C           <------比较
:0045A2BC 7502               jne 0045A2C0            <------如果你想强暴它这里是个好地方
```

"啪!"  
"谁扔我?"  
"扔你怎么样,这里不准发布 X 级的东东"  
"老兄,麻烦你单纯点,我是说强制暴破"  
"啪!","啊!你又扔!

```asm
:0045A2BE B301               mov bl, 01
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:0045A210(C), :0045A2BC(C)
```

```asm
:0045A2C0 33C0               xor eax, eax  <---注册名位数小于 4 就到了这,就跳过了运算.
:0045A2C2 5A                 pop edx
:0045A2C3 59                 pop ecx
:0045A2C4 59                 pop ecx
:0045A2C5 648910             mov dword ptr fs:[eax], edx
:0045A2C8 68E2A24500         push 0045A2E2
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0045A2E0(U)
```

```asm
:0045A2CD 8D45E4             lea eax, dword ptr [ebp-1C]
:0045A2D0 BA07000000         mov edx, 00000007
:0045A2D5 E83E93FAFF         call 00403618
:0045A2DA C3                 ret
```

### 5) 现总结一下

注册码的第一段固定不变，也就是 `1000-`。

注册码的第二段是注册名的第一位的 `ASCII` 加上前注册名长度减一位的 `ASCII` 的十进制总和（不知怎么表达，希望大家能看懂）。

注册码的第三段由注册名的第一位的 `ASCII` 和第四位的 `ASCII` 组成。

如果看不懂就看下面的算法吧：

```vb
Option Explicit

Sub Main() 
   On Error Resume Next 
   Dim yourname As String, initstr As String, esi As Long
```



以下是一段 VBScript 代码，用于生成注册码：

```vb
Dim i As Integer, s As Long, sn As String  

begin:  

   yourname = Trim(InputBox("请输入注册名,注册名必须不少于四位.", "Instyler Ex-it! 1.64 汉化版注册机请输入注册名", "AXiang"))  

   yourname = StrConv(yourname, 128) '因为是 VB 所以要解决 UNICODE 问题  

   If LenB(yourname) < 4 Then MsgBox "请输入一个长度大于三的英文名或大于一位的中文名!", 16, "注意!": GoTo begin  

   s = AscB(LeftB(yourname, 1))  

   For i = 1 To LenB(yourname) - 1 '实质上程序是进行了注册名长度次运算,但第一个字符运算了两
      s = s + AscB(MidB(yourname, i, 1))    '次,所以在前面先运算一次,这里减少一次.
   Next i  

   sn = "1000-" + CStr(s) + "-" + CStr(AscB(MidB(yourname, 1, 1))) + CStr(AscB(MidB(yourname, 4, 1)))  

   '"1000-"是程序内置的固定前缀.  

   MsgBox "您的注册码为:" + sn + Chr(13) + Chr(10) + "希望与大家交流,我的 E-MAIL 是:" + Chr(13) + Chr(10) + "yantuse.student@sina.com", 64, "作者涩郎恭喜你！"  

   Exit Sub  

End Sub
```

4dd466d0e1e0cc123afbe504da2462ca
