
`UINT` `L1`, `L2`, `L3`;

`UINT` `H1`, `H2`;

// `x` = (`x>>31`)^(`x>>30`)^(`x<<32`) (mod `2⁶³`)

```
L1 = (b1<<1)|(b0>>31);

L2 = (b1<<2)|(b0>>30);

H1 = (b1>>31);

H2 = (b1>>30);

b1 = H1^H2^b0 &0x7FFFFFFF;

b0 = L1^L2;
}
```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

这儿，我是通过一对 32bits 的变量来操作 63/64bits 的对象的。(我的编译器不支持纯天然的 64bits `int` 类型)

!!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```asm
; compiler 编译器

lea esi,[edx*2]     ; 1 U -
shr ebx,31          ; 2 U -
or  esi,ebx         ; 3 U -
mov ebx,eax         ; 4 U -
lea ecx,[edx*4]     ; 5 U
shr ebx,30          ; 5 V
or  ebx,ecx         ; 7 U - +agi
mov ecx,edx         ; 8 U -
shr ecx,31          ; 9 U -
shr edx,30          ;10 U -
xor edx,ecx         ;11 U -
xor edx,eax         ;12 U -
mov eax,esi         ;13 U
and edx,07FFFFFFFh  ;13 V
xor eax,ebx         ;14 U
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

手写的 asm:

!!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```asm
; by Paul Hsieh and Pisen Chiang

mov esi,edx         ; 1 U
xor cl,cl           ; 1 V
shld edx,eax,1      ; 5 NP  +4c*
shld esi,eax,2      ; 9 NP  +4c
adc  cl,0           ;10 U
xor  edx,esi        ;10 V
xchg eax,edx        ;12 NP  +2c
xor  dl,cl          ;13 U
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

这二个的占用的时钟数是很相当接近的(在奔腾 MMX 上的结果，事实上就是 MMX 预处理的过程把结果给拉进了，第一个 `shld` 指令需要花费一个额外的时钟用来解析特殊的 mmx 指令)，这个出人意料的结果很大一部分是因为慢腾腾的 `shld` 指令。而在奔腾 Pro/II, K6 和 6x86MX 处理器上, `shld` 指令是可以很明显的提高代码大小和代码速度上的性能的。

接下来这个，下面所做的优化，是完全针对奔腾处理器的:

!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```asm
; by hand for Pentiums

mov esi,edx         ; 1 U
xor cl,cl           ; 1 V
shl esi,2           ; 2 U
mov ebx,eax         ; 2 V
adc  cl,0           ; 3 U
lea  edx,[edx*2]    ; 3 V
shr  ebx,30         ; 4 U
xor  edx,esi        ; 4 V
xor  edx,ebx        ; 5 U
xor  al,cl          ; 5 V
shr  ebx,1          ; 6 U -
xchg eax,edx        ; 8 NP  +2c
xor  eax,ebx        ; 9 U
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

### 4.快速的泡沫排序

Core Designs(古墓丽影的公司)的 Andrew Howe 发给了我一个超精短的排序代码。原始代码是在 WATCOM C/C++下写的，我把它的主要部分再给剥离了出来，现在你看到的就是这段代码的最佳状态：

!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```asm
; by Andrew Howe

outerloop:  lea ebx,[edi+ecx*4]
            mov eax,[edi]

cmploop:    sub ebx,4
            cmp eax,[ebx]
            jle notyet
            xchg eax,[ebx]

notyet:     cmp ebx,edi
            jnz cmploop
            stosd
            loop outerloop
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

