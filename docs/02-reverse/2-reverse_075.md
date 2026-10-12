
# plugin sdk masm sample 注释翻译

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-720.htm`](https://www.pediy.com/kssd/pediy09/pediy09-720.htm)

> 最近想看看,OD 插件编写,顺手把它翻译了.我英语差,看的懂就行了.:)

> 还有个问题,我在工具版下的 plugin sdk masm 编译不通过,不知是什么回事,我用的是 masm32v8 和 radasm.谁能告诉原因.

> 顺便建议一下,能不能把 OD 开发的 Plugins.hlp 翻译了.

>

>

> Sample.asm:

> ;//////////////////////////////////////////////////////////////////////////////////
> ;//                                                                                 //
> ;//                 SAMPLE PLUGIN FOR OLLYDBG                                      //
> ;//                                                                                 //
> ;// 这个插件允许使用键盘快捷设立 10 code bookmarks 或者分拆 popup 菜单和很快的返回    //
> ;// bookmarks 使用的快捷键,popup 菜单或者 Bookmark 窗口.在文件 Bookmarks 保存 session 中.//
> ;//                                                                                 //
> ;//////////////////////////////////////////////////////////////////////////////////
>
> .386
> .model flat, stdcall ; 32 bit 内存模块
> option scoped ; local labels 被激活, global labels 在里面
> ;                       PROC 将被用 double 冒号定义(LABEL::)
> option casemap :none ; case sensitive
>
>
> include windows.inc
> include kernel32.inc
> include user32.inc
> include ..\plugin.inc
>
> includelib kernel32.lib
> includelib user32.lib
> includelib ..\ollydbg.lib
>
> .data?
>
> hinst                 HINSTANCE        ?  ; DLL 接口 instance
> hwmain                HWND             ?  ;OllDbg 主窗口句柄
> bookmarkwinclass      db 32 dup (?)    ;bookmark 窗口类的名称
>
> ; ollydbg 支持和广泛利用特殊的数据收集
> ; 所谓分类表. 表由广义表(t_table)和数据组成. 所有
> ; 数据元素具有相同的大小和开始为期 3dword header:地址、大小
> ; 和类型. 由地址自动分类项目表,不允许重叠.
> ;   我们 bookmark 表包含类型 t_bookmark 成分.
> t_bookmark struc 1
>     index   dword   ?       ;bookmark 索引(0..9)
>     size_   dword   ?       ;索引大小,在我们 case 总是 1
>     type_   dword   ?       ;入口的类型,总是 0
>     addr_   dword   ?       ;bookmark 的地址
> t_bookmark ends
>
> bookmark    t_table     <>  ;bookmark 表
>
> ; 在这个文件的函数或多或少"按顺序排列"的命令,
> ; 即由 ollydbg 先后次序 call. 这就需要向前
> ; 提交.
>
> Bookmarksortfunc proto C b1:ptr t_bookmark, b2:ptr t_bookmark, sort:dword
>
> Bookmarkwinproc proto hw:HWND, msg:UINT, wp:WPARAM, lp:LPARAM
>
> Createbookmarkwindow proto
>
> ;int Bookmarkgettext(char \*s,char \*mask,int \*select,t_sortheader \*ph,int column);
> ;void Createbookmarkwindow(void);
>
>
> .code
>
>
> ; Entry 指向插件 DLL. 许多系统调用的 DLL 接口
> ; 通过 dllentrypoint()作为参数之一. 记住了.
> ; 首选的方法是放置 initializations 到 ODBG_Plugininit()同时
> ; 清理 odbg_plugindestroy().
> DllEntryPoint proc hi:HINSTANCE, reason:dword, res:dword
>         .IF reason == DLL_PROCESS_ATTACH
>             m2m     hinst, hi                ; Mark plugin 接口
>         .ENDIF
>         return  1                    ; 返回成功
> DllEntryPoint endp
>
> ; odbg_plugindata()是"必须"有效 ollydbg 插件. 必须填写
> ; 插件名称和返回插件接口版本. 如果缺少这个函数,
> ; 或不兼容的版本,插件将不会被安装. Short name
> ; 在插件菜单识别它. 这个名称是 max. 31 包括字母或数字
> ; 性质或间隔+以'\0'终结. 为使用者简易维护
> ; 这个名称将被用 DLL 的名称描述和相关.
>
> ODBG_Plugindata proc C shortname:ptr byte
>         invoke  lstrcpy, shortname, CTEXT("Bookmarks")      ; Name of plugin
>         return  PLUGIN_VERSION;
> ODBG_Plugindata endp
>
> ; ollydbg 在一次启动期间必须 call 这个函数.放置所有
> ; 一次性初始化这里. 如果所有资源成功分配,
> ; 函数必须返回 0. 错误的话,必须释放部分分配的的资源
> ; 返回-1,在这种情况下插件将会被取消. 参数 ollydbgversion
> ; 是 ollydbg 的版本,它被用来告诉你,用你的插件是兼容
> ; 的; hw 是 ollydbg 主窗口句柄(handle),如果必要保持它.
> ; 参数 features 是为日后扩展预留的,不要使用它.
> ODBG_Plugininit proc C ollydbgversion:dword, hw:HWND, features:ptr dword
>
>         ; 测试 OllyDbg 版本是正确的.
>         .IF ollydbgversion < PLUGIN_VERSION
>             jmp     @@bad_exit
>         .ENDIF
>   ; 保持 ollydbg 主窗口句柄. 这个句柄是必要的,例如
>   ; 显示消息框.
>
>         m2m     hwmain, hw
>   ; 初始化 bookmark 数据. 数据以类型 t_bookmark 为基础组成,
>   ; 为 10 elements 我们预留空位. 如有必要,表将分配更多
>   ; 空间,但是,我们遇到的最大 bookmarks 数是 10. Elements 并不
>   ; 分配内存或其他资源,所以销毁是没有必要的.
>
>         invoke  Createsorteddata, offset bookmark.data, CTEXT("Bookmarks"),
>                             sizeof t_bookmark, 10, offset Bookmarksortfunc, NULL
>
>         .IF eax
>             jmp     @@bad_exit
>         .ENDIF
>
>   ; 为 MDI 窗口注册窗口类,这个将显示插件. 请
>   ; 注意这个类属于主 ollydbg 编程的接口
>   ; 不是插件 DLL. 字符串 bookmarkwinclass 取得唯一的新类名.
>   ; 保持它到建立窗口和关闭窗口 unregister.
>         invoke  Registerpluginclass, offset bookmarkwinclass, NULL, hinst, offset Bookmarkwinproc
>         .IF eax < 0
>             ; Failure! Destroy sorted data and exit.
>             invoke  Destroysorteddata, offset bookmark.data
>             jmp     @@bad_exit
>         .ENDIF
>
>   ; 插件初始化成功. 向日志窗口报告这一行为现在是最好的时机
>   ; . 要符合 ollydbg 外观,请用两句.
>   ; 第一,在 black,说明插件,第二变灰和缩进
>   ; 由两个字符、承担版权说明.
>         invoke  Addtolist, 0, 0, CTEXT("Bookmarks sample plugin v1.08 (plugin demo)")
>         invoke  Addtolist, 0, -1, CTEXT("  Copyright (C) 2001, 2002 Oleh Yuschuk")
>   ; ollydbg 用用属性 table_savepos 保存插件窗口位置
>   ; 到. INI 文件,但并不自动恢复他们. 这里让我们加上
>   ; 功能性. 无论是公开信息的窗口,当 ollydbg 终止时我在 olldbg.ini 保持信息无论窗口是打开
>   ; . 这一信息是保存在
>   ; odbg_pluginclose. 符合 ollydbg 标准,若选择打开被激活
>   ; 窗口被恢复.
>         invoke  Plugingetvalue, VAL_RESTOREWINDOWPOS
>         push    eax
>         invoke  Pluginreadintfromini, hinst, CTEXT("Restore bookmarks window"), 0
>         pop     ecx
>         .IF eax && ecx
>             invoke  Createbookmarkwindow
>         .ENDIF
>
>         return  0
> @@bad_exit:
>         return  -1
> ODBG_Plugininit endp
>
>
> ; 退出时 ollydbg call 这个可选函数. 此时,所有由 plugin 建立的 MDI
> ; 窗口已经被毁造(收到 WM_DESTROY
> ; 消息). 函数必须释放所有内部资源,如
> ; 窗口类,文件,内存等.
> ODBG_Plugindestroy proc C
>         invoke  Unregisterpluginclass, offset bookmarkwinclass
>         invoke  Destroysorteddata, offset bookmark.data
>         ret
> ODBG_Plugindestroy endp
>
> ; 当用户需要打开新的或重启动当前程序这个函数被 call.
> ; 插件重新安排内部变量和数据结构为初始状态.
> ODBG_Pluginreset proc C
>         invoke  Deletesorteddatarange, offset bookmark.data, 0, 0FFFFFFFFh
>         ret
> ODBG_Pluginreset endp
>
> ; 当用户想终止择 ollydbg,ollydbg call 这个可选函数.
> ; 所有由插件建立的 MDI 窗口仍然存在. 如果要安全的终止函数必须返回 0
> ; . 任何非零返回将停止结束序列.
> ; 不滥用这种可能性! 要告诉用户这个约定为什么
> ; 终止是不好的,并询问他决定!
> ODBG_Pluginclose proc C
>   ; 自动恢复打开窗口,mark 在. INI 文件
>   ; Bookmarks 窗口是否仍然打开.
>         mov     eax, bookmark.hw
>         .IF eax
>             mov     eax, 1
>         .ENDIF
>         invoke  Pluginwriteinttoini, hinst, CTEXT("Restore bookmarks window"), eax
>         return  0
> ODBG_Pluginclose endp
>
>
> ; 由一些标准严格分类数据种类,要提供分类函数
> ; 返回-1 如果第一要素是少于二,1 如果第一点是
> ; 包括周围和如果均等的依照分类标准 0. 通常
> ; 在窗口这一标准是零基指数.
> Bookmarksortfunc proc C b1:ptr t_bookmark, b2:ptr t_bookmark, sort:dword
> LOCAL i:dword
>
>         mov     ecx, 1
>         mov     esi, b1
>         mov     edi, b2
>
>         .IF sort == 1
>             mov     eax, [esi].t_bookmark.addr_
>             .IF eax < [edi].t_bookmark.addr_
>                 mov     ecx, -1
>             .ELSEIF eax > [edi].t_bookmark.addr_
>                 mov     ecx, 1
>             .ENDIF
>         .ENDIF
>
>         .IF ecx == 1
>             mov     eax, [esi].t_bookmark.index
>             .IF eax < [edi].t_bookmark.index
>                 mov     ecx, -1
>             .ELSEIF eax > [edi].t_bookmark.index
>                 mov     ecx, 1
>             .ENDIF
>         .ENDIF
>         return  ecx



> `Bookmarksortfunc endp`
>
> ; 标准函数 `painttable()` 使得 `ollydbg` 调整大部分窗口重新. 你
> ; 仅需要供应其他编写文本串函数,
> ; 可选择颜色. 定制窗口方案是比较复杂的,
> ; 请阅读文件.
>
> `Bookmarkgettext proc s:ptr byte, mask\_:ptr byte, select:ptr dword, ph:ptr t\_sortheader, column:dword`
>
> ```asm
>         mov     eax, column
>         .IF !eax        ;bookmark 的名字
>     ; bookmark 的栏 0 包含名在"Alt+n",其中 n 是
>     ; 数字从 0 到 9. 主要用于示范用途,我展示前缀
>     ; "Alt+"变灰和在正常文本数字. 标准表格窗口
>     ; 不用理会选择.
>             mov     eax, ph
>             invoke  wsprintf, s, CTEXT("Alt+%i"), [eax].t\_bookmark.index
>             mov     eax, select
>             mov     dword ptr [eax], DRAW\_MASK
>             invoke  RtlFillMemory, mask\_, 4, DRAW\_GRAY
>             mov     eax, mask\_
>             mov     byte ptr [eax + 4], DRAW\_NORMAL
>             invoke  lstrlen, s
>
>         .ELSEIF eax == 1    ; Address of bookmark
>             mov     eax, ph
>             invoke  wsprintf, s, CTEXT("%08X"), [eax].t\_bookmark.addr\_
>             invoke  lstrlen, s
>
>         .ELSEIF eax == 2    ;分拆(Disassembled)命令
>     ; 函数 Disasm()命令 calling 程序代码到
>     ; 分拆. 读内存代码. 最先确定可能的
>     ; 代码大小.
>             invoke  wsprintf, s, CTEXT("column %d"), eax
>             invoke  lstrlen, s
>
>         .ELSEIF eax == 3
>             mov     eax, ph
>             invoke  Findname, [eax].t\_bookmark.addr\_, NM\_COMMENT, s
>
>         .ELSE
>             xor     eax, eax
>         .ENDIF
>         ret
> ```
>
> `Bookmarkgettext endp`
>
> ; 每个窗口类需要自己窗口程序. 无论标准的和定制的
> ; `OllyDbg` 窗口必须通过一些系统和 `OllyDbg-defined` 定义的消息到
> `Bookmarkwinproc proc hw:HWND, msg:UINT, wp:WPARAM, lp:LPARAM`
> ; `LOCAL controlkey, shiftkey:dword`
> `LOCAL menu:HMENU`
> ; `LOCAL pb:ptr t\_bookmark`
>
> ```asm
>         mov     eax, msg
>
>  ; 标准消息. 你能及时处理他们,但要有绝对的把握-
>  ; 总是通过他们到 tablefunction().
>         .IF eax == WM\_DESTROY || eax == WM\_MOUSEMOVE || eax == WM\_LBUTTONDOWN || \
>             eax == WM\_LBUTTONDBLCLK || eax == WM\_LBUTTONUP || eax == WM\_RBUTTONDOWN || \
>             eax == WM\_RBUTTONDBLCLK || eax == WM\_HSCROLL || eax == WM\_VSCROLL || eax == WM\_TIMER || eax == WM\_SYSKEYDOWN
>
>             invoke  Tablefunction, offset bookmark, hw, msg, wp, lp
>  ; 通过信息到 defmdichildproc.
>  ; 为卷轴和选择定制可靠消息.用户绘制(User-drawn)
>  ; 窗口必须及时处理,标准 ollydbg 窗口没有额外
>  ; 功能通知他们到 tablefunction().
>         .ELSEIF eax == WM\_USER\_SCR || eax == WM\_USER\_VABS || eax == WM\_USER\_VREL || \
>                 eax == WM\_USER\_VBYTE || eax == WM\_USER\_STS || eax == WM\_USER\_CNTS || eax == WM\_USER\_CHGS
>             invoke  Tablefunction, offset bookmark, hw, msg, wp, lp
>             ret
>
>         .ELSEIF eax == WM\_USER\_MENU
>             invoke  CreatePopupMenu
>             mov     menu, eax
>
>     ; 查找选定 bookmark. 如果存在至少一个 bookmark 退出及被选定
>     ; 一些用 bookmark 操作仅仅产生判断. 记录分类数据
>     ; 有特殊分类表只在必要时更新.
>     ; getsortedbyselection()做这个
>     ; 有些数据整理函数
>     ; 不工作,你必须 call sortsorteddata(). 阅读文件!
>             invoke  Getsortedbyselection, offset bookmark.data, bookmark.data.selected
>             mov     esi, eax
>             .IF menu && eax
>                 invoke  AppendMenu, menu, MF\_STRING, 1, CTEXT("&Follow",9,"Enter")
>                 invoke  AppendMenu, menu, MF\_STRING, 2, CTEXT("&Delete",9,"Del")
>             .ENDIF
>     ; 即使菜单无效,call tablefunction 仍然有意义.
>             invoke  Tablefunction, offset bookmark, hw, WM\_USER\_MENU, 0, menu
>             push    eax
>             mov     eax, menu
>             .IF eax
>                 invoke  DestroyMenu, eax
>             .ENDIF
>             pop     eax
>
>             .IF eax == 1        ;在分拆跟随 bookmark Follow bookmark in Disassembler
>                 invoke  Setcpu, 0, [esi].t\_bookmark.addr\_, 0, 0, CPU\_ASMHIST or CPU\_ASMCENTER or CPU\_ASMFOCUS
>
>             .ELSEIF eax == 2    ; 删除 Delete bookmark
>                 invoke  Deletesorteddata, offset bookmark.data, [esi].t\_bookmark.index
>     ; 无自动更新窗口,你自己 do.
>                 invoke  InvalidateRect, hw, NULL, FALSE
>             .ENDIF
>             return  0
>
>         .ELSEIF eax == WM\_USER\_DBLCLK
>     ; 在分拆双精度行(Doubleclicking row)跟随 bookmark.
>             invoke  Getsortedbyselection, offset bookmark.data, bookmark.data.selected
>             .IF eax
>                 invoke  Setcpu, 0, [eax].t\_bookmark.addr\_, 0, 0, CPU\_ASMHIST or CPU\_ASMCENTER or CPU\_ASMFOCUS
>             .ENDIF
>             return  1   ; Doubleclick 处理 processed
>
>         .ELSEIF eax == WM\_USER\_CHALL || eax == WM\_USER\_CHMEM
>     ; 有些改变,重绘窗口.
>             invoke  InvalidateRect, hw, NULL, FALSE
>             return  0
>
>         .ELSEIF eax == WM\_PAINT
>     ; 由 Painttable()绘制所有 ollydbg 窗口. 使定制
>     ; 仅仅画出如果你有重要原因这样做.
>             invoke  Painttable, hw, offset bookmark, offset Bookmarkgettext
>             return  0
>
>         .ENDIF
>         invoke  DefMDIChildProc, hw, msg, wp, lp
>         ret
> ```
>
> `Bookmarkwinproc endp`
>
> ; 记录类型在 `ollydbg` 和插件中必须是唯一的. 最好
> ; 保证这是由 `OllDbg (Oleh Yuschuk)` 注册标明类型. 注册
> ; 完全免费,除电子邮件费用:)
>
> `TAG\_BOOKMARK        equ     0236D420Ah      ; Bookmark record type in .udd file`
>
> ; 为了节省时间保存数据到 `.udd` 文件! 这是由 `call pluginsaverecord()`
> ; 为每个数据项必须被保存. 全局性,面向过程的数据必须
> ; 主要 `.udd` 文件(名称由 `.exe` 确定); 模块相关资料必须
> ; 保存在模块文件. 勿忘保存各项相对地址到
> ; 模块基址, 即使模块变化也能使数据准确被恢复.
>
> `ODBG\_Pluginsaveudd proc C pmod:ptr t\_module, ismainmodule:dword`
> `LOCAL data[2]:dword`
>
> ```asm
>         .IF !ismainmodule       ; 仅仅 bookmarks 保存到主文件 Save bookmarks to main file only
>             ret
>         .ENDIF
>
>         mov     esi, bookmark.data.data
>         mov     ebx, bookmark.data.n
>
>         .WHILE ebx > 0
>             mov     eax, [esi].t\_bookmark.index
>             mov     data[0], eax
>             mov     eax, [esi].t\_bookmark.addr\_
>             mov     data[1 * sizeof dword], eax
>             invoke  Pluginsaverecord, TAG\_BOOKMARK, sizeof data, addr data
>             dec     ebx
>             add     esi, sizeof t\_bookmark
>         .ENDW
>         ret
> ```
>
> `ODBG\_Pluginsaveudd endp`
>
> ; `ollydbg` 恢复数据 `.udd` 文件. 如果属于记录插件,它必须
> ; 处理记录并返回 1 否则必须返回 0 通过纪录
> ; 其他插件. 注意模块由广义指向不完善的 `pmod`,
> ; 即不一定都含有信息, 尤其
> ; 从 `.udd` 文件.
>
> `ODBG\_Pluginuddrecord proc C pmod:ptr t\_module, ismainmodule:dword, tag:dword, size\_:dword, data:dword`
> `LOCAL mark:t\_bookmark`
>
> ```asm
>         mov     eax, ismainmodule
>         .IF !eax
>             return eax              ; 仅仅 bookmarks 保存到主文件 Bookmarks saved in main file only
>         .ENDIF
>
>         .IF tag != TAG\_BOOKMARK
>             return  0               ; 标记符没有验证 Tag is not recognized
>         .ENDIF
>
>         mov     ecx, data
>         mov     eax, [ecx]
>         mov     mark.index, eax
>         mov     mark.size\_, 1
>         mov     mark.type\_, 0
>         mov     eax, [ecx + 4]
>         mov     mark.addr\_, eax
>         invoke  Addsorteddata, offset bookmark.data, addr mark
>         return  1                   ; 记录进程  Record processed
> ```
>
> `ODBG\_Pluginuddrecord endp`
>
> ; 当建立标准 `MDI` 窗口时 `ollydbg` 进行大部分工作. 插件只需要作为一个整体
> ; 描述栏目数量,窗口的性质和性能.
>
> `Createbookmarkwindow proc`
> ```asm
>         ; Describe table columns. Note that column names are pointers, so strings
>         ; must exist as long as table itself.
>         .IF !bookmark.bar.nbar
>             ; Bar still uninitialized.
>             mov     bookmark.bar.name\_, CTEXT("Bookmark")   ; Name of bookmark
> ```



```asm
mov     bookmark.bar.defdx, 9  
mov     bookmark.bar.mode, 0  
mov     bookmark.bar.name_\[1 \* sizeof dword\], CTEXT("Address") ; Bookmark address  
mov     bookmark.bar.defdx\[1 \* sizeof dword\], 9  
mov     bookmark.bar.mode\[1 \* sizeof byte\], 0  
mov     bookmark.bar.name_\[2 \* sizeof dword\], CTEXT("Disassembly") ; Disassembled command  
mov     bookmark.bar.defdx\[2 \* sizeof dword\], 32  
mov     bookmark.bar.mode\[2 \* sizeof byte\], BAR_NOSORT  
mov     bookmark.bar.name_\[3 \* sizeof dword\], CTEXT("Comment")   ; Comment  
mov     bookmark.bar.defdx\[3 \* sizeof dword\], 256  
mov     bookmark.bar.mode\[3 \* sizeof byte\], BAR_NOSORT  
mov     bookmark.bar.nbar, 4  
mov     bookmark.mode, TABLE_COPYMENU or TABLE_SORTMENU or TABLE_APPMENU or TABLE_SAVEPOS;  
mov     bookmark.drawfunc, offset Bookmarkgettext  
.ENDIF  

