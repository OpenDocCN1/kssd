

# [原创]第一章:1.7、加法与减法的识别与优化原理

其实在大多数逆向工作中我们所面对的都是算法，也可以称之为一个软件的灵魂，而通常意义上的算法肯定由各种复杂的数学公式组成的，因此如果我们要真正的看懂一个加密、解密或压缩算法的话，那么学习运算符的逆向技巧还是非常必须的。下面我们就由简入繁，逐一为各位读者剖析逆向工程中数学运算的精要。

## 1.7.1、加法的识别与优化技巧

加法的优化相对来说比较简单，只有3种优化方案，下面我们就以一个简单的例子来说明这三个问题，先看源码：

```
int _tmain(int argc, _TCHAR* argv[])
{
        int nNum, nA = 8;
        nNum = argc + nA;      // 形式1
        printf("%d\r\n",nNum);
        nNum = argc + 9;       // 形式2
        printf("%d\r\n",nNum);
        nNum = nNum + 1;       // 形式3
        printf("%d\r\n",nNum);
        return 0;
}
```

Debug版反汇编代码：

```
.text:0041301E     mov [ebp+nA], 8                     ; nA = 8
.text:00413025     mov eax, [ebp+argc]                 ; eax=argc
.text:00413028     add eax, [ebp+nA]                   ; eax=eax+nA  <-- !!
.text:0041302B     mov [ebp+nNum], eax                 ; nNum=eax
.text:00413030     mov eax, [ebp+nNum]                 ; eax=nNum=argc+nA
.text:00413033     push eax
.text:00413034     push offset Format                  ; "%d\r\n"
.text:00413039     call ds:__imp__printf
.text:0041303F     add esp, 8
.text:00413049     mov eax, [ebp+argc]
.text:0041304C     add eax, 9                          ; eax=argc+9  <-- !!
.text:0041304F     mov [ebp+nNum], eax
.text:00413054     mov eax, [ebp+nNum]
.text:00413057     push eax
.text:00413058     push offset Format                  ; "%d\r\n"
.text:0041305D     call ds:__imp__printf
.text:00413063     add esp, 8
.text:0041306D     mov eax, [ebp+nNum]
.text:00413070     add eax, 1                          ; eax=nNum+1  <-- !!
.text:00413073     mov [ebp+nNum], eax
.text:00413078     mov eax, [ebp+nNum]
.text:0041307B     push eax
.text:0041307C     push offset Format                  ; "%d\r\n"
.text:00413081     call ds:__imp__printf
.text:00413087     add esp, 8
```

通过上面的反汇编代码我们可以看到三个在普通不过的加法计算，下面我们再看看Release版的反汇编代码：

```
.text:00401000 _main proc near                         ; CODE XREF: __tmainCRTStartup+10Ap
.text:00401000
.text:00401000 argc= dword ptr  4
.text:00401000 argv= dword ptr  8
.text:00401000 envp= dword ptr  0Ch
.text:00401000
.text:00401000     push esi
.text:00401001     mov esi, ds:__imp__printf
.text:00401007     push edi
.text:00401008     mov edi, [esp+8+argc]
.text:0040100C     lea eax, [edi+8]                    ; 优化后的加法
.text:0040100F     push eax
.text:00401010     push offset Format                  ; "%d\r\n"
.text:00401015     call esi ; __imp__printf
.text:00401017     add edi, 9                          ; 此处并没有什么优化
.text:0040101A     push edi
.text:0040101B     push offset Format                  ; "%d\r\n"
.text:00401020     call esi ; __imp__printf
.text:00401022     inc edi                             ; 优化后的加法
.text:00401023     push edi
.text:00401024     push offset Format                  ; "%d\r\n"
.text:00401029     call esi ; __imp__printf
.text:0040102B     add esp, 18h
.text:0040102E     pop edi
.text:0040102F     xor eax, eax
.text:00401031     pop esi
.text:00401032     retn
.text:00401032 _main endp
```

由上面两段反汇编代码我们可以总结出加法计算的以下优化方案：

`变量+变量 = lea Exx,[变量+变量]`

`变量+常量 = add 变量+常量`

`变量+1    = inc 变量`

## 1.7.2、减法的识别与优化技巧

减法的优化与加法大同小异，基本相同，因此与加法相同的地方笔者在本文中将不再多说，我们先看源代码：

```
int _tmain(int argc, _TCHAR* argv[])
{
        int nNum, nA = 8;
        nNum = argc - nA;      // 形式1
        printf("%d\r\n",nNum);
        nNum = argc - 9;       // 形式2
        printf("%d\r\n",nNum);
        nNum = nNum - 1;       // 形式3
        printf("%d\r\n",nNum);
        return 0;
}
```

这次我们直接看Release版反汇编代码：

```
.text:00401000 _main           proc near               ; CODE XREF: __tmainCRTStartup+10Ap
.text:00401000
.text:00401000 argc            = dword ptr  4
.text:00401000 argv            = dword ptr  8
.text:00401000 envp            = dword ptr  0Ch
.text:00401000
.text:00401000                 push    esi
.text:00401001                 mov     esi, ds:__imp__printf
.text:00401007                 push    edi
.text:00401008                 mov     edi, [esp+8+argc]
.text:0040100C                 lea     eax, [edi-8]    ; 减法优化
.text:0040100F                 push    eax
.text:00401010                 push    offset Format   ; "%d\r\n"
.text:00401015                 call    esi ; __imp__printf
.text:00401017                 add     edi, 0FFFFFFF7h ; 减法优化 <--!
.text:0040101A                 push    edi
.text:0040101B                 push    offset Format   ; "%d\r\n"
.text:00401020                 call    esi ; __imp__printf
.text:00401022                 dec     edi             ; 减法优化
.text:00401023                 push    edi
.text:00401024                 push    offset Format   ; "%d\r\n"
.text:00401029                 call    esi ; __imp__printf
.text:0040102B                 add     esp, 18h
.text:0040102E                 pop     edi
.text:0040102F                 xor     eax, eax
.text:00401031                 pop     esi
.text:00401032                 retn
.text:00401032 _main           endp
```

通过以上代码我们不难发现减法的优化方案与加法基本相同，唯一不同的就是在“形式2”上的体现，我们可以发现编译器将其优化成了一个加法，那么这究竟是为什么，而其原理又是怎样的呢？

仅从指令周期上来讲，老版本的CPU其加法指令周期要比减法短一些，因此这种优化也就这样一直沿袭了下来。而仅从本条指令上的优化来说，编译器将原本的减法转换成了加法，并将常量转成了其原本的补码，我们都知道减一个数与加这个数的补码所得到的结果都是一样的，因此编译器就利用了这个特性。

总结：

`变量-变量 = lea Exx,[变量-变量]`

`变量-常量 = add 变量+补码(常量)`

`变量-1    = dec 变量`

## 后记：

鉴于其他运算符的逆向的复杂性，因此后面的章节可能会有所改动，以求将篇幅调整合适，以便于读者们的阅读与学习。（笔者近几日一直在弄房子，因此耽搁些时间，还望各位读者见谅）

[【返回到目录】：http://bbs.pediy.com/showthread.php?t=113689](http://bbs.pediy.com/showthread.php?t=113689)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

