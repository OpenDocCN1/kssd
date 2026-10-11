
**注意**：使用高级指令 `Call` 获取 `IAT` 的话，一定要段上 `Create New IAT`

### 六、总结

1.  最好是选择程序里面的空白地址来放 `IAT` 表，但是，空间得足够大
2.  在设置搜索起始和结束地址的时候，如果你可以确定具体返回的话，尽量设置小一些的范围，这样可以提高处理速度，一般可以设置为 `Code` 段的开始地址 `[CodeBase]` 和 `Code` 段的结束地址 `[CodeBase+CodeSize]`
3.  本文介绍的是如何还原 `Jmp dword ptr ds:[API]`。`Call dword ptr ds:[API]` 类型的处理需要变换下代码。将在下文中介绍
4.  经测试，脱壳文件基本没有跨平台问题

### 七、脚本的使用

需要注意已经打了注释的个地方。

忽略除了特权指令和指定异常外的所有异常

```
/*
Script written by wynney

Date:   2007-04-20
Script: SoftWrap Fixer for VB
Environment : OllyDbg 1.1, ODBGScript 1.52,Winxp Sp2
Debugging options: Ignore all outside of "Invalid or privileged" and "Custom" exception
Thanks :
           kanxue      - author of HideOD        
           hnhuqiong   - author of ODbgScript 1.52
*/

var ImagBase
var ImagSize
var CBase
var CSize
var SrStart
var SrEnd
var IATNew
var temp
var Pointer
var DllBase
var OEPS
var fix

GetBase:
         dbh
         BPHWCALL
         GMI eip, ModuleBase
         cmp $RESULT,0
         je error
         mov ImagBase,$RESULT
         GMI eip,ModuleSize
         cmp $RESULT,0
         je error
         mov ImagSize,$RESULT
         GMI eip,CodeBase     
         cmp $RESULT,0
         je error
         mov CBase,$RESULT
         GMI eip, CodeSize
         cmp $RESULT,0
         je error
         mov CSize,$RESULT
         
WorkFor:
         mov SrStart,CBase
         mov temp,CBase
         add temp,CSize
         mov SrEnd,temp
         Alloc 10000
         mov DllBase,$RESULT
         mov temp,$RESULT
         add temp,4
         mov Pointer,temp
         add temp,4
         mov IATNEW,temp
      

GetTable:
         GPA "VirtualAlloc","kernel32.Dll"
         cmp $RESULT,0
         je error
         find $RESULT,#C21000#
         cmp $RESULT,0
         je error
         mov temp,$RESULT
         bp temp
         esto
         bc temp
         sto
         find eip,#85C00F84????????F6C3027421FFB5????????6A1050FFB5????????FFB5????????#
         cmp $RESULT,0
         je error
         add $RESULT,8
         mov temp,$RESULT
         BP temp,"x"
         esto
         BC temp
         mov [DllBase],ecx
         mov [Pointer],IATNEW  
         mov temp,eip
         mov [temp],#3A0D90909090#
         add temp,2
         mov [temp],DllBase
         add temp,4
         mov [temp],#740E#
         add temp,2
         mov [temp],#FE0590909090#
         add temp,2
         mov [temp],DllBase
         add temp,4
         mov [temp],#6683059090909004#  
         add temp,3
         mov [temp],Pointer
         add temp,5
         mov [temp],#8B1D90909090#
         add temp,2
         mov [temp],Pointer
         add temp,4
         mov [temp],#89036683059090909004#
         add temp,5
         mov [temp],Pointer          

GoToOEP:
         esto
         esto
         esto
         esto
         esto
         esto
         BPRM CBase,CSize
         esto
         esto
         BMC

ASKSt:
         MSGYN "是否到达 OEP？"  
         cmp $RESULT,0
         je Manuly
         jmp JmpAPI    
    
Manuly:
         msg "请手动到达 OEP，再继续！"         //若没到达 OEP，手动到达再点插件的"继续"
         Pause
         jmp JmpAPI

JmpAPI:   
 mov OEPS,eip
 PAUSE
 ask "请输入 Jmp 类型-Call Dword PTR DS:[Addr]中的 Addr"  //输入要修改的地址
 cmp $RESULT,0
 je error
 mov fix,$RESULT
 sti
 sti
 sti
 find eip,#68????????C3#
 cmp $RESULT,0
 je error
 mov temp,$RESULT
 bp temp
 esto
 bc temp
 sti
 sti
         find eip,#8B049635????????508A00040F#
         cmp $RESULT,0
         je error
         bp $RESULT
         esto
         bc $RESULT
         add $RESULT,8
         mov temp,$RESULT
         mov [Pointer],CBase
         mov [temp],#B990909090#
         add temp,1
         mov [temp],IATNEW
         add temp,4
         mov [temp],#3901740583C104EBF78BD181C200100000890283C41C3E8B042483E80666C700FF2583C00289109090#
         add temp,29
         mov [temp],#813D9090909090909090#
         add temp,2
         mov [temp],Pointer
         add temp,4
         mov [temp],SrEnd
         add temp,4
         mov [temp],#742C8B0D90909090#
         add temp,4
         mov [temp],Pointer
         add temp,4
         mov [temp],#668139FF1575178BC183C002813890909090#
         add temp,0E
         mov [temp],fix
         add temp,4
         mov [temp],#750A83C004A390909090FFE1FF0590909090EBC8EBFE90909090#
         add temp,6
         mov [temp],Pointer
         add temp,8
         mov [temp],Pointer
         add temp,6
         bp temp
         esto
         bc temp
         mov eip,OEPS

Done:
         msg "请使用 ImportREC 高级指令修复脱壳文件"
         ret          
  
error:
         msg "非支持版本"
         ret
```

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 04 月 23 日
