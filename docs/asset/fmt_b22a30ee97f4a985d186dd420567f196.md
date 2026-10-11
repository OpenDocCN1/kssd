
# 沸点邮件群发系统 v1.2 标准版

> 原文：[`www.pediy.com/kssd/pediy03/forum1239.htm`](https://www.pediy.com/kssd/pediy03/forum1239.htm)

软件下载：http://www.21swe.com/   

首先找到`mail.exe`的注册错误提示信息：“您的注册名称或注册号不对，不能注册”   

因为真假注册码比较以后才会出现这个提示   

========   

`W32Dasm`反汇编和`TRW2000`一起使用   

* 用`language`查看，程序没有加壳   

* 用`W32Dasm`反汇编，根据“串式参考”找到注册错误提示信息“您的注册名称或注册号不对，不能注册”，双击   

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`   

```
:00404D51(C)   

:00404FEC A100C94D00  mov eax, dword ptr [004DC900]   
:00404FF1 6A00  push 00000000   

* Possible StringData Ref from Data Obj ->"错误提示"   

:00404FF3 B9E2534D00  mov ecx, 004D53E2   

* Possible StringData Ref from Data Obj ->"您的注册名称或注册号不对，不能注册!"   

:00404FF8 BABE534D00  mov edx, 004D53BE   
:00404FFD 8B00  mov eax, dword ptr [eax]   
:00404FFF E8C0870B00  call 004BD7C4
```

发现是从`00404D51`跳转过来的，鼠标右键双击`00404D51`

```
:00404D1E 8945C8  mov dword ptr [ebp-38], eax   
:00404D21 8D55C8  lea edx, dword ptr [ebp-38]   
:00404D24 FF461C  inc [esi+1C]   
:00404D27 8B83E8020000  mov eax, dword ptr [ebx+000002E8]   
:00404D2D E8FE210700  call 00476F30   
:00404D32 8D45C8  lea eax, dword ptr [ebp-38]   
:00404D35 8D55FC  lea edx, dword ptr [ebp-04]   
:00404D38 E8F38C0B00  call 004BDA30   
:00404D3D 50  push eax   
:00404D3E FF4E1C  dec [esi+1C]   
:00404D41 8D45C8  lea eax, dword ptr [ebp-38]   
:00404D44 BA02000000  mov edx, 00000002   
:00404D49 E8128C0B00  call 004BD960   
:00404D4E 59  pop ecx   
:00404D4F 84C9  test cl, cl   
:00404D51 0F8495020000  je 00404FEC   
:00404D57 B201  mov dl, 01   
:00404D59 A1F4A44400  mov eax, dword ptr [0044A4F4]   
:00404D5E E8E9580400  call 0044A64C   
:00404D63 8BF8  mov edi, eax   
:00404D65 89BBF4020000  mov dword ptr [ebx+000002F4], edi   
:00404D6B BA02000080  mov edx, 80000002   
:00404D70 8BC7  mov eax, edi   
:00404D72 E85D8A0B00  call 004BD7D4   
:00404D77 66C746102C00  mov [esi+10], 002C
```

对照“风飘雪”的破解教程，发现可疑的关键`Call`在`00404D38`   

打开`TRW2000`   

在`mail.exe`中的注册栏中填入：用户名“mazhenyu”和注册号“87654321”，但不点击“立即注册”   

“Ctrl+N”激活`TRW2000`   

在`00404D35`处下断点：`bpx 00404D35`，回车，然后按“F5”退出   

点击`mail.exe`注册栏中的“立即注册”   

程序被中断   

按一下“F10”来到`00404D38`，没有发现注册码，按“F8”进入`Call`以进一步追踪   

```
0137:004BDA2F  RET   
0137:004BDA30  PUSH  EBP   
0137:004BDA31  MOV     EBP,ESP   
0137:004BDA33  PUSH  EBX   
0137:004BDA34  MOV     EAX,[EAX]   
0137:004BDA36  MOV     EDX,[EDX]   
0137:004BDA38  CALL  004ACE0C   
0137:004BDA3D  SETZ  AL   
0137:004BDA40  AND     EAX,BYTE +01   
0137:004BDA43  POP     EBX   
0137:004BDA44  POP     EBP   
0137:004BDA45  RET   
0137:004BDA46  NOP   
0137:004BDA47  NOP   
0137:004BDA48  PUSH  EBP
```

按“F10”向下移动绿色光条，结果在`004BDA38`处找到真假注册码：   

`d eax=87654321`   

`d edx=356088`   

将用户名“mazhenyu”和注册号“356088”进行注册，注册成功！   

太棒了！   

马震宇   

2001.8.16.  4dd466d0e1e0cc123afbe504da2462ca