注意代码里对 `xchg`，`stosd` 和 `loop` 的使用，这几个是你永远在 c 编译器里看不到的指令。我不会花额外的时间去探讨这个流程的时间性能。要是留心的话，会发现在有一些程序里，泡沫排序比其他一个支线很多麻烦很多的排序算法要合适的多。例如，3D 程序需要 z 轴-排序，考虑到时间地点的问题，这个上面总需要维护一个最大值的排序结果，这样就适合来点泡沫了。(从这个出发点延伸出来的另一个有趣的话题可以到这儿看 <<Sorting with less than all the facts>> http://www.azillionmonkeys.com/qed/case5.html)

### 5.一个速度更快的 strlen()代码

在最近，有人给我写了一个留言，说是 `strlen()` 是一个平时要用到很多次的函数，多到足够引起兴趣给它写一个性能优化的替代函数了。那么首先，不把它考虑到太难的方向，我不觉得有什么方法可以从算法的根本上改进它的性能。我在这点上是对的，不过在算法最底部基础的地方看看，可以改进性能的方法还是很丰富的。首先，算法的根本是按字节搜索 0 的，这样的话有一个标准性的改进方向是 c 编译器版本的会错过减少预载入到 cache 数据数量的机会:

!!!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!

```asm
; compiler

    mov edx,ebx
    cmp byte ptr [ebx],0
    je  l2
l1: mov ah,[ebx+1]       ; U
    inc ebx              ; V
    test ah,ah           ; U
    jne  l1              ; V +1brt
l2: sub ebx,edx
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

人手汇编

!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```asm
; by Paul Hsieh

    lea ecx,[ebx-1]
l1: inc ecx
    test ecx,3
    jz  l2
    cmp byte ptr [ecx],0
    jne l1
    jmp l6
l2: mov eax,[ecx]        ; U
    add ecx,4           ; V
    test al,al          ; U
    jz  l5              ; V
    test ah,ah          ; U
    jz  l4              ; V
    test eax,0ff0000h   ; U
    jz  l3              ; V
    test eax,0ff000000h ; U
    jnz l2              ; V +1brt
    inc ecx
l3: inc ecx
l4: inc ecx
l5: sub ecx,4
l6: sub ecx,ebx
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

这就是了。我牺牲了代码大小，来换取时间性能，通过从本质上把循环展开 4 次，如果输入的字符串足够的长(这种情况下性能就是个需要注意的东西了，测试是在奔腾 CPU 上的),每个字节需要执行的 asm 代码是 1.5 个时钟，c 的每字节则需要 3 个时钟。但当字符串不够长的时候，过多的跳转指令产生的分支误预测会使它比编译器的那个占点劣势。

update 补充:

在讨论 Sprite(图像里的一个东西)的数据复制时，我意识到有一种显著改善 32-bit X86 上分支误预测影响力的方法(P-II 和 athlon 上)

!!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```asm
; by Paul Hsieh

    lea ecx,[ebx-1]
l1: inc ecx              ;当有单个字节不等于 0 时，到这儿增加 1
    test ecx,3
    jnz l3
l2: mov edx,[ecx]        ; U
    mov eax,07F7F7F7Fh   ; V
    and eax,edx          ; U 把每个字节 0x7f 以下的数字留下来，0 当然还是 0
    add ecx,4            ; V ~~~~按 4 作为比较是否为 0 的跨步~~~
    add eax,07F7F7F7Fh   ; U 增加 0x7f。
    or  eax,edx          ; U 或上原字节。直观看是处理 0x81 和比 0x81 大的情况，但到这儿其实是让所有非 0 字节至少等于 0x80 或更大
    and eax,080808080h   ; U 到现在，这个字节除了是 0 之外，所有的其他的大小都会被变成 0x80。
    cmp eax,080808080h   ; U 和 0x80 判断即站点是否为 0
    je  l2               ; V +1brt
    sub ecx,4
l3: cmp byte ptr [ecx],0
    jne l1
    sub ecx,ebx
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

我认为这个代码在 32 bit X86 上通常都会提供更好的性能，它减少了很多的跳转。16 bits 的 X86CPU 很明显的可以用一个类似的代码,但很明显它也会比 32bits 的处理速度慢一倍。(还有，我真的开始喜欢这种 mask 技巧了!快看看下一个例子)还有，谢谢 ZiBin Yang (杨子斌 ?)指出来的我的代码里的一个错误。

这儿是 C 版本的相同功能的代码：

!!!!!!!C CODE!!!!!!!!!!!!!!!

```c
int fstrlen(char *s) {
    char *p;
    int d;
    #define M1 (0x7f7f7f7f)
    #define M2 (0x80808080)
    #define SW (sizeof(int)/sizeof(char))

        p=s-1;
        do {
            p++;
            if( (((int)p)&(SW-1))==0 ) {
                do {
                    d = *((int *)p);
                    p += SW;
                    d = (((d&M1)+M1)|d)&M2;
                } while( d==M2 );
                p -= SW;
            }
        } while( *p!=0 );
        return p-s;
}
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

它会编译成很大一篇幅的相同功能的代码(在 P6 上它应该也有相同的性能表现)，但它有一个小问题。它不能适应不同大小的 int 类型(`0x7f7f7f7f` 和 `0x80808080` 会被截断，或者不够用，也会把指针 `p` 指到一个无法确定位置的地方)，而且 C 版本的它也没一点点比 asm 版本的容易理解。

注意，虽然读取时候读取对齐的数据不是必须的，在现代 X86 上它还是会有一些影响，不对齐地址的读取可能造成内存读取失误。这个失误在读取到字符串结尾的时候就可能出现，但我们上面的代码没考虑这些，还是假设你的内存是安装 DWORD(32bits)大小对齐申请的。

update 补充:

Ryan Mack 发来了一个 MMX 版本的 strlen:

!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

```asm
; MMX version by Ryan Mack
```
