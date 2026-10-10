

# LabWindows CVI 8.0

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-112.htm`](https://www.pediy.com/kssd/pediy09/pediy09-112.htm)

> 【软件名称】LabWindows CVI 8.0  
> 【应用平台】Win2000 SP4  
> 【作者邮箱】chubing6143@sina.com  
> 【使用工具】OllyDbg1.10，FlexLM 7.2 SDK  
> 【软件限制】FLEXnet Licensing v10.1.0  
> 【软件简介】NI 的产品，做测试和虚拟仪器的大家可能都知道了。 CVI 是 NI 公司提供的一个 C 语言的 IDE，由于提供了许多虚拟仪器的控件，及操作简单，而且基于 C 语言，因此用户群非常庞大，虽然我本人还是觉得 VC 好用，但是项目经常指定 CVI 为开发工具，因此接触比较多。有测试专业的同志多交流啊。  
>   
> 1. 确定 FlexLm 版本号 BEHAVIOR Version  
> 由于我有破解 CVI7.0 和 7.1 版本的经验，知道其 FLEXLM 加密在 CVI.dll 上，利用 LMTOOLS-Ultilities-Browse 选中 CVI.dll，然后 Find Version 可以发现如下信息。  
> \---------  
> Version  
> \---------  
> FLEXnet Licensing v10.1.0 (lmgr.lib), Copyright (c) 1988-2004 by Macrovision Corporation. All rights reserved.  
>   
> 2. 先找 FEATURE  
> 目的：确定 FEATURE 和产品版本号（不是 FLEXLM 的版本号）  
> 方法：在字符串参考中查找'lm\_ckout.c'；总共也就出现几次，比较容易确定。  
>   
> 据 newsearch 讲“一般是挨着的两个才是；另外有四个挨着的不是。”  
> 根据这一原则，在 OD 中 BP 105C1FD3  
> 105C1FD3      68 3C267410                push CVI\_1.1074263C                 ; ASCII "lm\_ckout.c"  
>   
> ,然后 F9 运行，  
> 程序被中断下来，此时察看堆栈情况如下所示：  
> 0022DB88   |0022E930     ASCII "CVI\_FDS"  
> 0022DB8C   |0022E964     ASCII "8.0000"  
>   
>   
> 可以知道  
> FEATURE 版本 v8.0  
> FEATURE id，CVI\_FDS  
> vendor: nilm  
>   
> 3. 找 vendor 以及得到 License  
> 在反汇编以后，搜索“87654321”，在前面设断点  
> 105BAAFB      8D8D 80FDFFFF              lea ecx,dword ptr ss:\[ebp-280\]  
> 105BAB01      51                         push ecx  
> 105BAB02      8B95 6CFDFFFF              mov edx,dword ptr ss:\[ebp-294\]  
> 105BAB08      81C2 0C030000              add edx,30C                         ; 计算得到 VENDOR 地址  
> 105BAB0E      52                         push edx                            ; VENDOR 入栈！(ASCII "nilm")  
> 105BAB0F      8B85 6CFDFFFF              mov eax,dword ptr ss:\[ebp-294\]  
> 105BAB15      50                         push eax  
> 105BAB16      E8 F4B00000                call CVI\_1.105C5C0F                 ; 这个 call 就是著名的所谓 7648B98E 标志 call！后面要进去了  
> 105BAB1B      83C4 0C                    add esp,0C  
> 105BAB1E      81BD 84FDFFFF 21436587     cmp dword ptr ss:\[ebp-27C\],87654321  
> 105BAB28      74 0C                      je short CVI\_1.105BAB36  
> 105BAB2A      81BD 88FDFFFF 78563412     cmp dword ptr ss:\[ebp-278\],12345678  
> 105BAB34      75 5D                      jnz short CVI\_1.105BAB93  
>   
>   
>   
> 我们在 105BAAFB 停下 d EBP-280  
> 0022F660  00000004  ...  
> 0022F664  8A0CD7ED  碜.  
> 0022F668  6F2EAFE6  姣.o  
> 0022F66C  EFD9F039  9 鹳  
> 0022F670  FC65A731  1�+  
> 0022F674  9E0596B5  y�  
> 0022F678  DB6AD71A  媟  
> 0022F67C  0001000A  ...  UNICODE ":\\"  
> 0022F680  30310020   .10  
> 0022F684  0000302E  .0..  
> 我们得到 BEHAVIOR Version  V10.0  
>   
> VENDOR 为"nilm"  
>   
> 根据以前的经验，参考安装程序提供的试用版的 License 文件，构造假的 License 文件如下：  
> \======================================================================================================================  
> SERVER THIS\_HOST ANY 27000  
> VENDOR nilm     
> PACKAGE CVI\_FDS\_LWH nilm 8.0000 COMPONENTS="CVI\_FDS NIDAQ\_DAQAssistant:7.0000" SIGN=123456789012  
> INCREMENT CVI\_FDS\_LWH nilm 8.0000 permanent uncounted HOSTID=DISK\_SERIAL\_NUM=\*\*\*\*\*\*\*\* vendor\_info="cracked by laowanghai" TS\_OK SIGN=210546789302  
> \======================================================================================================================  
>   
> 暂停一下，大家可能要问了，怎么知道是这样的格式了，我参考了试用版的 License 了，下面给出试用版的 License：  
> SERVER THIS\_HOST ANY 27000  
> VENDOR nilm     
> PACKAGE CVI\_FDS\_PKG nilm 8.0000 COMPONENTS="CVI\_FDS NIDAQ\_DAQAssistant:7.0000" SIGN=08BC414CDD12  
> INCREMENT CVI\_FDS\_PKG nilm 8.0000 12-may-2007 uncounted HOSTID=DEMO VENDOR\_STRING="TMP\_DEMO FULL\_DEMO\_END=19-apr-2007" vendor\_info="URL=http://www.ni.com/cgi-bin/redirect.cgi?dest=infcoprod&openagent&code=exveqw TYPE=Application LABEL=Full Development System FAMILY=CVI\_ADE\_080000" TS\_OK SIGN=73FE12149B8E  
>   
> 下面详细解释：  
> 为了个性我将其 PACKAGE 与 INCREMENT 名称取作了 CVI\_FDS\_LWH（CVI\_FDS 是前面获得的 FEATURE 了，LWH 是 laowanghai 的缩写了），其实你可以改成你想要的名称了。  
> HOSTID=DISK\_SERIAL\_NUM=\*\*\*\*\*\*\*\*，\*\*\*\*\*\*\*\*是我的硬盘序列号，你可以利用 LMTOOLS 看到了。  
>   
> 通过前面一个星期的工作（可以参考我的“Multigen vega prime 2.0 详细跟踪”文章），我知道了 FLEXLM 加密中 l\_crypt\_private 就是传说中的检查 license 文件是否正确的地方了。因此分析重点就是查找 l\_crypt\_private 函数了，下面使出我的“FFFF”大法（我自己命名的，命名完全搞笑，如有雷同，纯属巧合），搜索字符串“FFFF”，来到下面的地方：  
> 105C7623      C745 10 AC277410           mov dword ptr ss:\[ebp+10\],CVI\_1.107427>; ASCII "FFFF"  
> 向上向下浏览代码，很容易定位到下面估计就是 l\_pubkey\_verify 关键函数调用部分了：  
>   
> 105C7BB0      51                         push ecx                               ; SIGN 你的假 SIGN“123456789012”  
> 105C7BB1      8B95 9CEEFFFF              mov edx,dword ptr ss:\[ebp-1164\]  
> 105C7BB7      52                         push edx                               ; 66D8B337  
> 105C7BB8      8B45 14                    mov eax,dword ptr ss:\[ebp+14\]  
> 105C7BBB      50                         push eax  
> 105C7BBC      8B4D FC                    mov ecx,dword ptr ss:\[ebp-4\]  
> 105C7BBF      8D95 C4EEFFFF              lea edx,dword ptr ss:\[ebp-113C\]  
> 105C7BC5      2BCA                       sub ecx,edx  
> 105C7BC7      51                         push ecx                               ; 3B  
> 105C7BC8      8D85 C4EEFFFF              lea eax,dword ptr ss:\[ebp-113C\]  
> 105C7BCE      50                         push eax  
> 105C7BCF      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C7BD2      51                         push ecx  
> 105C7BD3      E8 0A120000                call CVI\_1.105C8DE2                    ; 判断 License 文件是否正确的关键函数  
>   
> 在 call CVI\_1.105C8DE2  函数处 F7 进去：  
> 105C8DE2      55                         push ebp  
> 105C8DE3      8BEC                       mov ebp,esp  
> 105C8DE5      81EC D8010000              sub esp,1D8  
> 105C8DEB      53                         push ebx  
> 105C8DEC      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C8DEF      0FBE80 0C030000            movsx eax,byte ptr ds:\[eax+30C\]  
> 105C8DF6      99                         cdq  
> ......  
> 105C8E77      68 B8287410                push CVI\_1.107428B8                    ; ASCII "demo"  
> 105C8E7C      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C8E7F      81C2 0C030000              add edx,30C  
> 105C8E85      52                         push edx  
> 105C8E86      E8 75F10A00                call CVI\_1.10678000  
> 105C8E8B      83C4 08                    add esp,8  
> 105C8E8E      85C0                       test eax,eax  
> 105C8E90      75 1F                      jnz short CVI\_1.105C8EB1  
> 105C8E92      6A 1B                      push 1B  
> 105C8E94      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C8E97      50                         push eax  
> 105C8E98      E8 533C0200                call CVI\_1.105ECAF0  
> 105C8E9D      83C4 08                    add esp,8  
> 105C8EA0      3D 6A830000                cmp eax,836A  
> ......  
>   
> 105C906C      8B55 1C                    mov edx,dword ptr ss:\[ebp+1C\]  
> 105C906F      0395 64FEFFFF              add edx,dword ptr ss:\[ebp-19C\]  
> 105C9075      8B85 5CFEFFFF              mov eax,dword ptr ss:\[ebp-1A4\]  
> 105C907B      8A0A                       mov cl,byte ptr ds:\[edx\]  
> 105C907D      888C05 68FEFFFF            mov byte ptr ss:\[ebp+eax-198\],cl  
> 105C9084      8B95 5CFEFFFF              mov edx,dword ptr ss:\[ebp-1A4\]  
> 105C908A      83C2 01                    add edx,1  
> 105C908D      8995 5CFEFFFF              mov dword ptr ss:\[ebp-1A4\],edx  
> 105C9093      83BD 5CFEFFFF 02           cmp dword ptr ss:\[ebp-1A4\],2  
> 105C909A      75 48                      jnz short CVI\_1.105C90E4  
> 105C909C      C785 5CFEFFFF 00000000     mov dword ptr ss:\[ebp-1A4\],0  
> 105C90A6      8D85 70FEFFFF              lea eax,dword ptr ss:\[ebp-190\]  
> 105C90AC      50                         push eax  
> 105C90AD      68 C0287410                push CVI\_1.107428C0                    ; ASCII "%02X"  
> 105C90B2      8D8D 68FEFFFF              lea ecx,dword ptr ss:\[ebp-198\]  
> 105C90B8      51                         push ecx  
> 105C90B9      E8 EBDF0A00                call CVI\_1.106770A9  
> 105C90BE      83C4 0C                    add esp,0C  
> ......  
> 105C9147      E8 17560000                call CVI\_1.105CE763  
> 105C914C      83C4 04                    add esp,4  
> 105C914F      83BD 54FEFFFF 00           cmp dword ptr ss:\[ebp-1AC\],0  
> 105C9156      74 0C                      je short CVI\_1.105C9164  
> 105C9158      C785 88FEFFFF 00000000     mov dword ptr ss:\[ebp-178\],0  
> 105C9162      EB 0A                      jmp short CVI\_1.105C916E  
> 105C9164      C785 88FEFFFF C8287410     mov dword ptr ss:\[ebp-178\],CVI\_1.10742>; ASCII "ok"   
> //此处纯属误导，我刚开始看到 ok 就想让其在这返回，浪费我的大好精力啊！   
> 105C916E      E9 7F0D0000                jmp CVI\_1.105C9EF2  
> 105C9173      6A 08                      push 8  
> 105C9175      6A 00                      push 0  
> 105C9177      68 98358A10                push CVI\_1.108A3598  
> 105C917C      E8 DF980A00                call CVI\_1.10672A60  
> 105C9181      83C4 0C                    add esp,0C  
> 105C9184      8B45 10                    mov eax,dword ptr ss:\[ebp+10\]  
> 105C9187      99                         cdq  
> 105C9188      83E2 07                    and edx,7  
> 105C918B      03C2                       add eax,edx  
> 105C918D      C1F8 03                    sar eax,3  
> ......  
> 105C9503      74 07                      je short CVI\_1.105C950C  
> 105C9505      C745 18 8AEB9B28           mov dword ptr ss:\[ebp+18\],289BEB8A  
> 105C950C      C785 7CFEFFFF 00000000     mov dword ptr ss:\[ebp-184\],0  
> 105C9516      EB 0F                      jmp short CVI\_1.105C9527  
> 105C9518      8B8D 7CFEFFFF              mov ecx,dword ptr ss:\[ebp-184\]  
> 105C951E      83C1 01                    add ecx,1  
> 105C9521      898D 7CFEFFFF              mov dword ptr ss:\[ebp-184\],ecx  
> 105C9527      8B55 1C                    mov edx,dword ptr ss:\[ebp+1C\]  
> 105C952A      0395 7CFEFFFF              add edx,dword ptr ss:\[ebp-184\]  
> 105C9530      0FBE02                     movsx eax,byte ptr ds:\[edx\]  
> 105C9533      85C0                       test eax,eax  
> 105C9535      74 22                      je short CVI\_1.105C9559  
> 105C9537      8B4D 1C                    mov ecx,dword ptr ss:\[ebp+1C\]  
> 105C953A      038D 7CFEFFFF              add ecx,dword ptr ss:\[ebp-184\]  
> 105C9540      0FBE11                     movsx edx,byte ptr ds:\[ecx\]  
> 105C9543      52                         push edx  
> 105C9544      E8 FD3A0300                call CVI\_1.105FD046  
> 105C9549      83C4 04                    add esp,4  
> 105C954C      85C0                       test eax,eax  
> 105C954E      75 07                      jnz short CVI\_1.105C9557  
> 105C9550      33C0                       xor eax,eax  
> 105C9552      E9 A1090000                jmp CVI\_1.105C9EF8  
> 105C9557    ^ EB BF                      jmp short CVI\_1.105C9518               ; 取 SIGN 将其转化为二进制值  
> 105C9559      8B45 1C                    mov eax,dword ptr ss:\[ebp+1C\]  
> 105C955C      50                         push eax  
> 105C955D      8D8D 9CFEFFFF              lea ecx,dword ptr ss:\[ebp-164\]  
> 105C9563      51                         push ecx  
> 105C9564      E8 67B80A00                call CVI\_1.10674DD0  
> 105C9569      83C4 08                    add esp,8  
> 105C956C      8D95 9CFEFFFF              lea edx,dword ptr ss:\[ebp-164\]  
> 105C9572      52                         push edx  
> 105C9573      E8 08CF0200                call CVI\_1.105F6480  
> 105C9578      83C4 04                    add esp,4  
> 105C957B      83BD 8CFEFFFF 14           cmp dword ptr ss:\[ebp-174\],14  
> 105C9582      0F85 BB000000              jnz CVI\_1.105C9643  
> ......  
> 105C9CC3      83C4 04                    add esp,4  
> 105C9CC6      EB 0D                      jmp short CVI\_1.105C9CD5  
> 105C9CC8      68 98358A10                push CVI\_1.108A3598  
> 105C9CCD      E8 44040000                call CVI\_1.105CA116  
> 105C9CD2      83C4 04                    add esp,4  
> 105C9CD5      8B4D F4                    mov ecx,dword ptr ss:\[ebp-C\]  
> 105C9CD8      83C1 08                    add ecx,8  
> 105C9CDB      894D F4                    mov dword ptr ss:\[ebp-C\],ecx  
> 105C9CDE    ^ E9 EEFAFFFF                jmp CVI\_1.105C97D1  
> 105C9CE3      817D 18 37B3D866           cmp dword ptr ss:\[ebp+18\],66D8B337  
> 105C9CEA      0F85 96000000              jnz CVI\_1.105C9D86  
> 105C9CF0      33D2                       xor edx,edx  
> 105C9CF2      8A15 9F358A10              mov dl,byte ptr ds:\[108A359F\]  
> 105C9CF8      52                         push edx  
> 105C9CF9      E8 A6030000                call CVI\_1.105CA0A4  
> 105C9CFE      83C4 04                    add esp,4  
> 105C9D01      25 FF000000                and eax,0FF  
> 105C9D06      25 FF000000                and eax,0FF  
> 105C9D0B      A2 9F358A10                mov byte ptr ds:\[108A359F\],al  
> 105C9D10      33C0                       xor eax,eax  
> 105C9D12      A0 9E358A10                mov al,byte ptr ds:\[108A359E\]  
> 105C9D17      50                         push eax  
> 105C9D18      E8 87030000                call CVI\_1.105CA0A4  
> 105C9D1D      83C4 04                    add esp,4  
> 105C9D20      25 FF000000                and eax,0FF  
> 105C9D25      25 FF000000                and eax,0FF  
> 105C9D2A      A2 9E358A10                mov byte ptr ds:\[108A359E\],al  
> 105C9D2F      33C9                       xor ecx,ecx  
> 105C9D31      8A0D 9F358A10              mov cl,byte ptr ds:\[108A359F\]  
> 105C9D37      33D2                       xor edx,edx  
> 105C9D39      8A15 98358A10              mov dl,byte ptr ds:\[108A3598\]  
> 105C9D3F      03CA                       add ecx,edx  
> 105C9D41      81E1 FF000000              and ecx,0FF  
> 105C9D47      81E1 FF000000              and ecx,0FF  
> 105C9D4D      880D 98358A10              mov byte ptr ds:\[108A3598\],cl  
> 105C9D53      33C0                       xor eax,eax  
> 105C9D55      A0 9E358A10                mov al,byte ptr ds:\[108A359E\]  
> 105C9D5A      33C9                       xor ecx,ecx  
> 105C9D5C      8A0D 99358A10              mov cl,byte ptr ds:\[108A3599\]  
> 105C9D62      03C1                       add eax,ecx  
> 105C9D64      25 FF000000                and eax,0FF  
> 105C9D69      25 FF000000                and eax,0FF  
> 105C9D6E      A2 99358A10                mov byte ptr ds:\[108A3599\],al  
> 105C9D73      C605 9F358A10 00           mov byte ptr ds:\[108A359F\],0  
> 105C9D7A      8A15 9F358A10              mov dl,byte ptr ds:\[108A359F\]  
> 105C9D80      8815 9E358A10              mov byte ptr ds:\[108A359E\],dl  
> 105C9D86      C785 78FEFFFF 08000000     mov dword ptr ss:\[ebp-188\],8  
> 105C9D90      817D 18 37B3D866           cmp dword ptr ss:\[ebp+18\],66D8B337  
> 105C9D97      75 0F                      jnz short CVI\_1.105C9DA8  
> 105C9D99      8B85 78FEFFFF              mov eax,dword ptr ss:\[ebp-188\]  
> 105C9D9F      83E8 02                    sub eax,2  
> 105C9DA2      8985 78FEFFFF              mov dword ptr ss:\[ebp-188\],eax  
> 105C9DA8      C785 7CFEFFFF 00000000     mov dword ptr ss:\[ebp-184\],0  
> 105C9DB2      EB 0F                      jmp short CVI\_1.105C9DC3  
> 105C9DB4      8B8D 7CFEFFFF              mov ecx,dword ptr ss:\[ebp-184\]  
> 105C9DBA      83C1 01                    add ecx,1  
> 105C9DBD      898D 7CFEFFFF              mov dword ptr ss:\[ebp-184\],ecx  
> 105C9DC3      8B95 7CFEFFFF              mov edx,dword ptr ss:\[ebp-184\]  
> 105C9DC9      3B95 78FEFFFF              cmp edx,dword ptr ss:\[ebp-188\]  
> 105C9DCF      0F8D 02010000              jge CVI\_1.105C9ED7  
> 105C9DD5      8B85 7CFEFFFF              mov eax,dword ptr ss:\[ebp-184\]  
> 105C9DDB      8A8C45 9CFEFFFF            mov cl,byte ptr ss:\[ebp+eax\*2-164\]  
> 105C9DE2      888D 34FEFFFF              mov byte ptr ss:\[ebp-1CC\],cl  
> 105C9DE8      0FBE95 34FEFFFF            movsx edx,byte ptr ss:\[ebp-1CC\]  
> 105C9DEF      52                         push edx  
> 105C9DF0      E8 5B300300                call CVI\_1.105FCE50  
> 105C9DF5      83C4 04                    add esp,4  
> 105C9DF8      85C0                       test eax,eax  
> 105C9DFA      74 15                      je short CVI\_1.105C9E11  
> 105C9DFC      0FBE85 34FEFFFF            movsx eax,byte ptr ss:\[ebp-1CC\]  
> 105C9E03      83E8 30                    sub eax,30  
> 105C9E06      C1E0 04                    shl eax,4  
> 105C9E09      8885 30FEFFFF              mov byte ptr ss:\[ebp-1D0\],al  
> 105C9E0F      EB 13                      jmp short CVI\_1.105C9E24  
> 105C9E11      0FBE8D 34FEFFFF            movsx ecx,byte ptr ss:\[ebp-1CC\]  
> 105C9E18      83E9 37                    sub ecx,37  
> 105C9E1B      C1E1 04                    shl ecx,4  
> 105C9E1E      888D 30FEFFFF              mov byte ptr ss:\[ebp-1D0\],cl  
> 105C9E24      8B95 7CFEFFFF              mov edx,dword ptr ss:\[ebp-184\]  
> 105C9E2A      8A8455 9DFEFFFF            mov al,byte ptr ss:\[ebp+edx\*2-163\]  
> 105C9E31      8885 34FEFFFF              mov byte ptr ss:\[ebp-1CC\],al  
> 105C9E37      0FBE8D 34FEFFFF            movsx ecx,byte ptr ss:\[ebp-1CC\]  
> 105C9E3E      51                         push ecx  
> 105C9E3F      E8 0C300300                call CVI\_1.105FCE50  
> 105C9E44      83C4 04                    add esp,4  
> 105C9E47      85C0                       test eax,eax  
> 105C9E49      74 1A                      je short CVI\_1.105C9E65  
> 105C9E4B      0FBE95 34FEFFFF            movsx edx,byte ptr ss:\[ebp-1CC\]  
> 105C9E52      83EA 30                    sub edx,30  
> 105C9E55      8A85 30FEFFFF              mov al,byte ptr ss:\[ebp-1D0\]  
> 105C9E5B      02C2                       add al,dl  
> 105C9E5D      8885 30FEFFFF              mov byte ptr ss:\[ebp-1D0\],al  
> 105C9E63      EB 18                      jmp short CVI\_1.105C9E7D  
> 105C9E65      0FBE8D 34FEFFFF            movsx ecx,byte ptr ss:\[ebp-1CC\]  
> 105C9E6C      83E9 37                    sub ecx,37  
> 105C9E6F      8A95 30FEFFFF              mov dl,byte ptr ss:\[ebp-1D0\]  
> 105C9E75      02D1                       add dl,cl  
> 105C9E77      8895 30FEFFFF              mov byte ptr ss:\[ebp-1D0\],dl  
> 105C9E7D      83BD 90FEFFFF 00           cmp dword ptr ss:\[ebp-170\],0  
> 105C9E84      74 2A                      je short CVI\_1.105C9EB0  
> 105C9E86      8B85 7CFEFFFF              mov eax,dword ptr ss:\[ebp-184\]  
> 105C9E8C      33C9                       xor ecx,ecx  
> 105C9E8E      8A88 98358A10              mov cl,byte ptr ds:\[eax+108A3598\]  
> 105C9E94      51                         push ecx  
> 105C9E95      8B95 7CFEFFFF              mov edx,dword ptr ss:\[ebp-184\]  
> 105C9E9B      52                         push edx  
> 105C9E9C      8D85 30FEFFFF              lea eax,dword ptr ss:\[ebp-1D0\]  
> 105C9EA2      50                         push eax  
> 105C9EA3      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C9EA6      51                         push ecx  
> 105C9EA7      FF95 90FEFFFF              call dword ptr ss:\[ebp-170\]  
> 105C9EAD      83C4 10                    add esp,10  
> 105C9EB0      8B95 30FEFFFF              mov edx,dword ptr ss:\[ebp-1D0\]  
> 105C9EB6      81E2 FF000000              and edx,0FF  
> 105C9EBC      8B85 7CFEFFFF              mov eax,dword ptr ss:\[ebp-184\]  
> 105C9EC2      33C9                       xor ecx,ecx  
> 105C9EC4      8A88 98358A10              mov cl,byte ptr ds:\[eax+108A3598\]  
> 105C9ECA      3BD1                       cmp edx,ecx                            ; 看到没有，  
> //此处就是取你的 SIGN 与正确的 SIGN 逐字节的比较  
> 105C9ECC      74 04                      je short CVI\_1.105C9ED2  
> 105C9ECE      33C0                       xor eax,eax  
> 105C9ED0      EB 26                      jmp short CVI\_1.105C9EF8  
> 105C9ED2    ^ E9 DDFEFFFF                jmp CVI\_1.105C9DB4  
> 105C9ED7      8B55 18                    mov edx,dword ptr ss:\[ebp+18\]  
> 105C9EDA      52                         push edx  
> 105C9EDB      68 98358A10                push CVI\_1.108A3598  
> 105C9EE0      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C9EE3      50                         push eax  
> 105C9EE4      E8 E9000000                call CVI\_1.105C9FD2  
> 105C9EE9      83C4 0C                    add esp,0C  
> 105C9EEC      8985 88FEFFFF              mov dword ptr ss:\[ebp-178\],eax  
> 105C9EF2      8B85 88FEFFFF              mov eax,dword ptr ss:\[ebp-178\]  
> 105C9EF8      5B                         pop ebx  
> 105C9EF9      8BE5                       mov esp,ebp  
> 105C9EFB      5D                         pop ebp  
> 105C9EFC      C3                         retn  
> 在 105C9ECC      74 04                      je short CVI\_1.105C9ED2 处强制跳转可以得到第一个 SIGN 应该为 B362A92649B8  
> 先退出程序，修改 License 文件中的第一个 SIGN，然后重新加载程序，这次第一个 SIGN 比较正常通过了，然后同样的步骤可以得到第二个 SIGN 了。  
>   
> 到此，得到完整的 License 文件如下：  
> SERVER THIS\_HOST ANY 27000  
> VENDOR nilm     
> PACKAGE CVI\_FDS\_LWH nilm 8.0000 COMPONENTS="CVI\_FDS NIDAQ\_DAQAssistant:7.0000" SIGN=B362A92649B8  
> INCREMENT CVI\_FDS\_LWH nilm 8.0000 permanent uncounted HOSTID=DISK\_SERIAL\_NUM=\*\*\*\*\*\*\*\* vendor\_info="cracked by laowanghai" TS\_OK SIGN=78E2A6F618A\*（最后一位隐去）  
>   
> 附仔细跟踪了一遍程序后得到 laoqian 的明码比较办法的小小改动法,有兴趣的大家自己看吧:  
> 总结：学习 FLEXLM 加密就需要对其加密过程了解清楚了，这样即使你做不出正确的 License，爆破应该是问题不大了,我是菜鸟没什么追求了.  
>   
> 感谢:  
> 学习过程中大量参考了  
> tulipfan\[CCG\]大虾  
> laoqian  
> sln  
> newsearch  
> 的文章.   
> 感谢看雪上所有人.  
>   
>   
> 求助：1)谁有 FLEXnet Licensing v10.1.0 的 SDK 啊，或告诉我在那下啊，谢谢！  
>       2)FLEXnet Licensing v10.1.0 如何追加密的 SEED 啊.  
>   
> 大家看到这了，辛苦了，谢谢大家！！！  
>   
>   
> 2007 年 4 月 6 日 

> 附仔细跟踪了一遍程序后得到 laoqian 的明码比较办法的小小改动法,有兴趣的大家自己看吧:  
>   
> 1020438B      8D85 E0FEFFFF              lea eax,dword ptr ss:\[ebp-120\]  
> 10204391      68 54007010                push CVI\_1.10700054                    ; ASCII "FDS"  
> 10204396      50                         push eax  
> 10204397      E8 0D0D0100                call CVI\_1.102150A9  
> 1020439C      F7D8                       neg eax  
> 1020439E      1BC0                       sbb eax,eax  
> 102043A0      40                         inc eax  
> 102043A1      6A 01                      push 1  
> 102043A3      A3 98637A10                mov dword ptr ds:\[107A6398\],eax  
> 102043A8      E8 0ADBFFFF                call CVI\_1.10201EB7                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> 10201F3C      50                         push eax                               ;  ASCII "LabWindows/CVI Message"  
> 10201F3D      55                         push ebp                               ; ASCII "http://www.ni.com/cgi-bin/redirect.cgi?dest=infcoprod&openagent&code=exveqw"  
> 10201F3E      E8 77C0FEFF                call CVI\_1.101EDFBA                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> 101EDFF2      6A 01                      push 1  
> 101EDFF4      56                         push esi  
> 101EDFF5      FF7424 1C                  push dword ptr ss:\[esp+1C\]  
> 101EDFF9      FF7424 1C                  push dword ptr ss:\[esp+1C\]  
> 101EDFFD      68 B4856A10                push CVI\_1.106A85B4                    ; ASCII "CVI\_ADE\_080000"  
> 101EE002      56                         push esi  
> 101EE003      56                         push esi  
> 101EE004      6A 08                      push 8  
> 101EE006      FF3485 10EB6F10            push dword ptr ds:\[eax\*4+106FEB10\]  
> 101EE00D      E8 CEAD3A00                call CVI\_1.10598DE0                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> 10598DE0      6A FF                      push -1  
> 10598DE2      68 144C6910                push CVI\_1.10694C14  
> 10598DE7      64:A1 00000000             mov eax,dword ptr fs:\[0\]  
> 10598DED      50                         push eax  
> 10598DEE      64:8925 00000000           mov dword ptr fs:\[0\],esp  
> 10598DF5      81EC F8000000              sub esp,0F8  
> 10598DFB      8B9424 20010000            mov edx,dword ptr ss:\[esp+120\]  
> 10598E02      53                         push ebx  
> 10598E03      55                         push ebp  
> 10598E04      56                         push esi  
> 10598E05      33DB                       xor ebx,ebx  
> 10598E07      BD 0F000000                mov ebp,0F  
> 10598E0C      8BC2                       mov eax,edx  
> 10598E0E      57                         push edi  
> 10598E0F      894C24 10                  mov dword ptr ss:\[esp+10\],ecx  
> 10598E13      89AC24 90000000            mov dword ptr ss:\[esp+90\],ebp  
> 10598E1A      899C24 8C000000            mov dword ptr ss:\[esp+8C\],ebx  
> 10598E21      885C24 7C                  mov byte ptr ss:\[esp+7C\],bl  
> 10598E25      8D70 01                    lea esi,dword ptr ds:\[eax+1\]  
> 10598E28      8A08                       mov cl,byte ptr ds:\[eax\]  
> 10598E2A      40                         inc eax  
> 10598E2B      3ACB                       cmp cl,bl  
> 10598E2D    ^ 75 F9                      jnz short CVI\_1.10598E28  
> 10598E2F      2BC6                       sub eax,esi  
> 10598E31      50                         push eax  
> 10598E32      52                         push edx  
> 10598E33      8D8C24 80000000            lea ecx,dword ptr ss:\[esp+80\]  
> 10598E3A      E8 0430A9FF                call CVI\_1.1002BE43  
> 10598E3F      8BB424 2C010000            mov esi,dword ptr ss:\[esp+12C\]  
> 10598E46      8BC6                       mov eax,esi  
> 10598E48      899C24 10010000            mov dword ptr ss:\[esp+110\],ebx  
> 10598E4F      896C24 64                  mov dword ptr ss:\[esp+64\],ebp  
> 10598E53      895C24 60                  mov dword ptr ss:\[esp+60\],ebx  
> 10598E57      885C24 50                  mov byte ptr ss:\[esp+50\],bl  
> 10598E5B      8D78 01                    lea edi,dword ptr ds:\[eax+1\]  
> 10598E5E      8BFF                       mov edi,edi  
> 10598E60      8A08                       mov cl,byte ptr ds:\[eax\]  
> 10598E62      40                         inc eax  
> 10598E63      3ACB                       cmp cl,bl  
> 10598E65    ^ 75 F9                      jnz short CVI\_1.10598E60  
> 10598E67      2BC7                       sub eax,edi  
> 10598E69      50                         push eax  
> 10598E6A      56                         push esi  
> 10598E6B      8D4C24 54                  lea ecx,dword ptr ss:\[esp+54\]  
> 10598E6F      E8 CF2FA9FF                call CVI\_1.1002BE43  
> 10598E74      8BB424 28010000            mov esi,dword ptr ss:\[esp+128\]  
> 10598E7B      8BC6                       mov eax,esi  
> 10598E7D      C68424 10010000 01         mov byte ptr ss:\[esp+110\],1  
> 10598E85      896C24 48                  mov dword ptr ss:\[esp+48\],ebp  
> 10598E89      895C24 44                  mov dword ptr ss:\[esp+44\],ebx  
> 10598E8D      885C24 34                  mov byte ptr ss:\[esp+34\],bl  
> 10598E91      8D78 01                    lea edi,dword ptr ds:\[eax+1\]  
> 10598E94      8A08                       mov cl,byte ptr ds:\[eax\]  
> 10598E96      40                         inc eax  
> 10598E97      3ACB                       cmp cl,bl  
> 10598E99    ^ 75 F9                      jnz short CVI\_1.10598E94  
> 10598E9B      2BC7                       sub eax,edi  
> 10598E9D      50                         push eax  
> 10598E9E      56                         push esi  
> 10598E9F      8D4C24 38                  lea ecx,dword ptr ss:\[esp+38\]  
> 10598EA3      E8 9B2FA9FF                call CVI\_1.1002BE43  
> 10598EA8      8D8424 24010000            lea eax,dword ptr ss:\[esp+124\]  
> 10598EAF      50                         push eax  
> 10598EB0      8D4C24 74                  lea ecx,dword ptr ss:\[esp+74\]  
> 10598EB4      C68424 14010000 02         mov byte ptr ss:\[esp+114\],2  
> 10598EBC      E8 2F5D0000                call CVI\_1.1059EBF0  
> 10598EC1      8D8C24 20010000            lea ecx,dword ptr ss:\[esp+120\]  
> 10598EC8      51                         push ecx  
> 10598EC9      8D4C24 6C                  lea ecx,dword ptr ss:\[esp+6C\]  
> 10598ECD      E8 1E5D0000                call CVI\_1.1059EBF0  
> 10598ED2      8D9424 1C010000            lea edx,dword ptr ss:\[esp+11C\]  
> 10598ED9      52                         push edx  
> 10598EDA      8D8C24 98000000            lea ecx,dword ptr ss:\[esp+98\]  
> 10598EE1      E8 0A5D0000                call CVI\_1.1059EBF0  
> 10598EE6      8BB424 18010000            mov esi,dword ptr ss:\[esp+118\]         ; Feature "CVI\_FDS"  
> 10598EED      8BC6                       mov eax,esi  
> 10598EEF      896C24 2C                  mov dword ptr ss:\[esp+2C\],ebp  
> 10598EF3      895C24 28                  mov dword ptr ss:\[esp+28\],ebx  
> 10598EF7      885C24 18                  mov byte ptr ss:\[esp+18\],bl  
> 10598EFB      8D78 01                    lea edi,dword ptr ds:\[eax+1\]  
> 10598EFE      8BFF                       mov edi,edi  
> 10598F00      8A08                       mov cl,byte ptr ds:\[eax\]  
> 10598F02      40                         inc eax  
> 10598F03      3ACB                       cmp cl,bl  
> 10598F05    ^ 75 F9                      jnz short CVI\_1.10598F00  
> 10598F07      2BC7                       sub eax,edi                            ; Feature "CVI\_Base"  
> 10598F09      50                         push eax  
> 10598F0A      56                         push esi  
> 10598F0B      8D4C24 1C                  lea ecx,dword ptr ss:\[esp+1C\]  
> 10598F0F      E8 2F2FA9FF                call CVI\_1.1002BE43  
> 10598F14      8D4424 70                  lea eax,dword ptr ss:\[esp+70\]  
> 10598F18      50                         push eax  
> 10598F19      8D4C24 6C                  lea ecx,dword ptr ss:\[esp+6C\]  
> 10598F1D      51                         push ecx  
> 10598F1E      8D9424 9C000000            lea edx,dword ptr ss:\[esp+9C\]  
> 10598F25      52                         push edx  
> 10598F26      8D8C24 A8000000            lea ecx,dword ptr ss:\[esp+A8\]  
> 10598F2D      C68424 1C010000 03         mov byte ptr ss:\[esp+11C\],3  
> 10598F35      E8 865D0000                call CVI\_1.1059ECC0  
> 10598F3A      8B8C24 3C010000            mov ecx,dword ptr ss:\[esp+13C\]  
> 10598F41      8B9424 38010000            mov edx,dword ptr ss:\[esp+138\]  
> 10598F48      51                         push ecx  
> 10598F49      8B8C24 38010000            mov ecx,dword ptr ss:\[esp+138\]  
> 10598F50      52                         push edx  
> 10598F51      51                         push ecx  
> 10598F52      8D9424 84000000            lea edx,dword ptr ss:\[esp+84\]  
> 10598F59      52                         push edx  
> 10598F5A      8D4C24 5C                  lea ecx,dword ptr ss:\[esp+5C\]  
> 10598F5E      51                         push ecx  
> 10598F5F      8D5424 44                  lea edx,dword ptr ss:\[esp+44\]  
> 10598F63      52                         push edx  
> 10598F64      50                         push eax  
> 10598F65      8D4424 30                  lea eax,dword ptr ss:\[esp+30\]  
> 10598F69      50                         push eax  
> 10598F6A      8B4424 30                  mov eax,dword ptr ss:\[esp+30\]  
> 10598F6E      8B48 04                    mov ecx,dword ptr ds:\[eax+4\]  
> 10598F71      50                         push eax  
> 10598F72      C68424 34010000 04         mov byte ptr ss:\[esp+134\],4  
> 10598F7A      E8 11400000                call CVI\_1.1059CF90                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> ......  
> 1059D005      50                         push eax  
> 1059D006      57                         push edi  
> 1059D007      53                         push ebx  
> 1059D008      8BCE                       mov ecx,esi  
> 1059D00A      C78424 18010000 00000000   mov dword ptr ss:\[esp+118\],0  
> 1059D015      E8 A6FCFFFF                call CVI\_1.1059CCC0                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> ......  
> 1059CD00      8B4D 0C                    mov ecx,dword ptr ss:\[ebp+C\]  
> 1059CD03      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 1059CD06      51                         push ecx  
> 1059CD07      8B4C24 20                  mov ecx,dword ptr ss:\[esp+20\]  
> 1059CD0B      52                         push edx  
> 1059CD0C      8D4424 1C                  lea eax,dword ptr ss:\[esp+1C\]  
> 1059CD10      50                         push eax  
> 1059CD11      C64424 1F 00               mov byte ptr ss:\[esp+1F\],0  
> 1059CD16      E8 05FBFFFF                call CVI\_1.1059C820                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> 1059C950      8B5C24 14                  mov ebx,dword ptr ss:\[esp+14\]  
> 1059C954      8B4E 14                    mov ecx,dword ptr ds:\[esi+14\]  
> 1059C957      8B0CB9                     mov ecx,dword ptr ds:\[ecx+edi\*4\]  
> 1059C95A      6A 00                      push 0  
> 1059C95C      8D5424 2C                  lea edx,dword ptr ss:\[esp+2C\]  
> 1059C960      52                         push edx  
> 1059C961      8D9424 B4000000            lea edx,dword ptr ss:\[esp+B4\]  
> 1059C968      52                         push edx  
> 1059C969      83CB 01                    or ebx,1  
> 1059C96C      50                         push eax  
> 1059C96D      C68424 58030000 05         mov byte ptr ss:\[esp+358\],5  
> 1059C975      895C24 24                  mov dword ptr ss:\[esp+24\],ebx  
> 1059C979      E8 B29B0000                call CVI\_1.105A6530                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> ......  
> 105A6589      8B4C24 1C                  mov ecx,dword ptr ss:\[esp+1C\]  
> 105A658D      51                         push ecx  
> 105A658E      8BCE                       mov ecx,esi  
> 105A6590      E8 0BF8FFFF                call CVI\_1.105A5DA0                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> ......  
>   
> 105A5FE9      8B55 0C                    mov edx,dword ptr ss:\[ebp+C\]  
> 105A5FEC      8B7D 08                    mov edi,dword ptr ss:\[ebp+8\]  
> 105A5FEF      50                         push eax  
> 105A5FF0      52                         push edx  
> 105A5FF1      57                         push edi  
> 105A5FF2      E8 39D3FFFF                call CVI\_1.105A3330                   ; 第三次的时候 F7 进去  
> \----------------------------------------------------------------------------------------------  
> ......  
>   
> 105A33A1      51                         push ecx  
> 105A33A2      6A 08                      push 8  
> 105A33A4      8D56 08                    lea edx,dword ptr ds:\[esi+8\]  
> 105A33A7      52                         push edx  
> 105A33A8      6A 00                      push 0  
> 105A33AA      6A 01                      push 1  
> 105A33AC      53                         push ebx                               ; 8.0000  
> 105A33AD      50                         push eax                               ; FEATURE  
> 105A33AE      8B46 04                    mov eax,dword ptr ds:\[esi+4\]  
> 105A33B1      50                         push eax  
> 105A33B2      E8 E9EB0100                call CVI\_1.105C1FA0                    ; F7 进去  
>   
> \----------------------------------------------------------------------------------------------  
> ......  
> 105C200C      8B55 20                    mov edx,dword ptr ss:\[ebp+20\]  
> 105C200F      52                         push edx  
> 105C2010      8B45 1C                    mov eax,dword ptr ss:\[ebp+1C\]  
> 105C2013      50                         push eax  
> 105C2014      8B4D 18                    mov ecx,dword ptr ss:\[ebp+18\]  
> 105C2017      51                         push ecx  
> 105C2018      8B55 14                    mov edx,dword ptr ss:\[ebp+14\]  
> 105C201B      52                         push edx  
> 105C201C      8B45 10                    mov eax,dword ptr ss:\[ebp+10\]  
> 105C201F      50                         push eax                               ; 8.0000  
> 105C2020      8B4D 0C                    mov ecx,dword ptr ss:\[ebp+C\]  
> 105C2023      51                         push ecx                               ; CVI\_FDS  
> 105C2024      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C2027      52                         push edx  
> 105C2028      E8 B4000000                call CVI\_1.105C20E1                    ; F7 进去  
>   
>   
> \----------------------------------------------------------------------------------------------  
> ......  
> 105C20E1      55                         push ebp  
> 105C20E2      8BEC                       mov ebp,esp  
> 105C20E4      81EC 84010000              sub esp,184  
> 105C20EA      57                         push edi  
> 105C20EB      C745 F8 00000000           mov dword ptr ss:\[ebp-8\],0  
> 105C20F2      C745 F4 00000000           mov dword ptr ss:\[ebp-C\],0  
> 105C20F9      C745 FC 00000000           mov dword ptr ss:\[ebp-4\],0  
> 105C2100      837D 0C 00                 cmp dword ptr ss:\[ebp+C\],0  
> 105C2104      74 1B                      je short CVI\_1.105C2121  
> 105C2106      837D 08 00                 cmp dword ptr ss:\[ebp+8\],0  
> 105C210A      74 15                      je short CVI\_1.105C2121  
> 105C210C      6A 1E                      push 1E  
> 105C210E      8B45 0C                    mov eax,dword ptr ss:\[ebp+C\]  
> 105C2111      50                         push eax  
> 105C2112      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C2115      83C1 2C                    add ecx,2C  
> 105C2118      51                         push ecx  
> 105C2119      E8 E9030300                call CVI\_1.105F2507  
> 105C211E      83C4 0C                    add esp,0C  
> 105C2121      6A 17                      push 17  
> 105C2123      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C2126      52                         push edx  
> 105C2127      E8 C4A90200                call CVI\_1.105ECAF0  
> 105C212C      83C4 08                    add esp,8  
> 105C212F      3D 342A0000                cmp eax,2A34                  ;熟悉的面孔  
> 105C2134      74 59                      je short CVI\_1.105C218F  
>   
> ......  
> 105C2265      52                         push edx  
> 105C2266      8B45 1C                    mov eax,dword ptr ss:\[ebp+1C\]  
> 105C2269      50                         push eax  
> 105C226A      8B4D 18                    mov ecx,dword ptr ss:\[ebp+18\]  
> 105C226D      51                         push ecx  
> 105C226E      8B55 14                    mov edx,dword ptr ss:\[ebp+14\]  
> 105C2271      52                         push edx  
> 105C2272      8B45 10                    mov eax,dword ptr ss:\[ebp+10\]  
> 105C2275      50                         push eax                               ; 8.0000  
> 105C2276      8B4D 0C                    mov ecx,dword ptr ss:\[ebp+C\]  
> 105C2279      51                         push ecx                               ; CVI\_FDS  
> 105C227A      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C227D      52                         push edx  
> 105C227E      FF15 A0358A10              call dword ptr ds:\[108A35A0\]           ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> ......  
> 105C292B      52                         push edx  
> 105C292C      8B45 E0                    mov eax,dword ptr ss:\[ebp-20\]  
> 105C292F      8B88 24030000              mov ecx,dword ptr ds:\[eax+324\]  
> 105C2935      51                         push ecx  
> 105C2936      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C2939      52                         push edx  
> 105C293A      E8 50120000                call CVI\_1.105C3B8F                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
> 105C3B8F      55                         push ebp  
> 105C3B90      8BEC                       mov ebp,esp  
> 105C3B92      81EC B4050000              sub esp,5B4  
> 105C3B98      57                         push edi  
> 105C3B99      C785 70FDFFFF 00000000     mov dword ptr ss:\[ebp-290\],0  
> 105C3BA3      C785 74FDFFFF 00000000     mov dword ptr ss:\[ebp-28C\],0  
> 105C3BAD      C745 FC 00000000           mov dword ptr ss:\[ebp-4\],0  
> 105C3BB4      C785 7CFDFFFF 00000000     mov dword ptr ss:\[ebp-284\],0  
> 105C3BBE      C785 78FDFFFF 00000000     mov dword ptr ss:\[ebp-288\],0  
> 105C3BC8      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C3BCB      33C9                       xor ecx,ecx  
> 105C3BCD      83B8 50030000 00           cmp dword ptr ds:\[eax+350\],0  
> 105C3BD4      0F94C1                     sete cl  
> 105C3BD7      81E1 00000001              and ecx,1000000  
> 105C3BDD      85C9                       test ecx,ecx  
> 105C3BDF      74 1A                      je short CVI\_1.105C3BFB  
> 105C3BE1      8B55 0C                    mov edx,dword ptr ss:\[ebp+C\]  
> 105C3BE4      8B82 7C020000              mov eax,dword ptr ds:\[edx+27C\]  
> 105C3BEA      83E0 01                    and eax,1  
> 105C3BED      85C0                       test eax,eax  
> 105C3BEF      74 0A                      je short CVI\_1.105C3BFB  
> 105C3BF1      B8 01000000                mov eax,1  
> 105C3BF6      E9 8B0A0000                jmp CVI\_1.105C4686  
> 105C3BFB      68 7C020000                push 27C  
> 105C3C00      8B4D 10                    mov ecx,dword ptr ss:\[ebp+10\]  
> 105C3C03      51                         push ecx  
> 105C3C04      8D95 80FDFFFF              lea edx,dword ptr ss:\[ebp-280\]  
> 105C3C0A      52                         push edx  
> 105C3C0B      E8 10F00A00                call CVI\_1.10672C20  
> 105C3C10      83C4 0C                    add esp,0C  
> 105C3C13      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C3C16      8B88 50030000              mov ecx,dword ptr ds:\[eax+350\]  
> 105C3C1C      81E1 00008000              and ecx,800000  
> 105C3C22      85C9                       test ecx,ecx  
> 105C3C24      75 18                      jnz short CVI\_1.105C3C3E  
> 105C3C26      8D95 80FDFFFF              lea edx,dword ptr ss:\[ebp-280\]  
> 105C3C2C      52                         push edx  
> 105C3C2D      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C3C30      05 0C030000                add eax,30C  
> 105C3C35      50                         push eax  
> 105C3C36      E8 4A960200                call CVI\_1.105ED285  
> 105C3C3B      83C4 08                    add esp,8  
> 105C3C3E      8D8D 80FDFFFF              lea ecx,dword ptr ss:\[ebp-280\]  
> 105C3C44      51                         push ecx  
> 105C3C45      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C3C48      81C2 0C030000              add edx,30C  
> 105C3C4E      52                         push edx  
> 105C3C4F      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C3C52      50                         push eax                               ; nilm  
> 105C3C53      E8 B71F0000                call CVI\_1.105C5C0F  
> 105C3C58      83C4 0C                    add esp,0C  
> 105C3C5B      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C3C5E      8B91 1C040000              mov edx,dword ptr ds:\[ecx+41C\]  
> 105C3C64      83BA 780D0000 00           cmp dword ptr ds:\[edx+D78\],0  
> 105C3C6B      0F84 A6000000              je CVI\_1.105C3D17  
> 105C3C71      8B45 0C                    mov eax,dword ptr ss:\[ebp+C\]  
> 105C3C74      83B8 38030000 00           cmp dword ptr ds:\[eax+338\],0  
> 105C3C7B      0F85 96000000              jnz CVI\_1.105C3D17  
> 105C3C81      C685 39FDFFFF 00           mov byte ptr ss:\[ebp-2C7\],0  
> 105C3C88      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C3C8B      8B91 1C040000              mov edx,dword ptr ds:\[ecx+41C\]  
> 105C3C91      0FBE82 780D0000            movsx eax,byte ptr ds:\[edx+D78\]  
> 105C3C98      83C0 30                    add eax,30  
> 105C3C9B      8885 38FDFFFF              mov byte ptr ss:\[ebp-2C8\],al  
> 105C3CA1      8D8D 38FDFFFF              lea ecx,dword ptr ss:\[ebp-2C8\]  
> 105C3CA7      51                         push ecx  
> 105C3CA8      68 1C277410                push CVI\_1.1074271C                    ; ASCII "SIGN%s="  
> 105C3CAD      8D95 3CFDFFFF              lea edx,dword ptr ss:\[ebp-2C4\]  
> 105C3CB3      52                         push edx  
> 105C3CB4      E8 FEFE0A00                call CVI\_1.10673BB7                    ; 熟悉的面孔  
>   
> ......  
> 105C3FFF      50                         push eax                               ; SIGN  
> 105C4000      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C4003      51                         push ecx  
> 105C4004      E8 879A0200                call CVI\_1.105EDA90  
> 105C4009      83C4 08                    add esp,8  
> 105C400C      8985 7CFDFFFF              mov dword ptr ss:\[ebp-284\],eax  
> 105C4012      8D95 80FDFFFF              lea edx,dword ptr ss:\[ebp-280\]  
> 105C4018      52                         push edx  
> 105C4019      8B85 7CFDFFFF              mov eax,dword ptr ss:\[ebp-284\]  
> 105C401F      50                         push eax  
> 105C4020      8B4D 0C                    mov ecx,dword ptr ss:\[ebp+C\]  
> 105C4023      51                         push ecx  
> 105C4024      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C4027      52                         push edx  
> 105C4028      E8 6E2C0000                call CVI\_1.105C6C9B                    ; F7 进去  
> \----------------------------------------------------------------------------------------------  
>   
> 105C6C9B      55                         push ebp  
> 105C6C9C      8BEC                       mov ebp,esp  
> 105C6C9E      83EC 10                    sub esp,10  
> 105C6CA1      C745 FC 00000000           mov dword ptr ss:\[ebp-4\],0  
> 105C6CA8      C745 F8 00000000           mov dword ptr ss:\[ebp-8\],0  
> 105C6CAF      8B45 14                    mov eax,dword ptr ss:\[ebp+14\]  
> 105C6CB2      50                         push eax  
> 105C6CB3      8B4D 10                    mov ecx,dword ptr ss:\[ebp+10\]  
> 105C6CB6      51                         push ecx  
> 105C6CB7      8B55 0C                    mov edx,dword ptr ss:\[ebp+C\]  
> 105C6CBA      52                         push edx  
> 105C6CBB      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C6CBE      50                         push eax  
> 105C6CBF      E8 46020000                call CVI\_1.105C6F0A                    ; 这里应该就是 l\_crypt\_private 函数了  
> ......  
> \----------------------------------------------------------------------------------------------  
> ......  
> 105C714F      8B42 6C                    mov eax,dword ptr ds:\[edx+6C\]  
> 105C7152      81B8 38130000 8AEB9B28     cmp dword ptr ds:\[eax+1338\],289BEB8A                    ; 熟悉的面孔  
> 105C715C      74 6A                      je short CVI\_1.105C71C8  
> 105C715E      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C7161      8B51 6C                    mov edx,dword ptr ds:\[ecx+6C\]  
> 105C7164      81BA 38130000 37B3D866     cmp dword ptr ds:\[edx+1338\],66D8B337                    ; 熟悉的面孔  
> 105C716E      74 58                      je short CVI\_1.105C71C8  
> ......  
> 105C71FF      52                         push edx                               ; SIGN  
> 105C7200      E8 8BC60A00                call CVI\_1.10673890  
> 105C7205      83C4 04                    add esp,4  
> 105C7208      83F8 0C                    cmp eax,0C                             ; SIGN 的长度是否等于 12  
> 105C720B      7F 13                      jg short CVI\_1.105C7220  
> 105C720D      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C7210      8B88 50030000              mov ecx,dword ptr ds:\[eax+350\]  
> 105C7216      81E1 00020000              and ecx,200  
> 105C721C      85C9                       test ecx,ecx  
> 105C721E      74 0A                      je short CVI\_1.105C722A  
> 105C7220      C785 9CEEFFFF 8AEB9B28     mov dword ptr ss:\[ebp-1164\],289BEB8A  
> ......  
> 105C757D      51                         push ecx                               ; CVI\_FDS\_LWH  
> 105C757E      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C7581      52                         push edx  
> 105C7582      E8 4E130000                call CVI\_1.105C88D5  
> 105C7587      83C4 10                    add esp,10  
> 105C758A      8945 FC                    mov dword ptr ss:\[ebp-4\],eax  
> 105C758D      8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
> 105C7590      50                         push eax  
> 105C7591      8B8D B8EEFFFF              mov ecx,dword ptr ss:\[ebp-1148\]  
> 105C7597      51                         push ecx  
> 105C7598      E8 60290000                call CVI\_1.105C9EFD  
> 105C759D      83C4 08                    add esp,8  
> 105C75A0      8945 FC                    mov dword ptr ss:\[ebp-4\],eax  
> 105C75A3      8B55 FC                    mov edx,dword ptr ss:\[ebp-4\]  
> 105C75A6      52                         push edx  
> 105C75A7      8B85 B0EEFFFF              mov eax,dword ptr ss:\[ebp-1150\]  
> 105C75AD      50                         push eax  
> 105C75AE      E8 4A290000                call CVI\_1.105C9EFD  
> 105C75B3      83C4 08                    add esp,8  
> 105C75B6      8945 FC                    mov dword ptr ss:\[ebp-4\],eax  
> 105C75B9      8B4D 0C                    mov ecx,dword ptr ss:\[ebp+C\]  
> 105C75BC      83B9 68020000 00           cmp dword ptr ds:\[ecx+268\],0           ; COMPONENTS  
> 105C75C3      74 1F                      je short CVI\_1.105C75E4  
> 105C75C5      8B55 FC                    mov edx,dword ptr ss:\[ebp-4\]  
> 105C75C8      52                         push edx  
> 105C75C9      6A 00                      push 0  
> 105C75CB      8B45 0C                    mov eax,dword ptr ss:\[ebp+C\]  
> 105C75CE      8B88 68020000              mov ecx,dword ptr ds:\[eax+268\]  
> 105C75D4      51                         push ecx                               ; COMPONENTS  
> 105C75D5      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C75D8      52                         push edx  
> 105C75D9      E8 F7120000                call CVI\_1.105C88D5  
> 105C75DE      83C4 10                    add esp,10  
> 105C75E1      8945 FC                    mov dword ptr ss:\[ebp-4\],eax  
> 105C75E4      8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
> 105C75E7      50                         push eax  
> 105C75E8      8B4D 0C                    mov ecx,dword ptr ss:\[ebp+C\]  
> 105C75EB      81C1 2B020000              add ecx,22B  
> 105C75F1      51                         push ecx                               ; Start 日期  
> 105C75F2      E8 7C150000                call CVI\_1.105C8B73  
> 105C75F7      83C4 08                    add esp,8  
> 105C75FA      8945 FC                    mov dword ptr ss:\[ebp-4\],eax  
> 105C75FD      83BD 88EEFFFF 00           cmp dword ptr ss:\[ebp-1178\],0  
> 105C7604      74 63                      je short CVI\_1.105C7669  
> 105C7606      83BD BCEEFFFF 00           cmp dword ptr ss:\[ebp-1144\],0  
> 105C760D      74 14                      je short CVI\_1.105C7623  
> 105C760F      8B95 BCEEFFFF              mov edx,dword ptr ss:\[ebp-1144\]  
> 105C7615      52                         push edx  
> 105C7616      E8 75C20A00                call CVI\_1.10673890  
> 105C761B      83C4 04                    add esp,4  
> 105C761E      83F8 04                    cmp eax,4  
> 105C7621      7D 10                      jge short CVI\_1.105C7633  
> 105C7623      C745 10 AC277410           mov dword ptr ss:\[ebp+10\],CVI\_1.107427>; ASCII "FFFF"      
>                 ; 熟悉的面孔  
> ......  
> 105C7BB0      51                         push ecx                               ; 取得 SIGN  
> 105C7BB1      8B95 9CEEFFFF              mov edx,dword ptr ss:\[ebp-1164\]  
> 105C7BB7      52                         push edx                               ; 66D8B337  
> 105C7BB8      8B45 14                    mov eax,dword ptr ss:\[ebp+14\]  
> 105C7BBB      50                         push eax  
> 105C7BBC      8B4D FC                    mov ecx,dword ptr ss:\[ebp-4\]  
> 105C7BBF      8D95 C4EEFFFF              lea edx,dword ptr ss:\[ebp-113C\]  
> 105C7BC5      2BCA                       sub ecx,edx  
> 105C7BC7      51                         push ecx                               ; 3B  
> 105C7BC8      8D85 C4EEFFFF              lea eax,dword ptr ss:\[ebp-113C\]  
> 105C7BCE      50                         push eax  
> 105C7BCF      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C7BD2      51                         push ecx  
> 105C7BD3      E8 0A120000                call CVI\_1.105C8DE2                    ; 应该是 l\_pubkey\_verify 判断 License 文件是否正确的关键函数,F7 进去  
>   
>   
>   
> \----------------------------------------------------------------------------------------------  
> ......  
>   
> 105C8DE2      55                         push ebp  
> 105C8DE3      8BEC                       mov ebp,esp  
> 105C8DE5      81EC D8010000              sub esp,1D8  
> 105C8DEB      53                         push ebx  
> 105C8DEC      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C8DEF      0FBE80 0C030000            movsx eax,byte ptr ds:\[eax+30C\]  
> 105C8DF6      99                         cdq  
> 105C8DF7      B9 14000000                mov ecx,14  
> 105C8DFC      F7F9                       idiv ecx  
> 105C8DFE      8955 F8                    mov dword ptr ss:\[ebp-8\],edx  
> 105C8E01      C785 98FEFFFF 00000000     mov dword ptr ss:\[ebp-168\],0  
> 105C8E0B      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C8E0E      8B82 F8030000              mov eax,dword ptr ds:\[edx+3F8\]  
> 105C8E14      8985 74FEFFFF              mov dword ptr ss:\[ebp-18C\],eax  
> 105C8E1A      C785 80FEFFFF 00000000     mov dword ptr ss:\[ebp-180\],0  
> 105C8E24      C785 88FEFFFF 00000000     mov dword ptr ss:\[ebp-178\],0  
> 105C8E2E      C785 90FEFFFF 00000000     mov dword ptr ss:\[ebp-170\],0  
> 105C8E38      C745 FC 00000000           mov dword ptr ss:\[ebp-4\],0  
> 105C8E3F      6A 0E                      push 0E  
> 105C8E41      8B4D 08                    mov ecx,dword ptr ss:\[ebp+8\]  
> 105C8E44      51                         push ecx  
> 105C8E45      E8 A63C0200                call CVI\_1.105ECAF0  
> 105C8E4A      83C4 08                    add esp,8  
> 105C8E4D      3D ADDE0000                cmp eax,0DEAD  
> 105C8E52      74 15                      je short CVI\_1.105C8E69  
> 105C8E54      6A 14                      push 14  
> 105C8E56      8B55 08                    mov edx,dword ptr ss:\[ebp+8\]  
> 105C8E59      52                         push edx  
> 105C8E5A      E8 913C0200                call CVI\_1.105ECAF0  
> 105C8E5F      83C4 08                    add esp,8  
> 105C8E62      3D 55260000                cmp eax,2655  
> 105C8E67      75 3E                      jnz short CVI\_1.105C8EA7  
> 105C8E69      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C8E6C      0FBE88 0C030000            movsx ecx,byte ptr ds:\[eax+30C\]  
> 105C8E73      85C9                       test ecx,ecx  
> 105C8E75      74 3A                      je short CVI\_1.105C8EB1  
> 105C8E77      68 B8287410                push CVI\_1.107428B8                    ; ASCII "demo"  
>   
> 熟悉的面孔  
> ......  
> 105C9156     /74 0C                      je short CVI\_1.105C9164  
> 105C9158     |C785 88FEFFFF 00000000     mov dword ptr ss:\[ebp-178\],0  
> 105C9162     |EB 0A                      jmp short CVI\_1.105C916E  
> 105C9164     \\C785 88FEFFFF C8287410     mov dword ptr ss:\[ebp-178\],CVI\_1.10742>; ASCII "ok"  
> 105C916E      E9 7F0D0000                jmp CVI\_1.105C9EF2  
>   
> ......  
> 105C9D2A      A2 9E358A10                mov byte ptr ds:\[108A359E\],al  
> 105C9D2F      33C9                       xor ecx,ecx  
> 105C9D31      8A0D 9F358A10              mov cl,byte ptr ds:\[108A359F\]  
> 105C9D37      33D2                       xor edx,edx  
> 105C9D39      8A15 98358A10              mov dl,byte ptr ds:\[108A3598\]  
> 105C9D3F      03CA                       add ecx,edx  
> 105C9D41      81E1 FF000000              and ecx,0FF  
> 105C9D47      81E1 FF000000              and ecx,0FF  
> 105C9D4D      880D 98358A10              mov byte ptr ds:\[108A3598\],cl  
> 105C9D53      33C0                       xor eax,eax  
> 105C9D55      A0 9E358A10                mov al,byte ptr ds:\[108A359E\]  
> 105C9D5A      33C9                       xor ecx,ecx  
> 105C9D5C      8A0D 99358A10              mov cl,byte ptr ds:\[108A3599\]  
> 105C9D62      03C1                       add eax,ecx  
> 105C9D64      25 FF000000                and eax,0FF  
> 105C9D69      25 FF000000                and eax,0FF  
> 105C9D6E      A2 99358A10                mov byte ptr ds:\[108A3599\],al  
> 105C9D73      C605 9F358A10 00           mov byte ptr ds:\[108A359F\],0  
> 105C9D7A      8A15 9F358A10              mov dl,byte ptr ds:\[108A359F\]  
> 105C9D80      8815 9E358A10              mov byte ptr ds:\[108A359E\],dl  
> 105C9D86      C785 78FEFFFF 08000000     mov dword ptr ss:\[ebp-188\],8  
> 105C9D90      817D 18 37B3D866           cmp dword ptr ss:\[ebp+18\],66D8B337  
> 105C9D97     /75 0F                      jnz short CVI\_1.105C9DA8  
> 105C9D99     |8B85 78FEFFFF              mov eax,dword ptr ss:\[ebp-188\]  
> 105C9D9F     |83E8 02                    sub eax,2  
> 105C9DA2     |8985 78FEFFFF              mov dword ptr ss:\[ebp-188\],eax  
> 105C9DA8     \\C785 7CFEFFFF 00000000     mov dword ptr ss:\[ebp-184\],0  
> 105C9DB2      EB 0F                      jmp short CVI\_1.105C9DC3  
> 105C9DB4      8B8D 7CFEFFFF              mov ecx,dword ptr ss:\[ebp-184\]  
> 105C9DBA      83C1 01                    add ecx,1  
> 105C9DBD      898D 7CFEFFFF              mov dword ptr ss:\[ebp-184\],ecx  
> 105C9DC3      8B95 7CFEFFFF              mov edx,dword ptr ss:\[ebp-184\]  
> 105C9DC9      3B95 78FEFFFF              cmp edx,dword ptr ss:\[ebp-188\]  
> 105C9DCF      0F8D 02010000              jge CVI\_1.105C9ED7  
> 105C9DD5      8B85 7CFEFFFF              mov eax,dword ptr ss:\[ebp-184\]  
> 105C9DDB      8A8C45 9CFEFFFF            mov cl,byte ptr ss:\[ebp+eax\*2-164\]  
> 105C9DE2      888D 34FEFFFF              mov byte ptr ss:\[ebp-1CC\],cl  
> 105C9DE8      0FBE95 34FEFFFF            movsx edx,byte ptr ss:\[ebp-1CC\]  
> 105C9DEF      52                         push edx  
> 105C9DF0      E8 5B300300                call CVI\_1.105FCE50  
>   
> 在 call CVI\_1.105FCE50 之后,d 108a359e  
>   
> 108A359E  25C00000  ..?  
> 108A35A2  0000105C  \\..  
> 没看见，滚动数据窗口看到了吧：  
> 108A3596  62B30000  ..x(  
> 108A359A  B84926A9  ?I  
> 看到没有 SIGN=B362A92649B8  
> 呵呵，我估计这就是 laoqian 的明码比较办法在 FLEXnet Licensing v10.1.0 中的小小改通吧.  
> ......  
> 105C9EB0      8B95 30FEFFFF              mov edx,dword ptr ss:\[ebp-1D0\]  
> 105C9EB6      81E2 FF000000              and edx,0FF  
> 105C9EBC      8B85 7CFEFFFF              mov eax,dword ptr ss:\[ebp-184\]  
> 105C9EC2      33C9                       xor ecx,ecx  
> 105C9EC4      8A88 98358A10              mov cl,byte ptr ds:\[eax+108A3598\]  
> 105C9ECA      3BD1                       cmp edx,ecx                            ; 看到没有，明码比较  
> 105C9ECC      74 04                      je short CVI\_1.105C9ED2  
> 105C9ECE      33C0                       xor eax,eax  
> 105C9ED0      EB 26                      jmp short CVI\_1.105C9EF8  
> 105C9ED2    ^ E9 DDFEFFFF                jmp CVI\_1.105C9DB4  
> 105C9ED7      8B55 18                    mov edx,dword ptr ss:\[ebp+18\]  
> 105C9EDA      52                         push edx  
> 105C9EDB      68 98358A10                push CVI\_1.108A3598  
> 105C9EE0      8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
> 105C9EE3      50                         push eax  
> 105C9EE4      E8 E9000000                call CVI\_1.105C9FD2  
> 105C9EE9      83C4 0C                    add esp,0C                             ; 返回真正的 SIGN  
> 105C9EEC      8985 88FEFFFF              mov dword ptr ss:\[ebp-178\],eax  
> 105C9EF2      8B85 88FEFFFF              mov eax,dword ptr ss:\[ebp-178\]  
> 105C9EF8      5B                         pop ebx  
> 105C9EF9      8BE5                       mov esp,ebp  
> 105C9EFB      5D                         pop ebp  
> 105C9EFC      C3                         retn 

 4dd466d0e1e0cc123afbe504da2462ca