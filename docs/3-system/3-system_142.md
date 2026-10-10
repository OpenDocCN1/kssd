

# 菜鸟编写 PE 文件分析工具

> 原文：[`www.pediy.com/kssd/pediy12/127478.html`](https://www.pediy.com/kssd/pediy12/127478.html)

我是个初学者 ，就是拿这个练练手 ， 大牛不要鄙视我之余希望能给其他刚入门的小菜点帮助 呵呵！  
   
PE 文件的格式在这里我就不多说了（呵呵 ，其实我就会那些 都是看看雪老大的书才明白的）多说了也不会。  
   
虽说 PE 文件的格式在各种资料上都有，无论是图呀还是文字都描述的很清楚，但是让我真正来编写个工具来实现它，我还是遇到了一些不大不小的问题，也许这些问题在大牛那里根本就不叫问题 但在向我这样小菜的眼里 ，很有可能就成为我继续学习的拦路虎，在这里把我的一些问题写出来 希望能对大家有帮助，即使帮不到大家 那也能对我有个提醒吧。以下我是用 C++来说明。  
   
好了 废话说的太多了（一直以来语文作文就一直偏题，这个问题上升到人生来说就能反映出我这人做事不分主次，抓不到重点，希望这次能抓到重点） 啊~~ 又扯了这么多 咋回事呢~  
   
首先我要想写一个工具来分析一个 PE 文件， 那我是不是得先运行我的程序 然后将想要分析的 PE 文件加载到我编写的程序的内存中呢？  
   
这就是我遇到的第一个问题 这么将待分析的 PE 文件映射到内存中？  
   
第一只拦路虎 怎么将 PE 文件映射到内存  
   
其实看完看雪老大的加密与解密这本书你会发现老大已经给了我们提示 只不过我们没注意罢了（有墨迹了）  
   
我的代码里我分成了 4 步：  
   
第 1 步：选取准备分析的 PE 文件路径 （用 CFileDialog ，MFC 已经封装好的类）  
   
第 2 步：通过得到的路径打开文件并得到文件句柄（CreateFile 函数）  
   
第 3 步：通过第 2 步的文件句柄创建文件映射内核对象（CreateFileMapping）  
   
第 4 步：通过第 3 不得到的文件映射内核对象将此文将映射到当前应用程序的地址空间（MapViewOfFile）  
   
嘿嘿 第 4 步过后我们就将 PE 文件原封不动的从磁盘文件上映射到内存空间了 哈哈，现在就可以通过映射之后得到的指针来对 PE 文件进行分析啦。（下面是代码 我已将上面的 4 步在代码中做了注释 你们一定会发现看代码比我说的还清晰 呵呵）  
   
bool CStaticTools::SelectFileAndMapping( CString& strFilePath, CString& strFileName,LPVOID\* lppMemory )  
{  
if ( lppMemory == NULL )  
{  
MessageBox( NULL, "无传出参数", "qq", MB\_OK );  
return false;  
}  
\*lppMemory = NULL;  
   
CFileDialog cFileDialog( TRUE, NULL, NULL, OFN\_HIDEREADONLY|OFN\_OVERWRITEPROMPT|OFN\_ALLOWMULTISELECT, "All Files (\*.\*)|\*.\*||", AfxGetMainWnd());　　　　　　　　　　　　//第 1 步 取得路径  
if ( cFileDialog.DoModal() == IDOK )　　　　　　　　　　　　  
{  
strFilePath = cFileDialog.GetPathName();  
strFileName = cFileDialog.GetFileName();　　　　　　　//到这 第 1 步顺利完成了（战果：文件路径， 文件名）  
   
HANDLE hFile = CreateFile( strFilePath, 　　　　　　　 //第 2 步打开文件 得到文件句柄  
GENERIC\_READ/\*|GENERIC\_WRITE\*/,  
FILE\_SHARE\_READ|| FILE\_SHARE\_WRITE,  
NULL, OPEN\_EXISTING,  
FILE\_ATTRIBUTE\_NORMAL,  
NULL );  
if ( hFile == INVALID\_HANDLE\_VALUE )  
{  
MessageBox( NULL, "未打开目标文件", "qq", MB\_OK );  
return false;  
}  
   
HANDLE hFileMap = CreateFileMapping( hFile, NULL, PAGE\_READONLY/\*WRITE\*/, 0, 0, NULL ); //第 3 步 创建文件映射内核对象  
if ( hFileMap == NULL )  
{  
MessageBox( NULL, "没有产生文件映射表", "qq", MB\_OK );  
CloseHandle( hFile );  
return false;  
}  
   
\*lppMemory = MapViewOfFile( hFileMap,FILE\_MAP\_READ, 0, 0, 0 );　　　　　//第 4 步 将文件映射到内存得到映射后的文件首地址  
if ( \*lppMemory == NULL )  
{  
MessageBox( NULL, "没有将映像映射到内存中", "qq", MB\_OK );  
CloseHandle( hFileMap );  
CloseHandle( hFile );  
return false;  
}  
CloseHandle(hFileMap);  
CloseHandle(hFile);  
   
return true;  
}  
   
return false;  
}  
   
   
   
第二只拦路虎 物理地址偏移与虚拟地址偏移  
   
万事开头难，其实第一只拦路虎已经被消灭后我已经有了武松的感觉了 呵呵 其他的分析就简单了 ，只要按《加密与解密》书中的分析一点一点在可视化界面上显示就可以了，这个过程正是我们熟悉 PE 格式的过程，千万不能省略呀（其实把这个过程省略了 你还写个屁呀）  
   
当把 PE 文件映射到当前分析工具进程的内存后，其在内存中分布和在物理磁盘上是一样的，而分析的时候会用到一些数据，这些数据的内容是当 PE 文件执行时在内存中的虚拟地址偏移（RVA），但由于对齐方式的不同，所以需要把这个虚拟地址偏移（RVA）转换成物理地址偏移之后才能进行往下的分析。  
   
这里用到一个公式（同样看雪大大已经在书中说的很明白了）  
   
物理地址偏移 = 虚拟地址偏移 - 虚拟地址偏移所在的虚拟区块的偏移 + 与虚拟区块对应的物理区块的偏移  
   
这里要解释下 什么叫 虚拟区块的偏移 与 物理区块的偏移（ 哈哈 没错 这就是我自己为了描述方便自定义的名词 ）  
   
我所说的虚拟区块的偏移是 当 PE 文件运行时在内存中的区块相对于基地址的偏移   
   
物理区块的偏移就是磁盘上的 PE 文件的区块相对于基地址的偏移 就是因为他们与虚拟区块的对齐方式的不同才引出了这么多问题 哎  
   
（什么 还不明白 那我也没办法了，不是你智力不行 是我描述水平太差，别灰心 去看看雪大大的书 讲的很详细的）  
   
编程思路是在 PE 文件头中的 IMAGE\_FILE\_HEADER 结构中取得区块的数量 然后遍历各个区块的区块表 找到当前要转换的虚拟地址偏移在哪个虚拟区块中，然后用上面的公式计算 （哎 说不明白了 看代码吧 。有点恼羞成怒的味道了）  
   
bool CStaticTools::GetPhysicalAddress( const long& dwRVA, long& dwPhysicalAddress )//传入虚拟地址偏移(RVA),传出物理地址偏移  
{  
if ( dwRVA == 0 )　　　　//容错  
{  
return true;  
}  
   
long lSectionNumber = strtoul( stData.m\_strNumberOfSection, NULL, 16 ); //得到区块数量，就是取 IMAGE\_FILE\_HEADER 结构中的 NumberOfSections 的值  
   
for ( int i=0; i<lSectionNumber; ++i )　　　　　　　　　　　　　　　　//遍历各个区块找到传入参数的 RVA 所在的区块  
{  
long lVirtualAddress = strtoul( stData.m\_stSection\[i\].m\_strVirtualAddress, NULL, 16 );//取得虚拟区块地址偏移，取得方法参看区块表  
long lVirtualSize = strtoul( stData.m\_stSection\[i\].m\_strVirtualSize, NULL, 16 );//取得虚拟区块的大小，取得方法参看区块表  
if ( (lVirtualAddress <= dwRVA) && (dwRVA < (lVirtualAddress+lVirtualSize) ) )//找 RVA 在哪个区块的判断  
{  
long lPointerToRawData = strtoul( stData.m\_stSection\[i\].m\_strPointerToRawData, NULL, 16 );//取得相应的物理区块地址偏移，取得方法参看区块表  
dwPhysicalAddress = dwRVA - lVirtualAddress + lPointerToRawData;//用上面的公式计算物理地址偏移  
return true;  
}  
}  
   
return false;  
}  
   
通过上面的函数得到的物理地址偏移再加上我们第一步拦路虎中得到的内存映像起始地址就可以很轻松的进行数据的读取了。  
   
   
   
第三只拦路虎 输出表结构（IED）中的 Base 属性  
   
在写输出表的时候 对这个 IMAGE\_EXPORT\_DIRECTORY 中的 Base 属性理解出现过偏差，认为在通过 AddressOfNameOrdinals 查找输出 EAT 索引后再加上这个 Base 值才是这个输出函数地址在 EAT 中的索引值，但是后来才发现 Base 属性和那个 AddressOfNameOrdinals 一点关系都没有。AddressOfNameOrdinals 中查到的索引就是输出函数在 EAT 中的索引。不过可以将这个索引值加上 Base 属性就可以得到此输出函数的输出序数。  
   
好了 其实第 3 只拦路虎不是什么问题，只是我理解输出表时遇到的一个小误区，通过输出表的编写发现了这个我理解上的误区，改正过来了，呵呵 所以说有时候多动动手还是不错的 起码如果我不动手，那么这个问题也许我永远也不会真正的发现他。  
   
好了 一上就是我编写的 PE 工具的所有体会了 ，我这个 PE 工具编写的很简单，功能也很不强大，不过说了吗 是入门，以后等我能力强大了 我会改善它的，如果现在我一直在扣它 那我就真的有可能一直在这里处于入门阶段了 学不到其他技术了 呵呵。 这个工具就先这样吧，让他先告一段落 不过话又说回来 ，等我学的差不多 要打造我自己的工具时， 他还是很有参考意义的，它的很多代码会被我重用 呵呵不过那是后话了。本来想重新整理下代码，让他们更加健壮和利于扩展，但发现这个工具实在太小了 ，没有意义在弄得更复杂了 就这样吧。代码写得质量不高请大家见谅！

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || PE.rar  
 |