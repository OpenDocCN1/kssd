
Norbert Juffa 在学习 Compaq Visual Fortran 编译器的过程里，考虑了对一般性的位移的优化。在 Fortran 语言里面，一个一般性的位移总是向左位移的，但它可能会产生一个负数值的位移数量，意味着向右位移，还有，fortran 里位移的数量要小于 word 能包含的最多的 bit 数量(往哪个方向的位移都得小于这个)。我开始做优化的算法如下:

```
!!!!!!!!!!C CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!

    unsigned long gensh(unsigned long v, int x) {

    int a, b;

        a = (v << x) & -(((unsigned int)x) < 32);

        x = -x;

        b = (v >> x) & -(((unsigned int)x) < 32);

        return a|b;

    }

    unsigned long gensh(unsigned long v, int x) {

    int a, b;

        a = (v << x) & -(((unsigned int)x) < 32);

        x = -x;

        b = (v >> x) & -(((unsigned int)x) < 32);

        return a|b;

    }

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

ANSI C 标准，要求我们在右移里必须用 `unsigned int` 类型，因为接触到"C标准定义的"符号位的位移可能会产生一个未知的错误.这也是我在这儿提起这个的原因。

现在开始比拼汇编功底，这次把 MS VC++ 也参与进来了:

```
!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

    ; by Paul Hsieh

    mov ebx, eax ; 1

    shl eax, cl  ; 1

    cmp ecx, 32  ; 1

    sbb edx, edx ; 2 dep: CF

    neg ecx      ; 2 issue

    and eax, edx ; 3 dep: edx

    cmp ecx, 32  ; 3 dep: ecx

    sbb edx, edx ; 4 dep: CF

    shr ebx, cl  ; 3 dep: ecx

    and ebx, edx ; 5 dep: edx

    or  eax, ebx ; 6 dep:

    ; by Paul Hsieh

    mov ebx, eax ; 1

    shl eax, cl  ; 1

    cmp ecx, 32  ; 1

    sbb edx, edx ; 2 dep: CF

    neg ecx      ; 2 issue

    and eax, edx ; 3 dep: edx

    cmp ecx, 32  ; 3 dep: ecx

    sbb edx, edx ; 4 dep: CF

    shr ebx, cl  ; 3 dep: ecx

    and ebx, edx ; 5 dep: edx

    or  eax, ebx ; 6 dep:

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

```
!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

    ; Microsoft Visual C++

    cmp ecx, 32  ; 1

    mov esi, eax ; 1

    sbb edx, edx ; 2 dep: CF

    shl esi, cl  ; 2 dep: esi

    neg edx      ; 3 dep: edx ??

    neg edx      ; 4 dep: edx ??

    neg ecx      ; 1

    and edx, esi ; 5 dep: edx

    cmp ecx, 32  ; 2 dep: ecx

    sbb esi, esi ; 3 dep: CF

    sar eax, cl  ; 3 issue (ANSI)

    neg esi      ; 4 dep: esi ??

    neg esi      ; 5 dep: esi ??

    and eax, esi ; 6 dep: esi

    or  eax, edx ; 7 dep:

    ; Microsoft Visual C++

    cmp ecx, 32  ; 1

    mov esi, eax ; 1

    sbb edx, edx ; 2 dep: CF

    shl esi, cl  ; 2 dep: esi

    neg edx      ; 3 dep: edx ??

    neg edx      ; 4 dep: edx ??

    neg ecx      ; 1

    and edx, esi ; 5 dep: edx

    cmp ecx, 32  ; 2 dep: ecx

    sbb esi, esi ; 3 dep: CF

    sar eax, cl  ; 3 issue (ANSI)

    neg esi      ; 4 dep: esi ??

    neg esi      ; 5 dep: esi ??

    and eax, esi ; 6 dep: esi

    or  eax, edx ; 7 dep:

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

```
!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

    ; WATCOM C/C++

    cmp  edx, 32  ; 1

    setb cl       ; 2 dep: CF

    mov  ebx, ecx ; 4 dep: ecx, size

    mov  esi, eax ; 1

    and  ebx, 255 ; 5 dep: ebx

    mov  cl, dl   ; 1 rename cl

    neg  ebx      ; 6 dep: ebx

    shl  esi, cl  ; 2 dep: esi,cl

    neg  edx      ; 2 issue

    mov  ecx, esi ; 3 dep: esi

    and  ebx, esi ; 7 dep: ebx

    cmp  edx, 32  ; 3 dep: edx

    setb dh       ; 4 dep: CF

    xor  ecx, esi ; 4 dep: ecx ??

    mov  cl, dh   ; 5 dep: dh

    mov  esi, ecx ; 7 dep: ecx, size

    mov  cl, dl   ; 3 dep: edx

    neg  esi      ; 8 dep: esi

    sar  eax, cl  ; 5 issue (ANSI)

    and  eax, esi ; 9 dep: esi

    or   eax, ebx ; 10 dep:

    ; WATCOM C/C++

    cmp  edx, 32  ; 1

    setb cl       ; 2 dep: CF

    mov  ebx, ecx ; 4 dep: ecx, size

    mov  esi, eax ; 1

    and  ebx, 255 ; 5 dep: ebx

    mov  cl, dl   ; 1 rename cl

    neg  ebx      ; 6 dep: ebx

    shl  esi, cl  ; 2 dep: esi,cl

    neg  edx      ; 2 issue

    mov  ecx, esi ; 3 dep: esi

    and  ebx, esi ; 7 dep: ebx

    cmp  edx, 32  ; 3 dep: edx

    setb dh       ; 4 dep: CF

    xor  ecx, esi ; 4 dep: ecx ??

    mov  cl, dh   ; 5 dep: dh

    mov  esi, ecx ; 7 dep: ecx, size

    mov  cl, dl   ; 3 dep: edx

    neg  esi      ; 8 dep: esi

    sar  eax, cl  ; 5 issue (ANSI)

    and  eax, esi ; 9 dep: esi

    or   eax, ebx ; 10 dep:

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

