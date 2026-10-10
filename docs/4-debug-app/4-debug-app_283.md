

# 含沙量高的->连连看 v3.0 游戏修改器

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-184.htm`](https://www.pediy.com/kssd/pediy09/pediy09-184.htm)

> ![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif") └文章标题┐:连连看 v3.0 小试逆向 and 游戏修改器  
> └破文作者┐:大菜一号  
> └破解对象┐:连连看 v3.0  
> └下载地址┐:不知道  
> └对象大小┐:不知道  
> └加壳方式┐:upx  
> └保护方式┐:说是逆向啦!  
> └编写语言┐:vb  
> └使用工具┐:OD  
> └破解平台┐:D-XP  
> └破解声明┐:逆向也不错:)  
> \----------------------------------------------------------------------------------  
> └破解过程┐:  
>   这个游戏好像不太好找了！找到的都是 4.0 的，呵呵，压了也有 3m 多，所以临时做了个精简版，压了之后有 788k，传上来了！  
>   从未做过游戏修改器也，说也简单,找到地址，然后 WriteProcessMemory 一下就行!主要是如何找地址！这个程序脱壳 upx 老牌子衣服之后，可以看到它是用 VB 做的！呵呵\`\`代码就有点乱罢了！  
>   
>   游戏介绍一下:  
>   一共有 11 关，从 0 关到第 10 关，除了第 0 关游戏图片不会动之外，其它的要不就是整体向上，向左，左上右下什么什么的！很是烦！  
>   
>   玩一下游戏，发现几处想改的地方:  
> 1、重新洗牌数  
> 2、提示数  
> 3、跳关  
> 4、提示数不想让程序减了  
> 5、洗牌数也不想让程序减了  
> 6、提示数用完之后，那个菜单会变灰，就算我们用游戏修改器加了提示数，也一样，所以，这里也不想让它变灰了！  
> 7、重新洗牌数用完之后就 Game Over,没注意看洗牌数为 0 时，再使用就 Game Over,而不像提示数那个可以变灰！我们就不让它 Game Over!  
> 好了，要改的就这么些地方，有人要问了，连连看都有时间限制的吧，你怎么不改时间，让它停住不动，这要改很简单！只不过，改了的话就要被人 bs 了！！呵呵~~这里就放弃吧！  
>   
>   好，脱掉 upx 之后，用 OD 载入，提示加壳(汗，我想修正一下 ep 段就不会了吧)：  
>   
> 一、找提示数地址:  
>   我们发现游戏有三个难度(easy,normal,hard)，简单，正常和因难三种！就有了这三个关键字，我们用 w32dasm 载入，查找到字符串 easy,双击之后发现多处调用，当然这里是试用出来的！到这:  
> 00489480    55              push    ebp                               
> 00489481    8BEC            mov     ebp, esp  
> 00489483    83EC 08         sub     esp, 8  
> 00489486    68 861D4000     push    <jmp.&MSVBVM60.\_\_vbaExceptHandle>  
> 0048948B    64:A1 00000000  mov     eax, dword ptr fs:\[0\]  
> 00489491    50              push    eax  
> 00489492    64:8925 0000000>mov     dword ptr fs:\[0\], esp  
> 00489499    81EC EC000000   sub     esp, 0EC  
> 0048949F    53              push    ebx  
> 004894A0    56              push    esi  
> 004894A1    57              push    edi  
> 004894A2    8965 F8         mov     dword ptr \[ebp-8\], esp  
> 004894A5    C745 FC 0017400>mov     dword ptr \[ebp-4\], 00401700  
> 004894AC    8B15 B0314A00   mov     edx, dword ptr \[4A31B0\]  
> 004894B2    33F6            xor     esi, esi  
> 004894B4    8D8D 24FFFFFF   lea     ecx, dword ptr \[ebp-DC\]  
> 004894BA    8975 E4         mov     dword ptr \[ebp-1C\], esi  
> 004894BD    8975 E8         mov     dword ptr \[ebp-18\], esi  
> 004894C0    8975 DC         mov     dword ptr \[ebp-24\], esi  
> 004894C3    8975 E0         mov     dword ptr \[ebp-20\], esi  
> 004894C6    8975 D0         mov     dword ptr \[ebp-30\], esi  
> 004894C9    8975 CC         mov     dword ptr \[ebp-34\], esi  
> 004894CC    8975 C8         mov     dword ptr \[ebp-38\], esi  
> 004894CF    8975 C4         mov     dword ptr \[ebp-3C\], esi  
> 004894D2    8975 C0         mov     dword ptr \[ebp-40\], esi  
> 004894D5    8975 B0         mov     dword ptr \[ebp-50\], esi  
> 004894D8    8975 A0         mov     dword ptr \[ebp-60\], esi  
> 004894DB    8975 90         mov     dword ptr \[ebp-70\], esi  
> 004894DE    8975 80         mov     dword ptr \[ebp-80\], esi  
> 004894E1    89B5 70FFFFFF   mov     dword ptr \[ebp-90\], esi  
> 004894E7    89B5 60FFFFFF   mov     dword ptr \[ebp-A0\], esi  
> 004894ED    89B5 50FFFFFF   mov     dword ptr \[ebp-B0\], esi  
> 004894F3    89B5 3CFFFFFF   mov     dword ptr \[ebp-C4\], esi  
> 004894F9    89B5 24FFFFFF   mov     dword ptr \[ebp-DC\], esi  
> 004894FF    66:8935 CE304A0>mov     word ptr \[4A30CE\], si  
> 00489506    8935 34314A00   mov     dword ptr \[4A3134\], esi  
> 0048950C    8935 58314A00   mov     dword ptr \[4A3158\], esi  
> 00489512    FF15 B4114000   call    dword ptr \[<&MSVBVM60.\_\_vbaStrCo>; MSVBVM60.\_\_vbaStrCopy  
> 00489518    8B85 24FFFFFF   mov     eax, dword ptr \[ebp-DC\]  
> 0048951E    8B3D F4104000   mov     edi, dword ptr \[<&MSVBVM60.\_\_vba> ; \_\_vbaStrCmp 的 rva 传给 edi,主要是供下面的 call 调用  
> 00489524    50              push    eax                               ; 关键字"easy"进栈  
> 00489525    68 5C6C4000     push    00406C5C                          ; 点击的菜单项的标题进栈  
> 0048952A    FFD7            call    edi                               ; 比较，所取得的信息是否为"easy"  
> 0048952C    85C0            test    eax, eax                          ; 返回值是否为空  
> 0048952E    75 1E           jnz     short 0048954E                    ; 不为空则所取信息不为"easy"  
> 00489530    C705 D0304A00 0>mov     dword ptr \[4A30D0\], 43480000      ; 所选难度为 easy,则在这里初始化游戏  
> 0048953A    66:C705 94314A0>mov     word ptr \[4A3194\], 4              ; 提示次数传给 004a3194  
> 00489543    66:C705 D4304A0>mov     word ptr \[4A30D4\], 2              ; 洗牌次数传到 004a30d4  
> 0048954C    EB 5E           jmp     short 004895AC                    ; 跳过后面中级高级难度的比较  
> 0048954E    8B8D 24FFFFFF   mov     ecx, dword ptr \[ebp-DC\]            
> 00489554    51              push    ecx  
> 00489555    68 6C6C4000     push    00406C6C                         ; 如果不为 easy 级别，则到这里，normal 关键进栈  
> 0048955A    FFD7            call    edi                              ; 调用\_\_vbaStrCmp  
> 0048955C    85C0            test    eax, eax                         ; 是否为 normal  
> 0048955E    75 1E           jnz     short 0048957E                   ; 不是就跳  
> 00489560    C705 D0304A00 0>mov     dword ptr \[4A30D0\], 43480000     ; 是 normal 级别就在这里初始化游戏  
> 0048956A    66:C705 94314A0>mov     word ptr \[4A3194\], 6             ; 提示次数赋值  
> 00489573    66:C705 D4304A0>mov     word ptr \[4A30D4\], 3             ; 洗牌次数赋值  
> 0048957C    EB 2E           jmp     short 004895AC  
> 0048957E    8B95 24FFFFFF   mov     edx, dword ptr \[ebp-DC\]  
> 00489584    52              push    edx  
> 00489585    68 806C4000     push    00406C80                         ; 如果不为 normal 和 easy 级别就到这里  
> 0048958A    FFD7            call    edi                              ; 调用\_\_vbaStrCmp  
> 0048958C    85C0            test    eax, eax                         ; 是否为 hard  
> 0048958E    75 1C           jnz     short 004895AC                   ; 不等就跳  
> 00489590    C705 D0304A00 0>mov     dword ptr \[4A30D0\], 43700000     ; 是 hard 就初始化游戏  
> 0048959A    66:C705 94314A0>mov     word ptr \[4A3194\], 8             ; 提示次数赋值  
> 004895A3    66:C705 D4304A0>mov     word ptr \[4A30D4\], 4             ; 洗牌次数赋值  
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
> 我们可以知道,提示数的地址为 4a3194,洗牌数地址为 4a30d4  
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
>   
> 二、找跳关，扣除提示数和洗牌数的地址:  
>   我们知道提示数和洗牌数的地址了，如果游戏要扣除它们的话，就会访问并写进数据，我们在命令下 d 4a3194,在第一字节上下内存写入断点，再 d 4a30d4，在第一字节上下内存写入断点！这样游戏扣除提示数和洗牌数时我们就可以知道在什么地方了！  
>   
>   运行游戏，使用一次提示数,就到达这里:  
> 004A17F0    66:8B0D 94314A0>mov     cx, word ptr \[4A3194\]            ; 把提示数传给 cx  
> 004A17F7    BA 38884000     mov     edx, 00408838  
> 004A17FC    66:83E9 01      sub     cx, 1                            ; 倒扣提示数(使用一次当然是减一)  
> 004A1800    C745 E0 D000000>mov     dword ptr \[ebp-20\], 0D0  
> 004A1807    0F80 A3080000   jo      004A20B0                         ; jo->真是讨厌!  
> 004A180D    66:890D 94314A0>mov     word ptr \[4A3194\], cx            ; 传回去!  
>   
>  再使用一次洗牌数:  
> 00485AC7    66:A1 D4304A00  mov     ax, word ptr \[4A30D4\]            ; 把洗牌数传给 ax  
> 00485ACD    66:2D 0100      sub     ax, 1                            ; 倒扣洗牌次数  
> 00485AD1    0F80 68020000   jo      00485D3F                         ; 又是 jo  
> 00485AD7    66:3BC3         cmp     ax, bx                           ; 洗牌次数是否归 0(是否游戏失败)  
> 00485ADA    66:A3 D4304A00  mov     word ptr \[4A30D4\], ax            ; 传回去!  
> 00485AE0    0F8D C3010000   jge     00485CA9  
>   
>   我们不取消上面下的内存写入断点,继续把游戏玩到第二关,第一关所有图片都消除后，游戏会给你加一次洗牌数和提示数，这时又会往提款数和洗牌数地址里写入东西！我们又断下了：  
> 004842AC    66:A1 CE304A00  mov     ax, word ptr \[4A30CE\]            ;\[4a30ce\]里的内容是 0，传给 ax  
> 004842B2    66:8B0D D4304A0>mov     cx, word ptr \[4A30D4\]            ;洗牌数传给 cx  
> 004842B9    66:8B15 94314A0>mov     dx, word ptr \[4A3194\]            ;提示数传给 dx  
> 004842C0    33F6            xor     esi, esi  
> 004842C2    8B7D 08         mov     edi, dword ptr \[ebp+8\]  
> 004842C5    66:05 0100      add     ax, 1                            ;ax+1(关数加一)  
> 004842C9    0F80 5A170000   jo      00485A29  
> 004842CF    66:83C1 01      add     cx, 1                            ;洗牌数加一  
> 004842D3    66:A3 CE304A00  mov     word ptr \[4A30CE\], ax            ;传回去  
> 004842D9    0F80 4A170000   jo      00485A29  
> 004842DF    66:83C2 01      add     dx, 1                            ;提示数加一  
> 004842E3    66:890D D4304A0>mov     word ptr \[4A30D4\], cx            ;传回去  
> 004842EA    0F80 39170000   jo      00485A29  
>   
>   
> 一路 f8 下来会到这:  
> 00484430    66:833D CE304A0>cmp     word ptr \[4A30CE\], 0B            ; 比较是否过完全部的 11 关  
> 00484438    68 94314A00     push    004A3194  
> 0048443D    68 D0304A00     push    004A30D0  
> 00484442    68 48314A00     push    004A3148  
> 00484447    68 D4304A00     push    004A30D4  
> 0048444C    68 CE304A00     push    004A30CE  
> 00484451    0F8D 81030000   jge     004847D8                         ; 是就恭喜你一下!  
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
> 我们又可以知道程序是在 004a17fc 上扣除提示数的，在 485acd 上扣除洗牌数的,在 4a30ce 上表示关数的!  
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
>   
> 三、不让提示数归 0 时菜单项变灰:  
> 用 VBExplorer 找到点击菜单事件代码，来到这里:  
> 00486C24    FFD3            call    ebx  
> 00486C26    66:833D 94314A0>cmp     word ptr \[4A3194\], 0             ;\[4a3194\]是提示数地址,和 0 比较  
> 00486C2E    8B16            mov     edx, dword ptr \[esi\]  
> 00486C30    56              push    esi  
> 00486C31    7F 23           jg      short 00486C56                   ;小于就不跳，就变灰  
>   
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
> 知道 jg 是关键跳,改为 jmp 就不会变灰了!  
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
>   
> 四、洗牌数归 0 时也不会 Game Over:  
> 再次来到扣除洗牌数的地方:  
> 00485ACD    66:2D 0100      sub     ax, 1                            ; 倒扣洗牌次数地址  
> 00485AD1    0F80 68020000   jo      00485D3F  
> 00485AD7    66:3BC3         cmp     ax, bx                           ; 洗洗牌次数是否归 0(是否游戏失败)  
> 00485ADA    66:A3 D4304A00  mov     word ptr \[4A30D4\], ax              
> 00485AE0    0F8D C3010000   jge     00485CA9                         ; 小于就 Game Over;  
>   
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
> 知道 jge 是关键跳,改为 jmp 就不会变灰了!  
> %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
>   
> 要找的就这么多，简单吧！记下机器码，要改的机器码不知道的话可以在 OD 中改成想要的代码，变红的就是改变了的机器码了，一个字节一个字节的记下来，接下来，，我们用 mfc 实现游戏修改器吧:  
>   
> 定义变量:  
> public:  
>   int radio;  
>   BOOL if\_edit\_1,if\_edit\_2;  
>   int tishi,xipia,i;//定义用户输入的洗牌数和提示数,一个临时变量  
>   DWORD ProcessID;//进程 ID  
>   HANDLE hProcess;//进程句柄  
>   HWND hwnd;//窗口句柄  
>   BYTE tishi\_edit,tishi\_1,tishi\_2,tishi\_3,tishi\_4,  
>     tishi\_5,tishi\_6,tishi\_7,tishi\_8;//写入提示不减时的机器码  
>   BYTE xipia\_edit,xipia\_1,xipia\_2,xipia\_3,xipia\_4;  
>   BYTE game\_over\_1,game\_over\_2,game\_over\_3  
>     ,game\_over\_4,game\_over\_5,game\_over\_6,game\_over\_edit1,  
>     game\_over\_edit2,  
>     game\_over\_edit3,  
>     game\_over\_edit4,  
>     game\_over\_edit5,  
>     game\_over\_edit6;//程序 game\_over 时要写入的机器码和原始数据  
>   BYTE tishi\_menu\_gray,tishi\_menu;//菜单变灰的数据  
>   
> 构造函数中变量赋值:  
> hProcess=NULL;  
>   hwnd=NULL;  
>   i=0;  
>   ProcessID=0;  
>   tishi=0;  
>   xipia=0;  
>     
>   tishi\_edit=0x90;//------------给提示不减的机器码赋值  
>     
>   if\_edit\_1=FALSE;  
>   if\_edit\_2=FALSE;  
>   radio=0;  
>   z=FALSE;  
>   game\_over\_edit1=0x90;//在 OD 中所记下的一点机器码  
>   game\_over\_edit2=0x00;  
>   game\_over\_edit3=0x00;  
>   game\_over\_edit4=0x01;  
>   game\_over\_edit5=0xc4;  
>   game\_over\_edit6=0xe9;  
>   
>   tishi\_menu\_gray=0xeb;  
> //至于其它，我们用读的:  
>   
> 对话框初始化 OnInitDialog()函数:  
>   hwnd=::FindWindow(0,"连连看 v3.0");  
>   if(hwnd==0)  
>   {  
>     MessageBox("游戏并没有运行!",NULL,MB\_OK | MB\_ICONWARNING);  
>     CKawai3EditDlg::OnCancel();  
>   }  
>   else  
>   {  
>   GetWindowThreadProcessId(hwnd,&ProcessID);  
>   hProcess=::OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,ProcessID);  
>   if(hProcess==NULL)  
>   {  
>     MessageBox("打开进程失败!",NULL,MB\_OK | MB\_ICONWARNING);  
>     CKawai3EditDlg::OnCancel();  
>   }  
>   //先保存一下数据，以便用户恢复  
>   //保存原先提示不减数据  
>   ReadProcessMemory(hProcess,(LPCVOID)0x004a17fc,&tishi\_1,1,NULL);  
>   ReadProcessMemory(hProcess,(LPCVOID)0x004a17fd,&tishi\_2,1,NULL);  
>   ReadProcessMemory(hProcess,(LPCVOID)0x004a17fe,&tishi\_3,1,NULL);  
>   ReadProcessMemory(hProcess,(LPCVOID)0x004a17ff,&tishi\_4,1,NULL);  
>   ReadProcessMemory(hProcess,(LPCVOID)0x00486b71,&tishi\_5,1,NULL);  
>   ReadProcessMemory(hProcess,(LPCVOID)0x00486b72,&tishi\_6,1,NULL);  
>   ReadProcessMemory(hProcess,(LPCVOID)0x00486b73,&tishi\_7,1,NULL);  
>   ReadProcessMemory(hProcess,(LPCVOID)0x00486b74,&tishi\_8,1,NULL);  
>   //保存存原先洗牌不减数据  
>   ReadProcessMemory(hProcess,(LPVOID)0x485acd,&xipia\_1,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ace,&xipia\_2,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485acf,&xipia\_3,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ad0,&xipia\_4,1,NULL);  
>   //保存原先洗牌数为 0 时也不 Game Over 数据  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ae0,&game\_over\_1,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ae1,&game\_over\_2,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ae2,&game\_over\_3,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ae3,&game\_over\_4,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ae4,&game\_over\_5,1,NULL);  
>   ReadProcessMemory(hProcess,(LPVOID)0x485ae5,&game\_over\_6,1,NULL);  
>   //保存原先菜单会变灰的数据  
>   ReadProcessMemory(hProcess,(LPVOID)0x486af3,&tishi\_menu,1,NULL);  
>   }  
> 游戏修改过程:  
> void CKawai3EditDlg::OnOK()   
> {  
>   // TODO: Add extra validation here  
>   tishi=GetDlgItemInt(IDC\_EDIT1);  
>   xipia=GetDlgItemInt(IDC\_EDIT2);  
>   WriteProcessMemory(hProcess,(LPVOID)0x004a3194,&tishi,4,NULL);  
>   WriteProcessMemory(hProcess,(LPVOID)0x004a30d4,&xipia,4,NULL);   
>           
>   if((GetDlgItem(IDC\_CHECK1)->SendMessage(BM\_GETCHECK,0,0))>0)  
>   {              
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17fc,&tishi\_edit,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17fd,&tishi\_edit,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17fe,&tishi\_edit,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17ff,&tishi\_edit,1,NULL);  
>           //使用提示数不减        
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b71,&tishi\_edit,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b72,&tishi\_edit,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b73,&tishi\_edit,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b74,&tishi\_edit,1,NULL);   
>   }  
>   else//没选中'提示不减'的话,就恢复  
>   {  
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17fc,&tishi\_1,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17fd,&tishi\_2,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17fe,&tishi\_3,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x004a17ff,&tishi\_4,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b71,&tishi\_5,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b72,&tishi\_6,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b73,&tishi\_7,1,NULL);   
>     WriteProcessMemory(hProcess,(LPVOID)0x00486b74,&tishi\_8,1,NULL);  
>   }  
>   
>   if(GetDlgItem(IDC\_CHECK2)->SendMessage(BM\_GETCHECK,0,0)>0)//是否选中洗牌数不减  
>   {  
>           //先读出来，以便恢复  
>             
>     WriteProcessMemory(hProcess,(LPVOID)0x485acd,&tishi\_edit,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ace,&tishi\_edit,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485acf,&tishi\_edit,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ad0,&tishi\_edit,1,NULL);  
>   }  
>   else//没选中就恢复数据  
>   {  
>     WriteProcessMemory(hProcess,(LPVOID)0x485acd,&xipia\_1,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ace,&xipia\_2,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485acf,&xipia\_3,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ad0,&xipia\_4,1,NULL);  
>   }  
>   
>   if(GetDlgItem(IDC\_CHECK3)->SendMessage(BM\_GETCHECK,0,0)>0)  
>     //是否选中洗牌数归 0 也不 game over  
>   {  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae0,&game\_over\_edit6,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae1,&game\_over\_edit5,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae2,&game\_over\_edit4,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae3,&game\_over\_edit3,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae4,&game\_over\_edit2,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae5,&game\_over\_edit1,1,NULL);  
>   }  
>   else//没选中就恢复数据  
>   {  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae0,&game\_over\_1,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae1,&game\_over\_2,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae2,&game\_over\_3,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae3,&game\_over\_4,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae4,&game\_over\_5,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x485ae5,&game\_over\_6,1,NULL);  
>   }  
>   
>   if(GetDlgItem(IDC\_CHECK4)->SendMessage(BM\_GETCHECK,0,0)>0)//是否让提示菜单变灰  
>   {  
>     WriteProcessMemory(hProcess,(LPVOID)0x486c31,&tishi\_menu\_gray,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x486af3,&tishi\_menu\_gray,1,NULL);  
>   }  
>   else  
>   {  
>     WriteProcessMemory(hProcess,(LPVOID)0x486af3,&tishi\_menu,1,NULL);  
>     WriteProcessMemory(hProcess,(LPVOID)0x486c31,&tishi\_menu,1,NULL);  
>   }  
>   WriteProcessMemory(hProcess,(LPVOID)0x004a30ce,&radio,1,NULL);//跳关  
> }  
>   
>   
>   //CDialog::OnOK();  
>   
>   
>   
> void CKawai3EditDlg::OnRadio1()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=0;  
> }  
>   
> void CKawai3EditDlg::OnRadio2()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=1;  
> }  
>   
> void CKawai3EditDlg::OnRadio3()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=2;  
> }  
>   
> void CKawai3EditDlg::OnRadio4()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=3;  
> }  
>   
> void CKawai3EditDlg::OnRadio5()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=4;  
> }  
>   
> void CKawai3EditDlg::OnRadio6()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=5;  
> }  
>   
> void CKawai3EditDlg::OnRadio7()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=6;  
> }  
>   
> void CKawai3EditDlg::OnRadio8()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=7;  
> }  
>   
> void CKawai3EditDlg::OnRadio9()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=8;  
> }  
>   
> void CKawai3EditDlg::OnRadio10()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=9;  
> }  
>   
> void CKawai3EditDlg::OnRadio11()   
> {  
>   // TODO: Add your control notification handler code here  
>   radio=10;  
> }  
>   
> void CKawai3EditDlg::OnCancel()   
> {  
>   // TODO: Add extra cleanup here  
>     
>   CDialog::OnCancel();  
> }  
>   
> void CKawai3EditDlg::OnClose()   
> {  
>   // TODO: Add your message handler code here and/or call default  
>   CloseHandle(hProcess);  //关闭进程句柄  
>   CDialog::OnClose();  
> }  
>   
> 好了好了\`\`就这样完了！！很简单的东西~~  
> 对了，这个游戏的流程我也稍微跟了一下，没有个确切的结果，不放上来给大家看了！就是点击图片时游戏作出的反映，也就是游戏如何判断两个图片是否相同，路径是否可通过，是否可在窗口上消除这两个图片。  
> 大家可以 bp CreateThread，在创建线程函数上下数，当点击一个图片时就会断下了！  
> 有兴趣的朋友可以逆一下，大致来说（我跟了一下），是这样：  
> 先取出第一次点击的图片的位置，和 12 比较，(easy 级别图片版面横向只有 12 张),看看是否超越下标界限！  
> 没有的话经过算一算，得出结果 a,再取出第二次点击的图片的位置，再看看是否超越下标界限，没有的话再算一算，得到结果 b；  
> 两结果比较，相等的话再算一下路径是否可通过，可通过就消除这两张图片！  
>   
> 就这样！！~~呵呵~~  
> 可能不太对，，- -B\`\`  
> 还有哇\`这个游戏修改器修改提示数和洗牌数时我没限制的，改太多的话 jo 这家伙就溢出了！~~  
> 所以\`\`大家小心啦！呵呵呵呵~~  
> \----------------------------------------------------------------------------------  
> └经验总结┐:  
>   我本来想做个专业的自动连连器的，就像扫雷器一样，不过跟踪 VB 程序是在是太烦了！！想想还是和弃了，有耐心的朋友可以一试，弄懂算法就可以了！呵呵~~先这样吧！  
>     
>   中考摧残身心之娱乐之作！！呵呵~  
> \----------------------------------------------------------------------------------  
> └版权声明┐ 本文原创于看雪软件安全论坛, 转载请注明作者并保持文章的完整, 谢谢!  
>                                                            2007 年 6 月 17 日  20:30:41 

 4dd466d0e1e0cc123afbe504da2462ca