

# 关于自己写文件补丁和内存补丁 LPK

> 原文：[`www.pediy.com/kssd/pediy12/130697.html`](https://www.pediy.com/kssd/pediy12/130697.html)

这篇文章主要是讲解一下怎么自己编程实现内存补丁 LPK 和文件补丁,算是总结吧!  
  
软件的下载地址: http://www.vdisk.cn/liuqiangni  
  
   无意中在电脑上面发现这样个软件(屏幕画笔),都不知道是什么时候下载的,运行后,居然还要  
  
注册? 程序加了 ASPack 2.12 -> Alexey Solodovnikov 的壳,我也没脱,带着壳跑了下,下了个  
  
MessageBox 断点.断下后,搜索了字符串,程序很简单,有字符串提示,于是很快找到突破口,大致看  
  
了一下,改一个字节就可以了.  
  
原来的:  
004CBA00  |. /75 07         JNZ SHORT Epen\_exe.004CBA09  
004CBA02  |. |C605 B4214D00>MOV BYTE PTR DS:\[4D21B4\],1  
004CBA09  |> \\8BC3          MOV EAX,EBX  
  
改后的  
004CBA00     /75 00         JNZ SHORT Epen\_exe.004CBA02  
004CBA02  |. \\C605 B4214D00>MOV BYTE PTR DS:\[4D21B4\],1  
004CBA09  |>  8BC3          MOV EAX,EBX  
  
  
我这里不是分析算法的,算法我没看,但是跟出了注册码,是明码比较的.我这里是演示一下用  
  
LPK.dll 做内存补丁 和 在壳段改程序做文件补丁,自己编程实现.  
  
文章很简单,是给新手的,高手就不要笑话了...  
  
1.  
LPk.dll :  我就用看雪老大的模板了,就只是改了一点点,源代码后面会发上来,上面注释也很全  
  
//这个是补丁函数,把补丁代码写在里面就可以了  
void PatchProcess(HANDLE hProcess)  
{  
  DWORD Oldpp;  
    
  /\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  
  /\*补丁代码样例                                                                 
  
                                                                   \*/  
  /\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  
  unsigned char p4CBA00\[2\] = {    
    0x75, 0x00  
  };  
  VirtualProtectEx(hProcess, (LPVOID)0x4CBA00, 2, PAGE\_EXECUTE\_READWRITE,   
  
&Oldpp);  
  WriteProcessMemory(hProcess, (LPVOID)0x4CBA00, p4CBA00, 2, NULL);  
}  
  
  
2.  
文件补丁:  因为程序加壳了,我们知道程序代码是动态解压出来的,我们直接改是无法保存的,但  
  
是我们可以在壳段加入代码,间接的来修改我们程序中的代码.  
  
下面开始,我们用 Esp 定律可以来到程序的入口点  
    
0055E3B0   /75 08           JNZ SHORT Epen.0055E3BA  ;这里会跳过去  
0055E3B2   |B8 01000000     MOV EAX,1  
0055E3B7   |C2 0C00         RETN 0C  
0055E3BA   \\68 30C54C00     PUSH Epen.004CC530  ;将程序的入口点压栈  
0055E3BF    C3              RETN    ;这里是跳到入口点    
  
  
我们将上面几句 NOP 掉,写上我们自己的代码,也就是修改 004CBA00 处的 2 个自己为 7500,  
   
我们这样来改  
  
0055E3B0    C605 01BA4C00 00  MOV BYTE PTR DS:\[4CBA01\],0  
0055E3B7    90                NOP  
0055E3B8    90                NOP  
0055E3B9    90                NOP  
0055E3BA    68 30C54C00       PUSH Epen.004CC530  
0055E3BF    C3                RETN  
  
  
保存就可以实现注册...  
  
注意:注册码存放在\[HKEY\_CURRENT\_USER\\Software\\EPen\]下  
  
自己用代码写的补丁有好处:  1.不会被杀软盯上  2.生成的文件很小  
所以我很赞成自己来写补丁.  
  
下面我们编程来实现:(MFC)    
//我使用的是 CFile 类来读写文件的  
void patch()  
{  //这里是我们要修改的地址    
  char reg\[\] = {0xC6,0x05,0x01,0xBA,0x4C,0x00,0x00,0x90,0x90,0x90};  
  CFile myfile;    
  CopyFile("Epen.exe","Epen.exe.bak",TRUE);//备份文件  
  myfile.Open("Epen.exe",CFile::modeReadWrite|CFile::typeBinary,NULL);  
  myfile.Seek(0x0006E7B0,CFile::begin);//移动文件指针  
  myfile.Write(reg,sizeof(reg));  //写文件  
}  
  
  
这样,文件补丁就写好了,当然也可以用 APi 来写(CreateFile,SetFilePointer,WriteFile)  
来写.  
  
给一组正确的注册号码:  
  
LiuQiang\_China  
0BA1AB67A033304A8EEAD62086B971B9  
  
好了,就写到这了,谢谢观看,高手飘过...

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 文件补丁源码.rar  
 |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || lpk.rar  
 |