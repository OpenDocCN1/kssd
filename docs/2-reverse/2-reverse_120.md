

# OllyHTML 脚本详解（二）基本原理

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-293.htm`](https://www.pediy.com/kssd/pediy08/pediy8-293.htm)

【文章标题】: OllyHTML 脚本详解（二）基本原理  
【文章作者】: dreaman  
【作者邮箱】: dreaman\_163@163.com  
【作者主页】: http://dreaman.haolinju.net  
【软件名称】: OllyHTML  
【软件大小】: 800KB  
【下载地址】: http://dreaman.haolinju.net  
【加壳方式】: 无  
【保护方式】: 无  
【编写语言】: c++  
【使用工具】: vc7.1  
【操作平台】: win2000 以上系统  
【软件介绍】: Ollydbg 的插件，用以支持用 DHTML 作脚本  
【作者声明】: 个人觉得用 DHTML 作脚本是能做一些比较复杂的事情的，希望大家能用用：）  
  
OllyHTML 插件实际上是一个 IE 浏览器，我们利用 IE 对象模型的扩展能力扩充了 DOM(文档对象模型)，也就是加入了许多 Ollydbg 提供的功能，这样一来，  
我们可以用网页里的脚本操作扩展 DOM，也就是可以用脚本调用 Ollydbg 的功能了。  
IE 对象模型的扩充主要是通过 window.external 来引入，所以 OllyHTML 提供的功能都通过 window.external 来引入，我们将主要的功能  
封装成对象，并使用一个根对象作为这些对象的容器，这个根对象通过 window.external.Application 来访问。在脚本的包含文件  
const.js 中我们定义了一个全局变量 app:  
var app=window.external.Application;  
这样只要我们包含了这个文件，我们在脚本中就可以用 app 来访问插件提供的功能了。  
这便是插件如何为脚本提供 API 的基本原理，至于每个具体功能对象，这是我们后面各部分要详细讲述的内容，这里就不提了。  
  
一般说来，我们写的 OllyHTML 脚本有两种工作情形：  
一、一种情形是脚本提供一个 UI，当用户点击上面的功能按钮时执行对应的脚本，脚  
本再调用插件提供的 API，操纵 Ollydbg 执行某种行为或者从 Ollydbg 读取信息，这种方式一般用于静态分析;  
二、另一种情形是 Ollydbg 调试程序时产生了调试事件，由插件调用脚本里的事件处理，在事件处理中操纵 Ollydbg 执行某种行为或者从 Ollydbg 读取信息  
，这种方式用于动态调试，此时脚本所做的就像我们用快捷键或菜单命令控制 Ollydbg 设断点、单步执行、单步跟进等等，而调试事件的产生相当于 Ollydbg  
执行完了我们前面的一个操作。  
  
下面我们分别就两种情形各看一个例子。  
例 1（静态分析）：模糊查找反汇编指令  
<html>  
  <head>  
    <title>模糊查找反汇编指令</title>  
    <script src="res://OllyHTML.dll/const.js"></script>  
  </head>  
  <body>  
    <table style="font-size:14px;width:100%;height:100%;border-collapse:collapse" cellspacing="0" cellpadding="0" align="center">  
      <tr>  
        <td>  
          <textarea id="resultArea" style="width:100%;height:100%;" rows="10" ondblclick="gotoAddr()" WRAP="off"></textarea>  
        </td>  
      </tr>  
      <tr style="height:24px">  
        <td align="center">  
          ADDR:<input type="text" id="addrId" value="00401000">  
          ASM CODE:<input type="text" id="asmCode" value="">  
          <input type="button" value="run" onclick="findCode()">  
        </td>  
      </tr>        
    </table>  
    <script>  
      //在用户双击结果编辑框里的地址时让 Ollydbg 跳到相应反汇编地址  
      function gotoAddr()  
      {  
        var txtRange=document.selection.createRange();//获取当前选中的文本范围  
        var val=parseInt(txtRange.text,16);//将选中的文本字符串转成整数  
        if(val>0x01000)  
          app.Analyser.Cpu(val,val,0);//让 Ollydbg 的 CPU 窗口的反汇编与 DUMP 窗格显示选中地址的内容  
      }  
    </script>  
    <script>  
      var addr=0;  
      //这个函数是 DHTML window 对象的事件 onload 的处理，这是 DHTML 中一种事件处理函数的写法（大多数  
      //OllyHTML 脚本都需要使用这个事件处理）。  
      function window.onload()  
      {  
        var tinfo=app.Analyser.GetThreadInfo(app.CpuThreadId);//读取当前 CPU 窗口代码所属线程的线程信息  
        if(!tinfo)  
        {  
          return;  
        }  
        var minfo=app.Analyser.GetModuleInfo(tinfo.Entry);//读取当前线程入口点所在的模块的信息  
        var mi=app.Analyser.GetMemoryInfo(minfo.Base);//读取指定模块基地址所属内存段的信息（通常这是第一个段，也就是 PE 文件头）  
        mi=app.Analyser.GetMemoryInfo(mi.Base+mi.Size+1);//获取紧随第一个内存段的内存段的信息  
        addrId.value=window.external.IntToHex(mi.Base);//将第二个内存段的起始地址作为搜索的起始地址，初始化 UI 上的起始地址编辑框  
      }  
      //这个函数当用户点击 UI 上的 run 按钮时被调用  
      function findCode()  
      {  
        addr=window.external.HexToInt(addrId.value);//将 16 进制字符串转为整数，作为搜索的起始地址  
        var cond=app.StringFields.Disasm.Like(asmCode.value);//构造一个搜索条件，这是 OllyHTML 提供的用来构造搜索条件的  
                                                            //Lambda 表达式系列对象在后面的部分我们会详细介始它，现在我们不用深究  
        var lines=new Array();  
        for(;;)  
        {  
          var dasm=app.Analyser.FindDisasm2(addr,cond,true);//从指定地址开始搜索符合指定条件的汇编代码，忽略大小写（第三个参数是 0.7.0.1 版本的插件加入的，之前的版本没有这个参数）  
          if(!dasm)//找不到，退出  
          {  
            break;  
          }  
          lines.push(window.external.IntToHex(dasm.IP)+":"+dasm.Disasm);//将结果字符串添加到数组中，以供稍后一次性显示  
          addr=app.Analyser.GetNextOPAddr(dasm.IP,1);//取下一条指令地址  
          window.external.Title=""+lines.length;//用窗口标题栏显示进度信息，这只是个取巧的用法，也可以自己在 UI 提供一个  
                                                //状态显示控件  
        }  
        addrId.value=window.external.IntToHex(addr);      
        resultArea.value=lines.join("\\r\\n");//将所有搜索到的信息用回车换行分隔，然后显示到结果编辑框中  
      }  
    </script>  
  </body>  
</html>  
  
这个脚本比我们（一）中的 Hello world 要复杂一些，不过结构上是一样的，HTML HEAD 里除标题不一样，内容是一样的，就是引用我们的脚本  
常量资源文件。  
HTML BODY 里照例是 UI 与功能脚本两大块，UI 部分仍然使用 TABLE 来布局，二行一列，第一行是我们用来显示结果的多行文本编辑框，第二行  
我们放了两个文本框，一个按钮，分别用于输入起始地址、输入搜索代码以及执行搜索功能。  
注：我们在多行文本框上添加了一个双击事件处理，用于当用户双击某个地址时让 Ollydbg 跳到相应地址以便于观察上下文代码，gotoAddr()这个函数  
在我们的许多脚本里可以用上。  
这个脚本在执行前我们应该先装入被调试程序，并且最好已经分析过代码，让 CPU 窗口显示主程序代码，这几乎是所有静态分析脚本在执行前的要求。  
  
例 2（动态调试）：二次内存断点找 OEP 入口  
动态调试代码与静态分析代码的主要差别有三处：  
1、需要有一个允许调试事件的启动过程，这个步骤通过两个 API 调用实现：  
  
          app.EnableDebug();//允许脚本接收调试事件，也就是说之后插件会在产生调试事件时调用相应的 window.OnPaused  
                            //\[或 window.OnStep（这个一般不用，Ollydbg 会周期性调用它）,  
                            //window.OnReset（打开新被调试程序或重启被调试程序时调用）\]  
          app.Execution.StepInto();//单步跟进，这个主要是启动调试，也可以用 app.Execution.StepOver()或 app.Execution.ExceptionContinue()  
                            //、app.Execution.Run();  
  
2、提供一个调试事件处理，并在其中调用与 Ollydbg 进行调试会话的逻辑（就是判断当前是什么事件，然后依据不同情形让 Ollydbg 做点什么）:  
          function window.OnPaused(reason,extdata,reg,debugEvent)  
          {  
            ...  
            return 1;  
          }  
这是事件处理的写法，其中的参数：  
  reason --- 产生事件的原因，将它与 PP\_MAIN 相与可得：  
        PP\_EVENT 表明是调试事件，此常量值为 0  
        PP\_PAUSE 表明是用户操作请求的暂停   
        PP\_TERMINATED 表明是程序结束  
        对于调试事件，我们可以直接用下面常量值判断其类型：  
        PP\_BYPROGRAM --- 由于程序倒致的调试事件，比如程序里的 int3 指令（不是我们设置的断点）   
        PP\_INT3BREAK --- 我们设置的 int3   
        PP\_MEMBREAK  --- 内存断点   
        PP\_HWBREAK   --- 硬件断点  
        PP\_SINGLESTEP --- 单步   
        PP\_EXCEPTION --- 异常，就是我们通常想要异常继续或看异常处理的那种异常   
        PP\_ACCESS --- 访问异常，经常在 C++程序里看到的"内存不可读,0xc0000005"应该就是此类了   
        PP\_GUARDED --- 这个很特别，偶跟了一个 Ollydbg，它的访问断点就是产生这个事件，我们可以用 VirtualProtectEx 改变一个内存段属性，加上  
                        PAGE\_GUARD|PAGE\_EXECUTE\_READWRITE 标志，就会在访问到该内存时产生这个事件，而且这个事件是一个 OneShot 型的，只触发  
                        一次，之后除非再次设定内存段属性，否则就不产生这个事件了（这个做法与 OD 访问断点是一样的实现方法）。   
  extdata --- 这个参数没有用处，不用理它  
  reg --- 产生调试事件时的寄存器情况，这是一个对象，我们在其它部分详细说明  
  debugEvent --- 调试事件信息，这也是一个对象，我们在其它部分详细说明          
调试事件处理中只要我们处理了就都要返回 1，否则返回 0                    
  
3、在脚本结束时，禁用调试事件，就是调用 API：  
        app.DisableDebug();//之后插件就不会再调用 2 中的处理了，这个是必须要执行的，否则，后面我们手工调试也会调 2 中的处理，会很难受的：）  
          
好了，现在来看具体的脚本是什么样的：  
<html>  
  <head>  
    <title>模拟手动跟踪</title>  
    <script src="res://OllyHTML.dll/const.js"></script>  
  </head>  
  <body>  
    <table style="font-size:14px;width:100%;height:100%;border-collapse:collapse" cellspacing="0" cellpadding="0" align="center">  
      <tr style="height:160px">  
        <td style="color:red" align="center">(执行脚本前请隐藏 OD、清除已经设置的全部断点并重新启动被调试程序 CTRL+F2)  
        </td>  
      </tr>  
      <tr style="height:24px">  
        <td align="center">  
          <input type="button" value="run" onclick="findOEP()">  
        </td>  
      </tr>  
    </table>  
    <script>  
      var codeAddr=0x00401000;  
      var codeSize=0x1000;  
      var dataAddr=0x00402000;  
      var dataSize=0x1000;  
      var bpCount=0;//当前遇到的内存断点次数  
        
      function findOEP()  
      {  
        var tinfo=app.Analyser.GetThreadInfo(app.CpuThreadId);  
        if(!tinfo)  
        {  
          alert("请先打开要调试的程序！");  
          return;  
        }  
        var minfo=app.Analyser.GetModuleInfo(tinfo.Entry);  
        var mbase=minfo.Base;  
        var mi=app.Analyser.GetMemoryInfo(minfo.Base);  
        mi=app.Analyser.GetMemoryInfo(mi.Base+mi.Size);//除 PE 头外第一个段，脚本认定这是目标代码段  
        codeAddr=mi.Base;  
        codeSize=mi.Size;  
        mi=app.Analyser.GetMemoryInfo(mi.Base+mi.Size);//第二个段，脚本认定这是目标数据段  
        dataAddr=mi.Base;  
        dataSize=mi.Size;  
        app.BreakPoint.OnMemoryWrite(dataAddr,dataSize);//设置数据段内存写断点  
        bpCount=0;  
        app.EnableDebug();//允许调试事件  
        app.Execution.ExceptionContinue();//异常继续，加壳程序用这个方法启动调试比较好一些，对停在异常的程序也能正常调试          
      }  
      //这个便是事件处理了  
      function window.OnPaused(reason,extdata,reg,debugEvent)  
      {  
        if(reason==PP\_TERMINATED)//是程序终止吗？  
        {  
          app.BreakPoint.MemoryClear();//清除内存断点  
          app.DisableDebug();//禁止调试事件  
          alert("程序已经结束！");  
          return 1;  
        }  
        if(reason==PP\_MEMBREAK)//是内存断点吗？  
        {  
          if(bpCount<1)//是第一次吧？  
          {  
            bpCount++;                
            app.BreakPoint.MemoryClear();//清除内存断点  
            app.BreakPoint.OnMemoryAccess(codeAddr,codeSize);//设置代码段内存读写断点  
            app.Execution.Run();//继续执行被调试程序  
          }  
          else//已经是第二次了！  
          {                
            app.BreakPoint.MemoryClear();//清除内存断点  
            app.DisableDebug();//禁止调试事件  
            alert("已经到达二次内存断点，OEP:"+window.external.IntToHex(reg.Eip)+"！");              
          }  
        }  
        else  
        {  
          app.Execution.ExceptionContinue();  
        }  
        return 1;  
      }  
    </script>  
  </body>  
</html>          
  
脚本的总体结构仍然是 Hello world 那样的结构，HTML HEAD 部分还是引用一下常量资源文件，UI 部分我们是二行一列的布局，第一行显示提示信息，  
第二行是一个按钮，用于在用户点击时开始用二次内存断点法搜索 OEP。  
  
写教程真的好累啊，在此向长期以来一直为大家写作精彩教程的 CCDebuger 致敬！  
  
  
  
\--------------------------------------------------------------------------------  
  
                                                       2006 年 09 月 06 日 0:38:39 

 4dd466d0e1e0cc123afbe504da2462ca