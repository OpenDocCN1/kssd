
# Exe 程序尾部附加数据添加程序(含源码)

> 原文：[`www.pediy.com/kssd/pediy11/117209.html`](https://www.pediy.com/kssd/pediy11/117209.html)

![](img/cda69129882b40377a87144c782680e1.jpg "微笑")

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

〖软件说明〗  

添加 EXE 尾部附加数据 No 1.0  

by zhanshen\[DFCG\]\[RCT\]  

这个程序是在我处理一个有附加数据程序时有感而写的。原来我在处理 Exe 程序的尾部附加程序时，一般都是先用 `WinHex` 或其它 Hex 十六进制编辑器打开原程序，再将附加数据拷出来写入到目标程序中，一次二次还好，n 次以上就觉得烦了。所以就写下了一个程序自动添加尾部附加数据。  

这个程序操作很简单：选择两个文件，然后点击【开始写入】按钮就行了。这个程序除了可以直接输入文件名和浏览文件取得程序文件名外，还支持直接将文件拖到文本框中。  

这个程序的写入方式有：  

1、选取两个 exe 文件，一个要写入附加数据的目标 exe 文件，一个原 有附加数据的 exe 文件，该软件可以直接取出附加数据添加到目标程序中。  

2、选取目标 exe 文件，另一个选择一个\*.bin 文件，该程序将会将这个整个\*.bin 文件直接写入目标 exe 程序的尾部。  

3、其它的方式就由用户自行摸索（众：好像没有了吧？![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")作者：-\_-!，转身偷逃。![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")众：哪里跑![](img/7eab1bc95cbb1be9f15eb01bf012328f.jpg "狂怒")）  

附：本软件为开源免费软件，用户可以任意使用本软件，任何人均可从网上等地方获取到本软件的副本及其源码。用户如若转载，请保持本文档的完整，谢谢！本软件作者只为测试，任何人均不得将程序用于非法之处，否则由此引起 的问题一概由用户自负。  

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

![](img/9c213356ea332de873102f4486c3b5b7.jpg)  

**附件下载**  

`WriteOverlay.rar`  

```
/* WriteOverlay.cpp file */

#include "WriteOverlay.h"

int WINAPI WinMain(IN HINSTANCE hInstance,
             IN HINSTANCE hPrevInstance,
             IN LPSTR lpCmdLine,
             IN int nShowCmd )
{
  g_hInstance = hInstance;
  g_lpszCmdLine = lpCmdLine;
  DialogBox(hInstance,(char *)DLG_MAIN,0,(DLGPROC)MainDlgProc);
  return 0;
}

LRESULT CALLBACK MainDlgProc(HWND hDlg, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
  HWND l_hEdit = 0;
  OPENFILENAME l_stOFN;
  void *l_pBuff = NULL;
  int l_nStrLen;
  HANDLE l_hWriteFile;
  char l_szFileName[CHAR_MAX],l_szPatchFileName[CHAR_MAX],l_szOverlayFileName[CHAR_MAX];
  switch(uMsg)
  {
  case WM_INITDIALOG:
    g_hDlgMain = hDlg;
    g_hMainIcon = LoadIcon(g_hInstance,(char *)ICON_MAIN);
    SendMessage(hDlg,WM_SETICON,ICON_SMALL,(LPARAM)g_hMainIcon);
    l_hEdit = GetDlgItem(hDlg,EDIT_PATCH_TO_EXE);
    SetWindowLong(l_hEdit,GWL_USERDATA,GetWindowLong(l_hEdit,GWL_WNDPROC));
    SetWindowLong(l_hEdit,GWL_WNDPROC,(UINT)EditsProc);
    l_hEdit = GetDlgItem(hDlg,EDIT_DATA_TO_PATCH);
    SetWindowLong(l_hEdit,GWL_USERDATA,GetWindowLong(l_hEdit,GWL_WNDPROC));
    SetWindowLong(l_hEdit,GWL_WNDPROC,(UINT)EditsProc);
    g_hHandCur = LoadCursor(g_hInstance,(char *)CUR_HAND);
    l_hEdit = GetDlgItem(hDlg,EDIT_PRINTS);
    g_hPrintfEdit = l_hEdit;
    StringAdd(l_hEdit,lpszCopyRight);
    break;
  case WM_COMMAND:
    switch((WORD)wParam)
    {
    case BTN_PATHC_TO_EXE_OPEN:
      InitOpenFileName(0,hDlg,&l_stOFN,l_szFileName,sizeof(l_szFileName));
      l_stOFN.lpstrFilter = "exe files(\*.exe)\\0\*.exe\\0all files(\*.\*)\\0\*.\*\\0\\0";
      if (GetOpenFileName(&l_stOFN))
        SetDlgItemText(hDlg,EDIT_PATCH_TO_EXE,l_szFileName);
      break;
    case BTN_DATA_TO_PATHC_OPEN:
      InitOpenFileName(0,hDlg,&l_stOFN,l_szFileName,sizeof(l_szFileName));
      l_stOFN.lpstrFilter = "exe files(\*.exe)\\0\*.exe\\0bin files(\*.bin)\\0\*.bin\\0all files(\*.\*)\\0\*.\*\\0\\0";
      if (GetOpenFileName(&l_stOFN))
        SetDlgItemText(hDlg,EDIT_DATA_TO_PATCH,l_szFileName);
      break;
    case BTN_START_WRITE:
      if (!g_IsStartWrite)
      {
        SetWindowText(g_hPrintfEdit,"");
        g_IsStartWrite = TRUE;
      }
      ZeroMemory(&l_szPatchFileName,sizeof(l_szPatchFileName));
      ZeroMemory(&l_szOverlayFileName,sizeof(l_szOverlayFileName));
      if (!GetDlgItemText(hDlg,EDIT_PATCH_TO_EXE,l_szPatchFileName,sizeof(l_szPatchFileName)))
      {
        StringAdd(g_hPrintfEdit,"错误!，要添加尾部附加数据这一栏不能为空!!\\r\\n");
        return TRUE;
      }
      if (!GetDlgItemText(hDlg,EDIT_DATA_TO_PATCH,l_szOverlayFileName,sizeof(l_szOverlayFileName)))
      {
        StringAdd(g_hPrintfEdit,"错误!，要写入的尾部数据这一栏不能为空!!\\r\\n");
        return TRUE;
      }
      if (StartWriteOverlay(l_szPatchFileName,l_szOverlayFileName))
        StringAdd(g_hPrintfEdit,"Exe 尾部附加数据写入成功!!\\r\\n");
      else
        StringAdd(g_hPrintfEdit,"写入失败!!，详细信息请看信息输出窗口\\r\\n");
      break;
    case BTN_SAVE_LOG:
      if (!g_IsStartWrite)
      {
        StringAdd(g_hPrintfEdit,"信息窗口内容为空，不用保存!\\r\\n");
        return TRUE;
      }
      ZeroMemory(&l_szFileName,sizeof(l_szFileName));
      InitOpenFileName(0,hDlg,&l_stOFN,l_szFileName,sizeof(l_szFileName));
      l_stOFN.lpstrFilter = "txt files(\*.txt)\\0\*.txt\\0all files(\*.\*)\\0\*.\*\\0\\0";
      if (GetSaveFileName(&l_stOFN))
      {
        l_nStrLen = StringAdd(g_hPrintfEdit,"");
        l_pBuff = malloc(l_nStrLen+4);
        ZeroMemory(l_pBuff,l_nStrLen+4);
        l_nStrLen = GetWindowText(g_hPrintfEdit,(char *)l_pBuff,l_nStrLen+4);
        l_hWriteFile = MyCreateFile(l_szFileName,OPEN_ALWAYS);
        if (l_hWriteFile == INVALID_HANDLE_VALUE)
        {
          StringAdd(g_hPrintfEdit,"创新或打开文件失败!!\\r\\n");
          free(l_pBuff);
          return TRUE;
        }
        SetFilePointer(l_hWriteFile,0,0,FILE_END);
        if (MyWriteFile(l_hWriteFile,l_pBuff,l_nStrLen))
          StringAdd(g_hPrintfEdit,"保存文件成功!!\\r\\n");
        else
          StringAdd(g_hPrintfEdit,"保存文件失败!!\\r\\n");
        CloseHandle(l_hWriteFile);
        free(l_pBuff);
        return TRUE;
      }
      break;
    case BTN_ABOUT:
      DialogBox(g_hInstance,(char *)DLG_ABOUT,hDlg,(DLGPROC)AboutDlgProc);
      break;
    case BTN_EXIT:
      PostMessage(hDlg,WM_CLOSE,0,NULL);
      break;
    }
    break;
  case WM_CLOSE:
    EndDialog(hDlg,FALSE);
    break;
  default:
    return FALSE;
  }
  return TRUE;
}

LRESULT CALLBACK EditsProc(HWND hEdit,UINT uMsg, WPARAM wParam, LPARAM lParam)
{
  int l_uNumFiles = 0;
  char l_szFilesName[CHAR_MAX];
  int i;
  switch(uMsg)
  {
  case WM_DROPFILES:
    l_uNumFiles = DragQueryFile((HDROP)wParam,-1,l_szFilesName,sizeof(l_szFilesName));
    for (i =0; i < l_uNumFiles; i++)
    {
      DragQueryFile((HDROP)wParam,i,l_szFilesName,sizeof(l_szFilesName));
      SetWindowText(hEdit,l_szFilesName);
    }
    DragFinish((HDROP)wParam);
    break;
  default:
    return CallWindowProc((WNDPROC)GetWindowLong(hEdit,GWL_USERDATA),hEdit,uMsg,wParam,lParam);
  }
  return TRUE;
}

LRESULT CALLBACK StaticProc(HWND hStatic, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
  switch(uMsg)
  {
  case WM_NCHITTEST:
    return (LRESULT)TRUE;
    break;
  case WM_SETCURSOR:
    SetCursor(g_hHandCur);
    break;
  case WM_LBUTTONUP:
    switch(GetDlgCtrlID(hStatic))
    {
    case STATIC_BBS_PEDIY:
      ShellExecute(0,NULL,"http://bbs.pediy.com",NULL,NULL,0);
      break;
    case STATIC_BBS_DEBUGMAN:
      ShellExecute(0,NULL,"http://www.debugman.com",NULL,NULL,0);
      break;
    }
    break;
  default:
}
```
