

# 菜鸟注册机编写基础教程（六）完

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-227.htm`](https://www.pediy.com/kssd/pediy08/pediy8-227.htm)

六、效果篇  
  
    经常看到一些注册机的界面很炫，有的还带有音乐，这篇我们就来讨论如何编写炫一点的注册机，为注册机添加背景图片，网址链接以及背景音乐。  
一、  加入地址链接。  
比如我在注册机中加入我的 EMAIL 地址链接，要用到 WIN32 的 API 函数 ShellExecuteA。  
VB 中实现方法，先声明 API 函数：  
Private Declare Function ShellExecute Lib "shell32.dll" Alias "ShellExecuteA" (ByVal hWnd As Long, ByVal lpOperation As String, ByVal lpFile As String, ByVal lpParameters As String, ByVal lpDirectory As String, ByVal nShowCmd As Long) As Long  
调用：  
Call ShellExecute(Form1.hWnd, "Open", "mailto:langxang@126.com", "", App.Path, 1)  
同样，想加入网站链接的话：  
Call ShellExecute(Form1.hWnd, "open", "http://bbs.pediy.com", vbNullString, vbNullString, &H0)  
DELPHI 有点类似，利用 ShellAPI 单元中的 ShellExecute，在 uses 句中加入 ShellAPI。  
ShellExecute(handle,nil,pchar(′mailto:langxang@126.com′),nil,nil,SW\_SHOWNORMAL);  
VC 也是用 ShellExecute，比较麻烦：   
生成一个新的对话框。用 AppWizard 建立一个基于通用窗口的新类 CHyperLinkWnd，并修改代码如下：  
　　CHyperLinkWnd::CHyperLinkWnd() //构造函数  
　　{ m\_hLinkCursor = AfxGetApp()-〉LoadCursor(IDC\_LINK); //引入手指形状的光标 }  
　　void CHyperLinkWnd::OnMouseMove(UINT nFlags, CPoint point)  
　　{ ::SetCursor(m\_hLinkCursor);//当鼠标在窗口上移动，光标变成手指状  
　　 CWnd::OnMouseMove(nFlags, point);}  
　　BOOL CHyperLinkWnd::Create( const RECT&& rect, CWnd pParentWnd)　　//创建窗口  
　　{ return CWnd::Create ( NULL, NULL, WS\_CHILD|WS\_VISIBLE, rect, pParentWnd,NULL,NULL);}  
　　BOOL CHyperLinkWnd::SetLinkString(CString m\_LinkString)  
　　{ LinkString　=m\_LinkString;　　//设置窗口的提示信息  
　　 return TRUE; }  
　　void CHyperLinkWnd::SetLink(CString m\_Link)  
　　{ Link=m\_Link; //设置链接网址信息 }  
　　void CHyperLinkWnd::OnLButtonDown(UINT nFlags, CPoint point)  
　　{ ::ShellExecute(this-〉m\_hWnd,"open",Link,NULL,NULL,SW\_SHOWMAXIMIZED); }  
　　void CHyperLinkWnd::OnPaint()  
　　{ CFont Font=NULL;  
　　CPaintDC dc(this); //设置超级链接中的显示字体信息，即蓝色有下划线  
　　 dc.SetTextColor(RGB(0, 0, 255));  
  
　　 dc.SetBkColor(::GetSysColor(COLOR\_3DFACE));  
　　 m\_font=dc.GetCurrentFont();  
　　 LOGFONT lFont;  
　　 m\_font-〉GetObject(sizeof(lFont), &&lFont);  
　　 m\_font=NULL;  
　　 m\_font=new CFont();  
　　 lFont.lfUnderline=TRUE;  
　　 m\_font-〉CreateFontIndirect(&&lFont);  
　　 dc.SelectObject(m\_font);  
　　 dc.TextOut(0,0,LinkString);}  
//使用这个类的一个范例，即在一个对话框中使用这些超级链接  
　　BOOL CNetPrice::OnInitDialog()  
　　{ RECT rect;  
　　HyperLinkWnd\[0\]=new CHyperLinkWnd(); //使用这个类，可以先在对话框中用一个 CStatic 框定位，并去掉它的 Visible 属性，然后用代码取得它的位置，并将超级链接窗口放上去  
　　m\_Canton.GetWindowRect(&&rect);  
　　this-}ScreenToClient(&&rect);  
　　HyperLinkWnd\[0\]-}SetLinkString("广州太平洋电脑城");  
　　HyperLinkWnd\[0\]-}SetLink("www.pconline.com.cnt");  
　　HyperLinkWnd\[0\]-}Create(rect,this);  
　　……//略掉部分重复代码  
　　HyperLinkWnd\[6\]=new CHyperLinkWnd();//下面这个是使用邮件链接的例子  
　　m\_SendEMail.GetWindowRect(&&rect);  
　　this-}ScreenToClient(&&rect);  
　　HyperLinkWnd\[6\]-}SetLinkString("发邮件");  
　　HyperLinkWnd\[6\]-}SetLink("MailTo:langxang@126.com");  
　　HyperLinkWnd\[6\]-〉Create(rect,this);  
　　return TRUE;}  
  
二、  滚动效果  
滚动的原理都一样，就是将滚动条字幕的宽度与窗体的宽度对比，一旦超出范围又重新回到初始位置。  
VB 滚动字幕实例：  
l 在 Form1 中的控件：  
（1）Label1  
Caption="滚动字幕"  
Autosize=True  
 （2）为构成一种阴影效果，再复制一个 Label2，调整 Label2 的位置使其比 Label1 略高，并将它的文字颜色改得淡一些（设置为蓝色或红色）。  
（3）Timer1  
Interval=100 （0.1 秒）  
l 程序代码：  
Private Sub Form\_Load()  
Label1.ForeColor = RGB(128, 128, 128) '文字阴影设为深灰色  
Label2.ForeColor = RGB(255, 0, 255) '文字颜色设为粉红色  
Label1.Left = Width + 50 '文字的初始位置：在窗体右端外  
Label2.Left = Width  
End Sub  
  
Private Sub Timer1\_Timer()  
If Label1.Left < 0 - Label1.Width Then '如果文字移到窗体左端之外，  
Label1.Left = Width + 50 '则将它的位置立即移回到初始位置  
Label2.Left = Width  '又进入窗体  
End If  
Label1.Left = Label1.Left - 100 '否则文字位置向左移 100 个单位  
Label2.Left = Label2.Left - 100   
End Sub  
  
如果要改进滚动效果，使文字的左面一部分从窗体左端出去以后，立即从右端进入窗体，则可将程序代码作如下修改：  
Private Sub Form\_Load()  
Label1.ForeColor = RGB(255, 0, 255) '取消阴影，使两段文字完全相同  
Label2.ForeColor = RGB(255, 0, 255)  
Label1.Left = 0 '一段文字放在最左端  
Label2.Left = -Width '一段文字放在最左端的外面，与 Label1  
End Sub '相差一个窗体距离  
  
Private Sub Timer1\_Timer()  
Label1.Left = Label1.Left - 50 '两段文字同时向左移  
Label2.Left = Label2.Left - 50   
If Label1.Left <= -Width Then '当文字在窗体外再走完一个窗体的距离时，  
Label1.Left = Width '又进入窗体  
End If  
If Label2.Left <= -Width Then  
Label2.Left = Width  
End If  
End Sub  
  
DELPHI：  
procedure TForm1.Timer1Timer(Sender: TObject);  
begin  
 If Label1.Left < 0 – Label1.Width Then  
   begin  
Label1.Left := Panel1.Width ;  
   end  
 else  
Label1.Left := Label1.Left - 1 ;  
end;  
  
VC：  
在状态条中显示滚动文本。  
1. 从 CCstatusBar 派生 MyStatusBar。  
2. 在 MainFrm.h 中，改变状态条的类型：   
CMyStatusBar m\_wndStatusBar;  
3. 在 MainFrm.cpp 中，改变 indicator 数组：  
static UINT indicators\[\] =  
{  
ID\_SEPARATOR,   
IDS\_SCROLL\_PANE,//scrolling text  
ID\_INDICATOR\_CAPS,  
ID\_INDICATOR\_NUM,  
ID\_INDICATOR\_SCRL,  
};  
4. 在资源编辑器中，增加 IDS\_SCROLL\_PANE 字符串，其内容为若干空格。窗格的大小由要显示的滚动文本来决定。   
5. 增加下面成员到 CMyStatusBar: Cstring m\_strScrollText;   
6. 增加 Add OnTimer()到 CMyStatusBar:   
void CMyStatusBar::OnTimer(UINT nIDEvent)   
{  
// TODO: Add your message handler code here and/or call default  
if (m\_strScrollText.IsEmpty())  
{  
KillTimer(1);  
SetPaneText(CommandToIndex(IDS\_SCROLL\_PANE), "");  
return;  
}  
static UINT str\_idx = 0;        //offset into string  
//if end of string, return to top  
if (str\_idx >= (UINT) (m\_strScrollText.GetLength() / 2) - 1)  
{  
str\_idx = 0;  
}  
//display string  
SetPaneText(CommandToIndex(IDS\_SCROLL\_PANE), ((LPCSTR)   
m\_strScrollText)+str\_idx);   
//scroll one character  
str\_idx = str\_idx + 1;  
CStatusBar::OnTimer(nIDEvent);  
}  
7. 撤除定时器：   
  
void CMyStatusBar::OnDestroy()   
{  
CStatusBar::OnDestroy();  
// TODO: Add your message handler code here  
KillTimer(1);  
}  
8. 增加显示滚动文本的方法，该方法必须在主框架窗口创建后调用，通常在 CWinApp::InitInstance()中调用：  
void CMyStatusBar::StartDisplay(void)   
{  
//set text for scrolling  
m\_strScrollText = "   Hello! World.   "  
//to make it circular scroll  
m\_strScrollText += m\_strScrollText;  
KillTimer(1);  
VERIFY(SetTimer(1, 200, NULL) != 0);    //timer  
}  
三、  添加图片  
添加位图没什么好说的，都可以直接导入。VC 稍微麻烦点，可以在 insert resource 中选 import bitmap，在导入资源对话框中选择文件类型为所有文件（\*.\*），打开方式为 AUTO，然后选择要导入的 BMP 图片既可。  
四、  背景音乐  
    XM 是一种比较小的音乐，非常适合做注册机音乐。论坛中也有很多相关的文章，尤其是其中的 uFMOD v1.08 工具，非常好用，自动转化 XM 文件为各种程序的 OBJ 文件。eff.exe 工具可以直接转 XM 音乐为 DELPHI 的 PAS 文件，和 C 语言的.C 文件以及汇编的.ASM 文件，还生成 eff.inc 库文件。可以直接到论坛下载该工具，里面都有源码和说明。  
另外，论坛会员 tankaiha 写的文章“在 VC6 的工程中加入播放 xm 音乐的功能”讲的非常详细，可以对照那个在 MFC 中加入音乐。  
附件中是我做的三种语言的最终编译后的效果，大家可以看一下。  
  
  
【版权声明】:  本文由 langxang 原创于看雪论坛，转载请注明作者并保持文章的完整, 谢谢!

 4dd466d0e1e0cc123afbe504da2462ca