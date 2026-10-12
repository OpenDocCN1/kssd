
## 引导扇区代码分析

### 初始跳转与堆栈操作

```
000006aa: (                    ) : jz .+0x008a               ; 0f848a00  jz 00000738  
000006ae: (                    ) : mov dl, 0x80              ; b280  
000006b0: (                    ) : jmp .+0xff82              ; eb82  
000006b2: (                    ) : push bp                   ; 55  
000006b3: (                    ) : xor ah, ah                ; 32e4  
000006b5: (                    ) : mov dl, byte ptr ss:[bp]  ; 8a5600  
000006b8: (                    ) : int 0x13                  ; cd13  
000006ba: (                    ) : pop bp                    ; 5d  
000006bb: (                    ) : jmp .+0xff9c              ; eb9c  
```

### 引导结束标志检查

```
000006bd: (                    ) : cmp word ptr ds:0x7dfe, 0xaa55 ; 813ef    
000006c3: (                    ) : jnz .+0x006e              ; 756e           jnz 00000733 判断引导扇区结束标志
000006c5: (                    ) : push word ptr ss:[bp]     ; ff7600  
000006c8: (                    ) : call .+0x008a            ; e88a00         call Func1  
000006cb: (                    ) : jnz .+0x0015              ; 0f851500       jnz 000006e4  
```

### 端口通信与延迟（Delay）

*无论向 `0x60` 还是 `0x64` 端口写东西前都有等状态寄存器 `0BF` 变 0*

```
000006cf: (                    ) : mov al, 0xd1              ; b0d1  
000006d1: (                    ) : out 0x64, al              ; e664  
000006d3: (                    ) : call .+0x007f            ; e87f00         call Func1  
000006d6: (                    ) : mov al, 0xdf              ; b0df  
000006d8: (                    ) : out 0x60, al              ; e660  
000006da: (                    ) : call .+0x0078            ; e87800         call Func1  
000006dd: (                    ) : mov al, 0xff              ; b0ff  
000006df: (                    ) : out 0x64, al              ; e664  
000006e1: (                    ) : call .+0x0071            ; e87100  
000006e4: (                    ) : mov ax, 0xbb00            ; b800bb         int 0x1a / AH=0xBB 没有查到是什么作用
000006e7: (                    ) : int 0x1a                  ; cd1a            
000006e9: (                    ) : and eax, eax              ; 6623c0  
000006ec: (                    ) : jnz .+0x003b              ; 753b           jnz 00000729  
000006ee: (                    ) : cmp ebx, 0x41504354       ; 6681fb5443  
000006f5: (                    ) : jnz .+0x0032              ; 7532           jnz 00000729  
000006f7: (                    ) : cmp cx, 0x0102            ; 81f90201  
000006fb: (                    ) : jb .+0x002c              ; 722c           jb 00000729  
000006fd: (                    ) : push 0x0000bb07          ; 666807bb00  eax  
00000703: (                    ) : push 0x00000200          ; 6668000200  ecx  
00000709: (                    ) : push 0x00000008          ; 6668080000  edx  
0000070f: (                    ) : push ebx                  ; 6653        ebx  
00000711: (                    ) : push ebx                  ; 6653        esp  
00000713: (                    ) : push ebp                  ; 6655        ebp  
00000715: (                    ) : push 0x00000000          ; 6668000000  esi  
0000071b: (                    ) : push 0x00007c00          ; 6668007c00  edi   
00000721: (                    ) : popad                     ; 6661        AX = 0xBB00     
00000723: (                    ) : push 0x0000               ; 680000  
00000726: (                    ) : pop es                    ; 07  
00000727: (                    ) : int 0x1a                  ; cd1a     
00000729: (                    ) : pop dx                    ; 5a  
0000072a: (                    ) : xor dh, dh                ; 32f6  
0000072c: (                    ) : jmp far 0000:7c00         ; ea007c0000    jmp DBR 的代码中  
00000731: (                    ) : int 0x18                  ; cd18  
00000733: (                    ) : mov al, byte ptr ds:0x7b7 ; a0b707  
00000736: (                    ) : jmp .+0x0008              ; eb08  
00000738: (                    ) : mov al, byte ptr ds:0x7b6 ; a0b607  
0000073b: (                    ) : jmp .+0x0003              ; eb03         jmp 00000740  
0000073d: (                    ) : mov al, byte ptr ds:0x7b5 ; a0b507  
00000740: (                    ) : xor ah, ah                ; 32e4  
00000742: (                    ) : add ax, 0x0700            ; 050007  
00000745: (                    ) : mov si, ax                ; 8bf0  
00000747: (                    ) : lodsb al, byte ptr ds:[si] ; ac  
00000748: (                    ) : cmp al, 0x00              ; 3c00  
0000074a: (                    ) : jz .+0xfffc               ; 74fc   jz 00000748  
0000074c: (                    ) : mov bx, 0x0007            ; bb0700  
0000074f: (                    ) : mov ah, 0x0e              ; b40e  
00000751: (                    ) : int 0x10                  ; cd10  
00000753: (                    ) : jmp .+0xfff2              ; ebf2  jmp 00000747  
```

