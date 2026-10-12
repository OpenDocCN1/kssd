
#pragma warning(disable:4035) // no missing return value

inline unsigned int gcd (Uint32 x, Uint32 y) {
  __asm {
    mov   ecx, dword ptr [y]
    mov   edx, dword ptr [x]
    test  ecx, ecx
    mov   eax, edx
    je    done               ;// if (y = 0) -> return x
    test  eax, eax
    mov   eax, ecx
    je    done               ;// if (x = 0) -> return y
    push  edi
    bsf   ebx, eax           ;// ebx = trailingZeroes(y)
    bsf   ecx, edx           ;// ecx = trailingZeroes(x)
    cmp   ebx, ecx
    mov   edi, ecx
    cmovl edi, ebx           ;// edi = min(ebx,ecx)
    shr   edx, cl            ;// x >>= trailingzeroes(x)
    mov   ecx, ebx
    shr   eax, cl            ;// y >>= trailingzeroes(y)
    align 8
  mainloop:                  ;// for (;;)
    cmp   eax, edx
    mov   ecx, eax
    je    skiploop           ;// if (x == y) -> break
    cmovb eax, edx
    cmovb edx, ecx           ;// if (y > x) swap(x,y)
    sub   eax, edx           ;// x -= y
    bsf   ecx, eax
    shr   eax, cl            ;// x >>= trailingzeroes(x)
    jmp   mainloop
    align 8
  skiploop:
    mov   ecx, edi
    shl   eax, cl            ;// return x<<finalShiftLeft
    pop   edi
  done:                      ;// return value in eax
  }
}

#pragma warning(default:4035)

/*---------------------------------------------------//*!
 * brief   Computes GCD of two 32-bit unsigned integers
 * param   x First unsigned integer
 * param   y Second unsigned integer
 * return  gcd(x,y)
 * note    gcd(x,0) -> x, gcd(0,y) -> y
 * note    Implemented in x86 assembler (PentiumPro and
 *         above only as cmov instructions are used)
 * note    Send all comments/whines to wili@hybrid.fi
 * todo    [wili 021026] Implement another version that
 *         uses sbb trickery rather than cmov
 *         instructions
 *-----------------------------------------------------*/

#pragma warning(disable:4035) // no missing return value

inline unsigned int gcd (Uint32 x, Uint32 y) {
  __asm {
    mov   ecx, dword ptr [y]
    mov   edx, dword ptr [x]
    test  ecx, ecx
    mov   eax, edx
    je    done               ;// if (y = 0) -> return x
    test  eax, eax
    mov   eax, ecx
    je    done               ;// if (x = 0) -> return y
    push  edi
    bsf   ebx, eax           ;// ebx = trailingZeroes(y)
    bsf   ecx, edx           ;// ecx = trailingZeroes(x)
    cmp   ebx, ecx
    mov   edi, ecx
    cmovl edi, ebx           ;// edi = min(ebx,ecx)
    shr   edx, cl            ;// x >>= trailingzeroes(x)
    mov   ecx, ebx
    shr   eax, cl            ;// y >>= trailingzeroes(y)
    align 8
  mainloop:                  ;// for (;;)
    cmp   eax, edx
    mov   ecx, eax
    je    skiploop           ;// if (x == y) -> break
    cmovb eax, edx
    cmovb edx, ecx           ;// if (y > x) swap(x,y)
    sub   eax, edx           ;// x -= y
    bsf   ecx, eax
    shr   eax, cl            ;// x >>= trailingzeroes(x)
    jmp   mainloop
    align 8
  skiploop:
    mov   ecx, edi
    shl   eax, cl            ;// return x<<finalShiftLeft
    pop   edi
  done:                      ;// return value in eax
  }
}

#pragma warning(default:4035)

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
|||||||||||||||||||||||||||||||||||||||||

2.搜索对齐了的结构中非零的元素

假设，假设我们想在一个数组结构中查找它是否有一个特殊的项里面存着的是空的对象。就是说，假设我们需要写这样一个函数出来:

