

# anti trick Ⅱ －－ InString

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-633.htm`](https://www.pediy.com/kssd/pediy07/pediy7-633.htm)

anti trick Ⅱ －－ InString   
  
site:www.ptteam.com  
by 来自轻院的狼\[Immlep\]  
  
这个不是什么新鲜的东西，不过我还没有见过有人在壳中使用的，用这个来做 anti，它比 FindWindow 强很多了，不过这些东西说出来后，以后就没什么，可能以后你一看到类似的 anti 你就会很清楚了，另外这个 anti 调用的函数过多，代码长，挺不好的，本文中的例子（InString ）是我引用现成的代码的，只不过做了一下简单的修改。思路是使用 GetWindow 循环获取系统的窗口的标题，查看标题中是否包含了要查找的关键字，如果发现，就做坏事，这个来检测 Ollydbg 也挺好的，可以检测"- \[CPU - ",这样的字样，另外因为 Ollydbg 调试的时候会把被调试程序的名称显示在 Ollydbg 的窗口中，所以我们也可以检测“－debugme.exe-\[”这样的字符（debugme.exe 为被调试程序的名称，你可以用 GetFullFilename 等函数来获取被加壳后程序的名称，然后再处理一下），当然最好先给这些字符加密一下了。。  
  
还有你可以用 InString 来做不 anti 的其它事情，这样可以防止调试者在 InString 做手脚，另外这个 anti 调用的函数，你最好也让它干一些有用的东西，不要只拿来做 anti,很容易被 hook 的:)  
  
我测试了一下，anti 效果还是良好的，什么修改版的 Od 都被干掉了（当然不保证以后），SoftICE 的 loader 也不例外:)  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> searchtext PROC
> ；；；；；；获取系统中所有窗口的标题
>      invoke GetDesktopWindow
>      invoke GetWindow,eax,GW\_CHILD
> @1:   
>       mov hwindow,eax
>       invoke GetWindowText,hwindow,ADDR buff,200
>       cmp eax,0
>       jz skip1
>       ;lea edi,offset buff
>       push offset buff2       ；我们要查找的窗口标题中包含的关键字，如"- \[CPU - "
>       push offset buff        ；；找到的窗口的中标题
>       push 1                  ；从第一个字符找
>       call \_InString
>       add esp,0ch
>       ;invoke InString,1,addr buff,addr buff2
>       cmp eax,0
>       jz @F
>       invoke PostMessage,hwindow,WM\_QUIT,0,0  ;在壳中做坏事不要这样做，很容易被跟踪出来的，自己想点新鲜的。
>       ;invoke MessageBox,NULL,addr szdbtext1,addr szdbtext1,MB\_OK
>       skip1:
>     @@:
>       invoke GetWindow,hwindow,GW\_HWNDNEXT
>       cmp eax,NULL
>       jne @1 
>       jmp loc111
> ；；；；；；获取系统中所有窗口的标题
> 
> 
> ;;;;InString 我处理了重定位问题，，可以在壳中使用！！！代码好长！！汗个！！！！
> \_InString:    
>   jmp @F
>     StartPos dd 0
>     lpszString dd 0
>     lpszSubStr dd 0
>     lnStrng dd 0
>     lnSubSt dd 0
>     reg1 dd 0
>     reg2 dd 0
>     Byte1 byte 0
>     @@:
>     push eax
>     mov eax,DWORD PTR \[ESP+8\]
>     mov \[EBP+OFFSET StartPos\],eax
>     pop eax
>     
>     push eax
>     mov eax,DWORD PTR \[ESP+0ch\]
>     mov \[EBP+OFFSET lpszString\],eax
>     pop eax
>     
>     push eax
>     mov eax,DWORD PTR \[ESP+10h\]
>     mov \[EBP+OFFSET lpszSubStr\],eax
>     pop eax
> 
>     push esi
>     push edi
>     push ebx
> 
>     .if \[EBP+OFFSET StartPos\] < 1
>       mov eax, -2               ; set eax -2
>       jmp @@Get\_Outa\_Here       ; exit if less than 1
>     .endif
> 
>     dec \[EBP+OFFSET StartPos\]               ; correct to 0 based index
> 
>     mov eax,\[EBP+OFFSET lpszString\]
>     call \_lstrlen                          ；；；；；把原来的 lstrlen 函数也干掉！！！
>     mov \[EBP+OFFSET lnStrng\], eax
>     push eax
>     mov eax,\[EBP+OFFSET lpszSubStr\]
>     call  \_lstrlen
>     mov \[EBP+OFFSET lnSubSt\], eax
>     pop  eax
>      
>     sub eax, \[EBP+OFFSET lnSubSt\]           ; subtract substr len from main string
> 
>     ; The following order is important.
> 
>     .if eax >= \[EBP+OFFSET lnStrng\]          ; the substring is greater than the main string
>       mov eax,0
>       jmp @@Get\_Outa\_Here
>     .elseif \[EBP+OFFSET StartPos\] == eax     ; startpos at the last position accepted
>       jmp Same\_Size
>     .elseif \[EBP+OFFSET StartPos\] > eax      ; startpos greater than the last position accepted
>       mov eax, 0
>       jmp @@Get\_Outa\_Here
>     .elseif eax == 0            ; the two strings have the same size
>       jmp Same\_Size
>     .endif
> 
>     mov esi, \[EBP+OFFSET lpszSubStr\]         ; get 1st byte in substring
>     mov bl, \[esi\]
> 
>   ; -------------------------------------------------------
>   ; set maximum count as main string length minus substring
>   ; -------------------------------------------------------
> 
>     mov ecx, \[EBP+OFFSET lpszString\]
>     add ecx, \[EBP+OFFSET lnStrng\]
>     sub ecx, \[EBP+OFFSET lnSubSt\]
>     inc ecx
> 
>     mov esi, \[EBP+OFFSET lpszString\]         ; main string address
>     add esi, \[EBP+OFFSET StartPos\]           ; add starting position to esi
> 
>     cld                         ; read forward
> 
>     @@L1s:                      ; 8 cycles on no 1st char match
>       mov al, \[esi\]             ; 1
>       inc esi                   ; 1
>       cmp al, bl                ; 1     find 1st substring byte
>       je @F                     ; 1 - 3 compare subsequent bytes to
>     @@L1r:
>       cmp esi, ecx              ; 1
>       jne @@L1s                 ; 3 - 1
> 
>       mov eax, 0                ; return zero and exit if
>       jmp @@Get\_Outa\_Here         ; match not found in string
> 
>   ; ------------------------------------------------
>   ; do the comparison, main string is already in esi
>   ; ------------------------------------------------
> 
>     @@:
> 
>     mov \[EBP+OFFSET reg1\], ecx               ; 1
>     mov \[EBP+OFFSET reg2\], esi               ; 1
>     mov byte ptr\[EBP+OFFSET Byte1\], bl               ; 1
> 
>     mov ecx, \[EBP+OFFSET lnSubSt \]           ; sub string length
>     .if ecx == 1
>         inc esi
>         jmp @@GetRetVal
>     .endif
>     inc ecx                     ; compare correct number of bytes
>     dec esi                     ; back one to compare correct bytes
>     mov edi, \[EBP+OFFSET lpszSubStr\]
> 
>     repe cmpsb                  ; 9 if strings match, ecx will be 0
> 
>     cmp ecx, 0                  ; did the two strings match ??
>     jne @F
> 
>     ; jnz @F
> 
>     @@GetRetVal:
>       mov eax, \[EBP+OFFSET lpszString\]
>       sub esi, eax              ; subtract it from esi current value
>       sub esi, \[EBP+OFFSET lnSubSt\]          ; subtract the search string length
> 
>       mov eax, esi              ; put count in eax
>       add eax, \[EBP+OFFSET StartPos\]         ; add starting pos to get correct count
>       jmp @@Get\_Outa\_Here
>     @@:
> 
>     mov ecx, \[EBP+OFFSET reg1\]               ; 1
>     mov esi, \[EBP+OFFSET reg2\]               ; 1
>     mov bl,byte ptr\[EBP+OFFSET Byte1\]              ; 1
>     jmp @@L1r                   ; try again for match
> 
> 
> 
>     Same\_Size:
> 
>     mov esi, \[EBP+OFFSET lpszString \]        ; main string address
>     add esi, \[EBP+OFFSET StartPos \]          ; add starting position to esi
>     mov edi, \[EBP+OFFSET lpszSubStr\]         ; sub string address
>     mov ecx, \[EBP+OFFSET lnSubSt \]           ; sub string length
> 
>     cld
>     repe cmpsb
> 
>     .if ZERO?                   ; the two strings match
>         mov eax, \[EBP+OFFSET StartPos \]      ; get the return value
>         inc eax
>     .else
>         mov eax, 0
>     .endif
>    jmp @@Get\_Outa\_Here
> \_lstrlen:
>         pushfd
>         push ecx
>         push ebx
>   ;mov     eax,lpStr
>   lea     ecx,\[eax-1\]
> l1:     inc     ecx
>         test    ecx,3
>         jz      l2
>         cmp     byte ptr\[ecx\],0
>         jne     l1
>         jmp     l6
> l2:     mov     ebx,\[ecx\]       ; U
>         add     ecx,4           ;   V
>         test    bl,bl           ; U
>         jz      l5              ;   V
>         test    bh,bh           ; U
>         jz      l4              ;   V
>         test    ebx,0ff0000h    ; U
>         jz      l3              ;   V
>         test    ebx,0ff000000h  ; U
>         jnz     l2              ;   V +1brt
>         inc     ecx
> l3:     inc     ecx
> l4:     inc     ecx
> l5:     sub     ecx,4
> l6:     sub     ecx,eax
>         mov     eax,ecx 
>         pop     ebx
>         pop     ecx
>         popfd 
>       db 0C3h
>   @@Get\_Outa\_Here:
>     pop ebx
>     pop edi
>     pop esi 
>     ret
> searchtext endp
> 
> * * *
> 
> 
> ```

  
  
附件中是我写来测试这个 anti 的小例子，只能对付 ollydbg  
附件:crackme.rar  

 4dd466d0e1e0cc123afbe504da2462ca