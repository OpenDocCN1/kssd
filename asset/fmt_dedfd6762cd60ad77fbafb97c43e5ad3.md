
# 反汇编代码分析

## 0040654C 处的主循环逻辑
此循环用于通过注册名产生的 10 进制数串 `A` 来计算一个值 `B`，并将其存储在 `esp+8` 处。

### 代码段 1
```
0040654C   |. /E9 B7000000     jmp HOMEBANK.00406608           ;  下面的循环是用注册名产生的 10 进制数串 A 来得到一值 B 放在 esp+8 处  

00406551   |> |> 8A442E FF      /mov al,byte ptr ds:\[esi+ebp-1\] ;  依次取字符给 al  

00406555   |. |> 8BD0          |mov edx,eax          ;  eax 传给 edx  

00406557   |. |> 80C2 D0       |add dl,0D0  

0040655A   |. |> 80EA 0A       |sub dl,0A     ;  判断取出的是字母还是数字  

0040655D   |. |> 73 62         |jnb short HOMEBANK.004065C1   ;  若是字母则走  

0040655F   |. |> 8BF8          |mov edi,eax          ;  eax-->edi  

00406561   |. |> 81E7 FF000000  |and edi,0FF           ;  高位清 0  

00406567   |. |> 83EF 30       |sub edi,30        ;  ASCII 码变成 10 进制数  

0040656A   |. |> 837C24 0C 00    |cmp dword ptr ss:\[esp+C\],0  

0040656F   |. |> 75 09          |jnz short HOMEBANK.0040657A  

00406571   |. |> 837C24 08 00    |cmp dword ptr ss:\[esp+8\],0  

00406576   |. |> 72 49          |jb short HOMEBANK.004065C1  

00406578   |. |> EB 02          |jmp short HOMEBANK.0040657C  

0040657A   |> |> 7C 45          |jl short HOMEBANK.004065C1  

0040657C   |> |> 817C24 0C CCCCC> |cmp dword ptr ss:\[esp+C\],0CCCCCCC  

00406584   |. |> 75 0C          |jnz short HOMEBANK.00406592  

00406586   |. |> 817C24 08 CCCCC> |cmp dword ptr ss:\[esp+8\],CCCCCCCC  

0040658E   |. |> 76 04          |jbe short HOMEBANK.00406594  

00406590   |. |> EB 2F          |jmp short HOMEBANK.004065C1  

00406592   |> |> 7F 2D          |jg short HOMEBANK.004065C1  

00406594   |> |> 6A 00          |push 0  

00406596   |. |> 6A 0A          |push 0A                 ;  压入 A  

00406598   |. |> 8B4424 10       |mov eax,dword ptr ss:\[esp+10\]  ;第一次循环时 eax 为 0,以后第次就用前次的值来进行运算  

0040659C   |. |> 8B5424 14       |mov edx,dword ptr ss:\[esp+14\]  ;  同上句  

004065A0   |. |> E8 4F120000     |call HOMEBANK.004077F4      ;  此 call 是把 eax=eax\*A  

004065A5   |. |> 52              |push edx  

004065A6   |. |> 50              |push eax  

004065A7   |. |> 8BC7            |mov eax,edi  

004065A9   |. |> 99              |cdq  

004065AA   |. |> 030424          |add eax,dword ptr ss:\[esp\]   ; 设每次取出的字符计为 a(i)，a(i)+eax--->eax,下次循环用此值进行运算  

004065AD   |. |> 135424 04       |adc edx,dword ptr ss:\[esp+4\]  

004065B1   |. |> 83C4 08         |add esp,8  

004065B4   |. |> 894424 08       |mov dword ptr ss:\[esp+8\],eax    ;eax 传给\[esp+8\]  

004065B8   |. |> 895424 0C       |mov dword ptr ss:\[esp+C\],edx  

004065BC   |. |> 45              |inc ebp                  ;  循环控制  

004065BD   |. |> 33DB            |xor ebx,ebx              ;  ebx 清 0  

004065BF   |.^ EB >\\jmp short HOMEBANK.00406551  

004065C1   |>  807C24 10 00  cmp byte ptr ss:\[esp+10\],0  

004065C6   |.  74 17           je short HOMEBANK.004065DF  

004065C8   |.  8B4424 08      mov eax,dword ptr ss:\[esp+8\]  

004065CC   |.  8B5424 0C      mov edx,dword ptr ss:\[esp+C\]  

004065D0   |.  F7D8           neg eax  

004065D2   |.  83D2 00        adc edx,0  
```

