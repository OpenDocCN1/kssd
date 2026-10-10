

# Stolen Code Found script-->PESpin v0.3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6471.htm`](https://www.pediy.com/kssd/pediy06/pediy6471.htm)

昨天上这里看到了最新版的 peSpin v0.3，今天上班时玩了一下，所以出来这么个东东，不知道是这样好还是写脱文好(应该不要再交作业了吧).   
声明：由于近来已经有几位朋友说我的脚本没有用，所以我在这里申明，如果可以的话，最好看看你的脚本插件的版本（已经有几位朋友由于是低版本的，所以当然出错了），看看我的脚本里写的测试环境，如果还是出错再和我联系 OK??   
我自己写一个脚本一般经过 VC、VB、delphi 三种语言的测试再发布,没有通过的或专用的，我会在脚本里说明的.至于有朋友说看不懂，说我能不能在脚本的每一处写上注释，第一个我不想写因为个人认为没什么必要，只要懂得用法的话你就知道看它比看一个程序是多么的容易,如果用法都不知道的可以先去工具下载里下我汉化的脚本插件和中文 readme 或直接去作者那里下新版的 E 文说明.   
第二没必要，第三呢也是最主要的懒。欢迎大家和我交流！   
  
/\*   
//////////////////////////////////////////////////   
  PESpin v0.3 Stolen Code Finder v0.1   
  ( for 'Remove OEP' mode)   
  Author:  loveboom   
  Email : bmd2chen@tom.com   
  OS    : Win2kADV sp2,OllyDbg 1.1b,OllyScript v0.62   
  Date  : 2004-3-28   
  Config: Ignore other exceptions except 'Invalid or privileged instruction'   
  Note  : If you have one or more question, email me please,thank you!   
//////////////////////////////////////////////////   
\*/   
var bpaddr    //Break point address   
  
start:    //script start   
run   
  
lbl1:   
esto   
esto   
gpa "LoadLibraryA","kernel32.dll"  //GetProcAddress   
mov bpaddr,$RESULT   
bp bpaddr   
eob lbl2   
esto   
  
lbl2:   
bc bpaddr   
eob lbl3   
rtu   
  
lbl3:   
mov bpaddr,esp   
add bpaddr,4   
bphws bpaddr,"r"   
eob lbl4   
run   
  
lbl4:   
bphwc bpaddr   
  
end:   
cmt eip,"Stolen Code found,here start Stolen program's OEP Code.please patch OEP code and then dumped it!"   
msg "Script by loveboom\[DFCG\],Thank you for using my Script!"   
ret  
    
 4dd466d0e1e0cc123afbe504da2462ca