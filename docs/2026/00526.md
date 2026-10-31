
# [原创]windbg下断辅助

## 小软件说明

小软件说明，

昨晚玩QQ欢乐斗地主，豆输光了，

无聊之余写下这小东西，

只为了方便自己在调试没有符号文件的驱动时提供方便。

没啥技术含量，毋笑之！

2012.04.14

by:小覃

---

实现的过程就是按特征搜索函数的文件偏移，

再将偏移转换为`rva`，

就这么简单.........

遍历过程如下，

```
......
CString strBP="";
DWORD FuncOffset = m_lPESec.FindNextFunc();
  while ( FuncOffset )
  {
    CString strTemp="";
    DWORD rav = m_lPESec.OffsetToRva( FuncOffset );
    strTemp.Format("bu %s+%X", m_strName, rav );
    strBP += strTemp;
    strBP += "\r\n";
    FuncOffset = m_lPESec.FindNextFunc();
  }
```

`FindNextFunc`的实现过程，

```
DWORD LPE::FindNextFunc()
{
  BYTE funCharacteristic[] = {0x8B, 0xFF, 0x55, 0x8b, 0xEC};
  byte *start;
  if ( 0 == StartPosOffset )
    start = (byte*)pstMapFile->ImageBase;
  else
    start = (byte*)pstMapFile->ImageBase + StartPosOffset + 5;
  DWORD OldStartPosOffset = StartPosOffset;
  while( StartPosOffset < pstMapFile->dwFileSize && memicmp( start, funCharacteristic, 5) != 0 ){
    StartPosOffset++;
    start = (byte*)pstMapFile->ImageBase + StartPosOffset;
  }

  return OldStartPosOffset==StartPosOffset ? 0 : StartPosOffset;
}
```

代码很乱就没全部发上来丢人了，![](https://bbs.kanxue.com/view/img/face/16.gif)

思路就是酱紫了........

下面是对照调试自己写的有符号文件的驱动，

很明显进入所有函数时被拦截了，

第一次断在`INIT`的入口处，第二次断在`TEXT`段自己的代码入口。

如下图：

![](https://bbs.kanxue.com/upload/attach/201204/360361_owgykva18phyrl2.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [windbg下断辅助.rar](javascript:void(0);)
  （235.26kb，53次下载）

- [windbg下断辅助截图.jpg](javascript:void(0);)
  （161.32kb，22次下载）
