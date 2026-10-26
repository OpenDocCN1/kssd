
# 某QQ信息群发机 V1.0 注册码算法分析

日期：2005年8月22日　　破解人:wxb

-------------------------------------------------------------------------------------------------------------------------

『软件名称』：某QQ信息群发机 V1.0

『软件大小』：1111 KB

『下载地址』：软件涉及发送垃圾信息，下载地址免去。

『保护方式』：注册码保护+发送广告

『破解声明』：初学Crack，只是感兴趣,今天QQ里收到一个未注册版发送来带广告信息，无聊分析之，失误之处敬请诸位大侠赐教!

『破解工具』：OllyDbg.V1.10 聆风听雨汉化第二版、PeID 0.93

## 破解过程

### 一、查壳

`PeID 0.93`,查壳,`NsPack V1.4 -> LiuXingPing [Overlay] *`，试试脱掉它,OD载入:

```
0042AF61 阿> 9C             pushfd
0042AF62   60             pushad
0042AF63   E8 00000000       call qf.0042AF68   //F8到这里,ESP=0012FFA0
0042AF68   5D             pop ebp
0042AF69   B8 B1854000       mov eax,qf.004085B1
0042AF6E   2D AA854000       sub eax,qf.004085AA
```

根据ESP定律,命令行下断点:hr esp,F9运行:

```
0042B185   9D             popfd //断在这里,取消硬件断点
0042B186   - E9 A686FDFF       jmp qf.00403831   //JMP OEP
0042B18B   8BB5 A3FEFFFF     mov esi,dword ptr ss:[ebp-15D]
0042B191   0BF6           or esi,esi
```

断下后,取消断点,2下F7单步执行,来到:

```
00403831   /. 55           push ebp
00403832   |. 8BEC           mov ebp,esp
00403834   |. 6A FF         push -1
00403836   |. 68 F0624000     push qf.004062F0
0040383B   |. 68 A44C4000     push qf.00404CA4           ; SE 句柄安装
00403840   |. 64:A1 00000000   mov eax,dword ptr fs:[0]
00403846   |. 50           push eax
00403847   |. 64:8925 00000000   mov dword ptr fs:[0],esp
0040384E   |. 83EC 58         sub esp,58
00403851   |. 53           push ebx
00403852   |. 56           push esi
```

抓取修复后文件不能运行，查看文件竟然比脱壳前还要小，汗。Overlay数据没有修复，我等壳盲，带壳修行算了，F9运行……

### 二、注册验证分析

切换到注册界面，输入试炼码：1234567890 点击“注册”,提示：错误的注册码，有提示好啊，bp `MessageBoxA`，确定，OD中断在：

```
77D5050B U> 8BFF           mov edi,edi
77D5050D   55             push ebp
77D5050E   8BEC           mov ebp,esp
77D50510   833D 1C04D777 00   cmp dword ptr ds:[77D7041C],0
```

反回主模块,中断在`00427844`，向上来到:

