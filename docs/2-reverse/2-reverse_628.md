

# santmat's reservesme-3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6268.htm`](https://www.pediy.com/kssd/pediy06/pediy6268.htm)

我也贴一个,这是看雪以前的精华里包含的一个程序，然后我不是按照 Code\_inside 方法修改的   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
                      Target SantMat's ReverseMe 3   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
Author:        SantMat   
Protection:   None   
URL:        http://www.immortaldescendants.org/users/santmat/reversemes/reme3.zip   
Needs:        Hiew,WinHex,Some PE knowledge,Most important is Our Brain   
     
  我们的目标是 SantMat 的 ReverseMe3,你可以按以上 URL 下载,也可以在我的主页   
下载,我们要使它弹出一个 MessageBox.有关 PE 的知识你可以参看 Iczelion 的文章   
!都是 e 文的,当然你也可以到看学那里下载,里面有翻译好了的中文! 好了我们现在   
假设你已经了解了相关的 PE 知识.   
Let Begin.......   
  
  我们将在 offset 9FF 处增加长度,我们从 offset 220 处开始到 offset 3FF 选中   
它,然后复制到文件尾部,以便我们以后修改!增加了 offset A00 到 offset BDF 的   
长度!   
  好了现在我们开始修改 PE 头的内容:一个是 Section 部分,我们知道从 PE 头开始   
偏移 0x06 是 Section 的数目,至于 PE 头的起始是根据 IMAGE\_DOS\_HEADER 部分开始来   
推出来的!从文件头开始偏移 0x3C 是 e\_lfanew 字段,它指向 PE 头"PE",0,0,简单的   
方法我们可以直接用 UE 看到右边的 ASCII 中的 PE,那就是 PE 的开始的地方,我们偏   
移 0x06 修改 Section:   
000000B0   50 45 00 00 4C 01 03 00 1A 9A DB 39 00 00 00 00   PE..L....氎 9....   
Change To:                  @||@   
                             /   
000000B0   50 45 00 00 4C 01 04 00 1A 9A DB 39 00 00 00 00   PE..L....氎 9....   
  
  我们改变了 Section 的数目,现在我们来填充进去,看看 Section Table 的结构:   
IMAGE\_SECTION\_HEADER STRUCT    
   Name1 db IMAGE\_SIZEOF\_SHORT\_NAME dup(?) ;块名,8 个字节   
   union Misc    
      PhysicalAddress dd ?    
      VirtualSize dd ?    
   ends    
   VirtualAddress dd ? ;该块的 RVA   
   SizeOfRawData dd ? ;在文件对齐后的尺寸   
   PointerToRawData dd ? ;在文件中的偏移   
   PointerToRelocations dd ? ;无用   
   PointerToLinenumbers dd ? ;无用   
   NumberOfRelocations dw ? ;无用   
   NumberOfLinenumbers dw ? ;无用   
   Characteristics dd ? ;块属性   
IMAGE\_SECTION\_HEADER ENDS    
  
  我们可以知道块的大小是 40 个字节即 0x28,同时第一个块的起始位置是 PE 的 offset   
加上 0xF8,这里我们 PE 头的开始处是 0xB0,我们加上 0xF8,即 0xB0+0xF8=0x1A8,我们查看   
0x1A8 处,看到了吧,是块名".text",同理你可以在加上 0x28,我们就到了第二个块的位置   
0x1A8+0x28=0x1D0,看到了第二个块名称了吧是".rdata"同理我们可以在接下的 0x28 处   
得到第三个块的信息,在这个程序里面是 0x1D0+0x28=0x1F8,即".data",接下来是下一个   
0x28 处是我们的第四个 Section 的信息,所以我们要手动添加内容(Need PE Knowledge):   
  
+0 8byte Name1 ;块名 dengkeng   
+8 dword VirtualSize ;100,That's enough   
+C dword VirtualAddress ;0x4000   
10 dword SizeOfRawData ;0x200   
14 dword PointerToRawData ;我们的从 0xA00 开始   
18 dword PointerToRelocations ;一般为 0x00000000   
1C dword PointerToLinenumbers ;一般为 0x00000000   
20 word NumberOfRelocations ;一般为 0x0000   
22 word NumberOfLinenumbers ;一般为 0x0000   
24 dword Characteristics -----> ours  is write read execute and code.so it's E0000020   
  
  需要说明的地方是 PointerToRawData,我们从 0xA00 开始,因为第三个 Section 的 PointerToRawData   
是从 0x800 开始的,并且 SizeOfRawData 是 0x200,所以我们开始的位置是 0x800+0x200=0xA00.   
下面我们开始修改:0x1F8+0x28=0x220,是我们 Section 开始的地方   
  
