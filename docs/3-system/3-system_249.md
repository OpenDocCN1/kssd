

# NTFS 解析?仅仅解析

> 原文：[`www.pediy.com/kssd/pediy10/84344.html`](https://www.pediy.com/kssd/pediy10/84344.html)

这里不想多说,以免误导别人.  
资料:  
1,  
  
ReactOS 代码(drivers\\filesystem\\ntfs\\)  
  
2,  
  
这里提下在 ReactOS 下的 drivers\\filesystem\\ntfs\\note.txt 中提到的资料不错的说...  
  
note.txt  
  
/\*  
  
These are notes about the NT filesystem  
  
More documentation is available at:  
http://linux-ntfs.sourceforge.net/ntfs/index.html  
  
\*/  
就是这里:  
http://sourceforge.net/project/showfiles.php?group\_id=13956&package\_id=16543  
  
3,  
  
还有是 NT 源码  
nt4\\private\\ntos\\cntfs\\ntfs.h  
  
4,  
  
一个哥们写的<用 Winhex 手工定位 NTFS 文件系统下的文件>,好呀,不过文中的 16 进制和 10 进制有些不分,总之很好的资料的说...  
  
5,  
  
<NTFS 文件解析系统的简单分析> by:MTrickster  
  
学习过程中写的一个历遍全盘文件的测试的代码:  
(注:代码不全,仅仅解析了 A0 和 90 属性,其他掠过,详细删除,新建的见 MFC 版本(还没写完),  
将文件解析的代码整成 NTFSFindFirstFile 和 NTFSFindNextFile 的主意不错,赞一个)

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ParseNTFS.rar  
 |