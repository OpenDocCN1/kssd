
```asm
      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax== 40h

      lea eax,szPAGE\_EXECUTE\_READWRITE

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax== 80h

      lea eax,szPAGE\_EXECUTE\_WRITECOPY

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax== 1

      lea eax,szPAGE\_NOACCESS

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax== 2

      lea eax,szPAGE\_READONLY

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax== 4

      lea eax,szPAGE\_READWRITE

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax== 8

      lea eax,szPAGE\_WRITECOPY

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .else

      mov eax,dword ptr mbi+20

      call showeax

      lea eax,bufferl

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,5

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .endif

    mov eax,dword ptr mbi+24

    .if eax==1000000h

      lea eax,szMEM\_IMAGE

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,6

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax==40000h

      lea eax,szMEM\_MAPPED

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,6

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .elseif eax==20000h

      lea eax,szMEM\_PRIVATE

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,6

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .else

      ;mov eax,dword ptr [mbi.lType];Type

      mov eax,dword ptr mbi+24

      ;mov eax,[eax+24]

      call showeax

      lea eax,bufferl

      mov lvi.imask,LVIF\_TEXT

      mov lvi.pszText,eax

      mov lvi.iSubItem,6

      invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi

    .endif

    mov eax,mbi.RegionSize

    add eax,x

    mov x,eax

  .endw

  next:

  INVOKE CreateWindowEx,WS\_EX\_CLIENTEDGE,addr szstatic,0,WS\_CHILD or WS\_VISIBLE or SS\_LEFT or WS\_GROUP,0,350,700,150,hWnd,0,hInstance,0

  MOV hstatic,EAX

  invoke SendMessage,eax,WM\_SETTEXT,0,addr notice

.else

  invoke DefWindowProc,hWnd,uMsg,wParam,lParam

  ret;不能去掉。

.endif

xor eax,eax;这两行可以去掉?

ret

liuchunli endp

start:

invoke GetSystemInfo,addr lpsi

invoke wsprintf,addr szBuffer,addr szsi,\

       ;lpsi.wProcessorArchitecture,\;

       lpsi.dwPageSize,\

       lpsi.lpMinimumApplicationAddress,\

       lpsi.lpMaximumApplicationAddress,\

       lpsi.dwActiveProcessorMask,\

       lpsi.dwNumberOfProcessors ,\

       lpsi.dwProcessorType,\

       lpsi.dwAllocationGranularity,\

       lpsi.wProcessorLevel,\

       lpsi.wProcessorRevision

invoke MessageBox,0,addr szBuffer,addr szCaptionMain,0

mov    stMemInfo.dwLength,sizeof stMemInfo

invoke GlobalMemoryStatus,addr stMemInfo

invoke wsprintf,addr szBuffer,addr szInfo,\

       stMemInfo.dwTotalPhys,\

       stMemInfo.dwAvailPhys,\

       stMemInfo.dwTotalPageFile,\

       stMemInfo.dwAvailPageFile,\

       stMemInfo.dwMemoryLoad,\

       stMemInfo.dwTotalVirtual,\

       stMemInfo.dwAvailVirtual

invoke MessageBox,0,addr szBuffer,addr szCaptionMain,0

invoke InitCommonControls;少加这个按钮，SysListView32控件是建立不起来的。

invoke GetModuleHandle,0

mov hInstance,eax

mov pwndclassex+20,eax

invoke LoadIcon,hInstance,1;加载图标，在资源文件中定义。

mov pwndclassex+24,eax

invoke LoadCursor,0,32512;加载箭头鼠标。

mov pwndclassex+28,eax

invoke RegisterClassEx,addr pwndclassex

invoke CreateWindowEx,200h,offset szClassName,offset szCaptionMain,0Ca0000h,80000000h,80000000h,710,530,0,0,hInstance,0

;0Cf0000h,80000000h,80000000h,99h,99h,0,0,hInstance,0

mov hWinMain,eax

invoke ShowWindow,hWinMain,1;若不想显示，此行也可以去掉。

invoke UpdateWindow,hWinMain

again:invoke GetMessage,addr stMsg,0,0,0

  cmp eax,0

  je exit

  invoke TranslateMessage,addr stMsg;若不想处理字符信息，此行也可以去掉。此行主要是方便处理字符信息的，个人认为。

  invoke DispatchMessage,addr stMsg

jmp again

exit:invoke ExitProcess,0

end start

;made at 2011.03.02
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [3.rar](javascript:void(0);)

  （6.10kb，21次下载）
