
```
004016D8 . 52 push edx ; |FileName => "KwazyWeb.bit"
004016D9 . E8 1C010000 call <jmp.&KERNEL32.CreateFileA> ; \\CreateFileA
004016DE . 83F8 FF cmp eax, -1 ; 断下返回到此，EAX 是文件句柄
004016E1 . 74 64 je short PackMe.00401747 ; 没有注册文件就跳，跳了就完蛋
004016E3 . A3 44344000 mov dword ptr \[403444\], eax ; 保存文件句柄到全局变量
004016E8 . 6A 00 push 0 ; /pOverlapped = NULL
004016EA . 68 48344000 push PackMe.00403448 ; |pBytesRead = PackMe.00403448
004016EF . 6A 01 push 1 ; |BytesToRead = 1
004016F1 . 68 FA344000 push PackMe.004034FA ; |Buffer = PackMe.004034FA
004016F6 . FF35 44344000 push dword ptr \[403444\] ; |hFile = NULL
004016FC . E8 11010000 call <jmp.&KERNEL32.ReadFile> ; \\ReadFile　　从注册文件读取一个字节，送到 004034FA
00401701 . 0FB605 FA3440>movzx eax, byte ptr \[4034FA\] ; 把读来的字节扩展送 EAX
00401708 . 85C0 test eax, eax
0040170A . 74 3B je short PackMe.00401747 ; 字符为 0　就跳，跳就完蛋
0040170C . 6A 00 push 0 ; /pOverlapped = NULL
0040170E . 68 48344000 push PackMe.00403448 ; |pBytesRead = PackMe.00403448
00401713 . 50 push eax ; |BytesToRead
00401714 . 68 88324000 push PackMe.00403288 ; |Buffer = PackMe.00403288
00401719 . FF35 44344000 push dword ptr \[403444\] ; |hFile = NULL
0040171F . E8 EE000000 call <jmp.&KERNEL32.ReadFile> ; \\ReadFile 再接着读取 49 个字节，送到 00403288
00401724 . E8 D7F8FFFF call PackMe.00401000 ; 以首字节大小为次数，计算后面的字节数据和，取和的低 8 位
{00401000 /$ 33C0 xor eax, eax
00401002 |. 33D2 xor edx, edx
00401004 |. 33C9 xor ecx, ecx
00401006 |. 8A0D FA344000 mov cl, byte ptr \[4034FA\] ; 注册文件第一字节送 CL
0040100C |. BE 88324000 mov esi, PackMe.00403288 ; ASCII "23456789"
00401011 |> AC /lods byte ptr \[esi\]
00401012 |. 03D0 |add edx, eax ; 每字节数据累加存于 EDX
00401014 |.^ E2 FB \\loopd short PackMe.00401011 ; 循环次数是 CL=注册文件第一字节数值
00401016 |. 8815 FB344000 mov byte ptr \[4034FB\], dl ; DL 送 004034FB 保存
0040101C \\. C3 ret
}
00401729 . 6A 00 push 0 ; /pOverlapped = NULL
0040172B . 68 48344000 push PackMe.00403448 ; |pBytesRead = PackMe.00403448
00401730 . 6A 12 push 12 ; |BytesToRead = 12 (18.)
00401732 . 68 E8344000 push PackMe.004034E8 ; |Buffer = PackMe.004034E8
00401737 . FF35 44344000 push dword ptr \[403444\] ; |hFile = NULL
0040173D . E8 D0000000 call <jmp.&KERNEL32.ReadFile> ; \\ReadFile 再读取 18 个字节，存于 004034E8
00401742 . E8 82F9FFFF call PackMe.004010C9 ; 这里是一个算法，我们跟 F7 进去
{004010C9 处的函数，参见下面的“最核心算法”分析
也就是在这个算法里，我们一步一步观察 OD，就可以发现迷宫的数据源：
004010CF |. 68 65334000 push PackMe.00403365 ; /String2 = 
 
"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*C\*......\*...\*\*\*\*.\*.\*\*\*\*...\*....\*.\*..\*\*\*\*\*\*\*\*\*\*.\*..\*....\*...\*...\*\*.\*\*\*\*.\*.\*...\*\*\*\*.\*....\*.\*\*\*\*\*\*\*..\*.\*\*\*..\*..
 
...\*.\*..\*\*\*.\*\*.\*\*\*.\*...\*\*\*\*....\*X..\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"
/-------------------------------------------------------------------------------------------\\
00403365 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
00403375 43 2A 2E 2E 2E 2E 2E 2E 2A 2E 2E 2E 2A 2A 2A 2A C\*......\*...\*\*\*\*
00403385 2E 2A 2E 2A 2A 2A 2A 2E 2E 2E 2A 2E 2E 2E 2E 2A .\*.\*\*\*\*...\*....\*
00403395 2E 2A 2E 2E 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2E 2A .\*..\*\*\*\*\*\*\*\*\*\*.\*
004033A5 2E 2E 2A 2E 2E 2E 2E 2A 2E 2E 2E 2A 2E 2E 2E 2A ..\*....\*...\*...\*
004033B5 2A 2E 2A 2A 2A 2A 2E 2A 2E 2A 2E 2E 2E 2A 2A 2A \*.\*\*\*\*.\*.\*...\*\*\*
004033C5 2A 2E 2A 2E 2E 2E 2E 2A 2E 2A 2A 2A 2A 2A 2A 2A \*.\*....\*.\*\*\*\*\*\*\*
004033D5 2E 2E 2A 2E 2A 2A 2A 2E 2E 2A 2E 2E 2E 2E 2E 2A ..\*.\*\*\*..\*.....\*
004033E5 2E 2A 2E 2E 2A 2A 2A 2E 2A 2A 2E 2A 2A 2A 2E 2A .\*..\*\*\*.\*\*.\*\*\*.\*
004033F5 2E 2E 2E 2A 2A 2A 2A 2E 2E 2E 2E 2A 58 2E 2E 2A ...\*\*\*\*....\*X..\*
00403405 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A 2A \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\\-------------------------------------------------------------------------------------------/
}
00401747 > FF35 44344000 push dword ptr \[403444\] ; /hObject = NULL
0040174D . E8 A2000000 call <jmp.&KERNEL32.CloseHandle> ; \\CloseHandle
00401752 > EB 15 jmp short PackMe.00401769
00401754 > FF75 14 push dword ptr \[ebp+14\] ; /lParam; Default case of switch 004012D8
00401757 . FF75 10 push dword ptr \[ebp+10\] ; |wParam
0040175A . FF75 0C push dword ptr \[ebp+C\] ; |Message
0040175D . FF75 08 push dword ptr \[ebp+8\] ; |hWnd
00401760 . E8 17000000 call <jmp.&USER32.DefWindowProcA> ; \\DefWindowProcA
00401765 . C9 leave
00401766 . C2 1000 ret 10
00401769 > 33C0 xor eax, eax
0040176B . C9 leave
0040176C . C2 1000 ret 10
```

