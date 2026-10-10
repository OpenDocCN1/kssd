

# Detect all versions of Themida/WinLicense(更新……)

> 原文：[`www.pediy.com/kssd/pediy10/79412.html`](https://www.pediy.com/kssd/pediy10/79412.html)

大家好！  
    Merry Christmas！  
    生蛋节了，没什么拿得出手能送大家做礼物的   
    正好看到 What 的文章 Themida 1.9.1.0 - 2.0.5.0 (Finding Exact Version)，粗略看了一下，简单易学，不过每次都得重复操作一遍，不太方便，于是就想拿出来写个脚本，算是陪大家一起学习脚本写作吧  
  
    运行脚本，可以自动检测 TMD 加壳程序的确切版本号，便于你采取下一步行动！  
    没什么技术含量，高手掠过  
      
    感谢 goldsun 和 peaceclub 修正脚本；  
    感谢 stupidass 对特征码值的修正，同时对 KooJiSung 所做的测试工作给予充分肯定!  

代码:

```
/\*
FileName    :  Detect all versions of Themida/WinLicense        
Features    :  If your target is packed with Themida/WinLicense,this script can help you detect its version. 
               But you must note that this isn't a unpack-script for Themida.
Environment :  WinXP,ODBYdyk V1.10,OllyScript V1.65            
Support     :  Themida all versions (1.9.8.0-2.0.5.0)
Thanks      :  What/goldsun/peaceclub/stupidass/KooJiSung            
Author      :  Playboysen                                      
Date        :  2008-12-25  o\_0  Merry Christmas!  
\*/

var temp
var verStr
var verAddr
mov verAddr,0

bc                                  //先清除一下断点
gpa "ZwContinue", "ntdll.dll"       //bp ZwContinue
bp $RESULT
loop:
esto
cmp \[esp+0C\],0C0000096             //关键处的值应该为 C0000096   
jnz loop                           //循环比较关键值
bc
mov eax,\[esp+4\]
add eax,0B8
mov temp,\[eax\]
find temp,#000004000000#          //特征码
cmp $RESULT,0
jz exit
mov eax,$RESULT
add eax,6 
mov verStr,"Themida/winlicense version: "
mov verAddr,eax
READSTR \[verAddr\],5
add verStr,$RESULT
msg verStr

exit:
ret

```