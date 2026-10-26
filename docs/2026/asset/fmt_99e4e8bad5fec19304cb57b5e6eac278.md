
# [原创]ScreenHunter 4.2 Pro 注册算法分析

- **软件名称**：ScreenHunter 4.2 Pro
- **破文作者**：forever[RCT]
- **编程语言**：VC
- **保护方式**：简单运算
- **使用工具**：peid0.93, exescope6.5, ida4.6, ollydbg1.1
- **软件简介**：ScreenHunter 是一个非常好的屏幕抓取工具，除了基本常见的功能外，他还可以抓取椭圆的选取区，抓取Word文件，还可以自动产生文件名称，定时抓取屏幕，抓取DirectX游戏及屏幕保护程序的屏幕，转存JPEG、GIF、PNG、BMP文件。
- **下载地址**：`039K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6K6c8Q4x3X3g2K6K9%4W2U0L8W2)9J5k6h3y4G2L8g2)9J5c8Y4y4G2k6Y4c8Q4x3V1j5#2x3o6t1I4i4K6u0W2K9s2c8E0L8l9`.`.

## 破文正文

首先声明一下，这篇帖子是写给新手的。:)

突然发现，写破文其实是一件很辛苦的事，尤其是想把整个过程写清楚的时候。向那些写了大量破文的前辈们道声辛苦吧。;)

我尽量把过程写的易懂些，不过我还是假定你会使用基本的工具，懂得编程的基本知识。关于MFC的消息处理函数的查找过程

我前面有过一篇帖子<从对话框的构造函数找到消息处理过程>，如果你不太清楚下面的过程可以参考一下前面的帖子。下面我只写下整个过程。我不在这里对代码做逆向分析了，我只是在注释里尽量写的清楚些。这样可以省掉很多时间:)

首先检查软件的编程语言，peid侦测出是VC，大多数情况下这是可信的。用od和ida载入也证实确实是VC的。

软件启动时有个NAP窗口提示输入注册码，我用exescope找到这个窗口的资源id是`1797h`,ok按钮的id是`1d4h`。了解这些信息是有用的。

现在用ida载入软件分析。我以前做过一个mfc4.2的sig，如果你使用这个sig，会检测出更多的函数名。

在Name窗口里查找`CDialog::CDialog(unsigned int,class CWnd *)`,来到下面，

```
.text:0054A50F public: __thiscall CDialog::CDialog(unsigned int,class CWnd *) proc near

.text:0054A50F

.text:0054A50F arg_0           = dword ptr  0Ch

.text:0054A50F arg_4           = dword ptr  10h

.text:0054A50F

.text:0054A50F                 push    esi

.text:0054A510                 push    edi

.text:0054A511                 mov     esi, ecx

.text:0054A513                 call    CWnd::CWnd(void)

.text:0054A518                 lea     edi, [esi+3Ch]

.text:0054A51B                 push    20h             ; size_t

.text:0054A51D                 push    0               ; int

.text:0054A51F                 push    edi             ; void *

.text:0054A520                 mov     dword ptr [esi], offset off_5838D4

.text:0054A526                 call    _memset

.text:0054A52B                 mov     eax, [esp+0Ch+arg_4]

.text:0054A52F                 add     esp, 0Ch

.text:0054A532                 mov     [esi+50h], eax

.text:0054A535                 mov     eax, [esp+arg_0]

.text:0054A539                 movzx   ecx, ax

.text:0054A53C                 mov     [edi], eax

.text:0054A53E                 mov     [esi+40h], ecx

.text:0054A541                 mov     eax, esi

.text:0054A543                 pop     edi

.text:0054A544                 pop     esi

.text:0054A545                 retn    8

.text:0054A545 public: __thiscall CDialog::CDialog(unsigned int,class CWnd *) endp

