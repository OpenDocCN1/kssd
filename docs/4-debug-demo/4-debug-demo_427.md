

# 方正 Apabi Reader 阅读器 1.0c

> 原文：[`www.pediy.com/kssd/pediy04/kanxue431.htm`](https://www.pediy.com/kssd/pediy04/kanxue431.htm)

方正 Apabi Reader 多文档格式阅读器拷贝文字限制解除   
程序版本:1.0c 补丁版 build 1037   
软件下载:http://www.apabi.com/download/readerDL.htm   
    Crack by fwnl   
    工具:TRW2000 WINHEX   
    软件说明:一个可以阅读 CEB,PDF,TXT,HTML 的阅读器，最用之处就是看 PDF 文档可用金山词霸抓词   
和可以拷贝加了防拷贝密码保护的 PDF 文档.   
但是每次只能拷 64 个字，真是太少了，恰好我要把一个加了防拷密码的 PDF 文档里的内容拷出来，在网   
上找了好久才发现只有这个东西可以办到，哈哈，所以解其限制就是我下面要做的事了   
  
    每当选取超过 64 个字拷贝时就弹出警告对话框，所以从这个对话框下手.   
用 TRW2000 load 主程序，F5 返回后用 Apabi Reader 随便打开一段超过 64 个字的文字，全部选定文字后,   
Ctrl+N 叫出 TRw2000 后 bpx lockmytask,按 F5 返回, 按右键选拷贝文字,程序中断后 bd \*关闭断点.   
按一下 f11, 然后不断按 f12 直到警告窗口出现.   
点 0k 就到了 Trw2000 的画面，这时看一下程序发现 41b66f 可能是警告对话框 call,于是设断 41b66f 后，   
F5 返回主程序，又选取全文，然后右键点拷贝文字，程序立刻断在 41b66f.   
  
0167:0041B668  PUSH    ECX   
0167:0041B669  MOV      ECX,\[EBP+FFFFFF2C\]        
0167:0041B66F  CALL    0041B8E9                ===========警告对话框 CALL，F8 进入      
0167:0041B674  MOV      DWORD \[EBP\-04\],FFFFFFFF   
0167:0041B67B  LEA      ECX,\[EBP+FFFFFF34\]   
0167:0041B681  CALL    0042AB68   
0167:0041B686  MOV      ECX,\[EBP\-0C\]   
0167:0041B689  MOV      \[FS:00\],ECX   
0167:0041B690  MOV      ESP,EBP   
0167:0041B692  POP      EBP                    ==========点警告对话框上的 OK 后，光标所在地   
0167:0041B693  RET      04   
0167:0041B696  PUSH    EBP   
0167:0041B697  MOV      EBP,ESP   
0167:0041B699  SUB      ESP,BYTE +14   
  
      进入 0041B66F  CALL 后来到   
  
0167:0041B923  MOV      ECX,\[EBP\-04\]   
0167:0041B926  PUSH    ECX   
0167:0041B927  MOV      ECX,\[EBP\-08\]          
0167:0041B92A  CALL    0041B943              ===========警告对话框 CALL，F8 进入   
0167:0041B92F  JMP      SHORT 0041B93D   
0167:0041B931  MOV      EDX,\[EBP\-04\]   
0167:0041B934  PUSH    EDX   
0167:0041B935  MOV      ECX,\[EBP\-08\]   
0167:0041B938  CALL    0041C36D   
  
      进入 0041B92A  CALL  后来到   
  
0167:0041B9E3  MOV      \[EBP\-38\],EAX   
0167:0041B9E6  MOV      EAX,\[EBP\-38\]   
0167:0041B9E9  MOV      EDX,\[EAX\]   
0167:0041B9EB  MOV      ECX,\[EBP\-38\]   
0167:0041B9EE  CALL    NEAR \[EDX+0C\]          ===========警告对话框 CALL，F8 进入   
0167:0041B9F1  MOV      \[EBP\-18\],EAX   
0167:0041B9F4  CMP      DWORD \[EBP\-18\],BYTE +00   
0167:0041B9F8  JNZ      0041BA15   
0167:0041B9FA  MOV      DWORD \[EBP\-24\],01   
0167:0041BA01  MOV      DWORD \[EBP\-04\],FFFFFFFF   
0167:0041BA08  LEA      ECX,\[EBP\-20\]   
0167:0041BA0B  CALL    0041CC60   
  
      进 0041B9EE  CALL  后来到   
      不久发现到了 SmartReader.dll 里   
  
0167:02879745  CALL    0287551B   
0167:0287974A  JMP      SHORT 02879764   
0167:0287974C  MOV      ECX,\[EBP\-0C\]   
0167:0287974F  SUB      ECX,BYTE +18   
0167:02879752  CALL    0287656D              ===========警告对话框 CALL，F8 进入   
0167:02879757  JMP      SHORT 02879764   
0167:02879759  MOV      ECX,\[EBP\-0C\]   
0167:0287975C  SUB      ECX,BYTE +18   
0167:0287975F  CALL    028727FC   
0167:02879764  PUSH    BYTE +00   
0167:02879766  PUSH    BYTE +00   
0167:02879768  LEA      ECX,\[EBP\-08\]   
  
      进 0167:02879752  CALL  后来到   
  
0167:02876625  MOV      ECX,\[028A4AB8\]   
0167:0287662B  CALL    NEAR \[EDX+34\]   
0167:0287662E  CMP      DWORD \[EBP\-30\],80  //发现宝藏了(16 进制的 80 不是 64 吗,原来在这里比字数)    
0167:02876635  JNG      02876682            如里这里跳就解除了限制      
0167:02876637  LEA      ECX,\[EBP\-38\]   
0167:0287663A  CALL    02871A90   
0167:0287663F  MOV      DWORD \[EBP\-04\],00   
0167:02876646  PUSH    DWORD 7531   
0167:0287664B  LEA      ECX,\[EBP\-38\]   
0167:0287664E  CALL    02892B5D   
0167:02876653  PUSH    BYTE +00   
  
code on 发现 0287662e 处代码是 7E4B,要它跳就改成 EB4b，由于这段代码是在 SmartReader.dll 里   
于是用 winhex 在 Apabi Reader 的安装目录下找到 SmartReader.dll 打开,查 FF5234817DD0800000007E4B   
                                                              改 FF5234817DD080000000EB4B   
这样就解除了拷贝文字 64 个的限制,不过拷出来的东西最好复制在写字板，不要用记事本.   
  
不知有没有比这个更好的读 PDF 的东西，如果谁知道请告诉我一声,不要说 Adobe Acrobat 哟，这个   
东太烂了，限制太多，连金山词霸抓词都不行，在精华 3 看到一篇文章说可以改 Adobe Acrobat，可我看   
了 X\*N 次也没看懂.那位知道改的大侠一定要告诉我一下(fwnl@sina.com)，先谢谢了。   
  
      \*\*\*\*\*\*                        \*\*\*                        fwnl        
      \*\*\*                            \*\*                    2002.2.20            
  \*\*\*\*\*\*\*\*  \*\*\*\* \*\* \*\*\*  \*\*\*\*\*\*    \*\*                        长沙            
      \*\*      \*\*\* \* \*\*    \*\*  \*\*    \*\*                                  
    \*\*\*      \*\*\*\*\*\*\*    \*\*\*  \*\*    \*\*\*                                
    \*\*      \*\* \*\*      \*\*  \*\*\*    \*\*\*\*                                  
  \*\*\*                                                                                                                        fwnl 

 4dd466d0e1e0cc123afbe504da2462ca