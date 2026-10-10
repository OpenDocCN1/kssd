

# flashget 1.60a

> 原文：[`www.pediy.com/kssd/pediy06/pediy6551.htm`](https://www.pediy.com/kssd/pediy06/pediy6551.htm)

在论坛上看到 RoBa 修改 flashget 如此有意思,我也忍不住用自己的方法试了一下.  
  
                             修改 flashget 的另类方法(1.60a)  
   我的思路是这样的:在 flashget 的添加下载任务的窗口里,有许多文本框,下载目录就显示在其中的一个文本框里,默认是:c:\\downloads  ,我想,它总要调用 SetWindowTextA 函数来显示,那好,我就在它掉用那个函数的时候做手脚,让它以日期作为目录名来显示.  
    先是给 flashget 加入 kernel32.dll 里的 GetDateFormatA 函数,我们亲自动手,用 Winhex 打开 flashget.exe ,在文件偏移 3c 处看到:18 01 00 00 ,倒过头就是 pe header 的偏移地址:  
  00000118 ,那么导入表的指针地址就是: 118+80=198 ,看到偏移 198 处,是: 18 64 12 00 ,倒过来就是: 00126418 ,呵呵,这就是导入表的 RVA 了,在文件中的实际偏移刚好也是: 126418 .  
    从 126418 处往下搜索 kernel32.dll 找到在 127b38 ,rva 也是这个,好,再往上查找 16 进制值:387b1200 ,找到在 0012644c ,这里就是对应着 kernel32.dll 的 IID 的 Name 项,看到它后面的双字项么,那就是 First Thunk ,指向对应着 kernel32.dll 的 IAT 部分,它也是个 RVA,这里的值是:7cf10f00,倒过来就是:FF17C ,找到 FF17C,从 FF17C 向下找 00000000 ,把从 FF17c 到找到地址的区块复制到一块空白的地方,好了,再在另一块空白的地方写入 GetDateFormat 的子窜,假设其地址是 12a000,然后我们就在刚才复制的数据后面加上 00a01200,假设加入这个地址的所在地为 12b000 ,这个 12b000 是最有用的地方了,我们在程序中调用 GetDateFormat 的方法是:call dword ptr \[image base+0012b000\] ,好了,函数表修好了.  
  
    
  在 flashget 正常运行后,bpx setwindowtexta ,然后点击"添加"按钮,程序被中断,如下:  
004D60A9    8B41 38         MOV EAX,DWORD PTR DS:\[ECX+38\]  
004D60AC    85C0            TEST EAX,EAX  
004D60AE    75 0F           JNZ SHORT flashget.004D60BF  
004D60B0    FF7424 04       PUSH DWORD PTR SS:\[ESP+4\]  
004D60B4    FF71 1C         PUSH DWORD PTR DS:\[ECX+1C\]  
004D60B7    FF15 8CF64F00   CALL DWORD PTR DS:\[<&USER32.SetWindowTex>; USER32.SetWindowTextA  
004D60BD    EB 0E           JMP SHORT flashget.004D60CD  
  
setwindowtexta 函数有两个参数,原型是:BOOL SetWindowText（HWND hwnd，LPCTSTR lpStrjng）  
     压栈时是"first in last out" ,所以, 4d60b0 处是文本的地址,下面的是对象的句柄地址,看看它所指向的地址处,并非"c:\\downloads" ,最后发现这里是关键:  
00529780    FF75 0C         PUSH DWORD PTR SS:\[EBP+C\]  
00529783    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]  
00529786    FF15 8CF64F00   CALL DWORD PTR DS:\[<&USER32.SetWindowTex>; USER32.SetWindowTextA  
0052978C    5E              POP ESI  
0052978D    C9              LEAVE  
0052978E    C2 0800         RETN 8  
  
而其它文本的显示都在这个地方,关键是 ebp 的值不同,但这里用 ebp 的值作为判断是不行的,自己试试就知道为什么了.我把代码写在 529700 处,所以改为:  
004DCB15    74 0C           JE SHORT flashget.004DCB23  
004DCB17    E9 E4CB0400     JMP flashget.00529700  
004DCB1C    90              NOP  
004DCB1D    FF15 8CF64F00   CALL DWORD PTR DS:\[<&USER32.SetWindowTex>; USER32.SetWindowTextA  
004DCB23    5E              POP ESI  
004DCB24    C9              LEAVE  
004DCB25    C2 0800         RETN 8  
自己写入的代码如下:  
00529700    81FD C0E51200   CMP EBP,12E5C0  
00529706    75 78           JNZ SHORT flashget.00529780  
00529708    83FB 02         CMP EBX,2  
0052970B    75 73           JNZ SHORT flashget.00529780;判断控件  
0052970D    8B75 0C         MOV ESI,DWORD PTR SS:\[EBP+C\]  
00529710    BF 90985200     MOV EDI,flashget.00529890       
00529715    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES\];复制默认目录  
00529716    B8 90985200     MOV EAX,flashget.00529890   
0052971B    8A18            MOV BL,BYTE PTR DS:\[EAX\]  
0052971D    83C0 01         ADD EAX,1  
00529720    80FB 00         CMP BL,0  
00529723  ^ 75 F6           JNZ SHORT flashget.0052971B;获得地址,为合并作准备  
00529725    83E8 01         SUB EAX,1;减 1 才是正确的地址  
00529728    8BD8            MOV EBX,EAX  
0052972A    50              PUSH EAX  
0052972B    6A 0A           PUSH 0A  
0052972D    50              PUSH EAX  
0052972E    6A 00           PUSH 0  
00529730    6A 00           PUSH 0  
00529732    6A 01           PUSH 1  
00529734    6A 00           PUSH 0  
00529736    FF15 1E605B00   CALL DWORD PTR DS:\[<&kernel32.GetDateFor>; KERNEL32.GetDateFormatA 获得日期  
0052973C    68 90985200     PUSH flashget.00529890                   ; ASCII "c:\\downloads\\"  
00529741    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]  
00529744    FF15 8CF64F00   CALL DWORD PTR DS:\[<&USER32.SetWindowTex>; USER32.SetWindowTextA 显示目录  
0052974A    B8 00000000     MOV EAX,0  
0052974F    8903            MOV DWORD PTR DS:\[EBX\],EAX;清除手尾,否则目录会越变越长  
00529751    5E              POP ESI  
00529752    C9              LEAVE  
00529753    C2 0800         RETN 8 ;直接返回  
  
下面的是窗口中其它文本的显示处理:  
00529780    FF75 0C         PUSH DWORD PTR SS:\[EBP+C\]  
00529783    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]  
00529786    FF15 8CF64F00   CALL DWORD PTR DS:\[<&USER32.SetWindowTex>; USER32.SetWindowTextA  
0052978C    5E              POP ESI  
0052978D    C9              LEAVE  
0052978E    C2 0800         RETN 8 ;直接返回  
  
  
这样就完完全全可以自定默认目录了

 4dd466d0e1e0cc123afbe504da2462ca