; 如果窗口已经存在,quicktablewindow()不建立新窗口  
; 但为了恢复和扩展现有顶步. 这是最简单的方法,  
; newtablewindow()更为灵活,更复杂. 没有很重要的理由我不推荐  
; 定制(插件绘制)窗口.   
invoke  Quicktablewindow, offset bookmark, 15, 4, offset bookmarkwinclass, CTEXT("Bookmarks")  
ret  
Createbookmarkwindow endp  

; 如果你定义 odbg_pluginmainloop,在 ollydbg 窗口这个函数将每次从循环主窗口被 call.  
;  如果这是从调试请求的一些调试事件  
; ,debugevent 指向它,否则 NULL. 非必要  
; 不要申明这个函数,因为这可能产生负面影响到  
; 整体速度!   
;ODBG_Pluginmainloop proc C debugevent:ptr DEBUG_EVENT  
;ODBG_Pluginmainloop endp  
; 函数添加任一项目到 ollydbg 主菜单(origin=pm_main)或 popup  
; 菜单在标准 ollydbg 窗口之一. 当插件要加上自己的菜单  
; 项目,在数据中搜集菜单式样和返回 1, 否则必须  
; 返回 0. 除了静态主菜单,插件不要添加非活动项目.   
; 项目指数在 0..63 范围. 完全相同索引是明确允许的.   
ODBG_Pluginmenu proc C uses ebx origin:dword, data:ptr byte, item:dword  
LOCAL pb:ptr t_bookmark  
LOCAL pd:ptr t_dump  

