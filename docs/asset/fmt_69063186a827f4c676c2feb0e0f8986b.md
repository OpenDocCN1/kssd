
# Vmprotect 编译字节码的策略 - 真是太强了

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-117.htm`](https://www.pediy.com/kssd/pediy09/pediy09-117.htm)

> `char* str = "this is string!haha";`  
> `__declspec(naked) func2()`  
>  
> `{`  
> `  VMPBEGIN`  
> `  MessageBox(0,"1","",MB_OK);`  
> `  __asm`  
> `  {`  
> `    retn`  
> `  }`  
> `  VMPEND`  
> `}`  
>  
> `DWORD tmp = 0;`  
> `DWORD Eeax = 0;`  
>  
> `__declspec(naked) func3()`  
>  
> `{`  
> `  VMPBEGIN`  
> `  __asm`  
> `  {`  
> `    mov eax,Eeax`  
> `    jmp [eax]`  
> `  }`  
> `  VMPEND`  
> `}`  
>  
> `__declspec(naked) func()`  
>  
> `{`  
> `  VMPBEGIN`  
> `  goto begin;`  
> `rets:`  
> `  func2();`  
> `  goto ends;`  
> `begin:`  
> `  __asm`  
> `  {`  
> `    lea ebx,rets`  
> `    mov tmp,ebx`  
> `  }`  
> `  Eeax = (DWORD)&tmp;`  
> `  __asm`  
> `  {`  
> `    jmp func3`  
> `  }`  
> `ends:`  
> `  __asm`  
> `  {`  
> `    retn`  
> `  }`  
> `  VMPEND`  
> `}`  
>  
> `int _tmain(int argc, _TCHAR* argv[])`  
>  
> `{`  
> `  DWORD ss = Eeax;`  
> `  func();`  
> `  getchar();`  
> `  return 0;`  
> `}`  
>  
> 以为 `jmp [eax]` 这种方法可以刁难 Vmprotect 了。  
> 没想到它更胜一筹。  
> 再找看看还有没能强奸它的语法。  
> 
> `****`  
> 
> 稍微变了下代码  
> `  VMPBEGIN`  
> 
> `  goto begin;`  
> `rets:`  
> `  __asm`  
> `  {`  
> `    push func2`  
> `    retn`  
> `  }`  
> `  goto ends;`  
> `begin:`  
> `  __asm`  
> `  {`  
> `    lea ecx,rets`  
> `    mov tmp,ecx`  
> `    lea ebx,tmp`  
> `    mov Eeax,ebx`  
> `    mov eax,Eeax`  
> `    jmp [eax]`  
> `  }`  
> `ends:`  
> `  __asm`  
> `  {`  
> `    retn`  
> `  }`  
> `  VMPEND`  
> 
> 发现顺利的挂了。  
> 原因如下。  
> Vmprotect 在解释 `call` 时会先判断是否有其他代码引用当前地址。当没有引用时就直接编译为字节码，有引用时就将 `call` 放在原处。其他地址引用时也就不会出现错误了。  
> 
> 而改为 `push func2` 时，就像没想的一样把这里编译为字节码了，后面的 `jmp [eax]` 理所当然的就出问题了。  
> 
> 如果说要让 Vmprotect 连这种类似自修改代码都支持的话，的确有点困难。  
> 
> 不过，VC 编译 `switch` 代码时会出现  
> `jmp [401000+ebx]` 跳到 `case` 语句去。  
> 像这种不知道 Vmprotect 的支持性怎么样...

4dd466d0e1e0cc123afbe504da2462ca