## 子程序 004077F4
该子程序处理数值运算，主要涉及乘法操作 `eax*0A--->EAX`。

### 代码段 2
```
004077F4  /$  52            push edx  

004077F5  |.  50            push eax  

004077F6  |.  8B4424 10     mov eax,dword ptr ss:\[esp+10\]  

004077FA  |.  F72424        mul dword ptr ss:\[esp\]  

004077FD  |.  8BC8         mov ecx,eax  

004077FF  |.  8B4424 04     mov eax,dword ptr ss:\[esp+4\]  

00407803  |.  F76424 0C     mul dword ptr ss:\[esp+C\]  

00407807  |.  03C8         add ecx,eax  

00407809  |.  8B0424        mov eax,dword ptr ss:\[esp\]  

0040780C  |.  F76424 0C     mul dword ptr ss:\[esp+C\]  ;eax\*0A--->EAX  

00407810  |.  03D1         add edx,ecx  

00407812  |.  59            pop ecx  

00407813  |.  59            pop ecx  

00407814  \.  C2 0800        retn 8  
```

## 函数 0040A46C
此函数设置格式字符串 `%.0x`，并调用关键子程序 `HOMEBANK.0040B404`。

### 代码段 3
```
0040A46C  /$  55            push ebp  

0040A46D  |.  8BEC          mov ebp,esp  

0040A46F  |.  83C4 F0       add esp,-10  

0040A472  |.  6A 01         push 1                   ;  /Arg1 = 00000001  

0040A474  |.  8945 F0       mov dword ptr ss:\[ebp-10\],eax         ;  |  

0040A477  |.  C645 F4 00     mov byte ptr ss:\[ebp-C\],0            ;  |  

0040A47B  |.  8D45 08       lea eax,dword ptr ss:\[ebp+8\]          ;  |  

0040A47E  |.  8945 F8       mov dword ptr ss:\[ebp-8\],eax          ;  |  

0040A481  |.  C645 FC 10     mov byte ptr ss:\[ebp-4\],10            ;  |  

0040A485  |.  8D4D F0       lea ecx,dword ptr ss:\[ebp-10\]         ;  |  

0040A488  |.  8BC2          mov eax,edx                   ;  |  

0040A48A  |.  BA A4A44000    mov edx,HOMEBANK.0040A4A4   ;ASCII "%.0x"  

0040A48F  |.  E8 700F0000    call HOMEBANK.0040B404      ; 关键,进入!  

0040A494  |.  8BE5          mov esp,ebp  

0040A496  |.  5D            pop ebp  

0040A497  \.  C2 0800        retn 8  
```

## 关键子程序 0040B404
这是一个复杂的子程序，包含多次调用 `HOMEBANK.0040431C` 和 `HOMEBANK.0040AF90`。

