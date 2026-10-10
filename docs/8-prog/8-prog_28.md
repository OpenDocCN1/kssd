

# VC++在源码级下调试 MASM32

> 原文：[`www.pediy.com/kssd/pediy05/pediy50355.htm`](https://www.pediy.com/kssd/pediy05/pediy50355.htm)

许多其它编译器都附带了源代码级的调试器,,这使得用这些高级语言调试比较简单,但对于 masm 并没有一个集成的开发和调试环境,我们可以用 TRW 和 Softice 实现源码级的调试，这两种方法，参看 Hume 兄的文章：http://www.finalseraph.org/hume/asmdata/sdbg.htm   
  
下面提供一种用 VC++6.0 调试的方法，好像比前两种更简单些。   
  
方法如下：   
\--------   
1、将我们的汇编程序编译连接成调试模式：   
  \\masm32\\bin\\ml /c /coff /Cp /Zi Myprog.asm   
  \\masm32\\bin\\link /DEBUG /DEBUGTYPE:CV /subsystem:windows    
  myProg.obj   
2、用 VC++6.0 建立一个同 ASM 程序同名字的工程：myProg，并将它编译连接   
  成 debug 模式，此时，在 VC 工程 myProg\\debug 中会形成 myProg.exe。   
3、将第 1 步骤中用 Masm32 编译连接成的同名 exe 文件 myProg.exe 拷贝到：   
  VC++工程 myProg\\debug 中，覆盖 VC++形成 myProg.exe   
4、启动 VC++6.0，打开 myProg 工程，build->start debug->go，呵呵。。   
  进入了 Masm32 形成的 myProg.exe，并且是源码级调试。   
  
  至此，请随心所欲的用 VC 调试器的强大功能，调试你的 Masm32 程序吧！   
  
附：我一般都是用个批处理(L\_debug.bat)一下完成：   
c:\\masm32\\bin\\ml /c /coff /Cp /Zi Myprog.asm >Mydebug.txt   
c:\\masm32bin\\link /DEBUG /DEBUGTYPE:CV /subsystem:windows   
  myProg.obj   
  copy .\\myProg.exe VC 工程\\debug\\myProg.exe   
启动 VC 进行调试。。。   
  
                                        Spring.W   
                                        2002/11/25  4dd466d0e1e0cc123afbe504da2462ca