mov     eax, origin  
; 菜单创作很简单. 你只需填写数据与菜单格式.   
; 一些例子:  
; 0 Aaa,2 Bbb|3 Ccc|,,  - 线性菜单有 3 项,相对的 IDS 0 ,2 和  
; 3,分隔线在第二和第三项之间,最后  
;  
; 分隔线和逗号被忽略;   
; #A{0Aaa,B{1Bbb|2Ccc}} - 无条件的分隔,popup 菜单紧随  
; 一两个要件,二是 popup 用两  
; 级分离分子.   
.IF eax == PM_MAIN ;Plugin 菜单在主窗口 Plugin menu in main window  
invoke  lstrcpy, data, CTEXT("0 &Bookmarks|1 &About")  
;如果你的 plugin 是有价值,我也推荐到 include 帮助  
; If your plugin is more than trivial, I also recommend to include Help.  
return  1  

.ELSEIF eax == PM_DISASM ; Popup menu in Disassembler  
; 首先检查菜单适用性.  
mov     eax, item  
.IF !eax || !\[eax\].t_dump.size_  
return  0                         ; Window 空闲,不添加.Window empty, don't add  
.ENDIF  
; 从二级 popup 菜单.   
; Start second-level popup menu.  
invoke  wsprintf, data, CTEXT("Bookmark{")  
invoke  lstrlen, data  
add     eax, data  
mov     ebx, eax  

