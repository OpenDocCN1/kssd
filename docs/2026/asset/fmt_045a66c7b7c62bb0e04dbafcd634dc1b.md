
```
07C0:0184 2EFFB46704　　push word ptr cs:[si+0467]　;将原中断地址压入堆栈

07C0:0189 CB　　　　　　retf　　　　　　　　　　　　;转入系统中断处理程序继续处理

07C0:018A　　　　　　　 DB "CDBOOT: Cannot　boot from CD - Code: "  ;定义的信息变量

07C0:01AE　　　　　　　 DB "0", 0D, 0A, 00　　　　　　　　　

07C0:01B2　　　　　　　 DB "CDBOOT: Couldn't find NTLDR", 0D, 0A, 00

07C0:01D0　　　　　　　 DB "CDBOOT: Memory　overflow error", 0D, 0A, 00

;以下为前面的代码中使用的函数,如果想了解的话可以自己查 iso9660 里面有关目录项的内容去一行行详细分析

07C0:01F0 C606B10400　　mov byte ptr [04B1], 00 ;

07C0:01F5 90　　　　　　nop

07C0:01F6 8B0E9E04　　　mov cx, [049E]　　　　　　　;取出读取的字符数低位

07C0:01FA FC　　　　　　cld

07C0:01FB 33DB　　　　　xor bx, bx　　　　　　　　　;es:bx 指向目录开始

07C0:01FD 33D2　　　　　xor dx, dx

07C0:01FF 8B36AD04　　　mov si, [04AD]　　　　　　　;要搜索的字符串的地址

07C0:0203 268A17　　　　mov dl, es:[bx]　　　　　　 ;取该目录项长度

07C0:0206 80FA00　　　　cmp dl, 00　　　　　　　　　;如果长度为零,则跳过该项

07C0:0209 0F843B00　　　je 0248

07C0:020D 8BC3　　　　　mov ax, bx

07C0:020F 052100　　　　add ax, 0021　　　　　　　　;加 21H 指向目录项中文件名

07C0:0212 8BF8　　　　　mov di, ax

07C0:0214 51　　　　　　push cx

07C0:0215 33C9　　　　　xor cx, cx

07C0:0217 8A0EAF04　　　mov cl , [04AF]　　　　　　 ;cx 为字符串长度

07C0:021B F3　　　　　　repz

07C0:021C A6　　　　　　cmpsb　　　　　　　　　　　 ;比较字符串

07C0:021D 59　　　　　　pop cx

07C0:021E 0F846F00　　　je 0291　　　　　　　　　　 ;如果相同

07C0:0222 3BD1　　　　　cmp dx, cx　　　　　　　　　;比较目录项长度与总长度的低位

07C0:0224 0F833700　　　jnb 025F                     ;如果不足则处理总长度高位

07C0:0228 2BCA　　　　　sub cx, dx　　　　　　　　　;从总长度低位中减掉当前目录项长度

07C0:022A 803EB10401　　cmp byte ptr [04B1], 01

07C0:022F 0F841A00　　　je 024D

07C0:0233 03D3　　　　　add dx, bx                     ;偏移地址加本目录项指向下一目录项

07C0:0235 8BDA　　　　　mov bx, dx

07C0:0237 83E30F　　　　and bx, 000F

07C0:023A 51　　　　　　push cx

07C0:023B B104　　　　　mov cl, 04

07C0:023D D3EA　　　　　shr dx, cl

07C0:023F 59　　　　　　pop cx

07C0:0240 8CC0　　　　　mov ax, es

07C0:0242 03C2　　　　　add ax, dx

07C0:0244 8EC0　　　　　mov es, ax　　　　　　　　　;最小化远指针偏移

07C0:0246 EBB7　　　　　jmp 01FF

07C0:0248 BA0100　　　　mov dx, 0001                     ;设定长度为一,直接进入下次比较

07C0:024B EBD5　　　　　jmp 0222

07C0:024D 41　　　　　　inc cx

07C0:024E C606B10400　　mov byte ptr [04B1], 00

07C0:0253 90　　　　　　nop

07C0:0254 EBDD　　　　　jmp 0233

07C0:0256 C606B10401　　mov byte ptr [04B1], 01

07C0:025B 90　　　　　　nop

07C0:025C EB29　　　　　jmp 0287

07C0:025E 90　　　　　　nop

07C0:025F 833EA00400　　cmp word ptr [04A0], 0000 ;总长度高位是否为零

07C0:0264 0F850200　　　jne 026A                   ;如果不为零则继续处理

07C0:0268 F9　　　　　　stc                           ;没有找到

07C0:0269 C3　　　　　　ret

07C0:026A 832EA00401　　sub word ptr [04A0], 0001 ;高位减一

07C0:026F 03DA　　　　　add bx, dx                   ;偏移地址加本目录项指向下一目录项

07C0:0271 53　　　　　　push bx

07C0:0272 51　　　　　　push cx

07C0:0273 B104　　　　　mov cl, 04

07C0:0275 D3EB　　　　　shr bx, cl

07C0:0277 59　　　　　　pop cx

07C0:0278 8CC0　　　　　mov ax, es

07C0:027A 03C3　　　　　add ax, bx

07C0:027C 8EC0　　　　　mov es, ax　　　　　　　　;最小化远指针偏移

07C0:027E 5B　　　　　　pop bx

07C0:027F 83E30F　　　　and bx, 000F

07C0:0282 2BD1　　　　　sub dx, cx

07C0:0284 74D0　　　　　je 0256

07C0:0286 4A　　　　　　dec dx

07C0:0287 B8FFFF　　　　mov ax, FFFF

07C0:028A 2BC2　　　　　sub ax, dx

07C0:028C 8BC8　　　　　mov cx, ax                      ;得到新的总长度的地位剩余值

07C0:028E E96EFF　　　　jmp 01FF                   ;继续查找

07C0:0291 803EB00401　　cmp byte ptr [04B0], 01   ;此次查找是否是子目录

07C0:0296 0F840A00　　　je 02A4                   ;如果是子目录则转子目录属性检查

07C0:029A 26F6471902　　test byte ptr es:[bx+19], 02 ;查看目录属性是否为文件

07C0:029F 7581　　　　　jne 0222                   ;如果不是继续查找

07C0:02A1 EB0A　　　　　jmp 02AD

07C0:02A3 90　　　　　　nop

07C0:02A4 26F6471902　　test byte ptr es:[bx+19], 02 ;查看目录属性是否为子目录

07C0:02A9 0F8475FF　　　je 0222                   ;如果不是则继续查找

07C0:02AD A0AF04　　　　mov al, [04AF]

07C0:02B0 26384720　　　cmp es:[bx+20], al           ;比较文件名长度

07C0:02B4 0F856AFF　　　jne 0222                   ;不相同则继续比较

07C0:02B8 F8　　　　　　clc                           ;清除标记,查找成功

07C0:02B9 C3　　　　　　ret

;下面的几个函数使用了类 C 的参数传递方法

07C0:02BA 55　　　　　　push bp

07C0:02BB 8BEC　　　　　mov bp, sp

07C0:02BD 53　　　　　　push bx

07C0:02BE 56　　　　　　push si

07C0:02BF 52　　　　　　push dx

07C0:02C0 50　　　　　　push ax　　　　　　　　　　 ;保存寄存器

07C0:02C1 BB6D04　　　　mov bx, 046D　　　　　　　　;bx 指向扩展磁盘读结构

07C0:02C4 C60710　　　　mov byte ptr [bx], 10

07C0:02C7 C6470100　　　mov byte ptr [bx+01], 00

07C0:02CB C6470300　　　mov byte ptr [bx+03], 00

07C0:02CF C747040000　　mov word ptr [bx+04], 0000

07C0:02D4 C7470C0000　　mov word ptr [bx+0C], 0000

07C0:02D9 C7470E0000　　mov word ptr [bx+0E], 0000　;全部清除

07C0:02DE 8B460C　　　　mov ax, [bp+0C]                    ;将调用参数复制到临时数据区

07C0:02E1 A3A204　　　　mov [04A2], ax

07C0:02E4 8B460A　　　　mov ax, [bp+0A]

07C0:02E7 A3A404　　　　mov [04A4], ax　　　　　　　;读盘总扇区数,32 位,分两次处理

07C0:02EA 8B4608　　　　mov ax, [bp+08]

07C0:02ED A3A604　　　　mov [04A6], ax　　　　　　　;数据区的段地址

07C0:02F0 8B4606　　　　mov ax, [bp+06]

07C0:02F3 A3A804　　　　mov [04A8], ax

07C0:02F6 8B4604　　　　mov ax, [bp+04]

07C0:02F9 A3AA04　　　　mov [04AA], ax　　　　　　　;开始逻辑扇区号,32位

07C0:02FC 813EA4040000　cmp word ptr [04A4], 0000

07C0:0302 0F851900　　　jne 031F

07C0:0306 813EA2042000　cmp word ptr [04A2], 0020

07C0:030C 0F8F0F00　　　jg 031F　　　　　　　　　　 ;总扇区数是否小于 20

;保证每次读取的数据量不超过 64K.

;如果总扇区数小于 20H ,即 20H \* 800H = 10000H, 正好是 64K 一个数据段的尺寸

;注意,这里处理的是 CDROM 的扇区, 所以扇区大小是 800H, 2048 字节

07C0:0310 C6067D0400　　mov byte ptr [047D], 00　　 ;清除标记标示不需段处理

07C0:0315 90　　　　　　nop

07C0:0316 A1A204　　　　mov ax, word ptr [04A2]

07C0:0319 A2AC04　　　　mov byte ptr [04AC], al　　 ;直接读取总扇区数

07C0:031C EB0D　　　　　jmp 032B

07C0:031E 90　　　　　　nop

07C0:031F C6067D0401　　mov byte ptr [047D], 01　　 ;置标记标示需要段处理

07C0:0324 90　　　　　　nop

07C0:0325 C606AC0420　　mov byte ptr [04AC], 20　　 ;每次固定读取 20 个扇区,即 64K 数据

07C0:032A 90　　　　　　nop

07C0:032B A0AC04　　　　mov al, [04AC]　　　　　　　　　

07C0:032E 884702　　　　mov [bx+02], al　　　　　　 ;扇区数

07C0:0331 A1A604　　　　mov ax, [04A6]

07C0:0334 894706　　　　mov [bx+06], ax　　　　　　 ;数据区的段地址

07C0:0337 A1A804　　　　mov ax, [04A8]

07C0:033A 894708　　　　mov [bx+08], ax

07C0:033D A1AA04　　　　mov ax, [04AA]

07C0:0340 89470A　　　　mov [bx+0A], ax　　　　　　 ;开始逻辑扇区号

07C0:0343 BE6D04　　　　mov si, 046D

07C0:0346 B442　　　　　mov ah, 42

07C0:0348 8A169904　　　mov dl, [0499]　　　　　　　;取出引导磁盘号

07C0:034C CD13　　　　　int 13　　　　　　　　　　　;操作

07C0:034E 803E7D0401　　cmp byte ptr [047D], 01　　 ;是否还需要读取?

07C0:0353 0F852200　　　jne 0379

07C0:0357 832EA20420　　sub word ptr [04A2], 0020

07C0:035C 831EA40400　　sbb word ptr [04A4], 0000　 ;总扇区数减掉 20

07C0:0361 8106A6040010　add word ptr [04A6], 1000　 ;数据段地址减掉 1000, 指向下一个 64K.

07C0:0367 0F825AFD　　　jb 00C5　　　　　　　　　　 ;如果溢出则转错误

07C0:036B 8106A8042000　add word ptr [04A8], 0020

07C0:0371 8116AA040000　adc word ptr [04AA], 0000　 ;开始逻辑扇区号加 20

07C0:0377 EB83　　　　　jmp 02FC　　　　　　　　　　;开始下一次循环读取

07C0:0379 58　　　　　　pop ax　　　　　　　　　　　;恢复寄存器

07C0:037A 5A　　　　　　pop dx

07C0:037B 5E　　　　　　pop si

07C0:037C 5B　　　　　　pop bx

07C0:037D 8BE5　　　　　mov sp, bp

07C0:037F 5D　　　　　　pop bp

07C0:0380 C3　　　　　　ret

07C0:0381 55　　　　　　push bp                           ;从指定扇区开始读取指定的字节数到内存,调用 02BA 完成读取

07C0:0382 8BEC　　　　　mov bp, sp

07C0:0384 51　　　　　　push cx

07C0:0385 53　　　　　　push bx

07C0:0386 50　　　　　　push ax

07C0:0387 B10B　　　　　mov cl, 0B                   ;将字节数转换为逻辑扇区数,不足一个扇区按照一个扇区计算

07C0:0389 8B1EA004　　　mov bx, [04A0]

07C0:038D A19E04　　　　mov ax, [049E]

07C0:0390 0FADD8　　　　shrd ax, bx, cl

07C0:0393 D3EB　　　　　shr bx, cl　　　　　　　　　;将 bx:ax 右移 11 位，即除 2048

07C0:0395 F7069E04FF07　test word ptr [049E], 07FF　;低位是否为零

07C0:039B 7406　　　　　je 03A3

07C0:039D 050100　　　　add ax, 0001

07C0:03A0 83D300　　　　adc bx, 0000　　　　　　　　;不是则加一

07C0:03A3 50　　　　　　push ax

07C0:03A4 53　　　　　　push bx　　　　　　　　　　 ;总扇区数

07C0:03A5 FF7604　　　　push word ptr [bp+04]　　　 ;段地址

07C0:03A8 FF369A04　　　push word ptr [049A]

07C0:03AC FF369C04　　　push word ptr [049C]　　　　;开始逻辑扇区号

07C0:03B0 E807FF　　　　call 02BA　　　　　　　　　 ;读扇区

07C0:03B3 83C40A　　　　add sp, 000A

07C0:03B6 58　　　　　　pop ax　　　　　　　　　　　;恢复寄存器

07C0:03B7 5B　　　　　　pop bx

07C0:03B8 59　　　　　　pop cx

07C0:03B9 8BE5　　　　　mov sp, bp

07C0:03BB 5D　　　　　　pop bp

07C0:03BC C3　　　　　　ret

07C0:03BD 50　　　　　　push ax                            ;取出目录项的起始扇区和字节数

07C0:03BE 268B4702　　　mov ax, es:[bx+02]

07C0:03C2 A39A04　　　　mov [049A], ax

07C0:03C5 268B4704　　　mov ax, es:[bx+04]
```
