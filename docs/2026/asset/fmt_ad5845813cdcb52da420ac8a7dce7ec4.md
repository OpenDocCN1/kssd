
![](https://bbs.kanxue.com/upload/attach/201801/783210_SWHJCKBDA8FUTU7.jpg)

什么情况？没有重复 `Handler` 的调用？看官莫要急躁，且听小生细细说来。

![](https://bbs.kanxue.com/upload/attach/201801/783210_9NMMJQVYZBQTRED.jpg)

这里引入了一个 `BL` 寄存器解密。`BL` 的初始值就是 `PCODE` 起始地址的低8位。

`004043DB    46              INC ESI                                  ; test_vmp.00404CA0`

在该行代码写下断点，则每次中断的时候都能看到 `AL` 里面是经解密后的真正操作码了。

接下来走入一个 `Handler`（`0x67`号）

![](https://bbs.kanxue.com/upload/attach/201801/783210_EYSUBHEDQ64EC4C.jpg)

![](https://bbs.kanxue.com/upload/attach/201801/783210_SPBE68BPSBGEF8B.jpg)

`Handler` 里面也需要经过变幻解密操作数。

要写段代码解密我觉得应该也不困难，抠解密算法就行了。或者可以用上反汇编引擎，动态抠出来执行，适用范围较广。

一段简单的演示代码。静态抠 `Handler` 的办法也不是不可取，主要是工作量大，写代码在 `Handler` 表里面遍历抠出来应该也可以。

小生太菜，只能上一段相当笨的代码。

```
dwEBX equ <0404C9Ch> ;待解码PCODE起始处
dwPID equ <796> ;待解码的进程ID
dwHandler equ <040458Fh> ;Handler表地址
.data
szNtDll db 'ntdll.dll',0
szFunc db 'RtlAdjustPrivilege',0
dwCEBX dd dwEBX
.data?
dwEnable dd ?
bDCode db 1024 dup(?) ;解码缓冲区
.code
Decode_vPop proc bCode:BYTE
LOCAL b:BYTE ;解密结果
pushad
mov bl,byte ptr dwCEBX
mov al,bCode
ADD AL,BL
NOT AL
ADD AL,053h
ROR AL,03h
INC AL
NOT AL
SUB AL,079h                              ; 解密操作数
ADD BL,AL                                ; 更新密钥
mov byte ptr dwCEBX,bl
mov b,AL
popad
mov al,b
movzx eax,al
ret
Decode_vPop endp
Decode proc bCode:BYTE ;解码机器码过程，需要自己抠出来
LOCAL b:BYTE ;解密结果
pushad
mov al,bCode
mov bl,byte ptr dwCEBX
ADD AL,BL                                ; AL += BL
ROL AL,07h                               ; 右移7位
ADD AL,07Fh                              ; AL += 0x7F
XOR AL,021h                              ; 异或0x21
INC AL                                   ; +1
ADD BL,AL                                ; 更新BL密钥
mov byte ptr dwCEBX,bl
mov b,AL
popad
mov al,b
movzx eax,al
ret
Decode endp
Main proc
LOCAL hProcess:DWORD
LOCAL pHandler:DWORD
LOCAL bCode[255]:BYTE
invoke GetModuleHandle,offset szNtDll
invoke GetProcAddress,eax,offset szFunc
push offset dwEnable
push FALSE
push TRUE
push 014H
call eax
;提权
invoke OpenProcess,PROCESS_ALL_ACCESS,FALSE,dwPID
mov hProcess,eax
invoke ReadProcessMemory,hProcess,dwEBX,offset bDCode,2,NULL
invoke Decode,bDCode[0]
mov bDCode[0],al
;解码第一条指令的操作码
invoke Decode_vPop,bDCode[1]
mov bDCode[1],al
ret
Main endp
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [vmprotect1.1.rar](javascript:void(0);)

  （379.17kb，314次下载）
