
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401B5B(C)

```
:00401BBC 66C743105C00            mov [ebx+10],
005C
:00401BC2 33C0
  xor eax, eax
:00401BC4 8945C4
mov dword ptr [ebp-3C], eax
:00401BC7 8D55C4
lea edx, dword ptr [ebp-3C]
:00401BCA FF431C
inc [ebx+1C]
:00401BCD 8B86D8020000            mov eax, dword
ptr [esi+000002D8]
:00401BD3 E880A50D00              call
004DC158
:00401BD8 8D55C4
lea edx, dword ptr [ebp-3C]
:00401BDB 33C0
  xor eax, eax
:00401BDD 8B0A
  mov ecx, dword ptr [edx]
:00401BDF 8D55C8
lea edx, dword ptr [ebp-38]
:00401BE2 51
    push ecx
:00401BE3 8945C8
mov dword ptr [ebp-38], eax
:00401BE6 FF431C
inc [ebx+1C]
:00401BE9 8B86DC020000            mov eax, dword
ptr [esi+000002DC]
:00401BEF E864A50D00              call
004DC158
:00401BF4 8D45C8
lea eax, dword ptr [ebp-38]
:00401BF7 8B00
  mov eax, dword ptr [eax]
:00401BF9 33D2
  xor edx, edx
:00401BFB 8955FC
mov dword ptr [ebp-04], edx
:00401BFE 8D4DFC
lea ecx, dword ptr [ebp-04]
:00401C01 FF431C
inc [ebx+1C]
:00401C04 5A
    pop edx
:00401C05 E8762C0100              call
00414880
:00401C0A FF4B1C
dec [ebx+1C]
:00401C0D 8D45C4
lea eax, dword ptr [ebp-3C]
:00401C10 BA02000000              mov edx,
00000002
:00401C15 E8CE711200              call
00528DE8
:00401C1A FF4B1C
dec [ebx+1C]
:00401C1D 8D45C8
lea eax, dword ptr [ebp-38]
:00401C20 BA02000000              mov edx,
00000002
:00401C25 E8BE711200              call
00528DE8
:00401C2A 66C743105000            mov [ebx+10],
0050
:00401C30 66C743106800            mov [ebx+10],
0068
:00401C36 33C0
  xor eax, eax
:00401C38 8945C0
mov dword ptr [ebp-40], eax
:00401C3B 8D55C0
lea edx, dword ptr [ebp-40]
:00401C3E FF431C
inc [ebx+1C]
:00401C41 8B86E0020000            mov eax, dword
ptr [esi+000002E0]
:00401C47 E80CA50D00              call
004DC158
:00401C4C 8D55C0
lea edx, dword ptr [ebp-40]
:00401C4F 8D45FC
lea eax, dword ptr [ebp-04]
:00401C52 E875721200              call
00528ECC
:00401C57 50
    push eax
:00401C58 FF4B1C
dec [ebx+1C]
:00401C5B 8D45C0
lea eax, dword ptr [ebp-40]
:00401C5E BA02000000              mov edx,
00000002
:00401C63 E880711200              call
00528DE8            //恭喜你，你已成功注册（哈……哈……，快成功了！）
:00401C68 59
    pop ecx
:00401C69 84C9
  test cl, cl
:00401C6B 0F84C0000000            je 00401D31　　　　　　　//下
r fl z ,F5退出，可看到上面提示
:00401C71 66C743107400            mov [ebx+10],
0074
:00401C77 33C0
  xor eax, eax
:00401C79 8945BC
mov dword ptr [ebp-44], eax
:00401C7C 8D55BC
lea edx, dword ptr [ebp-44]
:00401C7F FF431C
inc [ebx+1C]
:00401C82 8B86D8020000            mov eax, dword
ptr [esi+000002D8]
:00401C88 E8CBA40D00              call
004DC158
:00401C8D 8D55BC
lea edx, dword ptr [ebp-44]
:00401C90 33C0
  xor eax, eax
:00401C92 8B0A
  mov ecx, dword ptr [edx]
:00401C94 8D55B8
lea edx, dword ptr [ebp-48]
:00401C97 51
    push ecx
:00401C98 8945B8
mov dword ptr [ebp-48], eax
:00401C9B FF431C
inc [ebx+1C]
:00401C9E 8B86DC020000            mov eax, dword
ptr [esi+000002DC]
:00401CA4 E8AFA40D00              call
004DC158
:00401CA9 8D55B8
lea edx, dword ptr [ebp-48]
:00401CAC 8B0A
  mov ecx, dword ptr [edx]
:00401CAE 51
    push ecx
:00401CAF E818340100              call
004150CC
:00401CB4 83C408
add esp, 00000008
:00401CB7 FF4B1C
dec [ebx+1C]
:00401CBA 8D45B8
lea eax, dword ptr [ebp-48]
:00401CBD BA02000000              mov edx,
00000002
//D EDX ,ALT+↑可看到注册码
:00401CC2 E821711200              call
00528DE8
:00401CC7 FF4B1C
dec [ebx+1C]
:00401CCA 8D45BC
lea eax, dword ptr [ebp-44]
:00401CCD BA02000000              mov edx,
00000002
:00401CD2 E811711200              call
00528DE8
:00401CD7 66C743108000            mov [ebx+10],
0080
:00401CDD 8D573D
lea edx, dword ptr [edi+3D]
:00401CE0 8D45B4
lea eax, dword ptr [ebp-4C]
:00401CE3 E8A06F1200              call
00528C88
:00401CE8 FF431C
inc [ebx+1C]
:00401CEB 8D45B4
lea eax, dword ptr [ebp-4C]
:00401CEE 33D2
  xor edx, edx
:00401CF0 8955B0
mov dword ptr [ebp-50], edx
:00401CF3 8D55B0
lea edx, dword ptr [ebp-50]
:00401CF6 FF431C
inc [ebx+1C]
:00401CF9 E8F6A60100              call
0041C3F4
:00401CFE 8D45B0
lea eax, dword ptr [ebp-50]
:00401D01 8B00
  mov eax, dword ptr [eax]
:00401D03 E8B03E0100              call
00415BB8                //弹出“恭喜你，你已成功注册”窗口
```

****************************************************************************

大家明白了吗？r fl z 并不会真的修改程序，只是起到模拟的作用，用它来验证某些 CALL 的功能非常好用，象我这种不太懂汇编的初学者来说，看到可疑的 CALL 与 jne,je 时，在jne/je 处下 r fl z ，再 F5 退出，看一看提示，就毋须象以前那样乱 D  or ？一通，甚至 F8 跟入，徒劳无功了。如象 00401C63 处，在 00401C6B je 00401D31 处下 r fl z,弹出成功窗口，说明这个call有问题，虽然跟进这个 call 只能看到正确注册码的前三位，但很多软件还是用这种办法直接找到注册码或 F8 跟入再找到注册码的。

注：1、各位验证某个 call 时最好先 BD * 中断之前的所有断点，再在这个 call 设断，下 r fl z ，F5 退出后，如果发现不是验证注册码的 call ，按确定一般都可以被 TRW2000,SOFT-ICE 拦截，以便继续验证。（因为验证可能不止一次，如上例，以免重复作无谓的劳动。）

2、00401C6B je 00401D31 处必须下 r fl z，否则你是不可能来到00401CBD mov edx, 00000002的，切记，切记！