; 添加项目"插入 bookmark n"如果释放 bookmarks 及部分  
; 选定分拆. 注意 ollydbg 正确演示  
; superfluos 逗号,分隔,并在一定程度上失去支撑.   
mov     esi, bookmark.data.data  
xor     ecx, ecx  
.WHILE ecx < bookmark.data.n  
.BREAK .IF \[esi\].t_bookmark.index != ecx  
add     esi, sizeof dword  
inc     ecx  
.ENDW  

mov     esi, item  
mov     eax, \[esi\].t_dump.sel1  
.IF ecx < 10 && (eax > \[esi\].t_dump.sel0)  
invoke  wsprintf, ebx, CTEXT("%i &Insert bookmark %i",9,"Alt+Shift+%i,"), ecx, ecx, ecx  
invoke  lstrlen, ebx  
add     ebx, eax  
.ENDIF  
; 添加项目"删除 bookmark n"每次可利用 bookmark. 菜单  
; 标识不一定连续.   
mov     esi, bookmark.data.data  
xor     ecx, ecx  
.WHILE ecx < bookmark.data.n  
mov     eax, \[esi\].t_bookmark.index  
add     eax, 10  
push    ecx  
invoke  wsprintf, ebx, CTEXT("%i Delete bookmark %i,"), eax, \[esi\].t_bookmark.index  
add     esi, sizeof dword  
invoke  lstrlen, ebx  
pop     ecx  
add     ebx, eax  
inc     ecx  
.ENDW  
; 加上分隔到菜单.   
mov     byte ptr \[ebx\], '|'  
inc     ebx  
; 添加项目"到达 bookmark n"每次可利用 bookmark. Bookmarks  
; 设置在搜集命令司令部不显示.   
mov     esi, bookmark.data.data  
xor     ecx, ecx  
.WHILE ecx < bookmark.data.n  
mov     eax, item  
mov     eax, \[eax\].t_dump.sel0  
.IF eax != \[esi\].t_bookmark.addr_  
mov     eax, \[esi\].t_bookmark.index  
add     eax, 20  
push    ecx  
invoke  wsprintf, ebx, CTEXT("%i Go to bookmark %i",9,"Alt+%i,"), eax, \[esi\].t_bookmark.index, \[esi\].t_bookmark.index  
add     esi, sizeof dword  
invoke  lstrlen, ebx  
pop     ecx  
add     ebx, eax  
.ENDIF  
inc     ecx  
.ENDW  
; 关闭 popup. 如果你忘记这样做,ollydbg 将设法纠正  
; 你的错误.   
mov     word ptr \[ebx\], '}'  
return  1  
.ENDIF  
return  0  
ODBG_Pluginmenu endp  

