

# EncryptPE V2.2007.4.11 非 S 方式 跳过注册的脚本

> 原文：[`www.pediy.com/kssd/pediy10/58817.html`](https://www.pediy.com/kssd/pediy10/58817.html)

///////////////////////////////////////////////////////////////////////////////////  
// FileName    :  skip\_reg2\_1.txt  
// Comment     :  EncryptPE V2.2007.4.11  非 S 方式 ---- skip\_reg  
// Environment :  WinXP SP2,OllyDbg V1.10, OllyScript   
// Author      :  d\_h  
// Date        :  2008-1-23  
// WebSite     :  http://www.pediy.com  
///////////////////////////////////////////////////////////////////////////////////  
  
var WPM  
var data1  
var data2  
var data3  
var data4  
  
// 先用 OD 调试 Explorer.exe，然后 LifeOD 注入 Explorer.exe，使用 skip\_reg2\_2 跳过 REG  
  
start1:  
    cmp $VERSION, "1.48"   
    jb version  
    gpa "WriteProcessMemory","kernel32.dll"  
    mov WPM, $RESULT  
    add WPM , 2  
//    bphws WPM ,"x"  //    奇怪，这里用硬件断点行不通  
  
    bpcnd WPM , "\[ESP\]>=7121000"  
  
next:  
    esto  
    cmp eip , WPM  
    jne next   
    mov data1,\[esp\]  
    cmp data1,71121000  
    jb next  
    cmp data1,7125F000  
    ja next  
    mov data2,\[esp+8\]  
    cmp data2,71121000  
    jb next  
    cmp data3,7125F000  
    ja next  
    mov data3,\[esp+0C\]  
    cmp data2,data3  
    jne next  
    mov data4,\[esp+10\]  
    cmp data4,0  
    je next  
    mov \[esp+10\],0  
      
    log data1  
    log data2  
    jmp next  
      
  
version:  
    msg "插件版本过低"    
    ret    
  
///////////////////////////////////////////////////////////////////////////////////  
// FileName    :  skip\_reg2\_2.txt  
// Comment     :  EncryptPE V2.2007.4.11 非 S 方式 ---- skip\_reg   
// Environment :  WinXP SP2,LifeDbg V1.4, OllyScript 1.65.2   
// Author      :  d\_h  
// Date        :  2008-1-23  
// WebSite     :  http://www.pediy.com  
///////////////////////////////////////////////////////////////////////////////////  
var patch1  
var patch2  
var patch3  
  
var p11  
var p12  
var p13  
var p14  
var p21  
var p22  
  
// 注意：须设置忽略 0EEDFADE 异常  
// 先用 OD 调试 Explorer.exe，需配合使用 skip\_reg2\_1 脚本，防止 Patch 被覆盖  
  
Start:  
    cmp $VERSION, "1.48"   
    jb version  
    gpa "IsDebuggerPresent","kernel32.dll"  
    bp $RESULT  
    esto  
    bc $RESULT  
  
next:      
    mov  patch1 ,7120350A  
    mov \[patch1\],#7400#   //#7407#  
    mov  patch2 ,71201642  
    mov \[patch2\],#EB5E#   //#745E#  
    mov  patch3, 711F9561  
    mov \[patch3\],#7400#   //#7406#  
  
    mov  p11, 711FF5F5      
    mov \[p11\],#EB7E#     //#757E#  
    mov  p12, 711F41F4  
    mov \[p12\],#750E#     //#7507#  
    mov  p13, 71201F8E  
    mov \[p13\],#7400#     //#740F#  
  
    mov  p21, 71201F9D  
    mov \[p21\],#EB0B#     //#750B#  
  
  
    esto  
    jmp next   
  
  
version:  
    msg "插件版本过低"    
    ret

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 1.rar \[解压密码：PEDIY\]   
 |