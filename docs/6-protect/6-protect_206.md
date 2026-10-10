

# 加密狗保护的 VFP 软件破解

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-462.htm`](https://www.pediy.com/kssd/pediy08/pediy8-462.htm)

【文章标题】: 加密狗保护的 VFP 软件破解  
【文章作者】: Rufus Xu  
【作者邮箱】: my\_home\_page@163.com  
【软件名称】: 工程造价 V93  
【软件大小】: 43.3M  
【下载地址】: http://www.itongsoft.com  
【加壳方式】: 狗壳  
【保护方式】: 加密狗  
【编写语言】: VFP  
【使用工具】: OLLDBG  
【操作平台】: Windows2000  
【软件介绍】: 建筑工程造价软件  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
      应一个朋友的请求，破解一个加密狗保护的工程造价软件。以前一直没有搞过加密狗，这次想试试手，  
  经过努力，终于搞定，希望对被加密狗困扰的朋友有所帮助。  
      为了搞定这条狗我花了几乎两个月的业余时间，从开始的一窍不通，到最后完美破解，走了很多弯路。这次学习的过程  
  主要分为三个阶段，下面详细介绍。  
      一、外壳调试  
          在程序目录下有 Vfp6r.dll、Vfp6rchs.dll、Vfp6renu.dll 三个文件，不用说，该程序是用 VFP 开发的，  
  PEID 查不到信息，深度搜索发现是用 VC++写的。拿出 OD，打开要调试的程序 ztc.exe,OD 停在入口点：  
    
  00407049 ztc.>  55            push ebp  
  0040704A        8BEC          mov ebp,esp  
  0040704C        83EC 7C       sub esp,7C  
  0040704F        53            push ebx  
  00407050        56            push esi  
  00407051        57            push edi  
  00407052        C745 C8 A5A50>mov dword ptr ss:\[ebp-38\],0A5A5  
  00407059        66:C745 C0 0A>mov word ptr ss:\[ebp-40\],0A  
  0040705F        E9 550D0000   jmp ztc.00407DB9  
  00407064        E9 4C070000   jmp ztc.004077B5  
  00407069        EB 01         jmp short ztc.0040706C  
    
  往下走两步，就是一个跳转，跟着走，来到：00407DB9 处，是一大片跳转表。  
    
  00407CE1        8B45 E0       mov eax,dword ptr ss:\[ebp-20\]  
  00407CE4        5F            pop edi  
  00407CE5        5E            pop esi  
  00407CE6        5B            pop ebx  
  00407CE7        8BE5          mov esp,ebp  
  00407CE9        5D            pop ebp  
  00407CEA        FF20          jmp dword ptr ds:\[eax\]  
  00407CEC        E9 CD000000   jmp ztc.00407DBE  
  00407CF1      ^ E9 E6FFFFFF   jmp ztc.00407CDC  
  00407CF6      ^ E9 E1FFFFFF   jmp ztc.00407CDC  
  00407CFB      ^ E9 9EF3FFFF   jmp ztc.0040709E  
  00407D00      ^ E9 D7FFFFFF   jmp ztc.00407CDC  
  00407D05      ^ E9 A5FFFFFF   jmp ztc.00407CAF  
  00407D0A      ^ E9 14FEFFFF   jmp ztc.00407B23  
  00407D0F      ^ E9 0FFEFFFF   jmp ztc.00407B23  
  00407D14      ^ E9 C3FFFFFF   jmp ztc.00407CDC  
  00407D19      ^ E9 05FEFFFF   jmp ztc.00407B23  
  00407D1E      ^ E9 B9FFFFFF   jmp ztc.00407CDC  
  00407D23      ^ E9 FBFDFFFF   jmp ztc.00407B23  
  00407D28      ^ E9 F6FDFFFF   jmp ztc.00407B23  
  00407D2D      ^ E9 F1FDFFFF   jmp ztc.00407B23  
  00407D32      ^ E9 DAFCFFFF   jmp ztc.00407A11  
  00407D37      ^ E9 D8FDFFFF   jmp ztc.00407B14  
  00407D3C      ^ E9 C4FDFFFF   jmp ztc.00407B05  
  00407D41      ^ E9 DDFDFFFF   jmp ztc.00407B23  
  00407D46      ^ E9 E7FCFFFF   jmp ztc.00407A32  
  00407D4B      ^ E9 4EF3FFFF   jmp ztc.0040709E  
  00407D50      ^ E9 CEFDFFFF   jmp ztc.00407B23  
  00407D55      ^ E9 C9FDFFFF   jmp ztc.00407B23  
  00407D5A      ^ E9 3FF3FFFF   jmp ztc.0040709E  
  00407D5F      ^ E9 97FDFFFF   jmp ztc.00407AFB  
  00407D64      ^ E9 29FAFFFF   jmp ztc.00407792  
  00407D69      ^ E9 23FFFFFF   jmp ztc.00407C91  
  00407D6E      ^ E9 79FDFFFF   jmp ztc.00407AEC  
  00407D73      ^ E9 46FFFFFF   jmp ztc.00407CBE  
  00407D78      ^ E9 88FBFFFF   jmp ztc.00407905  
  00407D7D      ^ E9 1EFFFFFF   jmp ztc.00407CA0  
  00407D82      ^ E9 55FFFFFF   jmp ztc.00407CDC  
  00407D87      ^ E9 42F6FFFF   jmp ztc.004073CE  
  00407D8C      ^ E9 88FBFFFF   jmp ztc.00407919  
  00407D91      ^ E9 8DFDFFFF   jmp ztc.00407B23  
  00407D96      ^ E9 41FFFFFF   jmp ztc.00407CDC  
  00407D9B      ^ E9 3CFFFFFF   jmp ztc.00407CDC  
  00407DA0      ^ E9 26F4FFFF   jmp ztc.004071CB  
  00407DA5      ^ E9 32FFFFFF   jmp ztc.00407CDC  
  00407DAA      ^ E9 2DFFFFFF   jmp ztc.00407CDC  
  00407DAF      ^ E9 EAF2FFFF   jmp ztc.0040709E  
  00407DB4      ^ E9 6AFDFFFF   jmp ztc.00407B23  
  00407DB9      ^ E9 F7F9FFFF   jmp ztc.004077B5  
    
  在最上面的一条跳转语句 00407CEA 处下断，F9 走，OD 停下来以后，发现 DS：\[EAX\]的值变成了 00405634，  
  此时 EAX=0040E0E0，表示下一步就要跳到该地址了，继续：  
    
  00405634     55            push ebp  
  ...................................  
  00405654     FF15 0CD14100 call dword ptr ds:\[<&USER32.Messag>; USER32.MessageBoxA  
    
  F9,跳出对话框，提示没有加密狗，程序退出。看来，程序已经加了狗壳，在最开始就检测有没有加密狗。  
    
  根据网上的一些教程，我们要在读狗的时候下断，读狗肯定要用到 CreateFileA 函数，在 OD 的命令插件中输入：  
  bpx CreateFileA，让 OD 重新调入程序，F9，程序中断在：  
    
  0040A4CF     55            push ebp  
  0040A4D0     8BEC          mov ebp,esp  
  0040A4D2     51            push ecx  
  0040A4D3     53            push ebx  
  0040A4D4     56            push esi  
  0040A4D5     57            push edi  
  0040A4D6     8B7D 08       mov edi,dword ptr ss:\[ebp+8\]  
  0040A4D9     8B9F 88000000 mov ebx,dword ptr ds:\[edi+88\]  
  0040A4DF     83C3 18       add ebx,18  
  0040A4E2     33F6          xor esi,esi  
  0040A4E4     6A 00         push 0  
  0040A4E6     68 80000000   push 80  
  0040A4EB     6A 03         push 3  
  0040A4ED     6A 00         push 0  
  0040A4EF     6A 07         push 7  
  0040A4F1     68 000000C0   push C0000000  
  0040A4F6     68 8CC44100   push zj9.0041C48C                  ; ASCII "\\\\.\\LPTDI1"  
  0040A4FB     FF15 B0D04100 call dword ptr ds:\[<&KERNEL32.Crea>; kernel32.CreateFileA  
    
  CTRL+F9 返回，几次返回以后，来到 0040943C，可以看到一些花指令，差不多到了源头了，  
    
  0040943C     55            push ebp  
  0040943D     8BEC          mov ebp,esp  
  0040943F     81EC 78010000 sub esp,178  
  00409445     53            push ebx  
  00409446     33C9          xor ecx,ecx  
    
  将 0040943C 处的代码改成  
    
  0040943C    33C0            XOR EAX,EAX  
  00409E62    C3              RETN   
    
  将改动保存到到文件。让 OD 重新加载程序，清除所有断点，F9 运行。还是提示没有找到加密狗。  
    
  因为 00407CEA 是关键跳转处，我们来看看程序是什么时候为 EAX 赋值的，mov eax,dword ptr ss:\[ebp-20\]  
  在 OD 的命令处输入 D 0040E0E0，下硬件写入断点，重新运行，在 00407A8B 处断下  
    
  00407A7A     E8 9D030000   call zj9.00407E1C  
  00407A7F     3B05 04E14000 cmp eax,dword ptr ds:\[40E104\]  
  00407A85     0F84 0A000000 je zj9.00407A95  
  00407A8B     C705 E0E04000>mov dword ptr ds:\[40E0E0\],zj9.00405634<-------------------MessageBox 入口  
    
  00407A85 处是关键跳转，NOP 掉,保存，重新运行，在 00407CEA（VFP 入口点），下断，F9 运行。OD 断下以后：  
    
  00407CEA      - FF20          jmp dword ptr ds:\[eax\]             ; ztc.00401760  
    
  已经不是 00405634 了，开始的壳检测狗已经跳过了。熟悉 VFP 程序的人都知道 00401760 是特有的 VFP 程序入口点。  
    
  F9 运行，程序已经不提示无狗了，但是打开主程序以后，还是提示没有注册码。看来，在 VFP 主文件里面还有对狗的调用。  
    
      二、VFP 程序处理  
    
      在讲这部分的时候，先要补一课，就是 VFP 程序的结构和执行次序：  
    
      VFP 编译生成的 EXE 文件主要由两部份组成,第一部份(前半部分)是一个标准的 PE 格式的可执行文件,  
  这个标准的 PE 格式文件又叫载入器. 第二部分(后半部分)是一个标准的 .app 文件, 这个 app 与在 VFP   
  集成开发环境中编译时选择编译为 app 文件时生成的 app 文件基本上是一样的,只是在这个 app 文件的尾部,  
  多了 14 字节的标识部份(这 14 字节的前 10 字节是 00 83 41 00 00 00 00 00 00 00,最后四字节是 app 文件  
  加上 14 字节的标识后的长度). 因此,可以认为 VFP 编译的 EXE 文件是由一个标准的 EXE 文件后面附上一个 .app 文件,  
  再加上 14 字节的标识部份组成. 而且, 前半部份的 EXE 与后半部分的 app 之间没有任何联系.  
      前面提到的载入器的作用非常简单,主要是检查运行环境,载入相应的 VFPxR.DLL 并调用 VFPxR.DLL   
  中的 DllWinMain 函数. 在调用 DllWinMain 函数时, 有两个参数, 第一个是一个串参数,必须是 VFP 编译的 EXE   
  文件的文件名, 第二个是一个 int 型的值,一般用十六进制的 FF 就可以. 因此,你可以用除 VFP 以外的任何编译语  
  言来编译写自己的载入器以达到加密程序的作用.  
      事实上,一但载入器调用了 VFPxR.DLL 中的  DllWinMain 函数后, VFP 就接管了程序的控制权,你就再也无法  
  用常规的方法控制程序的运行过程了. 下面说一下 VFP 的 DLL 的启动过程:首先,VFPxR.DLL 会读取文件的最后 14 字节.   
  并根据最后四字节来定位到 app 在 EXE 文件中的位置。  
    
     好了，课补完了，即然 VFPxR.DLL 要读文件,那么我们就在读文件的时候下断,在 OD 的命令处输入:bpx ReadFile,  
  F9 运行,在 00405F3B 处断下,  
    
  00405F3B        FF15 94D04100 call dword ptr ds:\[<&KERNEL32.Read>; kernel32.ReadFile  
    
  此时堆栈的数据为:  
    
  0012FA18    00000030  |hFile = 00000030  
  0012FA1C    0012FA40  |Buffer = 0012FA40  
  0012FA20    00000040  |BytesToRead = 40 (64.)  
  0012FA24    0012FA38  |pBytesRead = 0012FA38  
  0012FA28    00000000  \\pOverlapped = NULL  
  0012FA2C    004031C0  ztc.004031C0  
    
  我们看到 BytesToRead 大小为 40H,不是 E(14 字节),F9 继续,三次断下以后,  
    
  004061FF        FF15 94D04100 call dword ptr ds:\[<&KERNEL32.Read>; kernel32.ReadFile  
    
  此时堆栈的数据为:  
    
  0012F9E8    00000030  |hFile = 00000030  
  0012F9EC    0012FAE4  |Buffer = 0012FAE4  
  0012F9F0    0000000E  |BytesToRead = E (14.)  
  0012F9F4    0012FABC  |pBytesRead = 0012FABC  
  0012F9F8    00000000  \\pOverlapped = NULL  
    
  这就是我们要找的地方了,读出的数据保存在 0012FAE4,F8 走一步,在 OD 命令行输入 D 0012FAE4  
    
  0012FAE4  29 B3 CB 29 29 29 29 29  )乘)))))  
  0012FAEC  29 29 51 0A F3 29        ))Q.?.  
    
  但是这并不是我们要的数据,00 83 41 00 00 00 00 00 00 00,看来已经加密过了,往下看,  
    
  00406298        FF4D F4       dec dword ptr ss:\[ebp-C\]  
  0040629B        837D F4 00    cmp dword ptr ss:\[ebp-C\],0  
  0040629F        0F8C 1F000000 jl ztc.004062C4  
  004062A5        8B45 F4       mov eax,dword ptr ss:\[ebp-C\]  
  004062A8        8B4D AC       mov ecx,dword ptr ss:\[ebp-54\]  
  004062AB        33D2          xor edx,edx  
  004062AD        8A1408        mov dl,byte ptr ds:\[eax+ecx\]  
  004062B0        8A82 08E34000 mov al,byte ptr ds:\[edx+40E308\]  
  004062B6        8B4D F4       mov ecx,dword ptr ss:\[ebp-C\]  
  004062B9        8B55 AC       mov edx,dword ptr ss:\[ebp-54\]  
  004062BC        880411        mov byte ptr ds:\[ecx+edx\],al  
    
  这就是解密过程了,从 DS:\[40E308\]取出解密的数据,我们看看解密以后的数据:  
    
  0012FAE4  00 83 41 00 00 00 00 00  .傾.....  
  0012FAEC  00 00 EC 06 31 00        ..?1..  
    
  这就是 VFP 的尾部数据,它指出了 VFP 的 APP 文件的大小是 003106EC-E,  
  我们用 UE 打开主程序文件,找到 29 B3 CB 29 29 29 29 29 所在的位置,然后计算出  
  APP 的头部位置,然后就把从头部开始的大小为 003106EC-E 数据取出另存.  
    
  然后我们在看看明文存放的位置,在 OD 的命令行输入 D 0040E038,可以看到如下数据:  
    
  0040E308  44 5D 85 5B 15 07 EF A2  D\]匸铫  
  0040E310  5A DD 06 22 A3 3F 81 4E  Z?"?丯  
  0040E318  C0 D6 73 A4 C1 F5 E9 96  乐 sち蹰  
    
  我们用 UE 打开主程序搜索 44 5D 85 5B 15 07 EF A2,但是找不到,看来这部分数据是  
  动态生成的,不要紧,我们把现在内存中的数据导出来就 OK 了,有 OD 自带的插件 DUMP 出数据  
  另存为 DUMP.EXE,然后用 OD 打开,再搜索 44 5D 85 5B 15 07 EF A2,已经找到了.根据解  
  密算法,我们把从 44 5D 85 5B 15 07 EF A2 开始的数据取出 512 字节出来,因为 512 后面的字  
  节用不到.  
    
  然后写程序把数据解密,我是用 VB.NET 写的,  
              Dim mFile As File  
              If File.Exists(Application.StartupPath & "\\file1.dat") And File.Exists(Application.StartupPath & "\\file2.dat") Then  
                  Dim mRFileStream1 As FileStream = mFile.OpenRead(Application.StartupPath & "\\file1.dat") '密文  
                  Dim mRFileStream2 As FileStream = mFile.OpenRead(Application.StartupPath & "\\file2.dat") '明文  
                  Dim mWFileStream As FileStream = New FileStream(Application.StartupPath & "\\ztc93.app", FileMode.OpenOrCreate)  
                  Dim i As Integer  
                  For i = 0 To &H3106DD  
                      Dim mArray1(0) As Byte  
                      Dim mArray2(0) As Byte  
                      '读密文  
                      mRFileStream1.Seek(i, SeekOrigin.Begin)  
                      mRFileStream1.Read(mArray1, 0, 1)  
                      '找对应明文  
                      mRFileStream2.Seek(mArray1(0), SeekOrigin.Begin)  
                      mRFileStream2.Read(mArray2, 0, 1)  
                      '写入新文件  
                      mWFileStream.Seek(i, SeekOrigin.Begin)  
                      mWFileStream.Write(mArray2, 0, 1)  
                  Next  
    
                  mRFileStream1.Close()  
                  mRFileStream2.Close()  
                  mWFileStream.Close()  
              End If  
  得到的文件便是 VFP 的 APP 文件了,我们拿出 FOXTOOL,打开,呵呵,开心啊,源代码出来了.因为我不会 VFP,所以反编译出  
  来的代码我没有办法再重新编译,我试了很多次,编译不通过.所以得另想办法了.  
    
      三、模拟写狗  
        
    
      用 FOXTOOL 打开,可以看到 APP 文件中果然还有读狗的地方,  
    
  zy4000.dll 是读狗的 DLL  
    
  DECLARE INTEGER DogRead IN zy4000 INTEGER DogBytes, INTEGER DogAddr,  STRING @Data  
  DogAddr = 49  
  DogBytes = 1  
  Data= '12345678901123456789011'  
  i = DogRead(DogBytes,DogAddr,@Data)  
  if left(data,1)<> 'g' or eee = 99  
  thisform.Image1.width=thisform.width  
  thisform.Image1.height=thisform.height  
  thisform.Image1.visible=.t.  
  thisform.refresh()  
  do form 系列号  
    
  好了,有了源代码,模拟狗就得心应手了,下面是我用 VC++7.0 写的 DLL  
  #include "stdafx.h"  
  #include "string.h"  
    
  short public DogRead(unsigned short DOGBYTES,unsigned short DOGADDR,char \*pData )  
  {  
    switch(DOGBYTES)  
    {  
      case 6:  
        memcpy(pData,"999abc",6);  
        break;  
      case 2 :  
        memcpy(pData,"fg",2);  
        break;  
      case 3 :  
        memcpy(pData,"123",3);  
        break;  
      case 1:  
        memcpy(pData,"g",1);  
        break;  
      case 8:  
        memcpy(pData,"g",1);  
        break;  
      case 5:  
        memcpy(pData,"g",1);  
        break;  
      case 15:  
        memcpy(pData,"g",1);  
        break;  
      default :  
        pData="test!!!";  
        break;  
    }  
    return 0;  
  }  
    
  将编译好的 DLL 替换好 zy4000.dll,运行程序,OK,一切正常,至此,完美破解.  
\--------------------------------------------------------------------------------  
【经验总结】  
  1.壳中读狗部分的处理.  
  2.VFP 程序的结构及相关知识  
  3.常见的对 VFP 的 APP 的加密处理  
  4.模拟狗的操作.  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 06 月 24 日 15:36:09 

 4dd466d0e1e0cc123afbe504da2462ca