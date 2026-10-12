
# 论用 C++给 dll 加壳【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-572.htm`](https://www.pediy.com/kssd/pediy08/pediy8-572.htm)

写这篇文章,还是为了分析我写加壳程序的经历,大家一同进步.

给 dll 加壳在 kanxu 老大的书里面有很仔细的论述,不过都是汇编的,我看了有点晕,飞天诚信的书写的又是欲盖弥彰.所以我还是更多的参考了 bigboot 的文章.

dll 理论上和 exe 是一样的,但 dll 多了几个东西.

1.`export` 表,`exe` 是没有这个东西的(`borland` 编译的 `exe` 似乎也有这个东西).所有的函数要在 `export` 表中输出.这个东西在我们自己的 `loader`(`stub`)加载前就加载了,所以我们要维持这些字符串的原装,对每一个地址加上我们的偏移.当然也可以自己构造新的 `export` 结构. 总之处理方法和 `iat` 基本一致.

2. `reloc` 表,这个东西 `exe` 绝对没有了.`exe` 都是 `40000` 加载的,但 `dll` 默认是 `100000`,而且可能变化,所以一定要处理 `reloc`.我刚开始做的时候有点复杂了,后来想了一下,没有必要把我们自己 `stub` 的 `reloc` 和原来 `dll` 的 `reloc` 合并,因为我们如果不用线程什么的,`reloc` 就是原来的样子,不用合并.处理的方法还是遍历,把所有的 `thunk` 都走一遍,计算一下偏移,每个 `thunk` 都加上这个偏移.

3.`tls`,这个基本不用考虑,如果想做,就用 `tls` 代理技术吧,`bigboot` 和我前面的文章都有写.注意别把地址和数据搞混了.

4. 一个全局变量,我们可以设为 `bFirstRun`.一定我们的 `stub` 加载过一次,就让 `bFirstRun=false`,以后就不在进行 `stub` 的操作,直接跳转.

5. 跳转! 这个比较关键,因为和 `exe` 不一样,`dll` 还要回到其他程序中去,所以栈绝对不能出错,我自己就在这里走了不少弯路(`C++` 就是这点不好,要是 `asm` 自己可以控制栈了). 当 `dll` 被加载时,过程时 `exe->dllmain`, `dll` 中函数被调用是 `exe` -> `dllmain->` `dll` `function`. 那系统是怎么判断那个函数 `dllmain` 执行完毕的呢?就是栈.所以在跳转到 `dllmain`(其实是 `entrypoint`)前,栈中的数据一定要保证和刚刚开始一直. 这里就说一下 `c++` 的实现方法. 我们自己的 `oep` 中,主函数一定要用 `void __declspec(naked)`, 这样就没有 `prolog/epilog`, 也就是在进入函数时,栈没有变化. 当然里面的函数可以不用这样说明(事实上,就不能嵌套). 在 `stub` 处理过所有操作后,我们必须使用汇编跳转, `__asm jmp oep;`

代码应该是

```cpp
void __declspec(naked) StubEntryPoint()
{
  if(bFirstRun)
    {
     bFirstRun=false;
     // stub operation
    }
  __asm jmp oep;
}
```

总结,`C++` 做壳原理和 `asm` 完全一样,但细节由于语言的限制还是很不一样.`c` 的代码可读性很不错,如果和 `asm` 配合,应该可以写出很不错的壳来.

下面的代码是处理 `reloc` 的

```cpp
void PerformRelocations ()
{
  //see if no relocation records
  if ( gev.dwOriginalRelocVA == 0 )
    return;
  PIMAGE_DOS_HEADER pDosHdr;
  PIMAGE_NT_HEADERS pNtHdr;
  PIMAGE_SECTION_HEADER pSecHdr;
  DWORD dwSecStart;
  DWORD dwKatSup;
  LONG lJmp;
  WORD wNumSections;
  WORD wSizeO;
  pDosHdr = (PIMAGE_DOS_HEADER)dwLoadAddress;
  lJmp = pDosHdr->e_lfanew;
  dwKatSup = (DWORD)pDosHdr;
  dwKatSup += lJmp;
  pNtHdr = (PIMAGE_NT_HEADERS)dwKatSup;
  //compute offset
  DWORD reloc_offset = dwLoadAddress - pNtHdr->OptionalHeader.ImageBase;
  //if we're where we want to be, nothing further to do
  if ( reloc_offset == 0 )
    return;
  //gotta do it, compute the start
  IMAGE_BASE_RELOCATION* ibr_current = (IMAGE_BASE_RELOCATION*)(gev.dwOriginalRelocVA + dwLoadAddress );
  //compute the end
  IMAGE_BASE_RELOCATION* ibr_end = (IMAGE_BASE_RELOCATION*)(&((unsigned char*)ibr_current)[gev.dwOriginalRelocSize]);
  //loop through the chunks
  while ( ibr_current < ibr_end && ibr_current->VirtualAddress )
  {
    DWORD RVA_page = ibr_current->VirtualAddress;
    int nCountReloc = ( ibr_current->SizeOfBlock - IMAGE_SIZEOF_BASE_RELOCATION ) / sizeof(WORD);
    WORD awRelTypenIdx = (WORD)((unsigned char*)ibr_current + IMAGE_SIZEOF_BASE_RELOCATION);//???
    for ( int i = 0; i < nCountReloc; ++i )
    {
      WORD wType = awRelTypenIdx >> 12;
      WORD wValue = awRelTypenIdx & 0x0fff;
      if ( wType == IMAGE_REL_BASED_HIGHLOW )
      { //do it
        *((DWORD*)(RVA_page + wValue + dwLoadAddress)) += reloc_offset;
      }
    }
    ibr_current = (IMAGE_BASE_RELOCATION*)(&((unsigned char*)ibr_current)[ibr_current->SizeOfBlock]);
  }
}
```

下一个题目就是 `anti` 了,这个都是 `crack` 的高手,我得多花些时间了.

文章内容太简单,大家不要笑话

`4dd466d0e1e0cc123afbe504da2462ca`
