
# 物理内存的读写

> 原文：[`www.pediy.com/kssd/pediy03/forum1116.htm`](https://www.pediy.com/kssd/pediy03/forum1116.htm)

实现 ring0 倒不是很难。
如果你不想为此再写一个 vxd 的话，用 CIH 的技术吧。
不好意思，俺不懂 uixm
转一篇白云黄鹤上的帖子，供参考
【实际当中应这样写函数原型
extern "C" void `_stdcall` `ChangeSectorSize` (`BYTE` `SectorSize`);】

---------------------------------------------------

发信人: TBsoft (TBsoft), 信区: Virus
标 题: CIH 病毒原理的应用——物理内存的读写
发信站: 武汉白云黄鹤站 (Fri Mar 12 18:44:00 1999) , 站内信件

大家都很讨厌 CIH 病毒，可毛老人家教导我们：“要一分为二地看问题”，
CIH 病毒很可怕，但是它使用的某些特殊技术还是可以借鉴的，这样可以做到
变害为利

Windows 95/98 应用程序无法直接读写物理内存，如果使用 VxD 编程，可以
调用 VMM 功能 `_MapPhysToLinear` 将物理地址映射到线性地址再进行修改，但是这
样就必须单独写一个 VxD，比较麻烦，那么能不能在应用程序中直接调用 VMM 功
能呢？一般不能，因为 VMM 功能要在 Ring 0 上调用，而一般的应用程序工作在
Ring 3 上，那么为什么 CIH 病毒能够调用 VMM 功能呢，CIH 病毒使用了一种技术，
采用 Intel 处理器的中断从 Ring 3 转到 Ring 0，我们完全可以借鉴这种技术来调
用 VMM 功能，下面的程序演示了如何修改物理内存——以在 Windows 95 加密程序
中修改加密扇区大小（物理地址 0000：0525H）为例：

```
;*************************************************
;*    Windows 95 加密软件核心模块之一－－磁盘扇区大小修改程序                 *
;*                                                                             *
;*    本程序在 Windows 95 下修改内存物理地址 0000:0525 处的磁盘扇区大小字节， *
;* 为了能够修改物理地址，本程序使用了 VMM 功能 _MapPhysToLinear 将物理地址映射 *
;* 到线性地址进行修改。为了在应用程序中调用 VMM 功能，本程序使用了 CIH 病毒的 *
;* 原理，使用中断将系统由 Ring 3 转到 Ring 0，然后调用 VMM 功能。             *
;*    本程序中的过程 ChangeSectorSize 在 VC 中的原型可以写成：               *
;*                                                                             *
;*    void _stdcall ChangeSectorSize(BYTE SectorSize);                    *
;*************************************************

.386p

.model flat,stdcall

;修改的中断号，如果本中断号改成 3 则可以防止 Soft-ICE 跟踪！
HookExceptionNo EQU 05h

.data

        IDTR_1 db 6 dup(0) ;保存中断描述符表寄存器
        OldExceptionHook dd 0 ;保存原先的中断入口地址

.code

;修改扇区大小过程
ChangeSectorSize PROC SectorSize:BYTE

        push eax
        ;获取修改的中断的中断描述符（中断门）地址
        sidt IDTR_1
        mov eax,dword ptr IDTR_1+02h
        add eax,HookExceptionNo*08h+04h
        cli
        ;保存原先的中断入口地址
        push ecx
        mov ecx,dword ptr [eax]
        mov cx,word ptr [eax-04h]
        mov dword ptr OldExceptionHook,ecx
        pop ecx
        ;设置修改的中断入口地址为新的中断处理程序入口地址
        push ebx
        lea ebx,NewExceptionHook
        mov word ptr [eax-04h],bx
        shr ebx,10h
        mov word ptr [eax+02h],bx
        pop ebx
        ;执行中断，转到 Ring 0（与 CIH 病毒原理相似！）
        push ebx
        mov bl,byte ptr SectorSize ;扇区大小保存在 bl 寄存器中
        int HookExceptionNo
        pop ebx
        ;恢复原先的中断入口地址
        push ecx
        mov ecx,dword ptr OldExceptionHook
        mov word ptr [eax-04h],cx
        shr ecx,10h
        mov word ptr [eax+02h],cx
        pop ecx
        ;修改扇区大小过程结束
        sti
        pop eax
        ret

ChangeSectorSize ENDP

;新的中断处理程序
NewExceptionHook PROC

        push eax
        push ebx
        push ecx
        push edx
        push esi
        ;修改扇区大小
        push dword ptr 00000000h ;必须为 0
        push dword ptr 00000001h ;字节数
        push dword ptr 00000525h ;物理地址 0000:0525
        int 20h
        dd 0001006ch ;以上两条指令相当于 VMMCall _MapPhysToLinear
        pop esi
        pop esi
        pop esi
        mov byte ptr [eax],bl ;修改扇区大小
        ;中断处理程序结束
        pop esi
        pop edx
        pop ecx
        pop ebx
        pop eax
        iretd

NewExceptionHook ENDP

end
```

本过程可以被 C 语言调用，编译方法：`ml /c /coff w95enc.asm`。请用 MASM 6.11
以上版本编译，不需要 DDK。将编译生成的 OBJ 文件插入 VC 的工程中，并在 VC 程序中写
上函数原型说明，就可以调用了。

本斑竹这几天在集中精力研究 CIH 病毒，希望能够尽早分析出它的杀除方法。这
段文章算是一个“副产品”吧，有什么意见只管提呀。

好久没有发文章了……

--
天生我才，就做一篇锦绣文章。
风起云涌，就挽一回狂澜巨浪。
成也堂堂！
败也堂堂！

※ 修改:·TBsoft 於 Mar 12 18:45:48 修改本文·[FROM:    210.42.24.31]
※ 来源:·武汉白云黄鹤站 s1000e.whnet.edu.cn·[FROM: 210.42.24.31]  4dd466d0e1e0cc123afbe504da2462ca