```
00427767   |. 8B45 FC         mov eax,dword ptr ss:[ebp-4]     ; 试炼码
0042776A   |. 85C0           test eax,eax
0042776C   |. 75 05         jnz short qf.00427773
0042776E   |. B8 C7924000     mov eax,qf.004092C7
00427773   |> 50           push eax
00427774   |. 68 01000000     push 1
00427779   |. BB 30010000     mov ebx,130
0042777E   |. E8 48040000     call qf.00427BCB
00427783   |. 83C4 10         add esp,10
00427786   |. 8945 F4         mov dword ptr ss:[ebp-C],eax     ; 试炼码长度
00427789   |. 837D F4 0B       cmp dword ptr ss:[ebp-C],0B
0042778D   |. 0F85 71000000     jnz qf.00427804           ; 试炼码长度必需=11
00427793   |. 68 01030080     push 80000301
00427798   |. 6A 00         push 0
0042779A   |. 68 03000000     push 3
0042779F   |. 68 04000080     push 80000004
004277A4   |. 6A 00         push 0
004277A6   |. 8B45 FC         mov eax,dword ptr ss:[ebp-4]     ; 试炼码
004277A9   |. 85C0           test eax,eax
004277AB   |. 75 05         jnz short qf.004277B2       ; 试炼码不为空
004277AD   |. B8 C7924000     mov eax,qf.004092C7
004277B2   |> 50           push eax                   ; 试炼码
004277B3   |. 68 02000000     push 2                   ; 2
004277B8   |. BB 34010000     mov ebx,134
004277BD   |. E8 09040000     call qf.00427BCB           ; 取试炼码前3位
004277C2   |. 83C4 1C         add esp,1C
004277C5   |. 8945 EC         mov dword ptr ss:[ebp-14],eax     ; 试炼码前3位
004277C8   |. 68 F4FC4000     push qf.0040FCF4           ; ASCII "BIN"
004277CD   |. FF75 EC         push dword ptr ss:[ebp-14]
004277D0   |. E8 26A8FFFF     call qf.00421FFB
004277D5   |. 83C4 08         add esp,8
004277D8   |. 83F8 00         cmp eax,0
004277DB   |. B8 00000000     mov eax,0
004277E0   |. 0F95C0         setne al                   ; 前3位必需为BIN标志
004277E3   |. 8945 E8         mov dword ptr ss:[ebp-18],eax
004277E6   |. 8B5D EC         mov ebx,dword ptr ss:[ebp-14]     ; 试炼码前3位
004277E9   |. 85DB           test ebx,ebx
004277EB   |. 74 09         je short qf.004277F6         ; 不能为空
004277ED   |. 53           push ebx
004277EE   |. E8 D2030000     call qf.00427BC5
004277F3   |. 83C4 04         add esp,4
004277F6   |> 837D E8 00       cmp dword ptr ss:[ebp-18],0
004277FA   |. 0F85 04000000     jnz qf.00427804
00427800   |. 33C0           xor eax,eax
00427802   |. EB 05         jmp short qf.00427809
00427804   |> B8 01000000     mov eax,1
00427809   |> 85C0           test eax,eax
0042780B   |. 0F84 3B000000     je qf.0042784C             ; 必需跳，要求前3位为BIN
00427811   |. 68 04000080     push 80000004
00427816   |. 6A 00         push 0
00427818   |. 68 61FF4000     push qf.0040FF61
0042781D   |. 68 01030080     push 80000301
00427822   |. 6A 00         push 0
00427824   |. 68 10000000     push 10
00427829   |. 68 04000080     push 80000004
0042782E   |. 6A 00         push 0
00427830   |. 68 66FF4000     push qf.0040FF66
00427835   |. 68 03000000     push 3
0042783A   |. BB 00030000     mov ebx,300
0042783F   |. E8 87030000     call qf.00427BCB           ; MessageBoxA
00427844   |. 83C4 28         add esp,28                 ; 中断在这里
00427847   |. E9 B5010000     jmp qf.00427A01
0042784C   |> 6A FF         push -1
0042784E   |. 6A 08         push 8
00427850   |. 68 68010116     push 16010168
00427855   |. 68 01000152     push 52010001
0042785A   |. E8 78030000     call qf.00427BD7
0042785F   |. 83C4 10         add esp,10
00427862   |. 8945 F8         mov dword ptr ss:[ebp-8],eax     ; EAX指向试炼码
00427865   |. A1 180DE700     mov eax,dword ptr ds:[E70D18]     ; 已经计算好的注册码??
0042786A   |. 50           push eax                   ; 注册码
0042786B   |. FF75 F8         push dword ptr ss:[ebp-8]       ; 试炼码
0042786E   |. E8 88A7FFFF     call qf.00421FFB           ; 比较
00427873   |. 83C4 08         add esp,8
00427876   |. 83F8 00         cmp eax,0
00427879   |. B8 00000000     mov eax,0
0042787E   |. 0F95C0         setne al                   ; 比较标志
00427881   |. 8945 F4         mov dword ptr ss:[ebp-C],eax
00427884   |. 8B5D F8         mov ebx,dword ptr ss:[ebp-8]
00427887   |. 85DB           test ebx,ebx
00427889   |. 74 09         je short qf.00427894
0042788B   |. 53           push ebx
0042788C   |. E8 34030000     call qf.00427BC5
00427891   |. 83C4 04         add esp,4
00427894   |> 837D F4 00       cmp dword ptr ss:[ebp-C],0       ; 判断比较标志
00427898   |. 0F84 3B000000     je qf.004278D9             ; 爆破点
0042789E   |. 68 04000080     push 80000004
004278A3   |. 6A 00         push 0
004278A5   |. 68 61FF4000     push qf.0040FF61
004278AA   |. 68 01030080     push 80000301
004278AF   |. 6A 00         push 0
004278B1   |. 68 10000000     push 10
004278B6   |. 68 04000080     push 80000004
004278BB   |. 6A 00         push 0
004278BD   |. 68 66FF4000     push qf.0040FF66
004278C2   |. 68 03000000     push 3
004278C7   |. BB 00030000     mov ebx,300
004278CC   |. E8 FA020000     call qf.00427BCB
004278D1   |. 83C4 28         add esp,28
004278D4   |. E9 28010000     jmp qf.00427A01
004278D9   |> 68 04000080     push 80000004               ; 注册成功,保存注册信息
004278DE   |. 6A 00         push 0
004278E0   |. A1 180DE700     mov eax,dword ptr ds:[E70D18]
004278E5   |. 85C0           test eax,eax
004278E7   |. 75 05         jnz short qf.004278EE
004278E9   |. B8 C7924000     mov eax,qf.004092C7
004278EE   |> 50           push eax
004278EF   |. 68 04000080     push 80000004
004278F4   |. 6A 00         push 0
004278F6   |. 68 F8FC4000     push qf.0040FCF8           ; ASCII "softwore\binqqmsg\reg\"
004278FB   |. 68 01030080     push 80000301
00427900   |. 6A 00         push 0
00427902   |. 68 02000000     push 2
00427907   |. 68 03000000     push 3
0042790C   |. BB A4060000     mov ebx,6A4
00427911   |. E8 B5020000     call qf.00427BCB
```

分析代码结果发现按钮事件开始判断注册码的有效性(具体请看注释)，并没有计算注册码的过程，但在`00427865`处发现`ds:[E70D18]`存放的就是注册码的明码，在`0042786E`处进行明码比较，汗，在注册验证前注册码已经计算好了，注册按钮仅仅进行比较而已，现在关键是要找到`ds:[E70D18]`的数据是从那里来的，鼠标右键－搜索－全部常数：`00E70D18`:

参考位于 `qf:v0`到常数`E70D18`

地址     反汇编                                     注释

`00425D3B`   mov ebx,dword ptr ds:[E70D18]                     （初始 CPU 选择）
