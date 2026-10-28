
# 给LordPE查看输入表部分加上搜索功能

2005.11.

1. 稍改进了下LordPE查看输入表部分的搜索功能

2. 给LordPE查看输入表部分加右键菜单(仅复制`ThunkRVA`/`FirstThunk`列),本人比较懒,得到`FirstThunk`值时喜欢用鼠标复制粘贴.

3. 当点击LordPE查看输入表部分中"View always `FirstThunk`",保持光条在原来位置.(LordPE默认会将光条置到0行)

4. 修改FLC(File Location Calulator)窗口中各个文本框(`VA`, `RVA`, `Offset`)为只读属性,此时可以用鼠标复制里面的文本.(LordPE原来是将文本框禁止变灰,此时不可复制)

![](http://bbs.pediy.com/upload/2005/4/image/lordpef3.gif)

![](http://bbs.pediy.com/upload/2005/4/image/lordpef3-flc.gif)

[附件:`lordpe.rar`](https://bbs.kanxue.com/elink@b0dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6b7c8g2)9#2k6Y4c8G2L8$3I4K6i4K6u0r3e0r3!0J5k6s2m8W2i4K6u0r3e0q4m8q4i4K6u0V1c8p5I4j5i4K6u0W2h3V1W2b7)

附过程及`LordPlug.dll`源码:

```c
/*-------------------------------------------------
   LordPlug.dll -- LordPE功能扩展

               (c)cffK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0 by kanxue 2005.10.10

  -------------------------------------------------*/
#include <Windows.h>
#include <commctrl.h>

#define MAXINPUTLEN 512

//(1) 给LordPE查看输入表部分加上搜索功能

void _cdecl Searchimport(const DWORD reversed, HWND hWnd)
{
        try
        {
                TCHAR cSearchBuffer[MAXINPUTLEN]={0};
                TCHAR cItem[512]={0};
                int listcount,nSearchLength,iItem;
                LVITEM lvitem;
                HWND hWinList;
                POINT Point;

                nSearchLength=GetDlgItemText(hWnd,10222,cSearchBuffer,sizeof(cSearchBuffer)/sizeof(TCHAR)+1);
                if(nSearchLength==0){
                        return;
                }

                hWinList=GetDlgItem(hWnd,2202);
                listcount=SendMessage(hWinList, LVM_GETITEMCOUNT, 0, 0);
                if(0==listcount)
                        return;

                //iItem=SendMessage(hWinList,LVM_GETNEXTITEM,-1,LVNI_FOCUSED);        //得到所选的行
                iItem=0;
                do{
                        iItem++;
                        if(iItem==listcount)
                                break;

                        lvitem.cchTextMax=512;
                        lvitem.iSubItem=4;
                        lvitem.pszText=cItem;
                        SendMessage(hWinList, LVM_GETITEMTEXT, (WPARAM)iItem, (LPARAM)&lvitem);
                }
                while(memicmp(cSearchBuffer,cItem,nSearchLength)!=0);

                if(iItem!=listcount){
                        SetDlgItemText(hWnd,2203,cItem);
                        //选中ListView控件中的Item
                        lvitem.state=LVIS_SELECTED | LVIS_FOCUSED;
                        lvitem.stateMask=LVIS_SELECTED | LVIS_FOCUSED;
                        SendMessage(hWinList,LVM_SETITEMSTATE,(WPARAM)iItem, (LPARAM)&lvitem);
                        //滚动窗口
                        SendMessage(hWinList,LVM_GETITEMPOSITION,(WPARAM)iItem,(LPARAM) (POINT*) &Point);
                        SendMessage(hWinList,LVM_SCROLL,0,Point.y-50);
                }
                else
                        SetDlgItemText(hWnd,2203,"没找到指定项目");
                return;
        }
        catch (...)
        {
                return ;
        }
}

/*
0040D473   > \E9 5AAE0000         jmp     004182D2                         ;  Case 111 (WM_COMMAND) of switch 0040D2A7
0040D478      90                  nop
0040D479      90                  nop
0040D47A      90                  nop
0040D47B      90                  nop
0040D47C   .  3D 04400000         cmp     eax, 4004                        ;  Switch (cases 89C..5002)
004182D2      8B4424 18           mov     eax, [esp+18]
004182D6      25 FFFF0000         and     eax, 0FFFF
004182DB      3D EF270000         cmp     eax, 27EE
004182E0      74 3E               je      short 00418320 //是否操作了搜索框
004182E2      68 7CD44000         push    0040D47C
004182E7      C3                  retn
00418320   > \E8 0E000000   call    00418333
00418325   .  4C 6F 72 64 5>ascii   "LordPlug.dll",0
00418332   .  00            ascii   0
00418333   $  FF15 C0904100 call    [<&KERNEL32.LoadLibraryA>]       ; \LoadLibraryA
00418339   .  E8 11000000   call    0041834F
0041833E   .  53 65 61 72 6>ascii   "Searchimport",0
0041834B   .  00            ascii   0
0041834C   .  00            ascii   0
0041834D   .  00            ascii   0
0041834E   .  00            ascii   0
0041834F   $  50            push    eax                              ; |hModule
00418350   .  FF15 68914100 call    [<&KERNEL32.GetProcAddress>]     ; \GetProcAddress
00418356   .  83C4 0C       add     esp, 0C
00418359   .  FFD0          call    eax                             //调用LordPlug.dll中的Searchimport()函数
0041835B   .  83EC 0C       sub     esp, 0C
0041835E   .  68 7CD44000   push    0040D47C
00418363   .  C3            retn                                     ;  RET 用来作为跳转到 0040D47C
*/

/******************************************************************************************************************************/
//(2) 给LordPE查看输入表部分加右键菜单(仅复制ThunkRVA/FirstThunk列)

void _cdecl  PopMenuCopy(const DWORD reversed, HWND hWnd)
{
         try
         {
                 int iItem;
                 LVITEM lvitem;
             TCHAR cItem[512]={0};
                 HWND hWinList;
                 HGLOBAL clipbuffer;
                 char * buffer=0;

                 hWinList=GetDlgItem(hWnd,2202);
                 iItem=SendMessage(hWinList,LVM_GETNEXTITEM,-1,LVNI_FOCUSED);        //得到所选的行
                 if(-1==iItem)
                         return;

                 lvitem.cchTextMax=512;
                 lvitem.iSubItem=0;
                 lvitem.pszText=cItem;
                 SendMessage(hWinList, LVM_GETITEMTEXT, (WPARAM)iItem, (LPARAM)&lvitem);

                 //将取得文本放到剪贴簿
                 if(OpenClipboard(NULL))
                 {
                         EmptyClipboard();
                         clipbuffer = GlobalAlloc(GMEM_DDESHARE,lstrlen(cItem)+1);
                         if(clipbuffer==NULL) return;

                         buffer = (char*)GlobalLock(clipbuffer);
                         strcpy(buffer, LPCSTR(cItem));
                         GlobalUnlock(clipbuffer);
                         if(SetClipboardData(CF_TEXT,clipbuffer)!=0)
                                 clipbuffer = NULL;
                         CloseClipboard();
                         if (clipbuffer) GlobalFree(clipbuffer);
                 }
                  return;
         }
         catch (...)
         {
                 return ;
         }
}

/*
0040D6C4  |.  68 01500000   push    5001                             ; |ItemID = 5001 (20481.)
0040D6C9  |.  6A 00         push    0                                ; |Flags = MF_BYCOMMAND|MF_ENABLED|MF_STRING
0040D6CB  |.  50            push    eax                              ; |hMenu
0040D6CC  |.  A3 34E14100   mov     [41E134], eax                    ; |
0040D6D1  |.  FFD6          call    esi                              ; \AppendMenuA
0040D6D3  |.  A1 34E14100   mov     eax, [41E134]
0040D6D8  |.  68 7CBE4100   push    0041BE7C                         ; /pItem = "refresh"
0040D6DD  |.  68 02500000   push    5002                             ; |ItemID = 5002 (20482.)
0040D6E2  |.  6A 00         push    0                                ; |Flags = MF_BYCOMMAND|MF_ENABLED|MF_STRING
0040D6E4  |.  50            push    eax                              ; |hMenu => 002E0C87
0040D6E5  |.  FFD6          call    esi                              ; \AppendMenuA
00418370      68 80A64100   push    0041A680                         ;  ASCII "copy..."
00418375      68 00500000   push    5000
0041837A      6A 00         push    0
0041837C      50            push    eax
0041837D      A3 34E14100   mov     [41E134], eax
*/
```