### 代码段 4
```
0040A48F  |.  E8 700F0000    call HOMEBANK.0040B404  

0040B404  /$  55            push ebp  

0040B405  |.  8BEC          mov ebp,esp  

0040B407  |.  81C4 04F0FFFF  add esp,-0FFC  

0040B40D  |.  50            push eax  

0040B40E  |.  83C4 F4       add esp,-0C  

0040B411  |.  53            push ebx  

0040B412  |.  56            push esi  

0040B413  |.  894D F8       mov dword ptr ss:\[ebp-8\],ecx  

0040B416  |.  8955 FC       mov dword ptr ss:\[ebp-4\],edx  

0040B419  |.  8BF0          mov esi,eax  

0040B41B  |.  BB 02100000    mov ebx,1002  

0040B420  |.  8B45 FC       mov eax,dword ptr ss:\[ebp-4\]  

0040B423  |.  E8 F48EFFFF    call HOMEBANK.0040431C  

0040B428  |.  8BD3          mov edx,ebx  

0040B42A  |.  85D2          test edx,edx  

0040B42C  |.  79 03         jns short HOMEBANK.0040B431  

0040B42E  |.  83C2 03       add edx,3  

0040B431  |>  C1FA 02        sar edx,2  

0040B434  |.  8BCB          mov ecx,ebx  

0040B436  |.  2BCA          sub ecx,edx  

0040B438  |.  3BC1          cmp eax,ecx  

0040B43A  |.  7D 24         jge short HOMEBANK.0040B460  

0040B43C  |.  8B45 FC       mov eax,dword ptr ss:\[ebp-4\]  

0040B43F  |.  E8 D88EFFFF    call HOMEBANK.0040431C  

0040B444  |.  50            push eax  

0040B445  |.  8B45 F8       mov eax,dword ptr ss:\[ebp-8\]  

0040B448  |.  50            push eax  

0040B449  |.  8B45 08       mov eax,dword ptr ss:\[ebp+8\]  

0040B44C  |.  50            push eax  

0040B44D  |.  8B4D FC       mov ecx,dword ptr ss:\[ebp-4\]  

0040B450  |.  8BD3          mov edx,ebx  

0040B452  |.  4A            dec edx  

0040B453  |.  8D85 F6EFFFFF  lea eax,dword ptr ss:\[ebp-100A\]  

0040B459  |.  E8 32FBFFFF    call HOMEBANK.0040AF90   ;  关键 call,进入  

0040B45E  |.  EB 0C         jmp short HOMEBANK.0040B46C  

0040B460  |>  8B45 FC       mov eax,dword ptr ss:\[ebp-4\]  

0040B463  |.  E8 B48EFFFF    call HOMEBANK.0040431C  

0040B468  |.  8BD8          mov ebx,eax  

0040B46A  |.  8BC3          mov eax,ebx  

0040B46C  |>  8BD3          mov edx,ebx  

0040B46E  |.  4A            dec edx  

0040B46F  |.  3BC2          cmp eax,edx  

0040B471  |.  7C 43         jl short HOMEBANK.0040B4B6  

0040B473  |.  EB 30         jmp short HOMEBANK.0040B4A5  

0040B475  |>  03DB          /add ebx,ebx  

0040B477  |.  8BC6          |mov eax,esi  

0040B479  |.  E8 1E8CFFFF    |call HOMEBANK.0040409C  

0040B47E  |.  8BC6          |mov eax,esi  

0040B480  |.  8BD3          |mov edx,ebx  

0040B482  |.  E8 C991FFFF    |call HOMEBANK.00404650  

0040B487  |.  8B45 FC       |mov eax,dword ptr ss:\[ebp-4\]  

0040B48A  |.  E8 8D8EFFFF    |call HOMEBANK.0040431C  

0040B48F  |.  50            |push eax  

0040B490  |.  8B45 F8       |mov eax,dword ptr ss:\[ebp-8\]  

0040B493  |.  50            |push eax  

0040B494  |.  8B45 08       |mov eax,dword ptr ss:\[ebp+8\]  

0040B497  |.  50            |push eax  

0040B498  |.  8B4D FC       |mov ecx,dword ptr ss:\[ebp-4\]  

0040B49B  |.  8BD3          |mov edx,ebx  

0040B49D  |.  4A            |dec edx  
```

## 子程序 0040AF90
这是被 `0040B459` 调用的关键子程序。

### 代码段 5
```
0040AF90   $  55            push ebp  

0040AF91   .  8BEC          mov ebp,esp  
```
