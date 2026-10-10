

# 字符串函数的汇编实现

> 原文：[`www.pediy.com/kssd/pediy11/107189.html`](https://www.pediy.com/kssd/pediy11/107189.html)

在项目开发过程中，对字符串的处理是经常遇到的问题，稍有不慎，就会留下一些不易察觉的瑕疵；熟练使用字符串函数，理解其实现过程，对编程会大有裨益；  
  
在开始之前，请思考一下，是否真的明白了这样几个函数的用法。  
1. sizeof,这不是函数，而是一个运算符，在编译时就已经计算好了参数所占用的内存空间，并分配；  
2. strlen，函数，运行时计算参数中的元素个数，不包括'\\0'结束符  
对应的版本有两个：strlenA 窄字符版本,strlenW 宽字符版本  
3. strcpynA, strcpyA 的区别，strcpynA 中的 unsigned long len 长度表示的是内存长度还是元素个数,从什么地方开始计算  
4. 快速求出一个字符串的最后元素，可以用前面方法的组合，也可以自己实现一个函数 strendA  
5.在前面几个函数的基础上，就可以自己实现 strcatA 等一些扩展函数  
  
6.如果上面的都十分清楚，那可以分析下内部实现，增强功力；汇编、数据结构、这些都是作为一个程序员所应具备的功底。  
  
一点说明：WINDOWS 提供的大多库函数都是 fastcall 调用约定，顾名思义，就是为了提高传输参数的效率而使用的一个调用约定；我们知道，传递参数，要么用堆栈，要么用寄存器，而 fastcall 调用约定就是二者的结合；前两个参数用 ecx,edx 分别传递，之后的参数用堆栈从右向左依次传递。  
  
现在，感兴趣的可以看下面的代码。代码是我经过严格测试的，注释也是为了方便大家阅读加上去的。  
  

代码:

```

//
//  ANSI 字符串求元素个数
//

\_\_declspec(naked) unsigned long \_\_fastcall strlenA(const char \*s)
{
  \_\_asm
  {
    mov edx, edi    //保存 edi 的值，到后面用 mov edi,edx 恢复
    mov edi, ecx    //将参数 ecx 传递给 edi，也就是说 edi 指向的内存空间存储字符串 s
    or ecx, -1      //初始化 ecx 为 0FFFFFFFFh
    xor eax, eax    //初始化 eax =0

    repne scasb      //scan byte，字符串搜索，在 edi 指向的内存空间中搜索与 al 相等的值，也就是结束符 0      
    
    dec eax        // eax-1,这里，eax = -1,和 repne scasb 执行之前的 ecx 的值一样
    dec eax        // eax-1,相当于减去了'\\0'的长度

    sub eax, ecx    //执行 repne scasb 的过程中，每循环一次，ecx--,那么 sub eax,ecx 就是元素的个数了
              //这里比较巧妙，运算都是在负数范围内，元素的个数也就是 eax 与 ecx 相对与 0 的距离之差
    mov edi, edx    
    retn        //返回值在 eax 默认返回
  }
}

//
//  WIDE CHAR 字符串求元素个数
//
\_\_declspec(naked) unsigned long \_\_fastcall strlenW(const wchar\_t \*s)
{
  \_\_asm
  {
    mov edx, edi
    mov edi, ecx
    or ecx, -1
    xor eax, eax
    repne scasw    //scan word，结构和思想与 strlenA 函数一样，只是这里，我们是每次比较 word 长度的单位(2 字节)
          //每循环一次，ecx--,edi +=2
    dec eax
    dec eax
    sub eax, ecx
    mov edi, edx
    retn
  }
}

```

代码:

```
//
//  ANSI 字符串的 copy 函数
//  参数三：unsigned long len：表示要 copy 的长度，即元素个数  
//

\_\_declspec(naked) char\* \_\_fastcall strcpynA(char \*dest, const char \*src, unsigned long len)
{
  \_\_asm
  {
    push edi
    push esi
    mov edi, ecx      //fast call 调用约定，第一个参数用 ecx 传递，第二个参数用 edx 传递
    mov eax, ecx      //现在，edi 保存 dest 指向的内存地址

    mov esi, edx      //esi 保存 src 指向的内存地址

    mov edx, \[esp + 12\]                   //传递将要 copy 的长度 len，第三个参数用堆栈传递，\[esp+8\] 就是返回地址    
    mov ecx, edx      //字符串的长度转存到 ecx 中

    shr ecx, 2        //长度缩小 4 倍，其实就是循环次数缩小 4 倍，每次传递 4 个字节

    repnz movsd      //mov dword

    mov ecx, edx      //余数部分，每次只传递 1 个字节
    and ecx, 3      //取余数部分，mod 4 的余数

    repnz movsb
    
    mov byte ptr \[edi\], 0                  //字符串结尾处，填充 0
    
    pop esi        //按照进栈的顺序，弹栈，使堆栈平衡
    pop edi
    
    retn 4
  }
}

//
//  WIDE CHAR 字符串的 copy 函数
//  参数三：unsigned long len：表示要 copy 的长度，即元素个数
//

\_\_declspec(naked) wchar\_t\* \_\_fastcall strcpynW(wchar\_t \*dest, const wchar\_t \*src, unsigned long len)
{
  \_\_asm
  {
    push edi
    push esi
    mov edi, ecx
    mov eax, ecx      //让 eax 指向目的字符串的地址\[ecx\]
    mov esi, edx
    mov edx, \[esp + 12\]                   //保存要传递的元素个数

    mov ecx, edx
    shr ecx, 1        //循环次数缩小 2 倍，每次传输两个 word 长度，也就是 4 字节

    repnz movsd        //每次传递 4 个字节
    mov ecx, edx

    and ecx, 1        //取余数部分，mod 2 的余数

    repnz movsw        //每次循环传递一个 word 长度，也是最小单元传输长度                
    
    mov word ptr \[edi\], 0  //字符串结尾，填充 0
    
    pop esi
    pop edi
    
    retn 4          //返回的字符串保存于 eax
  }
}

```

代码:

```
//
// 求 ANSI 字符串的最后一个元素
//

\_\_declspec(naked) char\* \_\_fastcall strendA(const char \*s)
{
  \_\_asm
  {
    mov edx, edi
    mov edi, ecx
    or ecx, -1
    xor eax, eax
    repnz scasb
    lea eax, \[edi - 1\]  //此时，edi 已经指向字符串的\\0 处，\[edi-1\]指向字符串的最后一个元素
    mov edi, edx
    retn
  }
}

//
//  求 WIDE CHAR 字符串的最后一个元素
//
\_\_declspec(naked) wchar\_t\* \_\_fastcall strendW(const wchar\_t \*s)
{
  \_\_asm
  {
    mov edx, edi
    mov edi, ecx
    or ecx, -1
    xor eax, eax
    repnz scasw
    lea eax, \[edi - 2\]
    mov edi, edx
    retn
  }
}

char\* \_\_fastcall strcatA(char \*dest, const char \*src)
{
  strcpyA(strendA(dest), src);
  return dest;
}

wchar\_t\* \_\_fastcall strcatW(wchar\_t \*dest, const wchar\_t \*src)
{
  strcpyW(strendW(dest), src);
  return dest;
}

```

  
希望大家感兴趣。  
别说我古董，我大学还没毕业，不过也快了。但是自己深知道，编程这东西，理解的越透彻，越深入，才会游刃有余；  
现在大家知道，假如求一个字符串的长度，时间复杂度是多少，不用说，都是 O(N)啊，不要因为我们平时见不到这些内部实现，就认为 CPU 会做的多智能...  
顺祝大家春节快乐！