从 asm 旁边的注释可以看到三段 asm 各自的时钟占用(athlon x86 的 CPU)。看的时候还应该注意每个指令序列会引起的指令执行暂停的问题。暂停的情况就像这样，一个 X86 处理器的 4 条指令解码流水线都去解码了，但之后一个突发情况让它们解码出来的可能要执行的代码不被执行，然后就得重新来过了。从所有的情况看，MSVC++ 看起来在最终性能优化上做的相等不错，但它还是不能简化 `neg reg` 这个指令。标准的 asm 开发过程中，寄存器重分配会把指令之间的数据依赖分散掉(`mov esi, eax` 之后，这二个寄存器就是一样的，而且可以在后面的代码里交换对他们的使用，操作它们二个的指令就可以放在相邻的地方)，MSVC++ 在这个上面没有做优化。WATCOM C/C++ 在寄存器大小的配对上面碰到困难了，标准的 `sub reg8, reg32` 技巧也没用上。

### 9. 64bits BCD数字的加法

Norbert Juffa 给了我一些不可想见的成功的 64bits BCD 数字的加法代码:

```
!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

    ; by Norbert Juffa

      mov   eax, dword ptr [x]   ; x (lo)

      mov   ebx, dword ptr [y]   ; y (lo)

      mov   edx, dword ptr [x+4] ; x (hi)

      mov   ecx, dword ptr [y+4] ; y (hi)

      ; here: EDX:EAX = augend, ECX:EBX = addend

      mov   esi, eax             ; x

      lea   edi, [eax+66666666h] ; x + 0x66666666

      xor   esi, ebx             ; x ^ y

      add   eax, ebx             ; x + y

      shr   esi, 1               ; t1 = (x ^ y) >> 1

      add   edi, ebx             ; x + y + 0x66666666

      sbb   ebx, ebx             ; capture carry

      rcr   edi, 1               ; t2 = (x + y + 0x66666666) >> 1

      xor   esi, edi             ; t1 ^ t2

      and   esi, 88888888h       ; t3 = (t1 ^ t2) & 0x88888888

      add   eax, esi             ; x + y + t3

      shr   esi, 2               ; t3 >> 2

      sub   eax, esi             ; x + y + t3 - (t3 >> 2)

      sub   edx, ebx             ; propagate carry

      mov   esi, edx             ; x

      lea   edi, [edx+66666666h] ; x + 0x66666666

      xor   esi, ecx             ; x ^ y

      add   edx, ecx             ; x + y

      shr   esi, 1               ; t1 = (x ^ y) >> 1

      add   edi, ecx             ; x + y + 0x66666666

    ;;sbb   esi, esi             ; capture carry

      rcr   edi, 1               ; t2 = (x + y + 0x66666666) >> 1

      xor   esi, edi             ; t1 ^ t2

      and   esi, 88888888h       ; t3 = (t1 ^ t2) & 0x88888888

      add   edx, esi             ; x + y + t3

      shr   esi, 2               ; t3 >> 2

      sub   edx, esi             ; x + y + t3 - (t3 >> 2)

      ; here EDX:EAX = sum

      mov   edi, z

      mov   [edi], eax

      mov   [edi+4], edx

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

对 Carry Flag 的利用和对 `rcr` 指令的引入，再次说明 c 编程工具们在很多汇编轻易可以做到的优化上的无能为力。

你可以在这儿找到 BCD 算数的说明: 693K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4K6i4K6u0W2N6h3W2G2N6$3q4Q4x3X3g2W2k6s2g2Q4x3V1k6Q4y4@1g2B7L8$3&6W2M7#2)9J5c8X3u0U0k6q4)9J5c8X3W2F1k6r3g2^5i4K6u0W2K9s2c8E0L8l9``.

### 10. 像素 bashing

"Gordy" 在实现一个 软件实现图像库(e7dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6X3L8$3c8&6i4K6u0W2j5$3!0E0i4K6u0r3i4K6t1&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1%4i4K6R3@1i4@1t1$3i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1#2i4K6W2o6i4@1p5^5g2g2y4q4e0V1g2f1 上就很多常用 pixel bashing 流程的优化提出了问题。这儿是一些我提供的解决方案。

第一个是按特殊顺序压缩 4bits 像素到一个像素的 MMX 函数。这个转化过程就是简单的从每 4 个像素的最低处取出需要的那个。原始 4 个像素的 qword 是: 21436587 (别问我为什么这样那样-这是 Gordy 的库)。我们可以假设这 4 个像素是被 mask 了的(例如，其他不用的 bit 都被设置为 0 了)

```
!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

    MagicConst dd 48124812h,48124812h

    ;....

        movq     mm7, MagicConst

        pcmpeqb  mm6, mm6        ; 1111111111111111

        psllw    mm6, 12         ; 1111000000000000

    @quads:

        movq     mm0, [esi]      ; 0006000500080007
```
