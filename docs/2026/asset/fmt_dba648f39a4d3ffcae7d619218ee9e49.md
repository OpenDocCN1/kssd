
>             |
> :0040E879 FF15E4614100            Call dword ptr
> [004161E4]
> :0040E87F 6A00
>   push 00000000
> :0040E881 6856040000              push 00000456
> 
> :0040E886 8B4D08
> mov ecx, dword ptr [ebp+08]
> :0040E889 51
>   push ecx
> 
> * Reference To: `USER32.GetDlgItem`, Ord:0000h
> 
>             |
> :0040E88A FF1548624100            Call dword ptr
> [00416248]
> :0040E890 50
>   push eax
> 
> * Reference To: `USER32.SetWindowTextA`, Ord:0000h
> 
>             |
> :0040E891 FF1544624100            Call dword ptr
> [00416244]
> :0040E897 6A00
>   push 00000000
> :0040E899 6857040000              push 00000457
> 
> :0040E89E 8B5508
> mov edx, dword ptr [ebp+08]
> :0040E8A1 52
>   push edx
> 
> * Reference To: `USER32.GetDlgItem`, Ord:0000h
> 
>             |
> :0040E8A2 FF1548624100            Call dword ptr
> [00416248]
> :0040E8A8 50
>   push eax
> 
> * Reference To: `USER32.SetWindowTextA`, Ord:0000h
> 
>             |
> :0040E8A9 FF1544624100            Call dword ptr
> [00416244]
> :0040E8AF 6A00
>   push 00000000
> :0040E8B1 6858040000              push 00000458
> 
> :0040E8B6 8B4508
> mov eax, dword ptr [ebp+08]
> :0040E8B9 50
>   push eax
> 
> * Reference To: `USER32.GetDlgItem`, Ord:0000h
> 
>             |
> :0040E8BA FF1548624100            Call dword ptr
> [00416248]
> :0040E8C0 50
>   push eax
> 
> * Reference To: `USER32.SetWindowTextA`, Ord:0000h
> 
>             |
> :0040E8C1 FF1544624100            Call dword ptr
> [00416244]
> :0040E8C7 6A00
>   push 00000000
> :0040E8C9 6859040000              push 00000459
> 
> :0040E8CE 8B4D08
> mov ecx, dword ptr [ebp+08]
> :0040E8D1 51
>   push ecx
> 
> * Reference To: `USER32.GetDlgItem`, Ord:0000h
> 
>             |
> :0040E8D2 FF1548624100            Call dword ptr
> [00416248]
> :0040E8D8 50
>   push eax
> 
> * Reference To: `USER32.SetWindowTextA`, Ord:0000h
> 
>             |
> :0040E8D9 FF1544624100            Call dword ptr
> [00416244]
> :0040E8DF 33C0
>   xor eax, eax
> :0040E8E1 E9A4000000              jmp 0040E98A
> 
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:0040E827(C)
> |
> :0040E8E6 8D95D8FEFFFF            lea edx, dword
> ptr [ebp+FFFFFED8]
> :0040E8EC 52
>   push edx
> :0040E8ED E8A70D0000              call 0040F699
> 
> :0040E8F2 83C404
> add esp, 00000004
> 
> * Possible Reference to String Resource ID=00001: "\bL? u2e<?B?n>2L-n"
> 
>             |
> :0040E8F5 C70528D6410001000000    mov dword ptr [0041D628], 00000001
> 
> * Possible Reference to String Resource ID=00001: "\bL? u2e<?B?n>2L-n"
> 
>             |
> :0040E8FF C7052CD6410001000000    mov dword ptr [0041D62C], 00000001
> 
> :0040E909 6804010000              push 00000104
> 
> :0040E90E 8D85CCFCFFFF            lea eax, dword
> ptr [ebp+FFFFFCCC]
> :0040E914 50
>   push eax
> 
> * Possible Reference to String Resource ID=00199: "?"==>注册成功提示信息
> 
>             |
> :0040E915 68C7000000              push 000000C7
> 
> :0040E91A 8B0D48D64100            mov ecx, dword
> ptr [0041D648]
> :0040E920 51
>   push ecx
> 
> 跟踪进入 `call 00410B70` 后，汇编代码如下：
> 
> * Referenced by a CALL at Address:
> |:0040E81D
> |
> :00410B70 8B542404                mov
> edx, dword ptr [esp+04]==>edx 中放着你输入的注册码
> :00410B74 8B4C2408                mov
> ecx, dword ptr [esp+08]==>ecx 中放着真的注册码
> 
> * Possible Reference to String Resource ID=00003: ""
> 
>             |
> :00410B78 F7C203000000            test edx, 00000003
> 
> :00410B7E 753C
>   jne 00410BBC
> 
> * Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:00410BAC(C), :00410BD6(C), :00410BF2(U)
> |
> :00410B80 8B02
>   mov eax, dword ptr [edx]
> :00410B82 3A01
>   cmp al, byte ptr [ecx]
> :00410B84 752E
>   jne 00410BB4
> :00410B86 0AC0
>   or al, al
> :00410B88 7426
>   je 00410BB0
> :00410B8A 3A6101
> cmp ah, byte ptr [ecx+01]
> :00410B8D 7525
>   jne 00410BB4
> :00410B8F 0AE4
>   or ah, ah
> :00410B91 741D
>   je 00410BB0
> :00410B93 C1E810
> shr eax, 10
> :00410B96 3A4102
> cmp al, byte ptr [ecx+02]
> :00410B99 7519
>   jne 00410BB4
> :00410B9B 0AC0
>   or al, al
> :00410B9D 7411
>   je 00410BB0
> :00410B9F 3A6103
> cmp ah, byte ptr [ecx+03]
> :00410BA2 7510
>   jne 00410BB4
> :00410BA4 83C104
> add ecx, 00000004
> :00410BA7 83C204
> add edx, 00000004
> :00410BAA 0AE4
>   or ah, ah
> :00410BAC 75D2
>   jne 00410B80
> :00410BAE 8BFF
>   mov edi, edi
> 
> * Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:00410B88(C), :00410B91(C), :00410B9D(C), :00410BCE(C), :00410BE4(C)
> |:00410BED(C)
> |
> :00410BB0 33C0
>   xor eax, eax
> :00410BB2 C3
>   ret
> 
> 用 `TRW2000` 在 `00410B78` 处下断后，用 `d ecx` 命令就可以知道真的注册码。
> 
> 我的用户名是：`FLITHAWK`，机器号是：`574B-3147-3735-3935`，注册号是：`C1D7-B567-6484-F9E2`
> 
> 至此，该软件的破解全部结束，希望此篇能对破解初学者有所帮助！
> 
>                   Crack by 飞鹰[BCG]
> flithawk@263.net  2001.10.25
>                   欢迎光临汉化新世纪: http://www.hanzify.org