00000220   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
00000230   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
00000240   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
  
Change To:   
00000220   64 65 6E 67 6B 65 6E 67  00 01 00 00 00 40 00 00   dengkeng.....@..   
00000230   00 02 00 00 00 0A 00 00  00 00 00 00 00 00 00 00   ................   
00000240   00 00 00 00 20 00 00 E0  00 00 00 00 00 00 00 00   .... ..?.......   
     
  接下来我们要做的是增加一个 API,MessageBox,它在 User32.dll 中,好了,在这之前我们   
有必要了解一下,Import Table(输入表),我们可以在据 PE 头 0x80 处找到它,即 0xB0+0x80=0x130   
我们看到了如下内容:   
00000130   08 20 00 00 28 00 00 00  00 00 00 00 00 00 00 00   . ..(...........   
  我们看到在 0x130 处的内容是 0x00002008,是数据的起始地址,0x00000028 是它的大小,我们得到   
磁盘上的 offset,2080 在".rdata"Section 中,它的 Raw Offset 是 600,所以我们查看 608 处的内容:   
                             Begin=>||   
                                    /   
00000600   38 20 00 00 00 00 00 00  30 20 00 00 00 00 00 00   8 ......0 ......   
00000610   00 00 00 00 46 20 00 00  00 20 00 00 ($)00 00 00 00   ....F ... ......   
00000620   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
00000630   38 20 00 00 00 00 00 00  75 00 45 78 69 74 50 72   8 ......u.ExitPr   
00000640   6F 63 65 73 73 00 4B 45  52 4E 45 4C 33 32 2E 64   ocess.KERNEL32.d   
00000650   6C 6C 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ll..............   
00000660   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
00000670   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
00000680   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
  在 offset 0x608 处,Import Table 的内容,我们看一下 Import Table 结构:   
  
IMAGE\_IMPORT\_DESCRIPTOR STRUCT    
  union    
    Characteristics dd ?    
    OriginalFirstThunk dd ? ;IMAGE\_THUNK\_DATA 数组的 RVA,每个指针指向 IMAGE\_IMPORT\_BY\_NAME 结构   
  ends    
  TimeDateStamp dd ? ;32 位时间标志,忽略   
  ForwarderChain dd ? ;忽略   
  Name1 dd ? ;指向 Dll 名字   
  FirstThunk dd ? ;IMAGE\_THUNK\_DATA 数组的 RVA   
IMAGE\_IMPORT\_DESCRIPTOR ENDS    
  
  这里是:   
+0 OriginalFirstThunk ;0x00002038   
+4 TimeDateStamp ;0x00000000   
+8 ForwarderChain ;0x00000000   
+C Name1 ;0x00002046   
+10 FirstThunk ;0x00002000   
  
  FirstThunk,OriginalFirstThunk 都指向两个相同的数组 IMAGE\_THUNK\_DATA,这些数组称为   
输入名称表(Import Name Table,INT)和地址输入表(Import Address Table,IAT).两者的区别   
是 INT 不可改写,IAT 则是由 PE 装载器重写的.(具体操作请查有关资料)   
  IMAGE\_IMPORT\_DESCRIPTOR 结构是以 5 个 dd 全 0 的结构结束,在上面的'$'处就是结束的开始地方,   
接下来是 5 个全 0 的 dd.   
  我们可以看到这 OriginalFirstThunk,Name1,FirstThunk 中的内容:   
0x00002038 <======>offset 0x638 ;指向 IMAGE\_IMPORT\_BY\_NAME   
0x00002046 <======>offset 0x646 ;指向 KERNEL32.dll 字符串   
0x00002000 <======>offset 0x600 ;内容为 0x00002038(offset 0x638)   
  
IMAGE\_IMPORT\_BY\_NAME STRUCT    
  Hint dw ? ;是本函数在所其驻留 DLL 的输出表中的序号   
  Name1 db ? ;函数名,以 NULL 结尾   
IMAGE\_IMPORT\_BY\_NAME ENDS    
  
  OriginalFirstThunk 指向 IMAGE\_IMPORT\_BY\_NAME,0x0075 则是 Hint,后面的则是函数名.FirstThunk   
中的内容也是指向 0x638,即都是指向同一处.   
  接下来我们要在我们新的 Section 中增加内容:   
  
offset 00000A00    00000000   
offset 00000A04    00000000   
offset 00000A08    00000000   
offset 00000A0C    00002046   
offset 00000A10    00002000   
接下来是另一个 IID:   
offset 00000A14    00000000   
offset 00000A18    00000000   
offset 00000A1C    00000000   
offset 00000A20    00004044 ;这个地方是根据后面推出来的,先不急   
offset 00000A24    0000403C ;这个地方是根据后面推出来的,先不急   
  
我们开始添加:   
  
00000A00   00 00 00 00 00 00 00 00  00 00 00 00 46 20 00 00   ............F ..   
00000A10   00 20 00 00 00 00 00 00  00 00 00 00 00 00 00 00   . ..............   
00000A20   44 40 00 00 3C 40 00 00  00 00 00 00 00 00 00 00   D@..<@..........   
00000A30   00 00 00 00 00 00 00 00  00 00 00 00 50 40 00 00   ............P@..   
00000A40   00 00 00 00 75 73 65 72  33 32 2E 64 6C 6C 00 00   ....user32.dll..   
00000A50   00 00 4D 65 73 73 61 67  65 42 6F 78 41 00 00 00   ..MessageBoxA...   
00000A60   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................   
00000A70   43 72 61 63 6B 00 00 00  00 00 00 00 00 00 00 00   Crack...........   
00000A80   4D 6F 64 69 66 79 20 74  68 65 20 52 65 76 65 72   Modify the Rever   
00000A90   73 65 6D 65 20 42 79 20  64 65 6E 67 6B 65 6E 67   seme By dengkeng   
00000AA0   20 51 51 3A 32 38 38 39  35 37 35 31 00 00 00 00    QQ:28895751....   
  
  我们可以看到在我们添加的 2 个 IID 结构后面有 1 个全 0 的 IID,以标志其结束,在结束的标志   
后面我们添加了 DLL 名"user32.dll",所以第二个 IID 中的 Name1 字段填入 00004044,使其指向   
"user32.dll",接下来是添加我们所需的 API,MessageBoxA,为了方便我们在 0xA50 处添加,当然   
我们也可以在 0xA4F 处添加,那样填写 FirstThunk 就不好看了,所以我们从 0xA50 开始,前面两个   
字为 00 00,代表 Hint,接下来就是填写 MessageBoxA,好了现在我们在结束的 IID 后面添加一个   
API 的 RVA,即 00004050,使我们添加的第二个 IID 中的 FirstThunk 指向它,所以我们在第二个 IID   
中的 FirstThunk 中写入的内容为 0000403C.   
  还有重要的一点的说明是在结束的 IID 后面添加 API 函数的 RVA 应该以双字为 0 的内容结束,   
即 0xA40 到 0xA43 为 0,我就是因为少考虑了这一点最后修改的程序出了错误!切记,切记!!!!   
  好了我们在 0xA70 和 0xA80 添加 Caption 和 Text,好了现在我们需要做的就是修改 Import Table   
的起始地址和长度了:   
  在 PE 头偏移 0x80 处这里是 0xB0+0x80=0x130   
  
Change To:   
  
 修改起始 RVA=>||       ||<=Size,3 个 IID 长度   
              /       /   
00000130   00 40 00 00 3C 00 00 00  00 00 00 00 00 00 00 00   .@..,...........   
  最后我们需要修改的就是程序的入口地址了(病毒??),AddressOfEntryPoint 在 PEHeadr 的偏移   
0x28 处,我们这里是 0xB0+0x28=0xD8 处:   
  
               修改为 000040B0(RVA)=>||<=因为我们是在 0xAA0 结束,我们将在 0xAB0 开始 Write Code   
                                    /   
000000D0   00 04 00 00 00 00 00 00  B0 40 00 00 00 10 00 00   ........燖......   
  下面的工作就是写 Code 了,不多只要几句话,当然我们有必要了解一下,MessageBox 结构:   
int MessageBox(   
    HWND hWnd,  // handle of owner window   
    LPCTSTR lpText,  // address of text in message box   
    LPCTSTR lpCaption,  // address of title of message box     
    UINT uType   // style of message box   
);   
  我们拿出 Hiew,Go,Let Write Code....   
我们依次从右至左,Push   
push 0       ;uType   
push 404070  ;Caption   
push 404080  ;Text   
push 0       ;hWnd   
call d,\[40403C\] ;Call d,\[ImageBase+FirstThunkRVA\]   
push 401000     
ret          ;Function ret just as jmp 401000   
  F9 存盘运行,Yeah,看到弹出的 MessageBox 了吗???   
  
  
Made By dengkeng   
E-mail:shellc0de@sohu.com   
欢迎转载,请保持文章的完整性  
 4dd466d0e1e0cc123afbe504da2462ca