;=================================================================================
```

为什么要找这个函数呢？因为这个函数是对话框类的构造函数，自定义对话框在初始化时肯定要调用这个函数。

在名字`CDialog::CDialog`上右击，选择菜单Jump to xref to operand，来到下面，

```
Up   p sub_40E7A7+C                                                         call    CDialog::CDialog(uint,CWnd *)
Up   p sub_411987+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_414C64+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_437218+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_43F20C+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4460F4+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_44D3FD+1A                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_46F72B+C                                                         call    CDialog::CDialog(uint,CWnd *)
Up   p sub_486E87+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4A89B6+1D                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4AF524+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4BC73F+1B                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4C0301+1A                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4C2F1A+1A                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4C5B16+1A                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p sub_4C8639+1A                                                        call    CDialog::CDialog(uint,CWnd *)
Up   p CFileDialog::CFileDialog(int,char const *,char const *,ulong,char const *,CWnd *)+19 call    CDialog::CDia
Up   p CFontDialog::CFontDialog(tagLOGFONTA *,ulong,CDC *,CWnd *)+19        call    CDialog::CDialog(uint,CWnd *)
Up   p CFontDialog::CFontDialog(_charformat const &,ulong,CDC *,CWnd *)+19  call    CDialog::CDialog(uint,CWnd *)
Up   p CColorDialog::CColorDialog(ulong,ulong,CWnd *)+17                    call    CDialog::CDialog(uint,CWnd *)
Up   p CPageSetupDialog::CPageSetupDialog(ulong,CWnd *)+17                  call    CDialog::CDialog(uint,CWnd *)
Up   p CPrintDialog::CPrintDialog(int,ulong,CWnd *)+18                      call    CDialog::CDialog(uint,CWnd *)
Up   p CPrintDialog::CPrintDialog(tagPDA &)+7                               call    CDialog::CDialog(uint,CWnd *)
Up   p CFindReplaceDialog::CFindReplaceDialog(void)+17                      call    CDialog::CDialog(uint,CWnd *)
Down p CDocManager::OnFileNew(void)+3D                                      call    CDialog::CDialog(uint,CWnd *)
;=================================================================================
```

这里就是对`CDialog::CDialog`引用的地方了。注意找对话框资源号为`1797h`的函数。（使用exescope查看注册对话框资源）

来到下面：

```
.text:0043F20C sub_43F20C      proc near               ; CODE XREF: sub_46973F+31p
.text:0043F20C                 mov     eax, offset unknown_libname_1324
.text:0043F211                 call    __EH_prolog
.text:0043F216                 push    ecx
.text:0043F217                 push    ebx
.text:0043F218                 push    esi
.text:0043F219                 push    edi
.text:0043F21A                 mov     esi, ecx
.text:0043F21C                 push    dword ptr [ebp+8]
.text:0043F21F                 mov     [ebp-10h], esi
.text:0043F222                 push    1797h           ; 注册对话框
.text:0043F227                 call    CDialog::CDialog(uint,CWnd *) //构造注册对话框的地方，这就是
.text:0043F22C                 and     dword ptr [ebp-4], 0          //从CDialog::CDialog下手的原因
.text:0043F230                 lea     ebx, [esi+5Ch]
.text:0043F233                 mov     ecx, ebx
.text:0043F235                 call    CWnd::CWnd(void)
.text:0043F23A                 mov     edi, offset off_584E04
.text:0043F23F                 mov     [ebx], edi
.text:0043F241                 lea     ebx, [esi+98h]
.text:0043F247                 mov     byte ptr [ebp-4], 1
.text:0043F24B                 mov     ecx, ebx
.text:0043F24D                 call    CWnd::CWnd(void)
.text:0043F252                 mov     [ebx], edi
.text:0043F254                 lea     ebx, [esi+0D4h]
.text:0043F25A                 mov     byte ptr [ebp-4], 2
.text:0043F25E                 mov     ecx, ebx
.text:0043F260                 call    CWnd::CWnd(void)
.text:0043F265                 mov     [ebx], edi
.text:0043F267                 lea     ebx, [esi+110h]
.text:0043F26D                 mov     byte ptr [ebp-4], 3
.text:0043F271                 mov     ecx, ebx
.text:0043F273                 call    CWnd::CWnd(void)
.text:0043F278                 mov     [ebx], edi
.text:0043F27A                 lea     ebx, [esi+14Ch]
.text:0043F280                 mov     byte ptr [ebp-4], 4
.text:0043F284                 mov     ecx, ebx
.text:0043F286                 call    CWnd::CWnd(void)
.text:0043F28B                 mov     [ebx], edi
.text:0043F28D                 mov     eax, dword_5D537C
.text:0043F292                 lea     ebx, [esi+188h]
```
