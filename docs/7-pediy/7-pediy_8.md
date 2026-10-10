

# 〈〈一分钟测试〉〉软件破解补丁分析和 C#.NetSDK 源代码

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-290.htm`](https://www.pediy.com/kssd/pediy08/pediy8-290.htm)

<<一分钟测试>>软件是测试 CPU，内存、磁盘、Direct 等系统性能.是居家装机之良品.  
  
它是由 Visual C++编译的.该程序在处理算法方面在做了个小手脚(以前论坛有分析过类似的手脚),就是把算法处理程序注册成消息处理句柄,消息 ID 定义为 1403,再通过 SendMessage 等消息处理 API 调用消息回调函数而进行算法计算.  
还有一点点：注册算法成功与否还于该对象的一个实例成员 bool 变量\[ecx+1A0\]有约定：1)当 bool \[ecx+1A0\]=0 注册错误.2)当 bool \[ecx+1A0\]=1 注册成功.  
如果顺利成功的话,程序把注册信息放到一个 inf 后缀的名的文件中.每一次启动程序的时候都核对注册信息.见 C# SDK 补丁源代码.该算法全局变量太多了,时间又不多,直接来个补丁省时又省力,又不废脑细胞:=).  
\----qiweixue

附件下载  
  
进入 MFC 消息回调函数体 BOOL CWnd::OnWndMsg(UINT message, WPARAM wParam, LPARAM lParam, LRESULT\* pResult)  
  
0042A2B0  /$  B8 B0AA4300   mov     eax, 0043AAB0 ---新 SEH 链处理句柄  
0042A2B5  |.  E8 9EE6FEFF   call    00418958----------->构造函数 SEH 机制  
0042A2BA  |.  83EC 54       sub     esp, 54  
0042A2BD  |.  8365 F0 00    and     dword ptr \[ebp-10\], 0  
0042A2C1  |.  53            push    ebx  
0042A2C2  |.  8B5D 08       mov     ebx, \[ebp+8\]  
0042A2C5  |.  56            push    esi  
0042A2C6  |.  57            push    edi  
0042A2C7  |.  81FB 11010000 cmp     ebx, 111  ---->比较处理 WM\_COMMAND 消息                      
0042A2CD  |.  8BF9          mov     edi, ecx  
0042A2CF  |.  75 18         jnz     short 0042A2E9  
0042A2D1  |.  FF75 10       push    dword ptr \[ebp+10\]               
0042A2D4  |.  8B07          mov     eax, \[edi\]  
0042A2D6  |.  FF75 0C       push    dword ptr \[ebp+C\]  
0042A2D9  |.  FF50 78       call    \[eax+78\]----------------->进入 OnCommand(wParam, lParam)  
0042A2DC  |.  85C0          test    eax, eax  
0042A2DE  |.  0F84 55010000 je      0042A439  
0042A2E4  |.  E9 1D040000   jmp     0042A706  
0042A2E9  |>  83FB 4E       cmp     ebx, 4E------->---->比较处理 WM\_NOTIFY 消息   
0042A2EC  |.  75 28         jnz     short 0042A316  
0042A2EE  |.  8B45 10       mov     eax, \[ebp+10\]                      
0042A2F1  |.  8338 00       cmp     dword ptr \[eax\], 0  
0042A2F4  |.  0F84 3F010000 je      0042A439  
.  
.  
.  
.......  
直接来到事件处理句柄地址：  
00414319  /.  55            push    ebp  
0041431A  |.  8BEC          mov     ebp, esp  
0041431C  |.  81EC 10010000 sub     esp, 110  
00414322  |.  898D F0FEFFFF mov     \[ebp-110\], ecx  ----> ecx 为 CDialog 对象指针  
00414328  |.  6A 20         push    20  
0041432A  |.  68 C8424500   push    004542C8 ---->004542C8 保存注册油箱的全局变量     
0041432F  |.  8B8D F0FEFFFF mov     ecx, \[ebp-110\]  
00414335  |.  81C1 14010000 add     ecx, 114-->这儿有点意思:ecx+114 是在定位 CDialog 对象容器中的子控件对象!这个 ecx+114 正是控件 EDIT 的对象的指针。  
0041433B  |.  E8 477C0100   call    0042BF87---->这个函数是 CWnd::GetWindowText(String buffer,int Count),跳进入看看,  
...  
0042BF8E  |.  FF7424 08     push    dword ptr \[esp+8\]  --->Count=20h 个字符  
0042BF92  |.  FF7424 08     push    dword ptr \[esp+8\]  --->Buffer=004542C8 保存注册油箱的全局变量  
0042BF96  |.  FF71 1C       push    dword ptr \[ecx+1C\]  ------->\[ecx\]对象是控件 Edit 类,\[ecx+1C\]是 Edit 的句柄 hWnd  
0042BF99  |.  FF15 F4C34300 call    \[<&USER32.GetWindowTextA>\] ---->GetWindowTextA(hWnd,Buffer,Count)  
0042BF9F  |.  EB 12         jmp     short 0042BFB3----函数返回...  
  
继续...  
  
00414340  |.  A3 DC5F4500   mov     \[455FDC\], eax--->\[455FDC\]全部变量保存注册码的字节数量  
00414345  |.  6A 20         push    20  
00414347  |.  68 A83A4500   push    00453AA8                          
0041434C  |.  8B8D F0FEFFFF mov     ecx, \[ebp-110\]  
00414352  |.  81C1 D8000000 add     ecx, 0D8---------->注意:程序又在定位 CDialog 类的子控件对象,\[ecx+0D8\]同样也是 CEdit 对象,只不过它存放的是用户输入密码,上一个是存放的输入注册编码  
00414358  |.  E8 2A7C0100   call    0042BF87------这个函数也是 CWnd::GetWindowText(String buffer,int Count),在上边给出了函数体。  
0041435D  |.  8945 F8       mov     \[ebp-8\], eax  
00414360  |.  837D F8 18    cmp     dword ptr \[ebp-8\], 18--->密码要大于 0x18 个字符,不然 over 不处理  
00414364  |.  75 09         jnz     short 0041436F--->game over  
00414366  |.  833D DC5F4500>cmp     dword ptr \[455FDC\], 4----->输入注册嘛需要大于 4 个字符.  
0041436D  |.  7D 30         jge     short 0041439F--->game over  
0041436F  |>  68 A08C4400   push    00448CA0   ---->这个变量保存 unicode 字符串："输入错误" 注意而不是报告注册编错误,不是关键跳.                      
00414374  |.  E8 EBD5FFFF   call    00411964     ------->一共有三处调用此处理函数,比较关键.                  
00414379  |.  83C4 04       add     esp, 4---->C 约定调用函数,调用者清理栈.  
  
  
继续...  
用户输入邮箱号码复制 0x40 份保存在全局变量：\[00453AC8\]     
用户输入的假注册号复制 0x40 份保存在全局变量：\[004526A0\]  
分析出来的关键对象：  
\[ecx\]=00475588--->CDialog 类对象  
\[ecx+1C\] m\_hWnd = 003B01B8  
虚拟表指针和成员函数如下：  
00475588  D0 D7 43 00 01 00 00 00 00 00 00 00 00 00 00 00  凶 C............  
00475598  00 00 00 00 01 00 00 00 00 00 00 00 B8 01 3B 00  ...........?;.  
004755A8  00 00 00 00 00 00 00 00 4F E5 D3 77 00 00 00 00  ........O 逵 w....  
004755B8  00 00 00 00 00 00 00 00 00 00 00 00 64 00 00 00  ............d...  
004755C8  64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  d...............  
004755D8  00 00 00 00 00 00 00 00 00 00 00 00 28 00 00 00  ............(...  
004755E8  80 00 00 00 00 00 40 00 64 00 00 00 00 00 00 00  €.....@.d.......  
004755F8  00 00 00 00 3B C6 42 00 88 55 47 00 3A 5B 43 00  ....;艬.圲 G.:\[C.  
00475608  00 00 00 00 C4 94 44 00 00 00 00 00 3F 00 00 00  ....臄 D.....?...  
00475618  00 00 00 00 6B 10 00 00 84 11 00 00              ....k..?..婿  
  
\[ecx+114\]=0047569C --->CDialog 类容器中子 CUserTextEdit 用来保存注册邮箱信息  
\[ecx+1C\] m\_hWnd = 002F01E6  
虚拟表指针和成员函数如下：  
0047569C  5C F8 43 00 01 00 00 00 00 00 00 00 00 00 00 00  \\鳦............  
004756AC  00 00 00 00 01 00 00 00 00 00 00 00 E6 01 2F 00  ...........?/.  
004756BC  00 00 00 00 00 00 00 00 C4 B3 D3 77 00 00 00 00  ........某觲....  
004756CC  00 00 00 00 00 00 00 00 00 00 00 00              ............  
  
  
\[ecx+0D8\]=00475660 --->Dialog 类容器中子 CPassWordEdit 类用来保存注册码信息  
\[ecx+1c\]m\_hWnd = 00030220   
虚拟表指针和成员函数如下：  
00475660  5C F8 43 00 01 00 00 00 00 00 00 00 00 00 00 00  \\鳦............  
00475670  00 00 00 00 01 00 00 00 00 00 00 00 20 02 03 00  ........... .  
00475680  00 00 00 00 00 00 00 00 C4 B3 D3 77 00 00 00 00  ........某觲....  
00475690  00 00 00 00 00 00 00 00 00 00 00 00              ............  
  
    
到这里很有意思,这个程序把算法处理程序注册成消息处理函数了,  
00414412 >|> \\6A 00         push    0         ------------>lparam                        
00414414  |.  6A 00         push    0        --->wparam                          
00414416  |.  68 03140000   push    1403     ------message                         
0041441B  |.  8B8D F0FEFFFF mov     ecx, \[ebp-110\]    ---------->重要对象此对象 ecx 指向 CDialog 类对象,CDialog 类处理算法消息句柄             
00414421  |.  E8 BA0B0000   call    00414FE0   ------->CWnd::SendMessage(message,wparam,lparam) 主角登场了...进去看看,在下文...    
00414426  |.  8B85 F0FEFFFF mov     eax, \[ebp-110\]---->CDialog 对象指针.  
0041442C  |.  83B8 A0010000>cmp     dword ptr \[eax+1A0\], 0 ---关键比较用 CDialog 类对象约定的 bool 实例变量.当为 0,注册错误,为 1 注册正确！这个字段是在回调函数中赋值的！  
00414433  |.  0F84 DB000000 je      <loc\_414514>  
  
  
上边的 CWnd::SendMessage(message,wparam,lparam)调用 Win32 API SendMessageA  
  
  
00414FE0 >/$  55            push    ebp                               
00414FE1  |.  8BEC          mov     ebp, esp  
00414FE3  |.  51            push    ecx  
00414FE4  |.  894D FC       mov     \[ebp-4\], ecx----------->CDialog 类对象指针  
00414FE7  |.  8B45 10       mov     eax, \[ebp+10\]--->lparam=0  
00414FEA  |.  50            push    eax                              
00414FEB  |.  8B4D 0C       mov     ecx, \[ebp+C\]  ----->wparam=0  
00414FEE  |.  51            push    ecx                                
00414FEF  |.  8B55 08       mov     edx, \[ebp+8\]    ------>message=00001403                 
00414FF2  |.  52            push    edx                                
00414FF3  |.  8B45 FC       mov     eax, \[ebp-4\]--->局部变量\[ebp-4\]把 CDialog 对象指针给 eax                      
00414FF6  |.  8B48 1C       mov     ecx, \[eax+1C\]   ------->\[eax+1C\]为 CDialog 对象成员 m\_hWnd                   
00414FF9  |.  51            push    ecx                               
00414FFA  |.  FF15 50C54300 call    \[<&USER32.SendMessageA>\]--->SendMessage(m\_hWnd,message,wparam,lparam)  
00415000  |.  8BE5          mov     esp, ebp  
00415002  |.  5D            pop     ebp  
00415003  \\.  C2 0C00       retn    0C  
  
在上边程序 SendMessage(m\_hWnd,message,wparam,lparam)直接进入 CDialog 对象中,调用注册的消息算法处理句柄,它会直接 Call,而不是放到消息队列中等待.  
  
这里我把 CDialog 类的回调函数直接贴出来：  
00413C80  /.  55            push    ebp  
00413C81  |.  8BEC          mov     ebp, esp  
00413C83  |.  51            push    ecx  
00413C84  |.  894D FC       mov     \[ebp-4\], ecx  
00413C87  |.  817D 08 01140>cmp     dword ptr \[ebp+8\], 1401  
00413C8E  |.  75 1E         jnz     short 00413CAE  
00413C90  |.  68 A08B4400   push    00448BA0                         ; /Arg1 = 00448BA0 ASCII "AboutDlg: --------- Get WM\_ABOUTSURFACELOST!"  
00413C95  |.  E8 66DDFFFF   call    00411A00                         ; \\zwtest.00411A00  
00413C9A  |.  83C4 04       add     esp, 4  
00413C9D  |.  68 88554700   push    00475588  
00413CA2  |.  B9 F05B4700   mov     ecx, 00475BF0  
00413CA7  |.  E8 A62C0200   call    00436952  
00413CAC  |.  EB 17         jmp     short 00413CC5  
00413CAE  |>  817D 08 03140>cmp     dword ptr \[ebp+8\], 1403------->我们需要的消息 handle 到来了...  
00413CB5  |.  75 0E         jnz     short 00413CC5  
00413CB7  |.  E8 69080000   call    00414525-------->这是关键算法 Call...我不打算给出算法分析了,太多了,一会给个补丁很省时间的.  
00413CBC  |.  8B4D FC       mov     ecx, \[ebp-4\]--->CDialog 类对象指针  
00413CBF  |.  8981 A0010000 mov     \[ecx+1A0\], eax----把结果给 CDialog 类成员变量\[ecx+1A0\]保存,\[ecx+1A0\]为 bool 变量,当为 1 的时候注册成功,等于 0 注册错误！  
00413CC5  |>  8B55 10       mov     edx, \[ebp+10\]--->lparam  
00413CC8  |.  52            push    edx                               
00413CC9  |.  8B45 0C       mov     eax, \[ebp+C\] ---->wparam                      
00413CCC  |.  50            push    eax                                
00413CCD  |.  8B4D 08       mov     ecx, \[ebp+8\] -------->message                      
00413CD0  |.  51            push    ecx                                
00413CD1  |.  8B4D FC       mov     ecx, \[ebp-4\]                     
00413CD4  |.  E8 11600100   call    00429CEA      ----->默认消息 CWnd::DefWindowProc(message,wparam,lparam)               
00413CD9  |.  8BE5          mov     esp, ebp  
00413CDB  |.  5D            pop     ebp  
00413CDC  \\.  C2 0C00       retn    0C  
  
\=======================================  
给出 C#.Net SDK 补丁源代码：  
using System;  
using System.Drawing;  
using System.ComponentModel;  
using System.Windows.Forms;  
using System.IO;  
  
namespace Pediy  
{  
    public class PediyForm: System.Windows.Forms.Form  
    {  
        private Button button2;  
        private Label label1;  
        private Label label2;  
        private TextBox textBox1;  
        private TextBox textBox2;  
        private OpenFileDialog openFileDialog1;  
        private LinkLabel linkLabel1;  
        private Button button1;  
        private string drxpath;  
      
        public PediyForm()  
        {  
             
            this.InitializeComponent();  
        }  
  
        private void InitializeComponent()  
        {  
            this.button1 = new System.Windows.Forms.Button();  
            this.button2 = new System.Windows.Forms.Button();  
            this.label1 = new System.Windows.Forms.Label();  
            this.label2 = new System.Windows.Forms.Label();  
            this.textBox1 = new System.Windows.Forms.TextBox();  
            this.textBox2 = new System.Windows.Forms.TextBox();  
            this.openFileDialog1 = new System.Windows.Forms.OpenFileDialog();  
            this.linkLabel1 = new System.Windows.Forms.LinkLabel();  
            this.SuspendLayout();  
               
            this.button1.Location = new System.Drawing.Point(380, 12);  
            this.button1.Name = "button1";  
            this.button1.Size = new System.Drawing.Size(69, 23);  
            this.button1.TabIndex = 0;  
            this.button1.Text = "打开(&O)";  
            this.button1.UseVisualStyleBackColor = true;  
            this.button1.Click += new System.EventHandler(this.button1\_Click);  
              
            this.button2.Location = new System.Drawing.Point(380, 42);  
            this.button2.Name = "button2";  
            this.button2.Size = new System.Drawing.Size(69, 23);  
            this.button2.TabIndex = 1;  
            this.button2.Text = "确定(&T)";  
            this.button2.UseVisualStyleBackColor = true;  
            this.button2.Click += new System.EventHandler(this.button2\_Click);  
              
            this.label1.AutoSize = true;  
            this.label1.Location = new System.Drawing.Point(10, 17);  
            this.label1.Name = "label1";  
            this.label1.Size = new System.Drawing.Size(41, 12);  
            this.label1.TabIndex = 2;  
            this.label1.Text = "文件：";  
             
            this.label2.AutoSize = true;  
            this.label2.Location = new System.Drawing.Point(12, 53);  
            this.label2.Name = "label2";  
            this.label2.Size = new System.Drawing.Size(65, 12);  
            this.label2.TabIndex = 3;  
            this.label2.Text = "状态>>>>>>";  
              
            this.textBox1.Location = new System.Drawing.Point(57, 12);  
            this.textBox1.Name = "textBox1";  
            this.textBox1.Size = new System.Drawing.Size(305, 21);  
            this.textBox1.TabIndex = 4;  
             
            this.textBox2.BackColor = System.Drawing.SystemColors.ControlLight;  
            this.textBox2.Dock = System.Windows.Forms.DockStyle.Bottom;  
            this.textBox2.ForeColor = System.Drawing.SystemColors.Highlight;  
            this.textBox2.Location = new System.Drawing.Point(0, 68);  
            this.textBox2.Name = "textBox2";  
            this.textBox2.ReadOnly = true;  
            this.textBox2.Size = new System.Drawing.Size(461, 21);  
            this.textBox2.TabIndex = 5;  
          
            this.openFileDialog1.ReadOnlyChecked = true;  
            this.openFileDialog1.Title = "<请选择补丁文件>";  
            
            this.linkLabel1.AutoSize = true;  
            this.linkLabel1.Location = new System.Drawing.Point(298, 47);  
            this.linkLabel1.Name = "linkLabel1";  
            this.linkLabel1.Size = new System.Drawing.Size(47, 12);  
            this.linkLabel1.TabIndex = 6;  
            this.linkLabel1.TabStop = true;  
            this.linkLabel1.Text = "关于(&A)";  
            this.linkLabel1.LinkClicked += new System.Windows.Forms.LinkLabelLinkClickedEventHandler(this.linkLabel1\_LinkClicked);  
             
            this.AcceptButton = this.button2;  
            this.ClientSize = new System.Drawing.Size(461, 89);  
            this.Controls.Add(this.linkLabel1);  
            this.Controls.Add(this.textBox2);  
            this.Controls.Add(this.textBox1);  
            this.Controls.Add(this.label2);  
            this.Controls.Add(this.label1);  
            this.Controls.Add(this.button2);  
            this.Controls.Add(this.button1);  
            this.MaximizeBox = false;  
            this.MinimizeBox = false;  
            this.Name = "PediyForm";  
            this.StartPosition = System.Windows.Forms.FormStartPosition.CenterScreen;  
            this.Text = "www.pediy.com---qiweixue";  
            this.TopMost = true;  
            this.Load += new System.EventHandler(this.PediyForm\_Load);  
            this.ResumeLayout(false);  
            this.PerformLayout();  
  
        }  
  
        private void button1\_Click(object sender, EventArgs e)  
        {  
            openFileDialog1.ShowDialog();  
            textBox1.Text = openFileDialog1.FileName;  
              
              
           
        }  
  
        private void linkLabel1\_LinkClicked(object sender, LinkLabelLinkClickedEventArgs e)  
        {  
            MessageBox.Show("http:\\\\www.pediy.com---qiweixue");  
        }  
  
        private void button2\_Click(object sender, EventArgs e)  
        {  
            if (textBox1.Text != null)  
            {  
                byte\[\] pebuf1=new Byte\[\]{117,66};  
                byte\[\] pebuf2=new Byte\[\]{91,82,69,71,73,83,84,69,82,73,78,70,79,93  
                                        ,13,10,113,105,119,101,105,120,117,101,64,  
                                         49,50,54,46,99,111,109,32,113,105,119,101,  
                                          105,120,117,101,13,10};  
                FileStream fs1 = new FileStream(textBox1.Text,FileMode.Open,FileAccess.ReadWrite);  
                FileInfo fi = new FileInfo(textBox1.Text);  
                drxpath = fi.DirectoryName;  
                fs1.Seek(76385, SeekOrigin.Begin);  
                fs1.Write(pebuf1, 0, pebuf1.Length);  
                fs1.Close();  
  
                FileStream fs2= new FileStream(drxpath + "\\\\Zwtstetc.inf", FileMode.Append, FileAccess.Write);  
                fs2.Write(pebuf2, 0, pebuf2.Length);  
                fs2.Close();  
                textBox2.Text = "OK,Succeed:=)....\[wwww.pediy.com-qiweixue\]";  
            }  
  
        }  
  
        private void PediyForm\_Load(object sender, EventArgs e)  
        {  
            textBox2.Text = "PediyForm is running...........";  
        }  
    }  
  
    public class RunPediyForm  
    {  
        \[STAThread\]  
        public static void Main()  
        {  
            Application.Run(new PediyForm());  
        }  
    }  
}  
  
  
\==================  
Net FrameWork Distributable Package  2.0  
Net FrameWork SDK 2.0  
直接用 CSC.exe Pediy.cs 编译就 ok...  
有误之处,多谢指教  
Copyright © 2000 - 2006 PEdiy.com All Rights Reserved.By KanXue Studio 

引用:  *最初由** KAN **发布*  
c# 的不会，加点注译应该会好点，支持 

  
兄弟,C#跟 java 差很类似...  
代码不多,程序很简单,不好意思注释.  
  
一个地方是到 inf 文件输入注册信息.邮箱:qiweixue666@126.com 注册码:qiweixue  
byte\[\] pebuf2=new Byte\[\]{91,82,69,71,73,83,84,69,82,73,78,70,79,93  
,13,10,113,105,119,101,105,120,117,101,64,  
49,50,54,46,99,111,109,32,113,105,119,101,  
105,120,117,101,13,10};  
  
一个地方是补丁关键 code.注册成功:  
byte\[\] pebuf1=new Byte\[\]{117,66}; 

 4dd466d0e1e0cc123afbe504da2462ca