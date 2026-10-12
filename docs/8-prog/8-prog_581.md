
# 用 vc 实现 ＯＯ消息群发的关键代码解析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-690.htm`](https://www.pediy.com/kssd/pediy08/pediy8-690.htm)

寒假时做的一个 `QQ` 消息发送器，请各位见谅，由于我级别不够，不能将整个代码和图片传上来，所以现在将它的一些核心部分的技术拿出来细讲一下，希望能对读者来一点思路，做出更好的东西。

感谢网上的所有高手将自己的经验分享出来让我做了许多的参考

感谢班主任杨老师的指导

做 `QQ` 消息发送器，最重要的是要先找到 `QQ` 聊天的窗口，然后将自己想说的话写到 `QQ` 消息发送框里，在模拟键盘输入按下“发送”的按钮，这样就可以搞定了（呵呵，是不是很简单的啊），那好，现在就让我们来动手做吧。

## 1. 找窗口

想要找到 `QQ` 聊天的窗口，我们要用到 2 个函数 `FindWindowEx()` 和 `GetWindowText()`。首先我们用 `FindWindowEx()` 函数找到所有桌面的窗口，当然，我们只需要 `QQ` 的聊天窗口，其他的对我们没用。`FindWindowEx()` 只能找到一个窗口，怎么才能找到所有的窗口然后我们再判断是不是 `QQ` 的聊天窗口呢？这就要用到 `vc++` 给我们提供的一个很好的函数 `GetWindow()` 了。让我们看看下面的代码：

```
HWND hwnd=FindWindowEx(NULL,NULL,"#32770",NULL);

While (hwnd != NULL)

{
   If ()
     {
        //……..
     }
Else
{
     hwnd=::GetWindow(hwnd,GW_HWNEXT);
}
}
```

用以上的代码就可以将桌面上的窗口一个一个的找到，然后我们就对一个一个的窗口的名字进行判断，如果含有“与”（因为 `QQ` 聊天窗口的名字都是“与 XX 聊天中”）。

## 2. 避免发送错误

由于桌面上的窗口按照那个 `Z` 序列放置的，如果按照找到一个窗口，发送一句话，然后再找个窗口，再发送一句话的思路会出错的（我做的时候就是这样出错了，感谢我的班主任杨老师的指导使我明白了这个问题）。所以我们就要想个方法避免上面那个思路了。我在这用到了数组，在找到窗口后先保存起来，在找完所有的窗口后在集体实行发送消息。

```
CTypedPtrArray<CObArray, CWnd*> myArray;
```

这句就是我创建了一个窗口的 `array`（如果您对上面这句话不理解，请查阅 `MSDN`）。

OK 这就是整个的思想。现在我将核心代码拿来让大家看看。

## 3. 核心代码解析

```
void CQQView::OnButton1()

{
  // TODO: Add your control notification handler code here
  CTypedPtrArray<CObArray, CWnd*> myArray;
    // 这个 m_edit 是 Edit Box 窗口的一个变量，它存放您想发送的那句话
  // 如果 m_edit 为空，说明您还没写您想发送的话，它提示您先写上您想发送的话
    if(m_edit.IsEmpty())
    {
      MessageBox("please enter what do you want to say!!!",
             "Attention",
             MB_OK |  MB_ICONWARNING);
      return;
    }
    HWND hwnd=FindWindowEx(NULL,NULL,"#32770",NULL);
    while(hwnd)
    {
      CString string;
      char a[100];
      HWND send,pro,pro1,message;
      // 得到窗口名称
      ::GetWindowText(hwnd,a,100);
      string.Format("%s",a);
      // 如果含有"与"(因为 QQ 聊天窗口的名字都是"与 XX 聊天中");说明是 QQ 窗口
      if(string.Find("与")>-1)
      {
        pro=FindWindowEx(hwnd,NULL,"#32770",NULL);
        CWnd *pWnd=CWnd::FromHandle(pro);
        pro1=FindWindowEx(pWnd->m_hWnd,NULL,"AfxWnd42",NULL);
        CWnd *child=CWnd::FromHandle(pro1);
                // 找到 QQ 窗口的哪个写消息的文本框
        message=FindWindowEx(child->m_hWnd,NULL,"RICHEDIT",NULL);
            if(message)
            {
               CRichEditCtrl* rich
                     =  (CRichEditCtrl*)(CWnd::FromHandle(message));
               rich->ReplaceSel(m_edit);
            }
            else
            {
               return;
            }
        send=FindWindowEx(pWnd->m_hWnd,NULL,NULL,NULL);
        string="";
            while(send)
            {
              child=CWnd::FromHandle(send);
              child->GetWindowText(string);
              if(string.Find("发送")>-1)
              {
                           //加入指针数组
                myArray.Add(child);
                 send=NULL;
              }
              else
              {
                 send=FindWindowEx(pWnd->m_hWnd,send,NULL,NULL);
              }
            }
        hwnd=::GetWindow(hwnd,GW_HWNEXT);
      }
      else
        hwnd=::GetWindow(hwnd,GW_HWNEXT);
    }
        // 按照窗口数组里的窗口进行逐个发送信息
    for(int count=0;count<myArray.GetSize();count++)
    {
      CWnd* p=myArray.GetAt(count);
      p->SendMessage(BM_CLICK);
    }
        // 清空数组
    myArray.RemoveAll();
        // 将 m_edit 清空，因为下一次您要写上您想发送的话
    SetDlgItemText(IDC_EDIT1,"");
    m_edit="";
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