; 这个可择函数从在类型起源窗口的插件菜单接受命令.  
;  启动行为是从 odbg_pluginmenu()的菜单标识符. 如果用户建立  
; 启动自动进入主菜单,行动是 0.   
ODBG_Pluginaction proc C origin:dword, action:dword, item:dword  
LOCAL mark:t_bookmark  
;LOCAL pb:ptr t_bookmark  
;LOCAL pd:ptr t_dump  

mov     eax, origin  
.IF eax == PM_MAIN  
mov     eax, action  
.IF !eax  
; 项目菜单"Bookmarks",建立 bookmark 窗口.   
invoke  Createbookmarkwindow  

.ELSEIF eax == 1  
; 菜单项目"About",列出插件信息.   
invoke  MessageBox, hwmain, CTEXT("Bookmark plugin v1.08",13,10,"(demonstration of plugin capabilities)",13,10,"Copyright (C) 2001, 2002 Oleh Yuschuk",13,10,"MASM32 version by Eviloid, 2oo3"),\  
CTEXT("Bookmark plugin"),MB_OK or MB_ICONINFORMATION  
.ENDIF  
.ELSEIF eax == PM_DISASM  
mov     eax, action  
.IF eax >= 0 && eax < 10 ; Insert bookmark  
mov     mark.index, eax  
mov     mark.size_, 1  
mov     mark.type_, 0  
mov     eax, item  
m2m     mark.addr_, \[eax\].t_dump.sel0  
invoke  Addsorteddata, offset bookmark.data, addr mark  

.IF bookmark.hw  
invoke  InvalidateRect, bookmark.hw, NULL, FALSE  
.ENDIF  

.ELSEIF eax >= 10 && eax < 20 ; Delete bookmark  
sub     eax, 10  
mov     ebx, eax  
invoke  Findsorteddata, offset bookmark.data, eax  
.IF eax  
invoke  Deletesorteddata, offset bookmark.data, ebx  
.IF bookmark.hw  
invoke  InvalidateRect, bookmark.hw, NULL, FALSE  
.ENDIF  
.ENDIF  

.ELSEIF eax >= 20 && eax < 30 ; Go to bookmark  
sub     eax, 20  
; mov     ebx, eax  
```



# 文档结构说明

## 1. 汇编代码片段

```asm
        invoke  Findsorteddata, offset bookmark.data, eax
        .IF eax
            invoke  Setcpu, 0, \[eax\].t_bookmark.addr_, 0, 0, CPU_ASMHIST or CPU_ASMCENTER or CPU_ASMFOCUS
        .ENDIF
    .ENDIF
.ENDIF
ret
ODBG_Pluginaction endp

