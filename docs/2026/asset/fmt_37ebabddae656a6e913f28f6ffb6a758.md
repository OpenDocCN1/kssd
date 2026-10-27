
```
BOOL CSectionFlagsDlg::OnCommand(WPARAM wParam, LPARAM lParam)
{
    // TODO: 在此添加专用代码和/或调用基类
    DWORD wmEvent = 0;
    wmEvent = HIWORD(wParam);

    // 如果是点击事件(有可能是OK按钮的点击)
    if (wmEvent==BN_CLICKED)
    {
        WORD wmId      = LOWORD(wParam);
        // 获取单选框控件的状态
        BOOL bIsCheck = ::SendMessage(::GetDlgItem(this->m_hWnd,wmId),BM_GETCHECK,0,0);

        // 在这里做一次判断，看资源ID号是否在复选框ID号之间。因为BN_CLICKED会响应所有的点击
        if (wmId>= IDC_CHECK1 && wmId<=IDC_CHECK15)
        {
            // 得到点击的复选框的索引值
            DWORD dwIndex  = wmId-IDC_CHECK1;
            DWORD dwCurVal = 0;

            CString strCurVal;
            m_editCurrentVal.GetWindowText(strCurVal);
            swscanf_s(strCurVal,L"%x",&dwCurVal);

            // 根据是否选中，对标志进行加或减
            if (bIsCheck)
            {
                dwCurVal += g_dwFlagsAry[dwIndex];
            }
            else
            {
                dwCurVal -= g_dwFlagsAry[dwIndex];
            }

            // 改变文件缓存中的值
            m_pSecHeader->Characteristics = dwCurVal;

            strCurVal.Format(_T("%08X"),dwCurVal);
            m_editCurrentVal.SetWindowText(strCurVal);
        }
    }
    return CDialogEx::OnCommand(wParam, lParam);
}
```

Current Value 这个值在传入时会有默认值需要对响应的复选框进行勾选

```
CString strTemp;
strTemp.Format(_T("%08X"),m_pSecHeader->Characteristics);
m_editCurrentVal.SetWindowText(strTemp);

for (DWORD j=0;j<15;j++)
{
     if (g_dwFlagsAry[j] & m_pSecHeader->Characteristics)
     {
          // 设置选中状态
          ::SendMessage(::GetDlgItem(this->m_hWnd,IDC_CHECK1+j),BM_SETCHECK,BST_CHECKED,0);
     }
}
```

### 下拉框的设置

在设置下拉框时，对`Sort`属性要设置为`false`，否则控件会按照字符串进行排序，而不是插入顺序

初始化下拉框

```
    DWORD dwShow = 1;
    CString strShow = _T("默认");
    m_cbAlignment.AddString(strShow);

    m_cbAlignment.SetCurSel(0);
    strShow.Format(_T("%d"),dwShow);
    m_cbAlignment.AddString(strShow);

    for (DWORD j=1;j<14;j++)
    {
        dwShow *= 2;// 下拉框代表的节对齐大小
        strShow.Format(_T("%d"),dwShow);
        m_cbAlignment.AddString(strShow);
    }
```

响应下拉框change消息:

```
int           j  = m_cbAlignment.GetCurSel();
DWORD dwVal  = m_pSecHeader->Characteristics;
CString  strTemp;

// 清空原来位置的值
dwVal  = dwVal&(IMAGE_SCN_ALIGN_MASK^-1);
dwVal  += g_dwAlignmentAry[j];

m_pSecHeader->Characteristics = dwVal;
strTemp.Format(_T("%08X"),dwVal);
m_editCurrentVal.SetWindowText(strTemp);
```

### 列表控件（`ListCtrl`）

```
m_listSecInfo.SetExtendedStyle(LVS_EX_FULLROWSELECT|LVS_EX_GRIDLINES);
```

使用这个函数可以设置控件的扩展属性，上面的宏代表网格和选一行。这个属性在属性编辑时是找不到的。

### 列表右键弹出菜单

有两种方法可以实现，第一种是子类化`ListCtrl`控件，第二种则是在父窗口响应右键点击事件。

第二种方法

在类上添加两个成员变量

```
CMenu m_menu,*m_pSub;
```

在对话框的初始化函数中

```
m_menu.LoadMenu(IDR_MENU1); // IDR_MENU1 弹出菜单的资源号
m_pSub = m_menu.GetSubMenu(0);
```

在对话框中右键点击列表控件，选择添加事件处理程序

![](https://bbs.kanxue.com/upload/attach/201312/586874_e9fbbf8e326ef6fd.png)

在底部还有对消息的说明，`NM_CLICK`代表控件内点击鼠标左键。在事件处理程序中添加下列代码

```
int nIndex = m_listSecInfo.GetSelectionMark();
m_pSel = g_objPe.m_vecPSection[nIndex];

CPoint   point;
GetCursorPos(&point);
m_pSub->TrackPopupMenu(TPM_LEFTALIGN|TPM_RIGHTBUTTON,point.x,point.y,this,NULL);
```

然后对菜单项添加事件处理程序，注意添加要添加对话框到类上，这里默认会是这个，不要急着点确定

![](https://bbs.kanxue.com/upload/attach/201312/586874_12181eae2ee43366.png)

然后在响应程序添加处理代码。

### 总结

通过写PE编辑器我对PE文件理解更进一步，也发现了一些自己在编程方面的不足，一些看似简单的东西，往往有很多东西需要挖掘。印证了古人的一句话"纸上得来终觉浅，绝知此事要躬行"。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [LoadPe.rar](javascript:void(0);)

  （399.53kb，113次下载）
