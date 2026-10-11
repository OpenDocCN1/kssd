
# PE 文件格式学习小结 之 文件偏移地址计算器功能的实现

> 原文：[`www.pediy.com/kssd/pediy10/67955.html`](https://www.pediy.com/kssd/pediy10/67955.html)

【文章标题】: 谈一下 LoadPE 的 FLC 功能的编程实现  
【文章作者】: 美丽の破船  
【作者邮箱】: juxingting@163.com  
【作者主页】: http://hi.baidu.com/juxingting  
【编写语言】: C/C++  
【操作平台】: XP + VC6.0  
【作者声明】: 学 PE 文件格式的练习程序，发出来供象我一样菜的朋友共同进步！程序实现的算法很笨，往高手指教！

【详细内容】:

`LoadPE` 的功能强大想必是众所周知的！

小弟我最近在学习 PE 文件格式，为了巩固自己的所学，我也在模仿 `LoadPE` 的 PE 编辑器功能！

昨天，我把它的 FLC 功能给实现了，个人认为收获不小，颇为激动。

在失眠了一夜之后，终于耐不住、来发帖子，与大家分享我的所得！（说不定，再让我骗来个精华~![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") ）

程序用的是最笨的方法，外加笨人比较愚笨，代码写的也罗嗦无比！故高手飘过~~~![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

闲话不说了，说主题：

在学习《加密与解密》（第二版）的时候，书里说的根据 `RVA` 计算文件偏移的方法如下：

1. 先遍历区段表中每个区段的 `Virtual Offset` 得到每个节的起始 `RVA`，并根据 `SizeOfRawData` 属性得到每个节的结束 `RVA`
2. 再根据第一步的结果判断要转换的 `RVA` 属于哪个区段内
3. 然后根据区段表中相应区段的 `Virtual Offset` 和 `Raw Offset` 的值来计算出每个区段中，它们两个的差值：`△k`
4. 最后就可以根据公式转换了!

   `RVA = VA - ImageBase;`
   `File Offset = RVA - △k;`

可是怎么根据 `File Offset` 来计算 `RVA` 就不知道怎么办了~~

随着我对 PE 文件结构的了解越来越深，慢慢的我才知道：节的 `PointerToRawData` 属性就是它在文件中的偏移地址！![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

这样一来，它们之间转换的方法就变成了：

1. 先遍历区段表中每个区段的 `Virtual Offset` 得到每个节的起始 `RVA`，并根据 `SizeOfRawData` 属性得到每个节的结束 `RVA`
2. 再根据第一步的结果判断要转换的 `RVA` 属于哪个区段内
3. 然后用要转换的 `RVA` - 它所在的起始 `RVA` 得到 这个 `RVA` 位于起始 `RVA` 的偏移 `RVA'`
4. 最后取出它所在段的 `PointerToRawData` 属性值加上上一步算出来了 `RVA'` 便得到它的文件偏移地址了！

从文件偏移转换到 `RVA` 的方法也可想出来了，只不过是把上面的方法倒过来就可以了~~~![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

下面我给出我写的代码:

代码:

```
void CTESTDlg::OnExec() 
{
  UpdateData(TRUE);
  DWORD Temp;
  if(((CButton*)GetDlgItem(IDC_RVA))->GetCheck())
  {
    sscanf(m_RvaCNT,"%X",&Temp);
    m_VaCNT.Format("%.8x",(m_ImgBase + Temp));
    if(Temp <= m_Sections[m_NumberOfSections-1].VirtualAddress + m_Sections[m_NumberOfSections-1].SizeOfRawData)
    {
      for(int a = 0;a < m_NumberOfSections;a++)
      {
        if(Temp >= m_Sections[a].VirtualAddress && Temp <= (m_Sections[a].VirtualAddress + m_Sections[a].SizeOfRawData))
        {
          m_OffCNT.Format("%.8x",(Temp - m_Sections[a].VirtualAddress)+m_Sections[a].PointerToRawData);
          m_SecName.Format("%s",m_Sections[a].Name);
          sscanf(m_OffCNT,"%X",&Temp);
          m_Content = GetHexContent(Temp);
        }
      }
    }
    else
    {
      MessageBox("超出范围了！！！");
      m_VaCNT = "";
      m_RvaCNT = "";
      m_OffCNT = "";
    }
  }
  else if(BST_CHECKED == ((CButton*)GetDlgItem(IDC_VA))->GetCheck())
  {
    sscanf(m_VaCNT,"%X",&Temp);
    m_RvaCNT.Format("%.8x",(Temp - m_ImgBase));
    sscanf(m_RvaCNT,"%X",&Temp);
    if(Temp <= m_Sections[m_NumberOfSections-1].VirtualAddress + m_Sections[m_NumberOfSections-1].SizeOfRawData)
    {
      for(int a = 0;a < m_NumberOfSections;a++)
      {
        if(Temp >= m_Sections[a].VirtualAddress && Temp <= (m_Sections[a].VirtualAddress + m_Sections[a].SizeOfRawData))
        {
          m_OffCNT.Format("%.8x",(m_Sections[a].PointerToRawData + (Temp - m_Sections[a].VirtualAddress)));
          m_SecName.Format("%s",m_Sections[a].Name);
          sscanf(m_OffCNT,"%X",&Temp);
          m_Content = GetHexContent(Temp);
        }
      }
    }
    else
    {
      MessageBox("超出范围了！！！");
      m_VaCNT = "";
      m_RvaCNT = "";
      m_OffCNT = "";
    }
  }
  else if(BST_CHECKED == ((CButton*)GetDlgItem(IDC_OFF))->GetCheck())
  {
    sscanf(m_OffCNT,"%X",&Temp);
    if(Temp <= (m_Sections[m_NumberOfSections-1].PointerToRawData + m_Sections[m_NumberOfSections-1].SizeOfRawData))
    {
      for(int a = 0;a < m_NumberOfSections;a++)
      {
        if(Temp >= m_Sections[a].PointerToRawData && Temp <= (m_Sections[a].PointerToRawData + m_Sections[a].SizeOfRawData))
        {
          m_RvaCNT.Format("%.8x",(m_Sections[a].VirtualAddress + (Temp - m_Sections[a].PointerToRawData)));
          m_VaCNT.Format("%.8x",(m_Sections[a].VirtualAddress + (Temp - m_Sections[a].PointerToRawData) + m_ImgBase));
          m_SecName.Format("%s",m_Sections[a].Name);
          sscanf(m_OffCNT,"%X",&Temp);
          m_Content = GetHexContent(Temp);
        }
      }
    }
    else
    {
      MessageBox("超出范围了！！！");
      m_VaCNT = "";
      m_RvaCNT = "";
      m_OffCNT = "";
    }
  }
  m_RvaCNT.MakeUpper();
  m_VaCNT.MakeUpper();
  m_OffCNT.MakeUpper();

  UpdateData(FALSE);
}
```

具体的代码,大家可以看附件!

希望我这个烂文章能对向我一样梦想自己成为高手的朋友有所帮助!

最后,祝大家成功!!加油~~

【附件下载】:文件地址转换器及 VC 源码.rar

--------------------------------------------------------------------------------

2008 年 07 月 06 日 12:26:35
