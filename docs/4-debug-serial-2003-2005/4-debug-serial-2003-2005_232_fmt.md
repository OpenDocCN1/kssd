
# quickchm 2.49

> 原文：[`www.pediy.com/kssd/pediy05/pediy50424.htm`](https://www.pediy.com/kssd/pediy05/pediy50424.htm)

**软件**：`quickchm` v2.49

**简介**：一个编译与反编译 `chm` 文件的软件

原来我是想找能将 `windows help` 格式的文件转换成 `chm` 文件的工具，于是找到了这个软件。但后来发觉还是不行。

这个软件的主程序加了壳，用 `fi` 查看，是 `Aspack` v2.1。用 `unaspack` 脱壳，启动程序，程序一闪而过，证明程序有自较检。

反汇编程序，查找函数 `postquitmessage`。

```
* Referenced by a CALL at Addresses:
:004644A0   , :0049C69B   

* Reference To: user32.PostQuitMessage, Ord:0000h
                                                                 |
:004079C0 FF2598965000              Jmp dword ptr [00509698]  
:004079C6 8BC0                       mov eax, eax  
```

有两个调用处，`004644a0` 与 `0049c69b`。用 `trw2000` 载入主程序，先在第一处下断点。

```
:0046448D E81E35FAFF               Call 004079B0  
:00464492 85C0                     test eax, eax  
:00464494 7411                     je 004644A7  
:00464496 837DD812                 cmp dword ptr [ebp-28], 00000012  
:0046449A 750B                     jne 004644A7       －－－－－尝试改为 jmp，虽然跳过自较检，但程序不能退出。  
:0046449C 8B45DC                   mov eax, dword ptr [ebp-24]  
:0046449F 50                         push eax

* Reference To: user32.PostQuitMessage, Ord:0000h
                                                                 |
:004644A0 E81B35FAFF               Call 004079C0  
```

在第二处下断点。

```
:004FC2BC 83FA00                    cmp edx, 00000000  
:004FC2BF 7509                     jne 004FC2CA  
:004FC2C1 3D20A10700               cmp eax, 0007A120  
:004FC2C6 7610                     jbe 004FC2D8           －－－－－－－－|--这里有两个地方跳过自较检退出，可先载入未脱  
:004FC2C8 EB02                     jmp 004FC2CC                   | 壳的程序，观察原程序在哪个地方跳过。结果  
                                                                 | 发现原程序在 004fc2c6 处跳过。  
* Referenced by a (U)nconditional or (C)onditional Jump at Address:        | 改为 jmp 004fc2c6  
|:004FC2BF(C)                                                                      |  
|                                                                              |  
:004FC2CA 7E0C                       jle 004FC2D8           －－－－－－－－|

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004FC2C8(U)  
|  
:004FC2CC A1405E5000                 mov eax, dword ptr [00505E40]  
:004FC2D1 8B00                       mov eax, dword ptr [eax]  
:004FC2D3 E8B803FAFF                 call 0049C690  
```

用 `hex workshop` 编辑程序，将 `3d20a107007610eb02` 改为 `eb` 即可跳过自较检退出。

这个程序在启动时会要求注册，但反汇编看到这里好像并不较检注册码，只是将注册码保存起来。

以下是破解注册码部分。

```
:0050175B 55                         push ebp  
:0050175C 6890195000                 push 00501990  
:00501761 64FF30                     push dword ptr fs:[eax]  
:00501764 648920                     mov dword ptr fs:[eax], esp  
:00501767 8D55F8                     lea edx, dword ptr [ebp-08]  
:0050176A A1A45B5000                 mov eax, dword ptr [00505BA4]  
:0050176F 8B00                       mov eax, dword ptr [eax]  
:00501771 8B80F8020000               mov eax, dword ptr [eax+000002F8]  
:00501777 E8B0A1F7FF                 call 0047B92C  
:0050177C 8D45E4                     lea eax, dword ptr [ebp-1C]

* Possible StringData Ref from Code Obj ->"F0E1"       －－－－－－－－ 这个字符串“F0E1”与注册码的计算有关  
                                                                 |  
:0050177F BAA8195000                 mov edx, 005019A8               
:00501784 E84F33F0FF                 call 00404AD8  
:00501789 8D55F4                     lea edx, dword ptr [ebp-0C]  
:0050178C A1A45B5000                 mov eax, dword ptr [00505BA4]  
:00501791 8B00                       mov eax, dword ptr [eax]  
:00501793 8B8004030000               mov eax, dword ptr [eax+00000304]  
:00501799 E88EA1F7FF                 call 0047B92C  
:0050179E 8B45E4                     mov eax, dword ptr [ebp-1C]－－－－（ebp－1c）存放字符串“F0E1”  
:005017A1 E85A35F0FF                 call 00404D00  －－－0040d00 的这个 call 的作用是将 eax 中的字符串的长度存入 eax 中，后面常用到。  
:005017A6 8945F0                     mov dword ptr [ebp-10], eax－－－－（ebp－10）存放字符串“F0E1”的长度。  
:005017A9 33FF                       xor edi, edi    －－－－－－－－－edi 存放的是字符串“F0E1”的指针。  
:005017AB C745E801000000             mov [ebp-18], 00000001－－－－（ebp－18）中存放的是假注册码的指针，以后每次＋2  
:005017B2 BB80000000                 mov ebx, 00000080  －－－－－－令 ebx＝80，这与计算注册码的第一二个字符有关  
:005017B7 C645E300                   mov [ebp-1D], 00    －－－－－－初始化标志，（ebp－1d）存放的是注册是否成功的标志。  
:005017BB 8B45F8                     mov eax, dword ptr [ebp-08]－－－－将名字存入 eax 中  
:005017BE E83D35F0FF                 call 00404D00    －－－－－－名字长度存入 eax，又一次用到这个函数了。  
:005017C3 8BF0                       mov esi, eax  
:005017C5 83FE01                     cmp esi, 00000001   
:005017C8 0F8CA7000000               jl 00501875      －－－－－－如果名字长度小于 1，则 game over。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0050186F(C)  
|  
:005017CE 8B45F8                     mov eax, dword ptr [ebp-08]－－－－名字存入 eax，esi 存放的是名字的长度。  
:005017D1 0FB64430FF                 movzx eax, byte ptr [eax+esi-01]－－－－将名字的第 n 位（从最后一位开始取）放入 eax 中。  
:005017D6 03C3                       add eax, ebx      －－－－－－－－－eax＋ebx，（即字符的 ascii 码＋ebx），存放于 eax 中。  
:005017D8 B9FF000000                 mov ecx, 000000FF－－－－－－－－令 ecx＝ffh，（十进制的 255）  
:005017DD 99                         cdq  
:005017DE F7F9                       idiv ecx          －－－－－－－－eax÷ecx  
:005017E0 8BDA                       mov ebx, edx      －－－－－－－－将余数存入 ebx 中  
:005017E2 3B7DF0                     cmp edi, dword ptr [ebp-10]  
:005017E5 7D03                       jge 005017EA      －－－－－－－－  ]  
:005017E7 47                         inc edi           －－－－－－－－  ]  
:005017E8 EB05                       jmp 005017EF                       ]  
                                                                 ]->字符串“F0E1”的指针＋1，如果到尾了，就重新指向头部。  
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  ]  
|:005017E5(C)                                                                        ]  
|                                                                              ]  
:005017EA BF01000000                 mov edi, 00000001－－－－－－－－－]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:005017E8(U)  
|  
:005017EF 8B45E4                     mov eax, dword ptr [ebp-1C]－－－－－字符串“F0E1”放入 eax  
:005017F2 0FB64438FF                 movzx eax, byte ptr [eax+edi-01]－－将字符串“F0E1”的第 n 位（从头开始取）放入 eax 中。  
:005017F7 33D8                       xor ebx, eax               －－－异或，结果存入 ebx 中。  
:005017F9 8D45EC                     lea eax, dword ptr [ebp-14]  
:005017FC 50                         push eax  
:005017FD 895DD8                     mov dword ptr [ebp-28], ebx  
:00501800 C645DC00                   mov [ebp-24], 00  
:00501804 8D55D8                     lea edx, dword ptr [ebp-28]  
:00501807 33C9                       xor ecx, ecx

* Possible StringData Ref from Code Obj ->"%1.2x"        －－－－以下这一大段是比较代码，作用大概是：用计算出来的结果  
                                                                 （存入 ebx 中）转为 ascii 码，与假注册码的指针（指针存于 ebp－  
:00501809 B8B8195000                 mov eax, 005019B8                18 中）所指向的字符相比较（每次比较两位）。比较的结果影响  
:0050180E E8C98BF0FF                 call 0040A3DC                    标志 ebp－1d，相等则置 1，不相等则置 0，相等则令指针＋2，不  
:00501813 8D45D4                     lea eax, dword ptr [ebp-2C]       相等则令指针不动。  
:00501816 50                         push eax                          我个人认为这段代码写的有的笨，其实只要一发现比较不相等，  
:00501817 8B45EC                     mov eax, dword ptr [ebp-14]      就跳去 game over 就行了，不用将整个假的注册码比较完。  
:0050181A E8E134F0FF                 call 00404D00  
:0050181F 8BC8                       mov ecx, eax  
:00501821 8B55E8                     mov edx, dword ptr [ebp-18]  
```



:00501824 8B45F4 mov eax, dword ptr `[ebp-0C]`  
:00501827 E83437F0FF call `00404F60`  
:0050182C 8B45D4 mov eax, dword ptr `[ebp-2C]`  
:0050182F 8B55EC mov edx, dword ptr `[ebp-14]`  
:00501832 E81536F0FF call `00404E4C`  
:00501837 0F9445E3 sete byte ptr `[ebp-1D]`  
:0050183B 8D45D0 lea eax, dword ptr `[ebp-30]`  
:0050183E 50 push eax  
:0050183F 8B45EC mov eax, dword ptr `[ebp-14]`  
:00501842 E8B934F0FF call `00404D00`  
:00501847 8BC8 mov ecx, eax  
:00501849 8B55E8 mov edx, dword ptr `[ebp-18]`  
:0050184C 8B45F4 mov eax, dword ptr `[ebp-0C]`  
:0050184F E80C37F0FF call `00404F60`  
:00501854 8B55D0 mov edx, dword ptr `[ebp-30]`  
:00501857 8B45EC mov eax, dword ptr `[ebp-14]`  
:0050185A E8ED35F0FF call `00404E4C`  
:0050185F 750B jne `0050186C`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0050185F(C)  

:0050186C 4E dec esi  
:0050186D 85F6 test esi, esi  
:0050186F 0F8559FFFFFF jne `005017CE` －－－－－－－循环  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:005017C8(C)  

:00501875 807DE300 cmp byte ptr `[ebp-1D]`, 00  
:00501879 0F84D9000000 je `00501958` －－－－－－－比较标志，如果为 0，则 game over。  
:0050187F B201 mov dl, 01 －－－－－－－以下是注册成功的代码，将名字和注册码存放到  
:00501881 A1D0444400 mov eax, dword ptr `[004444D0]` 注册表中，以便启动软件时检查是否注册了。  
:00501886 E8452DF4FF call `004445D0`  
:0050188B 8BD8 mov ebx, eax  
:0050188D BA02000080 mov edx, `80000002`  
:00501892 8BC3 mov eax, ebx  
:00501894 E8D72DF4FF call `00444670`  
:00501899 B101 mov cl, 01  

\* Possible StringData Ref from Code Obj ->`"Software\\Microsoft\\DirectX"` －－竟然将名字和注册码存放在这里？？？奇怪！！！  

:0050189B BAC8195000 mov edx, `005019C8`  
:005018A0 8BC3 mov eax, ebx  
:005018A2 E8312EF4FF call `004446D8`  
:005018A7 84C0 test al, al  
:005018A9 741E je `005018C9`  
:005018AB 8B4DF8 mov ecx, dword ptr `[ebp-08]`  

\* Possible StringData Ref from Code Obj ->`"Label_N"` －－－－－这里放名字  

:005018AE BAEC195000 mov edx, `005019EC`  
:005018B3 8BC3 mov eax, ebx  
:005018B5 E83A34F4FF call `00444CF4`  
:005018BA 8B4DF4 mov ecx, dword ptr `[ebp-0C]`  

\* Possible StringData Ref from Code Obj ->`"Label_K"` －－－－－这里放注册码  

:005018BD BAFC195000 mov edx, `005019FC`  
:005018C2 8BC3 mov eax, ebx  
:005018C4 E82B34F4FF call `00444CF4`  

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－  

username: `ben007`  

regcode: `F1120744EF62`

以下是我个人写的注册机，本人乃菜鸟一只，编程水平不高，程序若有不妥之处，请指正。

```
data segment

msg1 db 'The keygen for the Quickchm v2.49,created by ben007.',0ah,0dh,'$'
input_msg db 'Please input the name:$'
inputname db 15,?,10 dup(?),0ah,0dh,'$'
regcode db 25 dup(?)
output_msg db 0dh,0ah,'The regcode is:$'
regcode_ptr dw 0
dat db 'F0E1'
lastnum dw 80h

data ends

stack segment para  stack 'stack'
db 10 dup(?)
stack ends

code segment
assume cs:code,ds:data,ss:stack

start:mov ax,data
     mov ds,ax
     mov ah,09h
     lea dx,msg1
     int 21h
     lea dx,input_msg
     int 21h
     mov ah,0ah
     lea dx,inputname
     int 21h
     xor ax,ax
     xor dx,dx
     xor di,di
     mov ch,0
     mov cl,inputname+1
     lea bx,inputname+2
     mov si,cx
     dec si

next: mov al,byte ptr[bx+si]
     cbw
     add ax,lastnum
     mov dl,0ffh
     div dl
     mov dl,ah
     push bx
     lea bx,dat
     mov al,byte ptr[bx+di]
     cbw
     xor ax,dx
     mov lastnum,ax
     call changetoascii
     inc di
     cmp di,3
     ja pointtohead
     jmp lp

pointtohead:mov di,0

lp:   pop bx
     dec si
     loop next
     mov ah,09h
     lea dx,output_msg
     int 21h
     lea bx,regcode
     mov di,regcode_ptr
     mov byte ptr[bx+di],'$'
     mov dx,bx
     int 21h
     mov ah,4ch
     int 21h

changetoascii proc
     push bx
     push di
     mov bh,0
     mov bl,10h
     div bl
     lea bx,regcode
     cmp al,0ah
     jae alplus37h
     add al,30h
     jmp next1

alplus37h:add al,37h

next1:cmp ah,0ah
     jae ahplus37h
     add ah,30h
     jmp next2

ahplus37h:add ah,37h

next2:mov di,regcode_ptr
     mov byte ptr[bx+di],al
     inc di
     mov byte ptr[bx+di],ah
     inc di
     mov regcode_ptr,di
     pop di
     pop bx
     ret

changetoascii endp

code ends

end start
```

ben007  
2003.2.15
