

# arm3.75 以后的从双进程到单进程转换的脚本 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6699.htm`](https://www.pediy.com/kssd/pediy06/pediy6699.htm)

  
/\*  
arm3.75 版以后的从双进程到单进程转换的 Script 自动运行脚本  
by fxyang  
2004.中秋节  
\*/  
  
dbh  
  
var address  
  
gpa "OpenMutexA","kernel32.dll"  
bp $RESULT  
run  
  
eoe code\_1  
code\_1:  
mov address,eip  //获取第一次 PREFIX LOCK:异常地址//  
esto  
  
lbl3:  
cmp eip,address   //自动越过异常//  
ja begin  
esto  
  
begin:  
  exec  
    PUSHAD  
    PUSH EDX  
    push 0  
    push 0  
    CALL kernel32.CreateMutexA  
    POPAD  
    jmp kernel32.OpenMutexA  
  ende  
//上面的代码就是在 Script 中运行从双进程到单进程的转换//  
  
   bc $RESULT  
  
lbl4:  
  gpa "VirtualProtect","kernel32.dll"  
  bp  $RESULT  
    
lbl5:  
  msg "完成后，Shift+F9 会中断在函数 kernel32.VirtualProtect 中，再见！"  
  ret   

 4dd466d0e1e0cc123afbe504da2462ca