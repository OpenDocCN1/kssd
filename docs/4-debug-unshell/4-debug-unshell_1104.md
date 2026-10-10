

# 对 Themida1.9.1.0 的通法破解一文的补充（再修正）

> 原文：[`www.pediy.com/kssd/pediy10/73425.html`](https://www.pediy.com/kssd/pediy10/73425.html)

【文章标题】: 对 Themida1.9.1.0 的通法破解一文的补充修正  
【下载地址】: 自己搜索下载  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
          前几天我的《Themida 1.9.1.0 的通法破解》一文的脚本对 VC++程序的解密存在一些问题(对汇编程序正常)。主要是解密后，函数在程序中的地址张冠李戴，脱壳后程序不能运行。当时实验的主程序是针对汇编的，附带实验了 VC++。由于 VC++动辄几百个函数，多则上千不可能一一核对其地址，故疏漏了一个大问题。  
          原来 Themida v1.9.1.0 加密程序时，对函数的“序列号装载”或“函数名装载”是分别在不同的地址中显示其真实地址的(与 1.8.5.5 版不同)，而我的脚本 1 搜索到 2 个(Addr\_1)地址，因汇编程序无序列号装载，另一个 Addr\_1 根本就不调用，故删除了，这就是问题所在。  
             一、Themida v1.9.1.0 通法破解的再修定  
  
    不好意思的一而再再而三的修定，Themida 的反跟踪能力比想象中的强，一个不起眼的“单步异常”常常让你的断点乱套，你不知道他什么时候跳出单步 SEH，为了通用性只好加强防范了。  
    这次给出的“脚本 1(断点侦测)”（兼容版）在与单步中断周旋上作了较大改进，同时也摘除了相应代码段的“代码扫描”，让侦测能够进行到底。改进后，用于在我收集的 Themida 加密软件中还没有出现“搜索失败”的提示。（1.8.0.0 以前的版本没有试验）  
    再就是发现：mov eax,ecx  的汇编代码既可以是“01 C8”，也可以是“03 C1”,所以使一些程序失败，现在增加了对 03C1 的搜索。  
    修改后的脚本如下:  
\-------------------------------  
    //Themida v1.8.0.0--1.9.1.0 兼容的断点设置脚本  
data:  
  var memory  
  var csize  
  var tmp  
  var temp  
  var tmpbp1  
  var tmpbp2  
  var mem  
  var mem0  
        var mem1  
  var mem2  
  var mem3  
   var mem4  
  var mem5  
  var Str0  
  var Str3  
  var Str4  
  var Str5  
  var Str1  
        var Str2  
  var Str11  
star:  
  gmi eip,CODEBASE  
        mov  memory,$RESULT  
        gmi eip,CODESIZE  
  mov csize,$RESULT  
        add csize,memory    
  find memory,#0000000000000000000000000000000000000000000000#  
  cmp $RESULT,0  
        jz  stop2  
  mov tmp,$RESULT  
  bphws tmp,"w"  
        esto  
  esto  
  esto  
  bphwc tmp  
        mov temp,eip  
C0:  
  sto  
        cmp eip,temp     //与单步 SEH 周旋  
        jz  C0  
  find memory,#0000000000000000000000000000000000000000000000#  
  mov tmp,$RESULT  
        sub csize,tmp  
        find memory,#909090909090909090909090909090909090#  
  cmp $RESULT,0  
        jz  stop2  
  mov temp,$RESULT+8  
  bphws temp,"w"  
        mov temp,eip  
C1:  
  run  
  cmp  eip,temp+2   //若是单步,则再 run  
        jz  C1  
  bphwc temp  
        mov  Str0,"断点地址："  
        add  Str0,#0D0A#  
  mov  mem,eip  
  mov  mem4,eip           //eip=Addr\_4  
  itoa eip  
  mov  Str4,"Addr\_4="  
        add  Str4,$RESULT  
        add  Str4,#0A0D#  
        find mem4,#83BD????????000F84??0000003007#   
  cmp $RESULT,0  
        jz  stop2   
        mov  mem5,$RESULT       //Addr\_5  
        bp   mem5  
        itoa mem5   
        mov  Str5,"Addr\_5="  
        add  Str5,$RESULT   
        sub  mem,4000   
  find mem,#83BD????????640F82????0000#  
  cmp $RESULT,0  
        jz  stop2  
        mov  mem0,$RESULT+7    //Addr\_0  
        mov \[mem0\],#90E9#      //改 jb 为 jmp,摘除断点扫描  
        itoa mem0  
        add  Str0,"Addr\_0="  
        add  Str0,$RESULT   
        add  Str0,#0D0A#  
        find mem0,#C1C003#  
        find $RESULT,#0385????????#,40  
  cmp $RESULT,0  
        jz  stop2  
        mov  mem3,$RESULT+6   //Addr\_3  
        itoa mem3  
        mov  Str3,"Addr\_3="  
        add  Str3,$RESULT   
        add  Str3,#0D0A#  
        mov  mem4,0          //mem4 改为计数  
        mov  mem2,mem0  
findon:  
  find mem0,#AD#       //从 Addr\_0 开始查找  
        cmp  $RESULT,0  
        jz   stop  
        mov  mem0,$RESULT  
  find mem0,#01C8#,40  //还有可能是#03C1#  
        cmp  $RESULT,0       
  jnz  findoff  
  find mem0,#03C1#,40    
        cmp  $RESULT,0       
  jnz  findoff  
        add  mem0,40  
        cmp  mem0,mem5  
        ja   stop            //超过 Addr\_5 后停止  
        jmp  findon  
findoff:  
        cmp  mem4,0  
        jnz  D1  
        mov  tmpbp1,$RESULT  
        bp   $RESULT  
        inc  mem4  
  add  mem0,40  
        jmp  findon  
D1:          
        mov  tmpbp2,$RESULT  
        bp   $RESULT  
        add  mem0,40  
        jmp  findon  
stop:   
        cmp  mem4,0  
        jz   stop2  
B2:  
  run  
  cmp  eip,tmpbp1   //若是单步,则再 run  
        jz  C2  
        cmp  eip,tmpbp2  
  jz  C2  
        jmp B2  
C2:  
  bc eip  
@1:          
        sti  
        gn   eax  
        cmp  $RESULT,0  
        jz  @1        
  itoa eip  
        mov  mem1,$RESULT  
        mov  Str1,"Addr\_1\_0="  
        add  Str1,$RESULT  
        add  Str1,#0D0A#  
  bpwm tmp,csize  
        run  
@3:  
  sti  
        cmp \[eax\],ecx  
        jnz @3  
  itoa eip         
  mov  Str2,"Addr\_2="  
  add  Str2,$RESULT  
  add  Str2,#0A0D#  
        mov  $RESULT,mem1  
        bpmc  
B3:  
  run  
  cmp  eip,tmpbp1   //若是单步,则再 run  
        jz  C3  
        cmp  eip,tmpbp2  
  jz  C3  
  cmp eip,mem5  
        jz  @4  
        jmp B3  
C3:  
  bc eip  
@2:          
        sti  
        gn   eax  
        cmp  $RESULT,0  
        jz  @2        
  itoa eip  
@4:  
        mov  Str11,"Addr\_1\_1="  
        add  Str11,$RESULT  
        add  Str11,#0D0A#  
        add  Str0,Str1  
  add  Str0,Str11  
        add  Str0,Str2  
        add  Str0,Str3  
        add  Str0,Str4  
        add  Str0,Str5  
        msg  Str0  
        bc  
        pause  
stop2:  
        msg  "查找失败"  
        bc  
        pause  
     
         二、网友们反映脚本不能运行的问题  
         脚本和提供的附件是经过无数次的实验运行的，绝对不是脚本的问题。唯一不同的是 OD 版本号，插件的使用。请下载我上文提到的版本和插件。okdodo 的插件“invisible”很管用，无须更改 OD 的类名称“AAAMAIN”。  
         特别提醒：运行脚本，必须用“ODbgScript”打开，绝不能用“OllyScript”打开。若没有 ODbgScript，请立即下载该插件。  
         须要注意的是，电脑中不能运行 softIce，不能同时运行两个 OD(不管是不是加载同一程序)。操作系统不应该是问题。  
     
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2008 年 09 月 25 日 22:35:21

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || temp.rar \[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=73425`](http://bbs.pediy.com/showthread.php?t=73425)\]   
 |