end DllEntryPoint
```

这个是我在 `plugin.inc` 中弄出来对照的的

## 2. 结构定义

- 排列测试结构：`t_aligntest`
- 字符串搜索结构：`t_hexstr`
- 汇编命令搜索模块结构：`t_asmmodel`
- 扩展命令搜索模块结构：`t_extmodel`
- 颜色配置结构：`t_scheme`
- 可选择颜色配置结构：`t_schemeopt`
- bar 结构：`t_bar`
- 分类数据域 header 结构：`t_sortheader`
- 广义的分类表结构：`t_sorted`
- 使用分类数据和 bar 窗口结构：`t_table`
- 上下文(context)引用结构：`t_reg`
- 操作数结构：`t_operand`
- 汇编监控结构：`t_disasm`
- 表达式赋值的 Result 结构：`t_result`
- 活动线程信息结构：`t_thread`
- 内存描述块结构：`t_memory`
- 广义堆块结构：`t_heap`
- 指向字符串资源的结构：`t_stringtable`
- 关于固定或修正的结构：`t_fixup`
- 调试数据符号变量结构：`t_symvar`
- 跳转数据元素结构：`t_jdest`
- 可执行模块描述结构：`t_module`
- 当前或通用状况 dump 数据的结构：`t_dump`
- 参考描述结构：`t_ref`
- INT3 断点描述结构：`t_bpoint`
- 硬件断点结构：`t_hardbpoint`
- 注入代码描述结构：`t_inject`
- 资源排序描述符结构：`t_sourceline`
- 窗口描述结构：`t_window`

## 3. 函数定义

### 3.1 常规函数

- 添加到序列：`Addtolist`
- 更新序列：`Updatelist`
- 建立索引窗口：`Createlistwindow`
- 错误：`Error`
- 消息：`Message`
- 信息线路：`Infoline`
- 前进：`Progress`
- 反射：`Flash`

### 3.2 数据格式化函数

- `Decodeaddress` proto C `addr_:dword`, `base:dword`, `addrmode:dword`, `symb:ptr byte`, `nsymb:dword`, `comment_:ptr byte`
- `Decoderelativeoffset` proto C `addr_:dword`, `addrmode:dword`, `symb:ptr byte`, `nsymb:dword`
- `Decodecharacter` proto C `s:ptr byte`, `c_:dword`
- `Printfloat4` proto C `s:ptr byte`, `f:real4`
- `Printfloat8` proto C `s:ptr byte`, `double_1:dword`, `double_2:dword`
- `Printfloat10` proto C `s:ptr byte`, `double_1:dword`, `double_2:dword`
- `Print3dnow` proto `s:ptr byte`, `f:ptr byte`
- `Printsse` proto `s:ptr byte`, `f:ptr byte`
- `Followcall` proto `addr_:dword`
- `IstextA` proto C `c_:byte`
- `IstextW` proto C `w:word`
- `Stringtotext` proto C `data:ptr byte`, `ndata:dword`, `text:ptr byte`, `ntext:dword`

### 3.3 数据输入函数

- `Getlong` proto C `title_:ptr byte`, `data:dword`, `datasize:dword`, `letter:byte`, `mode:dword`
- `Getlongxy` proto C `title_:ptr byte`, `data:dword`, `datasize:dword`, `letter:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Getregxy` proto C `title_:ptr byte`, `data:dword`, `letter:byte`, `x:dword`, `y:dword`
- `Getline` proto C `title_:ptr byte`, `data:dword`
- `Getlinexy` proto C `title_:ptr byte`, `data:dword`, `x:dword`, `y:dword`
- `Getfloat10` proto C `title_:ptr byte`, `fdata:ptr real10`, `tag:ptr byte`, `letter:byte`, `mode:dword`
- `Getfloat10xy` proto C `title_:ptr byte`, `fdata:ptr real10`, `tag:ptr byte`, `letter:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Getfloat` proto C `title_:ptr byte`, `fdata:dword`, `size_:dword`, `letter:byte`, `mode:dword`
- `Getfloatxy` proto C `title_:ptr byte`, `fdata:dword`, `size_:dword`, `letter:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Getasmfindmodel` proto C `model_:ptr t_asmmodel`, `letter:byte`, `searchall:dword`
- `Getasmfindmodelxy` proto C `model_:ptr t_asmmodel`, `letter:byte`, `searchall:dword`, `x:dword`, `y:dword`
- `Gettext` proto C `title_:ptr byte`, `text:ptr byte`, `letter:byte`, `type_:dword`, `fontindex:dword`
- `Gettextxy` proto C `title_:ptr byte`, `text:ptr byte`, `letter:byte`, `type_:dword`, `fontindex:dword`, `x:dword`, `y:dword`
- `Gethexstring` proto C `title_:ptr byte`, `hs:ptr t_hexstr`, `mode:dword`, `fontindex:dword`, `letter:byte`
- `Gethexstringxy` proto C `title_:ptr byte`, `hs:ptr t_hexstr`, `mode:dword`, `fontindex:dword`, `letter:byte`, `x:dword`, `y:dword`
- `Getmmx` proto C `title_:ptr byte`, `data:byte`, `mode:dword`
- `Getmmxxy` proto C `title_:ptr byte`, `data:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Get3dnow` proto C `title_:ptr byte`, `data:byte`, `mode:dword`
- `Get3dnowxy` proto C `title_:ptr byte`, `data:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Browsefilename` proto C `title_:ptr byte`, `name_:ptr byte`, `defext:ptr byte`, `getarguments:dword`
- `OpenEXEfile` proto C `path:ptr byte`, `dropped:dword`
- `Animate` proto C `animation:dword`

### 3.4 绘制或绘制过程函数

- `SORTFUNC` typedef proto `:ptr t_sortheader`, `:ptr t_sortheader`, `:dword`
- `PSORTFUNC` typedef ptr `SORTFUNC`
- `DRAWFUNC` typedef proto `:ptr byte`, `:ptr byte`, `:ptr dword`, `:ptr t_sortheader`, `:dword`
- `PDRAWFUNC` typedef ptr `DRAWFUNC`
- `DESTFUNC` typedef proto `:ptr t_sortheader`
- `PDESTFUNC` typedef ptr `DESTFUNC`

- `Createsorteddata` proto C `sd:ptr t_sorted`, `name_:ptr byte`, `itemsize:dword`, `nmax:dword`, `sortfunc:PSORTFUNC`, `destfunc:PDESTFUNC`
- `Destroysorteddata` proto C `sd:ptr t_sorted`
- `Addsorteddata` proto C `sd:ptr t_sorted`, `item:dword`
- `Deletesorteddata` proto C `sd:ptr t_sorted`, `addr_:dword`
- `Deletesorteddatarange` proto C `sd:ptr t_sorted`, `addr0:dword`, `addr1:dword`
- `Deletenonconfirmedsorteddata` proto C `sd:ptr t_sorted`
- `Findsorteddata` proto C `sd:ptr t_sorted`, `addr_:dword`
- `Findsorteddatarange` proto C `sd:ptr t_sorted`, `addr0:dword`, `addr1:dword`
- `Findsorteddataindex` proto C `sd:ptr t_sorted`, `addr0:dword`, `addr1:dword`
- `Sortsorteddata` proto C `sd:ptr t_sorted`, `sort:dword`
- `Getsortedbyselection` proto C `sd:ptr t_sorted`, `index:dword`
- `Defaultbar` proto C `pb:ptr t_bar`
- `Tablefunction` proto C `pt:ptr t_table`, `hw:HWND`, `msg:UINT`, `wp:WPARAM`, `lp:LPARAM`
- `Painttable` proto C `hw:HWND`, `pt:ptr t_table`, `getline:DRAWFUNC`
- `Gettableselectionxy` proto C `pt:ptr t_table`, `column:dword`, `px:ptr dword`, `py:ptr dword`
- `Selectandscroll` proto C `pt:ptr t_table`, `index:dword`, `mode:dword`

### 3.5 name 函数

- `Insertname` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Quickinsertname` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Mergequicknames` proto C
- `Discardquicknames` proto C
- `Findname` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Decodename` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Findnextname` proto C `name_:ptr byte`
- `Findlabel` proto C `addr_:dword`, `name_:ptr byte`
- `Deletenamerange` proto C `addr0:dword`, `addr1:dword`, `type_:dword`
- `Findlabelbyname` proto C `name_:ptr byte`, `addr_:ptr dword`, `addr0:dword`, `addr1:dword`
- `Findimportbyname` proto C `name_:ptr byte`, `addr0:dword`, `addr1:dword`
- `Demanglename` proto C `name_:ptr byte`, `type_:dword`, `undecorated:ptr byte`
- `Findsymbolicname` proto C `addr_:dword`, `fname:ptr byte`

### 3.6 解构函数

- `Disasm` proto C `src:ptr byte`, `srcsize:dword`, `srcip:dword`, `srcdec:ptr byte`, `disasm:ptr t_disasm`, `disasmmode:dword`, `threadid:dword`
- `Disassembleback` proto C `block:ptr byte`, `base:dword`, `size_:dword`, `ip:dword`, `n:dword`, `usedec:dword`
- `Disassembleforward` proto C `block:ptr byte`, `base:dword`, `size_:dword`, `ip:dword`, `n:dword`, `usedec:dword`
- `Issuspicious` proto C `cmd:ptr byte`, `size_:dword`, `ip:dword`, `threadid:dword`, `preg:ptr t_reg`, `s:ptr byte`
- `Isfilling` proto C `offset_:dword`, `data:ptr byte`, `size_:dword`, `align_:dword`

### 3.7 汇编函数



# 线程函数

8. `Createthreadwindow` proto C  
   `Findthread` proto `threadid:dword`  
   `Decodethreadname` proto C `s:dword`, `threadid:dword`, `mode_:dword`  
   `Getcputhreadid` proto C  
   `Runsinglethread` proto C `threadid:dword`  
   `Restoreallthreads` proto C  

# 内存函数

9. `Findmemory` proto C  
   `Guardmemory` proto C `base:dword`, `size_:dword`, `guard:dword`  
   `Havecopyofmemory` proto C `copy:ptr byte`, `base:dword`, `size_:dword`  
   `Readmemory` proto C `buf:dword`, `addr_:dword`, `size_:dword`, `mode:dword`  
   `Writememory` proto C `buf:dword`, `addr_:dword`, `size_:dword`, `mode:dword`  
   `Readcommand` proto C `ip:dword`, `cmd:ptr byte`  

# 模块函数

10. `Findmodule` proto C `addr_:dword`  
   `Findfixup` proto C `pmod:ptr t_module`, `addr_:dword`  
   `Finddecode` proto C `addr_:dword`, `psize:ptr dword`  
   `Findfileoffset` proto C `pmod:ptr t_module`, `addr_:dword`  
   `Decoderange` proto C `addr_:dword`, `size_:dword`, `s:ptr byte`  
   `Analysecode` proto C `pm:ptr t_module`  

# dump 函数

11. `SPECFUNC` typedef proto `:ptr byte`, `:dword`, `:dword`, `:dword`, `:ptr t_disasm`, `:dword`  
   `PSPECFUNC` typedef `ptr SPECFUNC`  

# 窗口函数

12. `Registerotclass` proto C `classname:ptr byte`, `iconname:ptr byte`, `classproc:dword`  
   `Newtablewindow` proto C `pt:ptr t_table`, `nlines:dword`, `maxcolumns:dword`, `winclass:ptr byte`, `wintitle:ptr byte`  
   `Quicktablewindow` proto C `pt:ptr t_table`, `nlines:dword`, `maxcolumns:dword`, `winclass:ptr byte`, `wintitle:ptr byte`  
   `Createdumpwindow` proto C `name_:ptr byte`, `base:dword`, `size_:dword`, `addr_:dword`, `type_:dword`, `specdump:PSPECFUNC`  
   `Setdumptype` proto C `pd:ptr t_dump`, `dumptype:dword`  
   `Dumpbackup` proto `pd:ptr t_dump`, `action:dword`  
   `Broadcast` proto C `msg:UINT`, `wp:WPARAM`, `lp:LPARAM`  

# 数据转换函数

13. `Compress` proto C `bufin:ptr byte`, `nbufin:dword`, `bufout:ptr byte`, `nbufout:dword`  
   `Getoriginaldatasize` proto C `bufin:ptr byte`, `nbufin:dword`  
   `Decompress` proto C `bufin:ptr byte`, `nbufin:dword`, `bufout:ptr byte`, `nbufout:dword`  
   `Calculatecrc` proto C `copy:ptr byte`, `base:dword`, `size_:dword`, `pmod:ptr t_module`, `fixupoffset:dword`  

# 参考和搜索函数

14. `Findreferences` proto C `base:dword`, `size_:dword`, `addr0:dword`, `addr1:dword`, `origin:dword`, `recurseonjump:dword`, `title_:ptr byte`  
   `Findstrings` proto C `base:dword`, `size_:dword`, `origin:dword`, `title_:ptr byte`  
   `Findalldllcalls` proto C `pd:ptr t_dump`, `origin:dword`, `title_:ptr byte`  
   `Findallcommands` proto C `pd:ptr t_dump`, `model_:ptr t_asmmodel`, `origin:dword`, `title_:ptr byte`  
   `Findallsequences` proto C `pd:ptr t_dump`, `model_:ptr t_extmodel`, `origin:dword`, `title_:ptr byte`  
   `Walkreference` proto C `dir:dword`  
   `Walkreferenceex` proto C `dir:dword`, `size_:dword`  

# 断点和追踪函数

15. `Setbreakpoint` proto C `addr_:dword`, `type_:dword`, `cmd:byte`  
   `Manualbreakpoint` proto C `addr_:dword`, `key:dword`, `shiftkey:dword`, `nametype:dword`, `font:dword`  
   `Deletebreakpoints` proto C `addr0:dword`, `addr1:dword`, `silent:dword`  
   `Getbreakpointtype` proto `addr_:dword`  
   `Getnextbreakpoint` proto C `addr_:dword`, `type_:ptr dword`, `cmd:ptr dword`  
   `Tempbreakpoint` proto C `addr_:dword`, `mode:dword`  
   `Hardbreakpoints` proto C `closeondelete:dword`  
   `Sethardwarebreakpoint` proto C `addr_:dword`, `size_:dword`, `type_:dword`  
   `Deletehardwarebreakpoint` proto C `index:dword`  
   `Deletehardwarebreakbyaddr` proto C `addr_:dword`  
   `Setmembreakpoint` proto C `type_:dword`, `addr_:dword`, `size_:dword`  
   `Findhittrace` proto C `addr_:dword`, `ptracecopy:ptr LPBYTE`, `psize:ptr byte`  
   `Modifyhittrace` proto C `addr0:dword`, `addr1:dword`, `mode:dword`  
   `Isretaddr` proto C `retaddr:dword`, `procaddr:LPDWORD`  
   `Creatertracewindow` proto C  
   `Settracecondition` proto C `cond:ptr byte`, `onsuspicious:dword`, `in0:dword`, `in1:dword`, `out0:dword`, `out1:dword`  
   `Startruntrace` proto C `preg:ptr t_reg`  
   `Deleteruntrace` proto C  
   `Runtracesize` proto C  
   `Findprevruntraceip` proto C `ip:dword`, `startback:dword`  
   `Findnextruntraceip` proto C `ip:dword`, `startback:dword`  
   `Getruntraceregisters` proto C `nback:dword`, `preg:ptr t_reg`, `pold:ptr t_reg`, `cmd:ptr byte`, `comment_:ptr byte`  
   `Getruntraceprofile` proto C `addr_:dword`, `size_:dword`, `profile:LPDWORD`  
   `Scrollruntracewindow` proto C `back:dword`  
   `Createprofilewindow` proto C `base:dword`, `size_:dword`  

# 代码注入函数

16. `Injectcode` proto C `threadid:dword`, `inject:ptr t_inject`, `data:ptr byte`, `datasize:dword`, `parm1:dword`, `parm2:dword`, `answerfunc: PINJECTANSWER`  

# CPU 细节函数

17. `Setcpu` proto C `threadid:dword`, `asmaddr:dword`, `dumpaddr:dword`, `stackaddr:dword`, `mode:dword`  
   `Setdisasm` proto C `asmaddr:dword`, `selsize:dword`, `mode:dword`  
   `Redrawdisassembler` proto C  
   `Getdisassemblerrange` proto C `pbase:LPDWORD`, `psize:LPDWORD`  
   `Findprocbegin` proto C `addr_:dword`  
   `Findprocend` proto C `addr_:dword`  
   `Findprevproc` proto C `addr_:dword`  
   `Findnextproc` proto C `addr_:dword`  
   `Getproclimits` proto C `addr_:dword`, `start:LPDWORD`, `end_:LPDWORD`  
   `Sendshortcut` proto C `where:dword`, `addr_:dword`, `msg_:dword`, `ctrl_:dword`, `shift_:dword`, `vkcode_:dword`  

# 进程函数

18. `Getstatus` proto C  
   `Go` proto `threadid:dword`, `tilladdr:dword`, `stepmode:dword`, `givechance:dword`, `backupregs:dword`  
   `Suspendprocess` proto C `processevents:dword`  

# 解码和论据函数

19. `Findknownfunction` proto C `addr_:dword`, `direct:dword`, `level:dword`, `fname:ptr byte`  
   `Decodeknownargument` proto C `addr_:dword`, `arg_:ptr byte`, `value:dword`, `valid:dword`, `s:ptr byte`, `mask_:ptr byte`, `pset:ptr byte`  
   `Findunknownfunction` proto C `ip:dword`, `code_:ptr byte`, `dec_:ptr byte`, `size_:dword`, `fname:ptr byte`  
   `Decodeascii` proto C `value:dword`, `s:ptr byte`, `len:dword`, `mode:dword`  
   `Decodeunicode` proto C `value:dword`, `s:ptr byte`, `len:dword`  

# 源代码支持函数

20. `Showsourcefromaddress` proto C `addr_:dword`, `show:dword`  
   `Getresourcestring` proto C `pm:ptr t_module`, `id:dword`, `s:ptr byte`  
   `Getlinefromaddress` proto C `addr_:dword`  
   `Getaddressfromline` proto C `addr0:dword`, `addr1:dword`, `path:ptr byte`, `line:dword`  
   `Getsourcefilelimits` proto C `nameaddr:dword`, `addr0:LPDWORD`, `addr1:LPDWORD`  
   `Decodefullvarname` proto C `pmod:ptr t_module`, `psym:ptr t_symvar`, `offset_:dword`, `name_:ptr byte`  
   `Getbprelname` proto C `pmod:ptr t_module`, `addr_:dword`, `offset_:dword`, `s:ptr byte`, `nsymb:dword`  

# 监控支持函数

21. `Createwatchwindow` proto C  
   `Deletewatch` proto C `indexone:dword`  
   `Insertwatch` proto C `indexone:dword`, `text:ptr byte`  
   `Getwatch` proto C `indexone:dword`, `text:ptr byte`  

# 窗口细节函数

22. `Createwinwindow` proto C  

# PLUGIN 细节函数

23. `Registerpluginclass` proto C `classname:ptr byte`, `iconname:ptr byte`, `dllinst:HINSTANCE`, `classproc:dword`  
   `Unregisterpluginclass` proto C `classname:ptr byte`  
   `Pluginwriteinttoini` proto C `dllinst:HINSTANCE`, `key:ptr byte`, `value:dword`  
   `Pluginwritestringtoini` proto `dllinst:HINSTANCE`, `key:ptr byte`, `s:ptr byte`  
   `Pluginreadintfromini` proto C `dllinst:HINSTANCE`, `key:ptr byte`, `def:dword`  
   `Pluginreadstringfromini` proto C `dllinst:HINSTANCE`, `key:ptr byte`, `s:ptr byte`, `def:ptr byte`  
   `Pluginsaverecord` proto C `tag:dword`, `size_:dword`, `data:dword`  
   `Plugingetvalue` proto C `type_:dword`



### 24. 输出 PLUGIN 回调函数

| 函数原型 |
| :--- |
| `_ODBG_Plugindata proto C shortname:ptr byte` |
| `_ODBG_Plugininit proto C ollydbgversion:dword, hw:HWND, features:ptr dword` |
| `_ODBG_Pluginmainloop proto C debugevent:ptr DEBUG_EVENT` |
| `_ODBG_Pluginsaveudd proto C pmod:ptr t_module, ismainmodule:dword` |
| `_ODBG_Pluginuddrecord proto C pmod:ptr t_module, ismainmodule:dword, tag:dword, size_:dword, data:dword` |
| `_ODBG_Pluginmenu proto C origin:dword, data:ptr byte, item:dword` |
| `_ODBG_Pluginaction proto C origin:dword, action:dword, item:dword` |
| `_ODBG_Pluginshortcut proto C origin:dword, ctrl:dword, alt:dword, shift:dword, key:dword, item:dword` |
| `_ODBG_Pluginreset proto C` |
| `_ODBG_Pluginclose proto C` |
| `_ODBG_Plugindestroy proto C` |

有错请原谅

4dd466d0e1e0cc123afbe504da2462ca