### 内部延迟函数

```
;Delay 无论向 0x60 还是 0x64 端口写东西前都有等状态寄存器 0BF 变 0
00000755: (                    ) : sub cx, cx                ; 2bc9  
00000757: (                    ) : in al, 0x64               ; e464  
00000759: (                    ) : jmp .+0x0000              ; eb00 jmp 0000075b  
0000075b: (                    ) : and al, 0x02              ; 2402  
0000075d: (                    ) : loopne .+0xfff8          ; e0f8  
0000075f: (                    ) : and al, 0x02              ; 2402  
00000761: (                    ) : ret                       ; c3  
```

## DBR 分析

```
00007c00: (                    ) : jmp .+0x0052              ; eb52  jmp 00007c54  
```

后面是数据

`nop`

`db "NTFS", 0`

### 初始化与文件系统检查

```
00007c54: (                    ) : cli                       ; fa  
00007c55: (                    ) : xor ax, ax                ; 33c0  
00007c57: (                    ) : mov ss, ax                ; 8ed0       堆栈段寄存器置为 0
00007c59: (                    ) : mov sp, 0x7c00            ; bc007c     把 0x7c00 置为栈顶
00007c5c: (                    ) : sti                       ; fb  
00007c5d: (                    ) : push 0x07c0               ; 68c007  
00007c60: (                    ) : pop ds                    ; 1f         数据段寄存器置为 0x07c0
00007c61: (                    ) : push ds                   ; 1e  
00007c62: (                    ) : push 0x0066               ; 686600  
00007c65: (                    ) : retf                      ; cb           相当于 jmp 07c0:0066 (0x07c0 << 4 + 0x0066 = 0x7c66)
00007c66: (                    ) : mov byte ptr ds:0xe, dl   ; 88160e00 Reserved sectors 置为 0x80     
00007c6a: (                    ) : cmp dword ptr ds:0x3, 0x5346544e ; 66813e03004e544653 判断 File system ID 是否是 ntfs
00007c73: (                    ) : jnz .+0x0015              ; 7515   jnz 00007c8a  
00007c75: (                    ) : mov ah, 0x41              ; b441  
00007c77: (                    ) : mov bx, 0x55aa            ; bbaa55  
00007c7a: (                    ) : int 0x13                  ; cd13   检查扩展 Int13H 是否存在，进位标志为 1 是不支持
00007c7c: (                    ) : jb .+0x000c              ; 720c   jb 00007c8a 不支持跳转
00007c7e: (                    ) : cmp bx, 0xaa55            ; 81fb55aa  
00007c82: (                    ) : jnz .+0x0006              ; 7506    jnz 00007c8a 不支持跳转       
00007c84: (                    ) : test cx, 0x0001          ; f7c10100  
00007c88: (                    ) : jnz .+0x0003              ; 7503    jnz 00007c8d  支持第二 api 子集
00007c8a: (                    ) : jmp .+0x00d2              ; e9d200  jmp 00007d5f    
```

### 支持 int 13H 第一个 api 子集

```
00007c8d: (                    ) : push ds                   ; 1e   
00007c8e: (                    ) : sub sp, 0x0018            ; 83ec18    
00007c91: (                    ) : push 0x001a               ; 681a00  
00007c94: (                    ) : mov ah, 0x48              ; b448  
00007c96: (                    ) : mov dl, byte ptr ds:0xe   ; 8a160e00  
00007c9a: (                    ) : mov si, sp                ; 8bf4  
```
