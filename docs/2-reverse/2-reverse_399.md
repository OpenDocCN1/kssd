

# 自己制做 IDA 签名文件批处理！外带 1 个问题

> 原文：[`www.pediy.com/kssd/pediy12/136243.html`](https://www.pediy.com/kssd/pediy12/136243.html)

IDA 很强大，签名文件更强大，帮助我们减少了好多工作。  
照这 tnttools 大侠的贴子自己也写了个批处理，在这感谢 tnttools 大侠！  
我这个批处理可以处理当前目录下所有的 LIB 文件，当然最后一步还是需要手工去处理错误文件！做好的文件会放在 pat 目录中！自己边学批处理边做的，呵呵，花了一上午时间，大家莫笑！  
前提是要将 Lib.exe sigmake.exe 两个文件放在搜索路径里面；  
  
  

代码:

```
md pat
for %%i in (\*.lib) do lib /NOLOGO /list %%i >%%i.txt & md %%i.obj
for %%i in (\*.txt) do (cd %%~ni.obj\\ & (for /f %%j in (..\\%%~nxi) do  LIB ..\\%%~ni /NOLOGO /EXTRACT:%%j)&pcf -g0 \*.obj ..\\pat\\%%~ni.pat & del \*.obj &cd..&rd %%~ni.obj & del %%i)
FOR %%i in (pat\\\*.pat) do sigmake -n"test" %%i pat\\%%~ni.sig

```

我还有 1 个问题  
1.需要对 VC 下的 lib 目录下的所有 LIB 文件制做签名吗？有没有必要？