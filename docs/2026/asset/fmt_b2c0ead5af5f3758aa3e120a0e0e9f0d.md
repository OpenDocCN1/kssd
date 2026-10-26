
```
00425D51   mov dword ptr ds:[E70D18],eax
00425D69   mov eax,dword ptr ds:[E70D18]                     [00E70D18]=001D1820
00425DEE   mov ebx,dword ptr ds:[E70D18]                     ds:[00E70D18]=001D1820, (ASCII "BIN97E69658")
00425E04   mov dword ptr ds:[E70D18],eax                     保存注册码,阴险,先计算好注册码^\_^
00425E70   mov eax,dword ptr ds:[E70D18]                     [00E70D18]=001D1820
00427865   mov eax,dword ptr ds:[E70D18]                     已经计算好的注册码??
004278E0   mov eax,dword ptr ds:[E70D18]                     [00E70D18]=001D1820
00427B58   mov ebx,dword ptr ds:[E70D18]                     ds:[00E70D18]=001D1820, (ASCII "BIN97E69658")
```

在`00425E04`处`mov dword ptr ds:[E70D18],eax`赋值，双击反汇编中跟随，向上来到：

```
00425B42   |. E8 84200000     call qf.00427BCB           ; 取机器特征码
00425B47   |. 83C4 04         add esp,4
00425B4A   |. 8945 E4         mov dword ptr ss:[ebp-1C],eax
00425B4D   |. DB45 E4         fild dword ptr ss:[ebp-1C]
00425B50   |. DD5D E4         fstp qword ptr ss:[ebp-1C]
00425B53   |. DD45 E4         fld qword ptr ss:[ebp-1C]
00425B56   |. DC05 E7FC4000     fadd qword ptr ds:[40FCE7]
00425B5C   |. DD5D DC         fstp qword ptr ss:[ebp-24]
00425B5F   |. 68 01060080     push 80000601
00425B64   |. FF75 E0         push dword ptr ss:[ebp-20]
00425B67   |. FF75 DC         push dword ptr ss:[ebp-24]
00425B6A   |. 68 01000000     push 1
00425B6F   |. BB 68010000     mov ebx,168
00425B74   |. E8 52200000     call qf.00427BCB
00425B79   |. 83C4 10         add esp,10
00425B7C   |. 8945 D8         mov dword ptr ss:[ebp-28],eax
00425B7F   |. 8B1D 140DE700     mov ebx,dword ptr ds:[E70D14]
00425B85   |. 85DB           test ebx,ebx
00425B87   |. 74 09         je short qf.00425B92
00425B89   |. 53           push ebx
00425B8A   |. E8 36200000     call qf.00427BC5
00425B8F   |. 83C4 04         add esp,4
00425B92   |> 8B45 D8         mov eax,dword ptr ss:[ebp-28]
00425B95   |. A3 140DE700     mov dword ptr ds:[E70D14],eax
00425B9A   |. 68 04000080     push 80000004
00425B9F   |. 6A 00         push 0
00425BA1   |. A1 140DE700     mov eax,dword ptr ds:[E70D14]
00425BA6   |. 85C0           test eax,eax
00425BA8   |. 75 05         jnz short qf.00425BAF
00425BAA   |. B8 C7924000     mov eax,qf.004092C7
00425BAF   |> 50           push eax
00425BB0   |. 68 01000000     push 1
00425BB5   |. BB 98010000     mov ebx,198
00425BBA   |. E8 0C200000     call qf.00427BCB
00425BBF   |. 83C4 10         add esp,10
00425BC2   |. 8945 EC         mov dword ptr ss:[ebp-14],eax
00425BC5   |. 68 05000080     push 80000005
00425BCA   |. 6A 00         push 0
00425BCC   |. 8B45 EC         mov eax,dword ptr ss:[ebp-14]
00425BCF   |. 85C0           test eax,eax
00425BD1   |. 75 05         jnz short qf.00425BD8
00425BD3   |. B8 6CF94000     mov eax,qf.0040F96C
00425BD8   |> 50           push eax
00425BD9   |. 68 01000000     push 1
00425BDE   |. BB 08000000     mov ebx,8
00425BE3   |. B8 05000000     mov eax,5
00425BE8   |. E8 F01F0000     call qf.00427BDD
00425BED   |. 83C4 10         add esp,10
00425BF0   |. 8945 E8         mov dword ptr ss:[ebp-18],eax
00425BF3   |. 8B5D EC         mov ebx,dword ptr ss:[ebp-14]
00425BF6   |. 85DB           test ebx,ebx
00425BF8   |. 74 09         je short qf.00425C03
00425BFA   |. 53           push ebx
00425BFB   |. E8 C51F0000     call qf.00427BC5
00425C00   |. 83C4 04         add esp,4
00425C03   |> 8B1D 140DE700     mov ebx,dword ptr ds:[E70D14]
00425C09   |. 85DB           test ebx,ebx
00425C0B   |. 74 09         je short qf.00425C16
00425C0D   |. 53           push ebx
00425C0E   |. E8 B21F0000     call qf.00427BC5           ; MD5,计算机器特征码MD5值
00425C13   |. 83C4 04         add esp,4
00425C16   |> 8B45 E8         mov eax,dword ptr ss:[ebp-18]
00425C19   |. A3 140DE700     mov dword ptr ds:[E70D14],eax
00425C1E   |. 68 01030080     push 80000301
00425C23   |. 6A 00         push 0
00425C25   |. 68 08000000     push 8
00425C2A   |. 68 04000080     push 80000004
00425C2F   |. 6A 00         push 0
00425C31   |. A1 140DE700     mov eax,dword ptr ds:[E70D14]
00425C36   |. 85C0           test eax,eax
00425C38   |. 75 05         jnz short qf.00425C3F
00425C3A   |. B8 C7924000     mov eax,qf.004092C7
00425C3F   |> 50           push eax
00425C40   |. 68 02000000     push 2
00425C45   |. BB 34010000     mov ebx,134
00425C4A   |. E8 7C1F0000     call qf.00427BCB           ; 取前8为作为机器码
00425C4F   |. 83C4 1C         add esp,1C
00425C52   |. 8945 EC         mov dword ptr ss:[ebp-14],eax
00425C55   |. 68 04000080     push 80000004
00425C5A   |. 6A 00         push 0
00425C5C   |. 8B45 EC         mov eax,dword ptr ss:[ebp-14]
00425C5F   |. 85C0           test eax,eax
00425C61   |. 75 05         jnz short qf.00425C68
00425C63   |. B8 C7924000     mov eax,qf.004092C7
00425C68   |> 50           push eax
00425C69   |. 68 01000000     push 1
00425C6E   |. BB 50010000     mov ebx,150
00425C73   |. E8 531F0000     call qf.00427BCB
00425C78   |. 83C4 10         add esp,10
00425C7B   |. 8945 E8         mov dword ptr ss:[ebp-18],eax     ; 机器码,也是由机器特征码的MD5值所得;
00425C7E   |. 8B5D EC         mov ebx,dword ptr ss:[ebp-14]
00425C81   |. 85DB           test ebx,ebx
00425C83   |. 74 09         je short qf.00425C8E
00425C85   |. 53           push ebx
00425C86   |. E8 3A1F0000     call qf.00427BC5
00425C8B   |. 83C4 04         add esp,4
00425C8E   |> 8B1D 140DE700     mov ebx,dword ptr ds:[E70D14]
00425C94   |. 85DB           test ebx,ebx
00425C96   |. 74 09         je short qf.00425CA1
00425C98   |. 53           push ebx
00425C99   |. E8 271F0000     call qf.00427BC5
00425C9E   |. 83C4 04         add esp,4
00425CA1   |> 8B45 E8         mov eax,dword ptr ss:[ebp-18]
00425CA4   |. A3 140DE700     mov dword ptr ds:[E70D14],eax
00425CA9   |. 68 EFFC4000     push qf.0040FCEF           ; /Arg2 = 0040FCEF ASCII "e622"
00425CAE   |. FF35 140DE700     push dword ptr ds:[E70D14]       ; |Arg1 = 001CE150 ASCII "F6D40AB6"
00425CB4   |. B9 02000000     mov ecx,2                 ; |
00425CB9   |. E8 13BCFFFF     call qf.004218D1           ; \qf.004218D1
00425CBE   |. 83C4 08         add esp,8
00425CC1   |. 8945 EC         mov dword ptr ss:[ebp-14],eax
00425CC4   |. 68 04000080     push 80000004
00425CC9   |. 6A 00         push 0
00425CCB   |. 8B45 EC         mov eax,dword ptr ss:[ebp-14]     ; 机器码+e622
00425CCE   |. 85C0           test eax,eax
00425CD0   |. 75 05         jnz short qf.00425CD7
00425CD2   |. B8 C7924000     mov eax,qf.004092C7
00425CD7   |> 50           push eax
00425CD8   |. 68 01000000     push 1
00425CDD   |. BB 98010000     mov ebx,198
00425CE2   |. E8 E41E0000     call qf.00427BCB
00425CE7   |. 83C4 10         add esp,10
00425CEA   |. 8945 E8         mov dword ptr ss:[ebp-18],eax
00425CED   |. 8B5D EC         mov ebx,dword ptr ss:[ebp-14]
00425CF0   |. 85DB           test ebx,ebx
00425CF2   |. 74 09         je short qf.00425CFD
00425CF4   |. 53           push ebx
00425CF5   |. E8 CB1E0000     call qf.00427BC5
00425CFA   |. 83C4 04         add esp,4
00425CFD   |> 68 05000080     push 80000005
00425D02   |. 6A 00         push 0
00425D04   |. 8B45 E8         mov eax,dword ptr ss:[ebp-18]
00425D07   |. 85C0           test eax,eax
00425D09   |. 75 05         jnz short qf.00425D10
00425D0B   |. B8 6CF94000     mov eax,qf.0040F96C
00425D10   |> 50           push eax
00425D11   |. 68 01000000     push 1
00425D16   |. BB 08000000     mov ebx,8
00425D1B   |. B8 05000000     mov eax,5
00425D20   |. E8 B81E0000     call qf.00427BDD           ; MD5
00425D25   |. 83C4 10         add esp,10
00425D28   |. 8945 E4         mov dword ptr ss:[ebp-1C],eax     ; EAX=MD5(机器码码+e622)
00425D2B   |. 8B5D E8         mov ebx,dword ptr ss:[ebp-18]     ; EAX=001D6CF0, (ASCII "97e69658d95de2831573bf8651504e06")
00425D2E   |. 85DB           test ebx,ebx
00425D30   |. 74 09         je short qf.00425D3B
00425D32   |. 53           push ebx
```