!!!!!C CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```
unsigned int existnzkey (struct foobar * table, unsigned int tablelength)
{
    int i;
    for(i=0;i<tablelength;i++)
        if( table[i].key !=0 ) return table[i].key;
    return 0;
}
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

显而易见的，要直接从汇编层次分析它的话，这个代码需要改进的问题实在是太多了。我们先透过一些创造力，一些可以在我的代码优化文章里找到的标准技巧处理下它:

!!!!C CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```
static unsigned int existnzkey (unsigned int tablelength, struct foobar * table)
{
    int i,c,d;
    d=i=0;
    c=tablelength;
    goto Start;
    do {
        d = table[i].key;
        if( d !=0 ) return d;
        i++;
        c--;
Start:;
    } while( c!=0 );
    return d;
}
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

这个新代码看起来更长了，而且看起来增加了一些不必要复杂流程，但实际上这样帮助编译器把变量和寄存器映射起来。这年头可能很多注释会说 goto 和 labels 是糟糕的习惯，因为编译器会给 goto 关掉优化或者什么什么的原因，不过呢，我的编译器看起来不吃这一套说法，这也是为什么我用起了 goto。

假设我们的结构是这样:

```
struct foobar {
    unsigned int key;
    void * content;
};
```

然后这个是编译器 vs 我手工汇编的情况

!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```
; compiler
    xor     eax,eax
    test    ecx,ecx
    je      L2
L1: mov     eax,[esi]   ; 1 U -
    test    eax,eax     ; 2 U
    jne     L2          ; 2   V
    add     esi,8       ; 3 U
    dec     ecx         ; 3 V
    jne     L1          ; 5 - V +1brt
L2:
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

!!!!!!!!!ASM CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```
; by Pisen Chiang and Paul Hsieh
    xor     eax,eax
    test    ecx,ecx
    je      L2
    sub     esi,8
L1: add     esi,8       ; 1 U
    sub     ecx,1       ; 1   V
    sbb     eax,eax     ; 2 U -
    or      eax,[esi]   ; 3 U
    jz      L1          ; 4   V +1brt
    mov     eax,[esi]
L2:
```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

编译器终于取得了一点优势，它在代码大小上赢了一次，不过我的代码要快很多，因为我避过了一个额外的 test，还在循环里避过了一个跳转。就像注释里说的，我的代码在每个循环里降低一个 CPU 时钟的占用(奔腾 CPU 上)。

再深入些？唔，也不要太深入了。要不然就成了学术上面的研究了(而非实际应用的)。这个流程真正的问题所在是，我们访问的是一个对齐了的结构里面的一部分元素。如果这个数组的大小非常大，那我们就因为把结构里其他的元素抛弃掉而输掉了性能(CPU 的 cache 中只能储存连贯的一串数据)。把结构分开，把数组中的所有结构的所有元素都带上 locality of reference(本地引用 ?:所有的元素都要在循环里有 读或写的操作访问到它)，然后会比上面这一串优化还改观更多的性能。但是即便在那种情况下，我也认为大部分编译器会用一个性能随便的 repz scasd 就打发掉你想要有的代码优化(原话---)

原话-----But even in that situation I don't think most compilers will use a repz scasd.

总结下我们在这儿的成果，如果说你要处理的是一个结构很糟糕的数组(就像是 MS D3D API 用的顶点缓冲数组)，上面的代码还是很有用的。

3. 63 bits 的 LFSR

下面的是一个标准的 CRC(cyclic redundancy check computing 循环冗余校验计算)。它基本可以确定就是一个迭代性处理随机不确定的 63bits 输入的函数，然后返回 63bits。这个算法可以用来高速度(安全性不行的)加密，随机数产生，数据完整性效验，或者就当做一个 hash 函数。

!!!!!!!!!C CODE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```
typedef unsigned long UINT;
void TestC(UINT &b0, UINT &b1)
{
```
