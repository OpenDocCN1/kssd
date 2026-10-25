
```
                ; 分拆. 读内存代码. 最先确定可能的

                ; 代码大小.

            invoke  wsprintf, s, CTEXT("column %d"), eax

            invoke  lstrlen, s

        .ELSEIF eax == 3

            mov     eax, ph

            invoke  Findname, [eax].t\_bookmark.addr\_, NM\_COMMENT, s

        .ELSE

            xor     eax, eax

        .ENDIF

        ret

Bookmarkgettext endp

; 每个窗口类需要自己窗口程序. 无论标准的和定制的

; OllyDbg窗口必须通过一些系统和OllyDbg-defined定义的消息到

Bookmarkwinproc proc hw:HWND, msg:UINT, wp:WPARAM, lp:LPARAM

;LOCAL controlkey, shiftkey:dword

LOCAL menu:HMENU

;LOCAL pb:ptr t\_bookmark

        mov     eax, msg

        ; 标准消息. 你能及时处理他们,但要有绝对的把握-

        ; 总是通过他们到tablefunction().

        .IF eax == WM\_DESTROY || eax == WM\_MOUSEMOVE || eax == WM\_LBUTTONDOWN || \

            eax == WM\_LBUTTONDBLCLK || eax == WM\_LBUTTONUP || eax == WM\_RBUTTONDOWN || \

            eax == WM\_RBUTTONDBLCLK || eax == WM\_HSCROLL || eax == WM\_VSCROLL || eax == WM\_TIMER || eax == WM\_SYSKEYDOWN

            invoke  Tablefunction, offset bookmark, hw, msg, wp, lp

        ; 通过信息到defmdichildproc.

        ; 为卷轴和选择定制可靠消息.用户绘制(User-drawn)

        ; 窗口必须及时处理,标准ollydbg窗口没有额外

        ; 功能通知他们到tablefunction().

        .ELSEIF eax == WM\_USER\_SCR || eax == WM\_USER\_VABS || eax == WM\_USER\_VREL || \

                eax == WM\_USER\_VBYTE || eax == WM\_USER\_STS || eax == WM\_USER\_CNTS || eax == WM\_USER\_CHGS

            invoke  Tablefunction, offset bookmark, hw, msg, wp, lp

            ret

        .ELSEIF eax == WM\_USER\_MENU

            invoke  CreatePopupMenu

            mov     menu, eax

                ; 查找选定bookmark. 如果存在至少一个bookmark退出及被选定

                ; 一些用bookmark操作仅仅产生判断. 记录分类数据

                ; 有特殊分类表只在必要时更新.

                ; getsortedbyselection()做这个

                ; 有些数据整理函数

                ; 不工作,你必须call sortsorteddata(). 阅读文件!

            invoke  Getsortedbyselection, offset bookmark.data, bookmark.data.selected

            mov     esi, eax

            .IF menu && eax

                invoke  AppendMenu, menu, MF\_STRING, 1, CTEXT("&Follow",9,"Enter")

                invoke  AppendMenu, menu, MF\_STRING, 2, CTEXT("&Delete",9,"Del")

            .ENDIF

                ; 即使菜单无效,call tablefunction仍然有意义.

            invoke  Tablefunction, offset bookmark, hw, WM\_USER\_MENU, 0, menu

            push    eax

            mov     eax, menu

            .IF eax

                invoke  DestroyMenu, eax

            .ENDIF

            pop     eax

            .IF eax == 1        ;在分拆跟随bookmark Follow bookmark in Disassembler

                invoke  Setcpu, 0, [esi].t\_bookmark.addr\_, 0, 0, CPU\_ASMHIST or CPU\_ASMCENTER or CPU\_ASMFOCUS

            .ELSEIF eax == 2    ; 删除Delete bookmark

                invoke  Deletesorteddata, offset bookmark.data, [esi].t\_bookmark.index

                ; 无自动更新窗口,你自己do.

                invoke  InvalidateRect, hw, NULL, FALSE

            .ENDIF

            return  0

        .ELSEIF eax == WM\_USER\_DBLCLK

                ; 在分拆双精度行(Doubleclicking row)跟随bookmark.

            invoke  Getsortedbyselection, offset bookmark.data, bookmark.data.selected

            .IF eax

                invoke  Setcpu, 0,[eax].t\_bookmark.addr\_, 0, 0, CPU\_ASMHIST or CPU\_ASMCENTER or CPU\_ASMFOCUS

            .ENDIF

            return  1   ; Doubleclick 处理processed

        .ELSEIF eax == WM\_USER\_CHALL || eax == WM\_USER\_CHMEM

                ; 有些改变,重绘窗口.

            invoke  InvalidateRect, hw, NULL, FALSE

            return  0

        .ELSEIF eax == WM\_PAINT

                ; 由Painttable()绘制所有ollydbg窗口. 使定制

                ; 仅仅画出如果你有重要原因这样做.

            invoke  Painttable, hw, offset bookmark, offset Bookmarkgettext

            return  0

        .ENDIF

        invoke  DefMDIChildProc, hw, msg, wp, lp

        ret

Bookmarkwinproc endp

; 记录类型在ollydbg和插件中必须是唯一的. 最好

; 保证这是由OllDbg (Oleh Yuschuk)注册标明类型. 注册

; 完全免费,除电子邮件费用:)

TAG\_BOOKMARK        equ     0236D420Ah      ; Bookmark record type in .udd file

; 为了节省时间保存数据到.udd文件! 这是由call pluginsaverecord()

; 为每个数据项必须被保存. 全局性,面向过程的数据必须

; 主要.udd文件(名称由.exe确定); 模块相关资料必须

; 保存在模块文件. 勿忘保存各项相对地址到

; 模块基址, 即使模块变化也能使数据准确被恢复.

ODBG\_Pluginsaveudd proc C pmod:ptr t\_module, ismainmodule:dword

LOCAL data[2]:dword

        .IF !ismainmodule       ; 仅仅bookmarks保存到主文件Save bookmarks to main file only

            ret

        .ENDIF

         mov     esi, bookmark.data.data

         mov     ebx, bookmark.data.n

         .WHILE ebx > 0

             mov     eax, [esi].t\_bookmark.index

             mov     data[0], eax

             mov     eax, [esi].t\_bookmark.addr\_

             mov     data[1 \* sizeof dword], eax

             invoke  Pluginsaverecord, TAG\_BOOKMARK, sizeof data, addr data

             dec     ebx

             add     esi, sizeof t\_bookmark

        .ENDW

        ret

ODBG\_Pluginsaveudd endp

; ollydbg恢复数据.udd文件. 如果属于记录插件,它必须

; 处理记录并返回1 否则必须返回0通过纪录

; 其他插件. 注意模块由广义指向不完善的pmod,

; 即不一定都含有信息, 尤其

; 从.udd文件.

ODBG\_Pluginuddrecord proc C pmod:ptr t\_module, ismainmodule:dword, tag:dword, size\_:dword, data:dword

LOCAL mark:t\_bookmark

        mov     eax, ismainmodule

        .IF !eax

            return eax              ; 仅仅bookmarks保存到主文件Bookmarks saved in main file only

        .ENDIF

        .IF tag != TAG\_BOOKMARK

            return  0               ;标记符没有验证 Tag is not recognized

        .ENDIF

        mov     ecx, data

        mov     eax, [ecx]

        mov     mark.index, eax

        mov     mark.size\_, 1

        mov     mark.type\_, 0

        mov     eax, [ecx + 4]

        mov     mark.addr\_, eax

        invoke  Addsorteddata, offset bookmark.data, addr mark

        return  1                   ;记录进程  Record processed

ODBG\_Pluginuddrecord endp

; 当建立标准MDI窗口时ollydbg进行大部分工作. 插件只需要作为一个整体

; 描述栏目数量,窗口的性质和性能.

Createbookmarkwindow proc

        ; Describe table columns. Note that column names are pointers, so strings

        ; must exist as long as table itself.

        .IF !bookmark.bar.nbar

            ; Bar still uninitialized.

            mov     bookmark.bar.name\_, CTEXT("Bookmark")   ; Name of bookmark

            mov     bookmark.bar.defdx, 9

            mov     bookmark.bar.mode, 0

            mov     bookmark.bar.name\_[1 \* sizeof dword], CTEXT("Address") ; Bookmark address

            mov     bookmark.bar.defdx[1 \* sizeof dword], 9

            mov     bookmark.bar.mode[1 \* sizeof byte], 0

            mov     bookmark.bar.name\_[2 \* sizeof dword], CTEXT("Disassembly") ; Disassembled command

            mov     bookmark.bar.defdx[2 \* sizeof dword], 32

            mov     bookmark.bar.mode[2 \* sizeof byte], BAR\_NOSORT

            mov     bookmark.bar.name\_[3 \* sizeof dword], CTEXT("Comment")   ; Comment

            mov     bookmark.bar.defdx[3 \* sizeof dword], 256

            mov     bookmark.bar.mode[3 \* sizeof byte], BAR\_NOSORT

            mov     bookmark.bar.nbar, 4

            mov     bookmark.mode, TABLE\_COPYMENU or TABLE\_SORTMENU or TABLE\_APPMENU or TABLE\_SAVEPOS;

            mov     bookmark.drawfunc, offset Bookmarkgettext

        .ENDIF

        ; 如果窗口已经存在,quicktablewindow()不建立新窗口

        ; 但为了恢复和扩展现有顶步. 这是最简单的方法,

        ; newtablewindow()更为灵活,更复杂. 没有很重要的理由我不推荐

        ; 定制(插件绘制)窗口.

        invoke  Quicktablewindow, offset bookmark, 15, 4, offset bookmarkwinclass, CTEXT("Bookmarks")

        ret

Createbookmarkwindow endp

; 如果你定义odbg\_pluginmainloop,在ollydbg窗口这个函数将每次从循环主窗口被call.

;  如果这是从调试请求的一些调试事件

; ,debugevent指向它,否则NULL. 非必要

; 不要申明这个函数,因为这可能产生负面影响到

; 整体速度!

;ODBG\_Pluginmainloop proc C debugevent:ptr DEBUG\_EVENT

;ODBG\_Pluginmainloop endp

; 函数添加任一项目到ollydbg主菜单(origin=pm\_main)或popup

; 菜单在标准ollydbg窗口之一. 当插件要加上自己的菜单

; 项目,在数据中搜集菜单式样和返回1, 否则必须

; 返回0. 除了静态主菜单,插件不要添加非活动项目.

; 项目指数在0..63范围. 完全相同索引是明确允许的.

ODBG\_Pluginmenu proc C uses ebx origin:dword, data:ptr byte, item:dword

LOCAL pb:ptr t\_bookmark

LOCAL pd:ptr t\_dump
```
