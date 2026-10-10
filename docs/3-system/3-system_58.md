

# 手写仅包含一个节的可执行文件

> 原文：[`www.pediy.com/kssd/pediy10/66417.html`](https://www.pediy.com/kssd/pediy10/66417.html)

学习 PE 文件格式有一段时间了。下面是一个只包含一个节的 exe 文件，文件大小一共为 1024 字节。所写的东西没有太多新意，只能被初学的看官  
  
当作厕中佳品哈哈！~~  
  
\->DOS Header  
   e\_magic:         0x5A4D    //'MZ'  
   e\_cblp:          0x0000  
   e\_cp:            0x0000  
   e\_crlc:          0x0000  
   e\_cparhdr:       0x0000  
   e\_minalloc:      0x0000  
   e\_maxalloc:      0x0000  
   e\_ss:            0x0000  
   e\_sp:            0x0000  
   e\_csum:          0x0000  
   e\_ip:            0x0000  
   e\_cs:            0x0000  
   e\_lfarlc:        0x0000  
   e\_ovno:          0x0000  
   e\_res:           0x0000000000000000  
   e\_oemid:         0x0000  
   e\_oeminfo:       0x0000  
   e\_res2:          0x0000000000000000000000000000000000000000  
   e\_lfanew:        0x00000040  
  
\->Signature         0x00004550   //'PE'  
  
\->File Header  
   Machine:                  0x014C  (I386)  
   NumberOfSections:         0x0001  //只包含一个 section  
   TimeDateStamp:            0x00000000  
   PointerToSymbolTable:     0x00000000  
   NumberOfSymbols:          0x00000000  
   SizeOfOptionalHeader:     0x00E0  
   Characteristics:          0x010F  (为 exe 文件)  
  
\->Optional Header  
   Magic:                         0x010B  
   MajorLinkerVersion:            0x00  
   MinorLinkerVersion:            0x00  
   SizeOfCode:                    0x00000200  
   SizeOfInitializedData:         0x00000200  
   SizeOfUninitializedData:       0x00000000  
   AddressOfEntryPoint:           0x00001000  
   BaseOfCode:                    0x00001000  
   BaseOfData:                    0x00000000(试验知这个值在 xp 下任意)  
   ImageBase:                     0x00400000  
   SectionAlignment:              0x00001000  
   FileAlignment:                 0x00000200  
   MajorOperatingSystemVersion:   0x0004  
   MinorOperatingSystemVersion:   0x0000  -> 4.00  
   MajorImageVersion:             0x0000  
   MinorImageVersion:             0x0000  
   MajorSubsystemVersion:         0x0004  
   MinorSubsystemVersion:         0x0000  -> 4.00  
   Win32VersionValue:             0x00000000  
   SizeOfImage:                   0x00002000 //只含一个节，包括文件头部一共是 2000h  
   SizeOfHeaders:                 0x00000200 //从文件头到第一节的原始数据的偏移量  
   CheckSum:                      0x00000000  
   Subsystem:                     0x0002  (WINDOWS\_GUI)  
   DllCharacteristics:            0x0000  
   SizeOfStackReserve:            0x00100000  
   SizeOfStackCommit:             0x00001000  
   SizeOfHeapReserve:             0x00100000  
   SizeOfHeapCommit:              0x00001000  
   LoaderFlags:                   0x00000000  
   NumberOfRvaAndSizes:           0x0000000F  
  
   DataDirectory (F)             RVA             Size  
   -------------                 ----------      ----------  
   ExportTable                   0x00000000 0x00000000  
   ImportTable                   0x00001100 0x0000003C  //把导入表放在 0X300 处  
   Resource                      0x00000000 0x00000000  
   Exception                     0x00000000 0x00000000  
   Security                      0x00000000 0x00000000  
   Relocation                    0x00000000 0x00000000  
   Debug                         0x00000000 0x00000000  
   Copyright                     0x00000000 0x00000000  
   GlobalPtr                     0x00000000 0x00000000  
   TLSTable                      0x00000000 0x00000000  
   LoadConfig                    0x00000000 0x00000000  
   BoundImport                   0x00000000 0x00000000  
   IAT                           0x00000000 0x00000000    
   DelayImport                   0x00000000 0x00000000  
   COM                           0x00000000 0x00000000  
   Reserved                      0x00000000 0x00000000  
  
（二）下面是节表了  
Name     VSize   VAddress    RSize    Roffset    characterstics  
.text    200     1000        200       200         60000020 （代码节属性）  
剩余字段置 0x00  
  
  
（三）代码  
因为我们的代码只有 25byte,所以可以将要显示的字符串放置在 0x200 + 25 = 0x225 再加一个字节 0x00，干脆放到 0x230  
0x230： 'xiep'，其余字节用 0x00 填充  
  
接着是代码:  
PUSH       0               6A 00  
push       'xiep'          68 30104000    //指向'xiep'  
PUSH       'xiep'          68 30104000   
push       0               6A 00  
call       MessageBoxA     E8 07000000    //向后 7 个字节  
push       0               6A 00  
call       ExitProcess     E8 06000000    //向后 6 个字节  
jmp        MessageBoxA     FF25 F0104000  //跳到 IMAGE\_THUNK\_DATA  
jmp        ExitProcess     FF25 F8104000  //将 IMAGE\_THUNK\_DATA 放在 0x2F8 处  
从 0x200 处开始将上面的 Opcode 按序输入，再以 0x00 填充到 0x300  
  
（四）导入表  
导入表的原理：  
http://photo.store.qq.com/http\_imgload.cgi?/rurl2=8e75544ae53199819508a139094a4fdcdbb3d8e95b418608dd5c285f4e1a64770e981f01782b85f3d8cfc33b36186d8de7bea4f2b46e58091b25612590557da46c5b02abddc6dbadc213378830db3b4c355debe3  
  
如上图：0040100E 是 Call MessageBoxA，当 CPU 执行这条指令跳转到 0040101A，0040101A 再跳转到 00402008 处包含的地址，而这个地址在文件被  
  
加载之前并不是一个合法的地址，而是一个指向 IMPORT\_BY\_NAME 结构的 RVA（假设以函数名导入），当文件被加载的时候，操作系统会依据  
  
IMPORT\_BY\_NAME 将 00402008 处替换为 MessageBoxA 的地址。  
  
具体的过程如下：  
1）OS 根据 PE 文件头找到 IMAGE\_DIRECTORY\_ENTRY\_IMPORT 数据目录  
2）再通过 IMAGE\_DIRECTORY\_ENTRY\_IMPORT 的成员变量找到 IMAGE\_IMPORT\_DESCRIPTOR,该结构的 name 是一个指向 DLL 文件名的 RVA  
3）IMAGE\_IMPORT\_DESCRIPTOR 的 FirstThunk 指向 00402008，而\[00402008\]是一个 IMAGE\_THUNK\_DATA 结构，假设它的最高位是 0，即以名称导入，  
  
此时它的低四位就是一个指向 IMPORT\_BY\_NAME 结构的 RVA  
4）IMPORT\_BY\_NAME 包含需要导入的函数的名称以及导入序号  
5）OS 根据 IMPORT\_BY\_NAME 包含的名称，以及 IMAGE\_IMPORT\_DESCRIPTOR 中 name 指向的 DLL 文件名，在 user32dll 中查找函数 MessageBoxA 的地址，  
  
然后将\[00402008\]替换成 MessageBoxA 的实际地址  
  
程序中引入了两个函数，而前面的 ImportTable 我们设为 1100 对应文件偏移 0x300.因为 IMAGE\_IMPOR\_DESCRIPTOR 结构的大小为 5\*4=20 字节，我们  
  
从 user32.dll 和 kernel32.dll 分别导入 MessageBoxA 和 ExitProcess,所以加上一个以全 0 结束的 IMAGE\_IMPOR\_DESCRIPTOR，一共是 3\*20=60 字节(3  
  
个 IMAGE\_IMPOR\_DESCRIPTOR 结构)，所以我们可以在 0x300+60(十进制)=0x33C 处放置 DLL 文件名和 IMAGE\_IMPORT\_BY\_NAME 结构。  
  
我们先在 0x33C 处，  
Hint  0x00           33C--->113C--->0040113C  //函数的导入序号  
'MessageBoxA'        //IMPORT\_BY\_NAME  
填充一个字节 0x00  
'User32.dll'         34A--->114A--->0040114A  //DLL 文件名  
填充一个字节 0x00  
接下来：  
Hint  0x00           355--->1155--->00401155  //函数的导入序号  
'ExitProcess'        //IMPORT\_BY\_NAME  
'Kelnel32.dll'       363--->1163--->00401163  //DLL 文件名  
  
而上面我们在代码中是这样写的：  
jmp        MessageBoxA     FF25 F0104000  //跳到 IMAGE\_THUNK\_DATA  
jmp        ExitProcess     FF25 F8104000  //将 IMAGE\_THUNK\_DATA 放在 0x2F8 处  
所以我们需要将 IMAGE\_THUNK\_DATA 放在 004010F0 即 0x2F0 和 004010F8 即 0x2F8 处。  
所以在 0x2F0 处：  
3C110000  (0x33C 处是 MessageBoxA 的 IMPORT\_BY\_NAME)  
00000000  (以全 0 结束)  
55110000  (0x355 处是 ExitProcess 的 IMPORT\_BY\_NAME)  
00000000  (以全 0 结束)  
  
只剩下 IMAGE\_IMPORT\_DIRECTORY  
Name1                                FirstThunk  
4A110000   //0x34A 处‘user32.dll’    F0100000  //0x2F0 处的 IMPORT\_BY\_NAME  
63110000   //0x363 处‘kernel32.dll’  F8100000  //0x2F8 处的 IMPORT\_BY\_NAME  
其余字段可以置 0，以 0x00 对齐到 3ff，保存为.exe 文件即可。  
  
如有错误之处，请不吝指出！