/-----------------------------------最核心的算法-----------------------------------------------\\  

代码:

```
 
00401033 $ 55 push ebp
00401034 . 8BEC mov ebp, esp
00401036 . 83C4 F8 add esp, -8
00401039 . 8B15 84314000 mov edx, dword ptr \[403184\] ; 一看就是一个全局变量，用直接寻址，设为 lpCursor(因为一会儿就会发现，这是保存当前所在迷宫位置的指针)
0040103F . 8955 FC mov dword ptr \[ebp-4\], edx ; 一看\[EBP-4\]就是用局部变量，设为\_dwCur，即 dwCursor=\_dwCur
00401042 . 0AC0 or al, al ; 或运算，就是检查 AL 是否为 0; Switch (cases 0..2)
00401044 . 75 09 jnz short PackMe.0040104F ; AL 不为 0 就跳
00401046 . 832D 84314000>sub dword ptr \[403184\], 10 ; lpCursor-10h，因为迷宫一行是 16 个字符，所以可知道是向上移动一步。说明 0 是向上; Case 0 of switch 00401042
0040104D . EB 1F jmp short PackMe.0040106E ; 移动完毕，跳之
0040104F > 3C 01 cmp al, 1
00401051 . 75 08 jnz short PackMe.0040105B ; AL 不为 1 就跳
00401053 . FF05 84314000 inc dword ptr \[403184\] ; AL 为 1 就 lpCursor+1,即向右移动一位，说明 AL=1 是向右移动; Case 1 of switch 00401042
00401059 . EB 13 jmp short PackMe.0040106E ; 移动完毕，跳之
0040105B > 3C 02 cmp al, 2
0040105D . 75 09 jnz short PackMe.00401068 ; AL 不为 2 就跳
0040105F . 8305 84314000>add dword ptr \[403184\], 10 ; lpCursor+10h,一行迷宫是 16 个字符，所以可知 AL=2 是向下移动一位; Case 2 of switch 00401042
00401066 . EB 06 jmp short PackMe.0040106E ; 移动完毕，跳之
00401068 > FF0D 84314000 dec dword ptr \[403184\] ; 因为 AL 是对 3 求余，所以取值只能为 0~3，运行到这条代码，说明只能 AL=3 了，说明是向左移动一位，即 lpCursor-1; Default case of switch 00401042
0040106E > 8B15 84314000 mov edx, dword ptr \[403184\] ; 在此 F4，运行到此
00401074 . 8A02 mov al, byte ptr \[edx\] ; 本条和上面一条代码一起看，发现，这不就是取指针嘛！果然明白了，取移动后，当前所以地的值。
00401076 . 3C 2A cmp al, 2A ; 与 2Ah 即'\*'比较
00401078 . 75 06 jnz short PackMe.00401080 ; 撞墙啦，说明没走出迷宫，隔屁了。
0040107A . 33C0 xor eax, eax
0040107C . C9 leave
0040107D . C3 ret ; 隔屁(就是挂了的意思。)
0040107E . EB 33 jmp short PackMe.004010B3
00401080 > 3C 58 cmp al, 58 ; 58h 即 ASCII 的'X',观察迷宫可知道，X 多半是表示出口。
00401082 ^\\75 90 jnz short PackMe.00401014 ; 不是出口，说明还要继续走(注意，此处可以爆破)
00401084 . 6A 00 push 0 ; /Style = MB\_OK|MB\_APPLMODAL
00401086 . 8D15 59334000 lea edx, dword ptr \[403359\] ; |
0040108C . 52 push edx ; |Title => "Success.."
0040108D . 8D15 EC324000 lea edx, dword ptr \[4032EC\] ; |
00401093 . 52 push edx ; |Text => "Congratulations!",LF,CR,"Mail me (KwazyWebbit@hotmail.com) how you did it.",LF,CR,"Dont forget to include your keyfile! =\]"
00401094 . 6A 00 push 0 ; |hOwner = NULL
00401096 . 8D15 AC174000 lea edx, dword ptr \[4017AC\] ; |
0040109C . FFD2 call edx ; \\MessageBoxA
0040109E . 8D15 7B324000 lea edx, dword ptr \[40327B\]
004010A4 . 52 push edx ; /Text => "Cracked by : 23456789"
004010A5 . FF35 20344000 push dword ptr \[403420\] ; |hWnd = 000204C2 ('UNREGISTERED!',class='Edit',parent=000204CA)
004010AB . 8D15 DC174000 lea edx, dword ptr \[4017DC\] ; |
004010B1 . FFD2 call edx ; \\SetWindowTextA
004010B3 > 8B15 84314000 mov edx, dword ptr \[403184\] ;PackMe.004031DC
004010B9 . C602 43 mov byte ptr \[edx\], 43
004010BC . 8B55 FC mov edx, dword ptr \[ebp-4\]
004010BF . C602 20 mov byte ptr \[edx\], 20
004010C2 . B8 01000000 mov eax, 1
004010C7 . C9 leave
004010C8 . C3 ret
 
```

先爆破之，看看未来胜利的果实：  

代码:
