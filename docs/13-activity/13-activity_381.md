

# 第一阶段 第四题 脱壳流程+脚本(9 楼)+修复工具源码(15 楼)

> 原文：[`www.pediy.com/kssd/pediy11/123520.html`](https://www.pediy.com/kssd/pediy11/123520.html)

贴一个我自己生成的导入表信息吧，应该和大家的都差不多  
  

代码:

```
KERNEL32.DLL
    \[1EF688\] :  DeleteFileA
    \[1EF68C\] :  lstrcatA
    \[1EF690\] :  lstrcpyA
    \[1EF694\] :  GetCurrentThreadId
    \[1EF698\] :  GetCurrentThread
    \[1EF69C\] :  lstrcmpiA
    \[1EF6A0\] :  lstrcmpA
    \[1EF6A4\] :  GlobalDeleteAtom
    \[1EF6A8\] :  GlobalAlloc
    \[1EF6AC\] :  GlobalLock
    \[1EF6B0\] :  GetModuleFileNameA
    \[1EF6B4\] :  CloseHandle
    \[1EF6B8\] :  WaitForSingleObject
    \[1EF6BC\] :  SetEvent
    \[1EF6C0\] :  SuspendThread
    \[1EF6C4\] :  CreateEventA
    \[1EF6C8\] :  FreeResource
    \[1EF6CC\] :  LoadResource
    \[1EF6D0\] :  FindResourceA
    \[1EF6D4\] :  LockResource
    \[1EF6D8\] :  GlobalFree
    \[1EF6DC\] :  GlobalUnlock
    \[1EF6E0\] :  GetLastError
    \[1EF6E4\] :  GetProfileStringA
    \[1EF6E8\] :  GetLocaleInfoW
    \[1EF6EC\] :  SetEnvironmentVariableA
    \[1EF6F0\] :  SetStdHandle
    \[1EF6F4\] :  GetUserDefaultLCID
    \[1EF6F8\] :  EnumSystemLocalesA
    \[1EF6FC\] :  GetLocaleInfoA
    \[1EF700\] :  IsValidCodePage
    \[1EF704\] :  IsValidLocale
    \[1EF708\] :  IsBadCodePtr
    \[1EF70C\] :  Sleep
    \[1EF710\] :  GetStringTypeW
    \[1EF714\] :  GetStringTypeA
    \[1EF718\] :  SetConsoleCtrlHandler
    \[1EF71C\] :  SetUnhandledExceptionFilter
    \[1EF720\] :  CompareStringW
    \[1EF724\] :  CompareStringA
    \[1EF728\] :  LCMapStringW
    \[1EF72C\] :  LCMapStringA
    \[1EF730\] :  VirtualAlloc
    \[1EF734\] :  HeapReAlloc
    \[1EF738\] :  HeapAlloc
    \[1EF73C\] :  FatalAppExitA
    \[1EF740\] :  VirtualFree
    \[1EF744\] :  HeapFree
    \[1EF748\] :  HeapCreate
    \[1EF74C\] :  HeapDestroy
    \[1EF750\] :  GetVersionExA
    \[1EF754\] :  GetEnvironmentVariableA
    \[1EF758\] :  GetFileType
    \[1EF75C\] :  SetHandleCount
    \[1EF760\] :  GetEnvironmentStringsW
    \[1EF764\] :  GetEnvironmentStrings
    \[1EF768\] :  FreeEnvironmentStringsW
    \[1EF76C\] :  FreeEnvironmentStringsA
    \[1EF770\] :  UnhandledExceptionFilter
    \[1EF774\] :  GetLocalTime
    \[1EF778\] :  GetSystemTime
    \[1EF77C\] :  GetTimeZoneInformation
    \[1EF780\] :  GetACP
    \[1EF784\] :  GetStdHandle
    \[1EF788\] :  DebugBreak
    \[1EF78C\] :  RaiseException
    \[1EF790\] :  ExitThread
    \[1EF794\] :  CreateThread
    \[1EF798\] :  HeapValidate
    \[1EF79C\] :  TerminateProcess
    \[1EF7A0\] :  ExitProcess
    \[1EF7A4\] :  GetCommandLineA
    \[1EF7A8\] :  GetStartupInfoA
    \[1EF7AC\] :  RtlUnwind
    \[1EF7B0\] :  lstrcpyW
    \[1EF7B4\] :  CopyFileA
    \[1EF7B8\] :  GlobalSize
    \[1EF7BC\] :  SetFileAttributesA
    \[1EF7C0\] :  SystemTimeToFileTime
    \[1EF7C4\] :  LocalFileTimeToFileTime
    \[1EF7C8\] :  GetFileSize
    \[1EF7CC\] :  GetTickCount
    \[1EF7D0\] :  lstrlenW
    \[1EF7D4\] :  FindResourceExA
    \[1EF7D8\] :  FormatMessageA
    \[1EF7DC\] :  GetDiskFreeSpaceA
    \[1EF7E0\] :  GetFileTime
    \[1EF7E4\] :  SetFileTime
    \[1EF7E8\] :  GetTempFileNameA
    \[1EF7EC\] :  GetFileAttributesA
    \[1EF7F0\] :  GetShortPathNameA
    \[1EF7F4\] :  GetStringTypeExA
    \[1EF7F8\] :  GetFullPathNameA
    \[1EF7FC\] :  GetVolumeInformationA
    \[1EF800\] :  FindFirstFileA
    \[1EF804\] :  GetProcAddress
    \[1EF808\] :  FindClose
    \[1EF80C\] :  lstrlenA
    \[1EF810\] :  MoveFileA
    \[1EF814\] :  SetEndOfFile
    \[1EF818\] :  UnlockFile
    \[1EF81C\] :  LockFile
    \[1EF820\] :  FlushFileBuffers
    \[1EF824\] :  SetFilePointer
    \[1EF828\] :  WriteFile
    \[1EF82C\] :  ReadFile
    \[1EF830\] :  CreateFileA
    \[1EF834\] :  GetCurrentProcess
    \[1EF838\] :  DuplicateHandle
    \[1EF83C\] :  SetErrorMode
    \[1EF840\] :  GetOEMCP
    \[1EF844\] :  GetCPInfo
    \[1EF848\] :  OutputDebugStringA
    \[1EF84C\] :  GetThreadLocale
    \[1EF850\] :  SizeofResource
    \[1EF854\] :  GetProfileIntA
    \[1EF858\] :  VirtualProtect
    \[1EF85C\] :  FileTimeToLocalFileTime
    \[1EF860\] :  FileTimeToSystemTime
    \[1EF864\] :  IsBadReadPtr
    \[1EF868\] :  IsBadWritePtr
    \[1EF86C\] :  IsBadStringPtrA
    \[1EF870\] :  IsBadStringPtrW
    \[1EF874\] :  GetProcessVersion
    \[1EF878\] :  GetCurrentDirectoryA
    \[1EF87C\] :  WritePrivateProfileStringA
    \[1EF880\] :  GetPrivateProfileStringA
    \[1EF884\] :  GetPrivateProfileIntA
    \[1EF888\] :  GlobalFlags
    \[1EF88C\] :  TlsGetValue
    \[1EF890\] :  LocalReAlloc
    \[1EF894\] :  TlsSetValue
    \[1EF898\] :  EnterCriticalSection
    \[1EF89C\] :  GlobalReAlloc
    \[1EF8A0\] :  LeaveCriticalSection
    \[1EF8A4\] :  TlsFree
    \[1EF8A8\] :  GlobalHandle
    \[1EF8AC\] :  DeleteCriticalSection
    \[1EF8B0\] :  TlsAlloc
    \[1EF8B4\] :  InitializeCriticalSection
    \[1EF8B8\] :  LocalAlloc
    \[1EF8BC\] :  LocalFree
    \[1EF8C0\] :  lstrcpynA
    \[1EF8C4\] :  MultiByteToWideChar
    \[1EF8C8\] :  WideCharToMultiByte
    \[1EF8CC\] :  InterlockedDecrement
    \[1EF8D0\] :  InterlockedIncrement
    \[1EF8D4\] :  MulDiv
    \[1EF8D8\] :  SetLastError
    \[1EF8DC\] :  ResumeThread
    \[1EF8E0\] :  GetThreadPriority
    \[1EF8E4\] :  SetThreadPriority
    \[1EF8E8\] :  LoadLibraryA
    \[1EF8EC\] :  FreeLibrary
    \[1EF8F0\] :  GetVersion
    \[1EF8F4\] :  GlobalGetAtomNameA
    \[1EF8F8\] :  GlobalAddAtomA
    \[1EF8FC\] :  GlobalFindAtomA
    \[1EF900\] :  GetModuleHandleA

USER32.DLL
    \[1EFAF8\] :  MessageBeep
    \[1EFAFC\] :  CopyAcceleratorTableA
    \[1EFB00\] :  CharNextA
    \[1EFB04\] :  GetDialogBaseUnits
    \[1EFB08\] :  DestroyCursor
    \[1EFB0C\] :  SetCursorPos
    \[1EFB10\] :  GetAsyncKeyState
    \[1EFB14\] :  GetClipboardFormatNameA
    \[1EFB18\] :  LoadStringA
    \[1EFB1C\] :  UnpackDDElParam
    \[1EFB20\] :  ReuseDDElParam
    \[1EFB24\] :  DestroyMenu
    \[1EFB28\] :  TranslateAcceleratorA
    \[1EFB2C\] :  LoadAcceleratorsA
    \[1EFB30\] :  GetWindowThreadProcessId
    \[1EFB34\] :  WaitMessage
    \[1EFB38\] :  ReleaseCapture
    \[1EFB3C\] :  OemToCharA
    \[1EFB40\] :  CharToOemA
    \[1EFB44\] :  CheckMenuRadioItem
    \[1EFB48\] :  GetMenuContextHelpId
    \[1EFB4C\] :  SetMenuContextHelpId
    \[1EFB50\] :  LoadMenuIndirectA
    \[1EFB54\] :  LoadMenuA
    \[1EFB58\] :  RemoveMenu
    \[1EFB5C\] :  ModifyMenuA
    \[1EFB60\] :  InsertMenuA
    \[1EFB64\] :  GetSubMenu
    \[1EFB68\] :  GetMenuItemInfoA
    \[1EFB6C\] :  GetMenuStringA
    \[1EFB70\] :  GetMenuState
    \[1EFB74\] :  GetMenuItemID
    \[1EFB78\] :  GetMenuItemCount
    \[1EFB7C\] :  GetMenuDefaultItem
    \[1EFB80\] :  SetMenuDefaultItem
    \[1EFB84\] :  EnableMenuItem
    \[1EFB88\] :  CheckMenuItem
    \[1EFB8C\] :  AppendMenuA
    \[1EFB90\] :  DeleteMenu
    \[1EFB94\] :  IsMenu
    \[1EFB98\] :  CreatePopupMenu
    \[1EFB9C\] :  CreateMenu
    \[1EFBA0\] :  ScrollDC
    \[1EFBA4\] :  GrayStringA
    \[1EFBA8\] :  GetTabbedTextExtentA
    \[1EFBAC\] :  DrawTextA
    \[1EFBB0\] :  DrawFocusRect
    \[1EFBB4\] :  DrawFrameControl
    \[1EFBB8\] :  DrawEdge
    \[1EFBBC\] :  DrawStateA
    \[1EFBC0\] :  DrawIcon
    \[1EFBC4\] :  InvertRect
    \[1EFBC8\] :  FrameRect
    \[1EFBCC\] :  FillRect
    \[1EFBD0\] :  ExcludeUpdateRgn
    \[1EFBD4\] :  WindowFromDC
    \[1EFBD8\] :  GetSysColorBrush
    \[1EFBDC\] :  SubtractRect
    \[1EFBE0\] :  UnionRect
    \[1EFBE4\] :  InflateRect
    \[1EFBE8\] :  SetRectEmpty
    \[1EFBEC\] :  SetRect
    \[1EFBF0\] :  PtInRect
    \[1EFBF4\] :  IsRectEmpty
    \[1EFBF8\] :  OpenIcon
    \[1EFBFC\] :  CloseWindow
    \[1EFC00\] :  LoadCursorA
    \[1EFC04\] :  PostThreadMessageA
    \[1EFC08\] :  GetWindowContextHelpId
    \[1EFC0C\] :  SendNotifyMessageA
    \[1EFC10\] :  GetForegroundWindow
    \[1EFC14\] :  SetForegroundWindow
    \[1EFC18\] :  ShowCaret
    \[1EFC1C\] :  HideCaret
    \[1EFC20\] :  SetCaretPos
    \[1EFC24\] :  GetCaretPos
    \[1EFC28\] :  CreateCaret
    \[1EFC2C\] :  GetClipboardViewer
    \[1EFC30\] :  GetClipboardOwner
    \[1EFC34\] :  GetOpenClipboardWindow
    \[1EFC38\] :  OpenClipboard
    \[1EFC3C\] :  SetClipboardViewer
    \[1EFC40\] :  ChangeClipboardChain
    \[1EFC44\] :  FlashWindow
    \[1EFC48\] :  WindowFromPoint
    \[1EFC4C\] :  SetParent
    \[1EFC50\] :  FindWindowA
    \[1EFC54\] :  ChildWindowFromPoint
    \[1EFC58\] :  ShowScrollBar
    \[1EFC5C\] :  GetNextDlgTabItem
    \[1EFC60\] :  GetNextDlgGroupItem
    \[1EFC64\] :  DlgDirSelectComboBoxExA
    \[1EFC68\] :  DlgDirSelectExA
    \[1EFC6C\] :  DlgDirListComboBoxA
    \[1EFC70\] :  DlgDirListA
    \[1EFC74\] :  GetDesktopWindow
    \[1EFC78\] :  SetCapture
    \[1EFC7C\] :  KillTimer
    \[1EFC80\] :  SetTimer
    \[1EFC84\] :  EnableScrollBar
    \[1EFC88\] :  RedrawWindow
    \[1EFC8C\] :  LockWindowUpdate
    \[1EFC90\] :  GetDCEx
    \[1EFC94\] :  ShowOwnedPopups
    \[1EFC98\] :  IsWindowVisible
    \[1EFC9C\] :  ValidateRgn
    \[1EFCA0\] :  CharUpperA
    \[1EFCA4\] :  InvalidateRect
    \[1EFCA8\] :  GetUpdateRgn
    \[1EFCAC\] :  GetUpdateRect
    \[1EFCB0\] :  UpdateWindow
    \[1EFCB4\] :  ReleaseDC
    \[1EFCB8\] :  IsWindowUnicode
    \[1EFCBC\] :  GetDC
    \[1EFCC0\] :  EndPaint
    \[1EFCC4\] :  BeginPaint
    \[1EFCC8\] :  ClientToScreen
    \[1EFCCC\] :  BringWindowToTop
    \[1EFCD0\] :  GetWindowRgn
    \[1EFCD4\] :  SetWindowRgn
    \[1EFCD8\] :  ArrangeIconicWindows
    \[1EFCDC\] :  IsZoomed
    \[1EFCE0\] :  HiliteMenuItem
    \[1EFCE4\] :  GetSystemMenu
    \[1EFCE8\] :  DrawMenuBar
    \[1EFCEC\] :  SetMenu
    \[1EFCF0\] :  GetMenu
    \[1EFCF4\] :  ShowWindow
    \[1EFCF8\] :  MoveWindow
    \[1EFCFC\] :  SetWindowTextA
    \[1EFD00\] :  IsDialogMessageA
    \[1EFD04\] :  ScrollWindowEx
    \[1EFD08\] :  IsDlgButtonChecked
    \[1EFD0C\] :  SetDlgItemTextA
    \[1EFD10\] :  SetDlgItemInt
    \[1EFD14\] :  GetDlgItemTextA
    \[1EFD18\] :  GetDlgItemInt
    \[1EFD1C\] :  CheckRadioButton
    \[1EFD20\] :  CheckDlgButton
    \[1EFD24\] :  LoadIconA
    \[1EFD28\] :  SendDlgItemMessageA
    \[1EFD2C\] :  GetClientRect
    \[1EFD30\] :  MapWindowPoints
    \[1EFD34\] :  GetSysColor
    \[1EFD38\] :  SetFocus
    \[1EFD3C\] :  AdjustWindowRectEx
    \[1EFD40\] :  ScreenToClient
    \[1EFD44\] :  EqualRect
    \[1EFD48\] :  DeferWindowPos
    \[1EFD4C\] :  BeginDeferWindowPos
    \[1EFD50\] :  CopyRect
    \[1EFD54\] :  EndDeferWindowPos
    \[1EFD58\] :  ScrollWindow
    \[1EFD5C\] :  GetScrollInfo
    \[1EFD60\] :  SetScrollInfo
    \[1EFD64\] :  GetScrollRange
    \[1EFD68\] :  SetScrollRange
    \[1EFD6C\] :  GetScrollPos
    \[1EFD70\] :  SetScrollPos
    \[1EFD74\] :  GetTopWindow
    \[1EFD78\] :  IsChild
    \[1EFD7C\] :  GetCapture
    \[1EFD80\] :  WinHelpA
    \[1EFD84\] :  wsprintfA
    \[1EFD88\] :  GetClassInfoA
    \[1EFD8C\] :  RegisterClassA
    \[1EFD90\] :  TrackPopupMenu
    \[1EFD94\] :  SetWindowPlacement
    \[1EFD98\] :  GetWindowTextLengthA
    \[1EFD9C\] :  GetWindowTextA
    \[1EFDA0\] :  GetClassNameA
    \[1EFDA4\] :  GetDlgCtrlID
    \[1EFDA8\] :  CreateWindowExA
    \[1EFDAC\] :  GetClassLongA
    \[1EFDB0\] :  SetPropA
    \[1EFDB4\] :  UnhookWindowsHookEx
    \[1EFDB8\] :  GetPropA
    \[1EFDBC\] :  CallWindowProcA
    \[1EFDC0\] :  RemovePropA
    \[1EFDC4\] :  DefWindowProcA
    \[1EFDC8\] :  GetMessageTime
    \[1EFDCC\] :  GetMessagePos
    \[1EFDD0\] :  SetWindowLongA
    \[1EFDD4\] :  RegisterWindowMessageA
    \[1EFDD8\] :  OffsetRect
    \[1EFDDC\] :  IntersectRect
    \[1EFDE0\] :  SystemParametersInfoA
    \[1EFDE4\] :  IsIconic
    \[1EFDE8\] :  GetWindowPlacement
    \[1EFDEC\] :  GetWindowRect
    \[1EFDF0\] :  MapDialogRect
    \[1EFDF4\] :  SetWindowPos
    \[1EFDF8\] :  GetWindow
    \[1EFDFC\] :  SetWindowContextHelpId
    \[1EFE00\] :  EndDialog
    \[1EFE04\] :  GetActiveWindow
    \[1EFE08\] :  SetActiveWindow
    \[1EFE0C\] :  CreateDialogIndirectParamA
    \[1EFE10\] :  DestroyWindow
    \[1EFE14\] :  GetDlgItem
    \[1EFE18\] :  TabbedTextOutA
    \[1EFE1C\] :  PostMessageA
    \[1EFE20\] :  IsWindow
    \[1EFE24\] :  GetMenuCheckMarkDimensions
    \[1EFE28\] :  LoadBitmapA
    \[1EFE2C\] :  SetMenuItemBitmaps
    \[1EFE30\] :  GetFocus
    \[1EFE34\] :  GetMessageA
    \[1EFE38\] :  TranslateMessage
    \[1EFE3C\] :  DispatchMessageA
    \[1EFE40\] :  GetKeyState
    \[1EFE44\] :  CallNextHookEx
    \[1EFE48\] :  ValidateRect
    \[1EFE4C\] :  DestroyIcon
    \[1EFE50\] :  RegisterClipboardFormatA
    \[1EFE54\] :  wvsprintfA
    \[1EFE58\] :  InSendMessage
    \[1EFE5C\] :  InvalidateRgn
    \[1EFE60\] :  IsClipboardFormatAvailable
    \[1EFE64\] :  PeekMessageA
    \[1EFE68\] :  GetCursorPos
    \[1EFE6C\] :  SetWindowsHookExA
    \[1EFE70\] :  GetWindowLongA
    \[1EFE74\] :  GetParent
    \[1EFE78\] :  GetLastActivePopup
    \[1EFE7C\] :  IsWindowEnabled
    \[1EFE80\] :  SendMessageA
    \[1EFE84\] :  MessageBoxA
    \[1EFE88\] :  EnableWindow
    \[1EFE8C\] :  SetCursor
    \[1EFE90\] :  PostQuitMessage
    \[1EFE94\] :  GetWindowDC
    \[1EFE98\] :  DefDlgProcA
    \[1EFE9C\] :  GetSystemMetrics
    \[1EFEA0\] :  ChildWindowFromPointEx
    \[1EFEA4\] :  UnregisterClassA

COMCTL32.DLL
    \[1EF1E0\] :  ImageList\_Merge
    \[1EF1E4\] :  ImageList\_Read
    \[1EF1E8\] :  ImageList\_Write
    \[1EF1EC\] :  8000000E
    \[1EF1F0\] :  ImageList\_GetImageCount
    \[1EF1F4\] :  ImageList\_Add
    \[1EF1F8\] :  ImageList\_AddMasked
    \[1EF1FC\] :  ImageList\_Remove
    \[1EF200\] :  ImageList\_Replace
    \[1EF204\] :  ImageList\_ReplaceIcon
    \[1EF208\] :  ImageList\_GetIcon
    \[1EF20C\] :  ImageList\_Draw
    \[1EF210\] :  ImageList\_SetBkColor
    \[1EF214\] :  ImageList\_GetBkColor
    \[1EF218\] :  ImageList\_SetOverlayImage
    \[1EF21C\] :  ImageList\_GetImageInfo
    \[1EF220\] :  ImageList\_BeginDrag
    \[1EF224\] :  ImageList\_EndDrag
    \[1EF228\] :  ImageList\_DragMove
    \[1EF22C\] :  ImageList\_SetDragCursorImage
    \[1EF230\] :  ImageList\_DragShowNolock
    \[1EF234\] :  ImageList\_GetDragImage
    \[1EF238\] :  ImageList\_DragEnter
    \[1EF23C\] :  ImageList\_DragLeave
    \[1EF240\] :  80000008
    \[1EF244\] :  PropertySheetA
    \[1EF248\] :  DestroyPropertySheetPage
    \[1EF24C\] :  CreatePropertySheetPageA
    \[1EF250\] :  8000000D
    \[1EF254\] :  ImageList\_Destroy
    \[1EF258\] :  ImageList\_Create
    \[1EF25C\] :  80000011
    \[1EF260\] :  ImageList\_LoadImageA

OLEDLG.DLL
    \[1F0198\] :  80000004
    \[1F019C\] :  80000009
    \[1F01A0\] :  80000005
    \[1F01A4\] :  80000006
    \[1F01A8\] :  80000007
    \[1F01AC\] :  80000003
    \[1F01B0\] :  80000008

OLEPRO32.DLL
    \[1EFA84\] :  800000FD

GDI32.DLL
    \[1EF2A8\] :  CreatePenIndirect
    \[1EF2AC\] :  ExtCreatePen
    \[1EF2B0\] :  CreateSolidBrush
    \[1EF2B4\] :  CreateHatchBrush
    \[1EF2B8\] :  CreateBrushIndirect
    \[1EF2BC\] :  CreatePatternBrush
    \[1EF2C0\] :  CreateDIBPatternBrushPt
    \[1EF2C4\] :  CreateFontIndirectA
    \[1EF2C8\] :  CreateFontA
    \[1EF2CC\] :  CreateBitmapIndirect
    \[1EF2D0\] :  SetBitmapBits
    \[1EF2D4\] :  GetBitmapBits
    \[1EF2D8\] :  SetBitmapDimensionEx
    \[1EF2DC\] :  GetBitmapDimensionEx
    \[1EF2E0\] :  CreateCompatibleBitmap
    \[1EF2E4\] :  CreateDiscardableBitmap
    \[1EF2E8\] :  CreatePalette
    \[1EF2EC\] :  CreateHalftonePalette
    \[1EF2F0\] :  GetPaletteEntries
    \[1EF2F4\] :  SetPaletteEntries
    \[1EF2F8\] :  AnimatePalette
    \[1EF2FC\] :  GetNearestPaletteIndex
    \[1EF300\] :  ResizePalette
    \[1EF304\] :  CreateRectRgn
    \[1EF308\] :  CreateRectRgnIndirect
    \[1EF30C\] :  CreateEllipticRgn
    \[1EF310\] :  CreateEllipticRgnIndirect
    \[1EF314\] :  CreatePolygonRgn
    \[1EF318\] :  CreatePolyPolygonRgn
    \[1EF31C\] :  CreateRoundRectRgn
    \[1EF320\] :  PathToRegion
    \[1EF324\] :  ExtCreateRegion
    \[1EF328\] :  GetRegionData
    \[1EF32C\] :  SetRectRgn
    \[1EF330\] :  CombineRgn
    \[1EF334\] :  EqualRgn
    \[1EF338\] :  OffsetRgn
    \[1EF33C\] :  GetRgnBox
    \[1EF340\] :  PtInRegion
    \[1EF344\] :  RectInRegion
    \[1EF348\] :  CreateDCA
    \[1EF34C\] :  CreateICA
    \[1EF350\] :  CreateCompatibleDC
    \[1EF354\] :  GetDeviceCaps
    \[1EF358\] :  GetBrushOrgEx
    \[1EF35C\] :  SetBrushOrgEx
    \[1EF360\] :  EnumObjects
    \[1EF364\] :  SelectObject
    \[1EF368\] :  GetNearestColor
    \[1EF36C\] :  RealizePalette
    \[1EF370\] :  UpdateColors
    \[1EF374\] :  GetBkColor
    \[1EF378\] :  GetBkMode
    \[1EF37C\] :  GetPolyFillMode
    \[1EF380\] :  GetROP2
    \[1EF384\] :  GetStretchBltMode
    \[1EF388\] :  GetTextColor
    \[1EF38C\] :  GetMapMode
    \[1EF390\] :  GetViewportOrgEx
    \[1EF394\] :  GetViewportExtEx
    \[1EF398\] :  GetWindowOrgEx
    \[1EF39C\] :  GetWindowExtEx
    \[1EF3A0\] :  DPtoLP
    \[1EF3A4\] :  LPtoDP
    \[1EF3A8\] :  FillRgn
    \[1EF3AC\] :  FrameRgn
    \[1EF3B0\] :  InvertRgn
    \[1EF3B4\] :  CreatePen
    \[1EF3B8\] :  PtVisible
    \[1EF3BC\] :  RectVisible
    \[1EF3C0\] :  GetCurrentPositionEx
    \[1EF3C4\] :  Arc
    \[1EF3C8\] :  GetObjectType
    \[1EF3CC\] :  Chord
    \[1EF3D0\] :  Ellipse
    \[1EF3D4\] :  Pie
    \[1EF3D8\] :  Polygon
    \[1EF3DC\] :  PolyPolygon
    \[1EF3E0\] :  Rectangle
    \[1EF3E4\] :  RoundRect
    \[1EF3E8\] :  PatBlt
    \[1EF3EC\] :  BitBlt
    \[1EF3F0\] :  StretchBlt
    \[1EF3F4\] :  GetPixel
    \[1EF3F8\] :  SetPixel
    \[1EF3FC\] :  FloodFill
    \[1EF400\] :  ExtFloodFill
    \[1EF404\] :  TextOutA
    \[1EF408\] :  GetTextExtentPoint32A
    \[1EF40C\] :  GetTextAlign
    \[1EF410\] :  GetTextFaceA
    \[1EF414\] :  GetTextMetricsA
    \[1EF418\] :  GetTextCharacterExtra
    \[1EF41C\] :  GetCharWidthA
    \[1EF420\] :  GetAspectRatioFilterEx
    \[1EF424\] :  Escape
    \[1EF428\] :  SetBoundsRect
    \[1EF42C\] :  GetBoundsRect
    \[1EF430\] :  ResetDCA
    \[1EF434\] :  GetOutlineTextMetricsA
    \[1EF438\] :  GetCharABCWidthsA
    \[1EF43C\] :  GetFontData
    \[1EF440\] :  GetKerningPairsA
    \[1EF444\] :  GetGlyphOutlineA
    \[1EF448\] :  StartDocA
    \[1EF44C\] :  StartPage
    \[1EF450\] :  EndPage
    \[1EF454\] :  SetAbortProc
    \[1EF458\] :  AbortDoc
    \[1EF45C\] :  EndDoc
    \[1EF460\] :  MaskBlt
    \[1EF464\] :  PlgBlt
    \[1EF468\] :  SetPixelV
    \[1EF46C\] :  AngleArc
    \[1EF470\] :  GetArcDirection
    \[1EF474\] :  PolyPolyline
    \[1EF478\] :  GetColorAdjustment
    \[1EF47C\] :  GetCurrentObject
    \[1EF480\] :  PolyBezier
    \[1EF484\] :  DrawEscape
    \[1EF488\] :  ExtEscape
    \[1EF48C\] :  GetCharABCWidthsFloatA
    \[1EF490\] :  GetCharWidthFloatA
    \[1EF494\] :  AbortPath
    \[1EF498\] :  BeginPath
    \[1EF49C\] :  CloseFigure
    \[1EF4A0\] :  EndPath
    \[1EF4A4\] :  FillPath
    \[1EF4A8\] :  FlattenPath
    \[1EF4AC\] :  GetMiterLimit
    \[1EF4B0\] :  GetPath
    \[1EF4B4\] :  SetMiterLimit
    \[1EF4B8\] :  StrokeAndFillPath
    \[1EF4BC\] :  StrokePath
    \[1EF4C0\] :  WidenPath
    \[1EF4C4\] :  GdiComment
    \[1EF4C8\] :  PlayEnhMetaFile
    \[1EF4CC\] :  DeleteDC
    \[1EF4D0\] :  SaveDC
    \[1EF4D4\] :  RestoreDC
    \[1EF4D8\] :  SelectPalette
    \[1EF4DC\] :  SetBkMode
    \[1EF4E0\] :  SetPolyFillMode
    \[1EF4E4\] :  SetROP2
    \[1EF4E8\] :  SetStretchBltMode
    \[1EF4EC\] :  SetMapMode
    \[1EF4F0\] :  SetViewportOrgEx
    \[1EF4F4\] :  OffsetViewportOrgEx
    \[1EF4F8\] :  SetViewportExtEx
    \[1EF4FC\] :  ScaleViewportExtEx
    \[1EF500\] :  SetWindowOrgEx
    \[1EF504\] :  OffsetWindowOrgEx
    \[1EF508\] :  SetWindowExtEx
    \[1EF50C\] :  ScaleWindowExtEx
    \[1EF510\] :  SelectClipRgn
    \[1EF514\] :  ExcludeClipRect
    \[1EF518\] :  IntersectClipRect
    \[1EF51C\] :  OffsetClipRgn
    \[1EF520\] :  MoveToEx
    \[1EF524\] :  LineTo
    \[1EF528\] :  SetTextAlign
    \[1EF52C\] :  SetTextJustification
    \[1EF530\] :  SetTextCharacterExtra
    \[1EF534\] :  SetMapperFlags
    \[1EF538\] :  ArcTo
    \[1EF53C\] :  SetArcDirection
    \[1EF540\] :  PolyDraw
    \[1EF544\] :  PolylineTo
    \[1EF548\] :  SetColorAdjustment
    \[1EF54C\] :  UnrealizeObject
    \[1EF550\] :  GetStockObject
    \[1EF554\] :  GetObjectA
    \[1EF558\] :  SetBkColor
    \[1EF55C\] :  CloseEnhMetaFile
    \[1EF560\] :  SetTextColor
    \[1EF564\] :  GetClipBox
    \[1EF568\] :  GetDCOrgEx
    \[1EF56C\] :  Polyline
    \[1EF570\] :  ExtTextOutA
    \[1EF574\] :  CreateBitmap
    \[1EF578\] :  CreateMetaFileA
    \[1EF57C\] :  CloseMetaFile
    \[1EF580\] :  CreateEnhMetaFileA
    \[1EF584\] :  PaintRgn
    \[1EF588\] :  PolyBezierTo
    \[1EF58C\] :  DeleteObject
    \[1EF590\] :  GetClipRgn
    \[1EF594\] :  SelectClipPath
    \[1EF598\] :  ExtSelectClipRgn
    \[1EF59C\] :  PlayMetaFileRecord
    \[1EF5A0\] :  EnumMetaFile
    \[1EF5A4\] :  PlayMetaFile
    \[1EF5A8\] :  StretchDIBits
    \[1EF5AC\] :  EnumFontFamiliesExA
    \[1EF5B0\] :  CopyMetaFileA
    \[1EF5B4\] :  DeleteMetaFile
    \[1EF5B8\] :  CreateDIBitmap
    \[1EF5BC\] :  GetTextExtentPointA

WINSPOOL.DRV
    \[1EFF90\] :  DocumentPropertiesA
    \[1EFF94\] :  OpenPrinterA
    \[1EFF98\] :  ClosePrinter

COMDLG32.DLL
    \[1EFFC8\] :  GetSaveFileNameA
    \[1EFFCC\] :  GetFileTitleA
    \[1EFFD0\] :  ChooseColorA
    \[1EFFD4\] :  PageSetupDlgA
    \[1EFFD8\] :  PrintDlgA
    \[1EFFDC\] :  CommDlgExtendedError
    \[1EFFE0\] :  GetOpenFileNameA

ADVAPI32.DLL
    \[1EF174\] :  RegSetValueA
    \[1EF178\] :  RegCreateKeyExA
    \[1EF17C\] :  RegOpenKeyExA
    \[1EF180\] :  RegQueryValueExA
    \[1EF184\] :  RegSetValueExA
    \[1EF188\] :  RegDeleteValueA
    \[1EF18C\] :  RegDeleteKeyA
    \[1EF190\] :  RegOpenKeyA
    \[1EF194\] :  RegEnumKeyA
    \[1EF198\] :  RegCreateKeyA
    \[1EF19C\] :  RegQueryValueA
    \[1EF1A0\] :  SetFileSecurityA
    \[1EF1A4\] :  GetFileSecurityA
    \[1EF1A8\] :  RegCloseKey

SHELL32.DLL
    \[1EFAB4\] :  SHGetFileInfoA
    \[1EFAB8\] :  DragQueryFileA
    \[1EFABC\] :  DragFinish
    \[1EFAC0\] :  DragAcceptFiles
    \[1EFAC4\] :  ExtractIconA

OLE32.DLL
    \[1F0014\] :  OleSetClipboard
    \[1F0018\] :  OleFlushClipboard
    \[1F001C\] :  CreateItemMoniker
    \[1F0020\] :  CreateGenericComposite
    \[1F0024\] :  CreateFileMoniker
    \[1F0028\] :  GetClassFile
    \[1F002C\] :  CreateStreamOnHGlobal
    \[1F0030\] :  CoGetMalloc
    \[1F0034\] :  StgCreateDocfile
    \[1F0038\] :  StgOpenStorage
    \[1F003C\] :  StgIsStorageFile
    \[1F0040\] :  OleLockRunning
    \[1F0044\] :  OleSetContainedObject
    \[1F0048\] :  OleCreateFromData
    \[1F004C\] :  OleCreateLinkFromData
    \[1F0050\] :  OleCreateStaticFromData
    \[1F0054\] :  OleCreateFromFile
    \[1F0058\] :  OleCreateLinkToFile
    \[1F005C\] :  OleCreate
    \[1F0060\] :  OleLoad
    \[1F0064\] :  OleSave
    \[1F0068\] :  StgIsStorageILockBytes
    \[1F006C\] :  GetHGlobalFromILockBytes
    \[1F0070\] :  OleGetIconOfClass
    \[1F0074\] :  WriteClassStm
    \[1F0078\] :  OleSaveToStream
    \[1F007C\] :  OleIsRunning
    \[1F0080\] :  OleQueryCreateFromData
    \[1F0084\] :  OleQueryLinkFromData
    \[1F0088\] :  OleIsCurrentClipboard
    \[1F008C\] :  OleSetMenuDescriptor
    \[1F0090\] :  DoDragDrop
    \[1F0094\] :  OleRegEnumVerbs
    \[1F0098\] :  OleRegGetMiscStatus
    \[1F009C\] :  OleCreateMenuDescriptor
    \[1F00A0\] :  OleDestroyMenuDescriptor
    \[1F00A4\] :  CreateOleAdviseHolder
    \[1F00A8\] :  CreateDataAdviseHolder
    \[1F00AC\] :  OleTranslateAccelerator
    \[1F00B0\] :  IsAccelerator
    \[1F00B4\] :  GetRunningObjectTable
    \[1F00B8\] :  CoLockObjectExternal
    \[1F00BC\] :  CreateBindCtx
    \[1F00C0\] :  OleDuplicateData
    \[1F00C4\] :  CoFreeUnusedLibraries
    \[1F00C8\] :  OleUninitialize
    \[1F00CC\] :  OleInitialize
    \[1F00D0\] :  CoDisconnectObject
    \[1F00D4\] :  OleRun
    \[1F00D8\] :  CoCreateInstance
    \[1F00DC\] :  CoTaskMemAlloc
    \[1F00E0\] :  CoTaskMemFree
    \[1F00E4\] :  StringFromGUID2
    \[1F00E8\] :  CreateILockBytesOnHGlobal
    \[1F00EC\] :  StgCreateDocfileOnILockBytes
    \[1F00F0\] :  StgOpenStorageOnILockBytes
    \[1F00F4\] :  CoGetClassObject
    \[1F00F8\] :  CLSIDFromString
    \[1F00FC\] :  CLSIDFromProgID
    \[1F0100\] :  CoRevokeClassObject
    \[1F0104\] :  CoRegisterClassObject
    \[1F0108\] :  CoRegisterMessageFilter
    \[1F010C\] :  ReleaseStgMedium
    \[1F0110\] :  CoTreatAsClass
    \[1F0114\] :  StringFromCLSID
    \[1F0118\] :  ReadFmtUserTypeStg
    \[1F011C\] :  ReadClassStg
    \[1F0120\] :  OleRegGetUserType
    \[1F0124\] :  WriteClassStg
    \[1F0128\] :  WriteFmtUserTypeStg
    \[1F012C\] :  OleGetClipboard
    \[1F0130\] :  SetConvertStg

OLEAUT32.DLL
    \[1EF9AC\] :  80000006
    \[1EF9B0\] :  8000000C
    \[1EF9B4\] :  80000005
    \[1EF9B8\] :  80000002
    \[1EF9BC\] :  80000018
    \[1EF9C0\] :  80000017
    \[1EF9C4\] :  80000013
    \[1EF9C8\] :  80000014
    \[1EF9CC\] :  80000012
    \[1EF9D0\] :  80000011
    \[1EF9D4\] :  80000028
    \[1EF9D8\] :  8000000F
    \[1EF9DC\] :  80000096
    \[1EF9E0\] :  80000095
    \[1EF9E4\] :  80000068
    \[1EF9E8\] :  80000071
    \[1EF9EC\] :  8000005E
    \[1EF9F0\] :  80000072
    \[1EF9F4\] :  8000001B
    \[1EF9F8\] :  80000025
    \[1EF9FC\] :  80000024
    \[1EFA00\] :  8000000A
    \[1EFA04\] :  800000B9
    \[1EFA08\] :  80000016
    \[1EFA0C\] :  80000010
    \[1EFA10\] :  80000027
    \[1EFA14\] :  80000026
    \[1EFA18\] :  8000000E
    \[1EFA1C\] :  80000007
    \[1EFA20\] :  800000A1
    \[1EFA24\] :  80000009
    \[1EFA28\] :  80000004
    \[1EFA2C\] :  80000019
    \[1EFA30\] :  80000094
    \[1EFA34\] :  8000001A
    \[1EFA38\] :  80000015

```

引用: *最初由 **findlakes**发布* ![查看帖子](img/showthread.php?p=878759#post878759) 我想知道 这题第 5 步  
  
谁没有使用 SpiNano.exe 修复  而是手工修复的。 

自己写的工具算不算工具修复？

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || fixgame.rar  
 |

脱壳流程  
准备：OD，FixGame.dll，fkpespin.txt  
  
后面两个看附件  
  
脱壳流程：  
OD 载入到入口，跑脚本 fkpespin.txt，停下来后，当前线程注入 FixGame.dll，执行完 DllMain 后，dump 下来，就完成了

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || test.rar  
 |

代码:

```
34501000    8B5424 04             MOV     EDX,DWORD PTR SS:\[ESP+4\]
34501004    83C8 FF               OR      EAX,FFFFFFFF
34501007    8A0A                  MOV     CL,BYTE PTR DS:\[EDX\]
34501009    84C9                  TEST    CL,CL
3450100B    74 27                 JE      SHORT 34501034
3450100D    81E1 FF000000         AND     ECX,0FF
34501013    33C1                  XOR     EAX,ECX
34501015    B9 08000000           MOV     ECX,8
3450101A    A8 01                 TEST    AL,1
3450101C    74 09                 JE      SHORT 34501027
3450101E    D1E8                  SHR     EAX,1
34501020    35 2083B8ED           XOR     EAX,EDB88320
34501025    EB 02                 JMP     SHORT 34501029
34501027    D1E8                  SHR     EAX,1
34501029    49                    DEC     ECX
3450102A  ^ 75 EE                 JNZ     SHORT 3450101A
3450102C    8A4A 01               MOV     CL,BYTE PTR DS:\[EDX+1\]
3450102F    42                    INC     EDX
34501030    84C9                  TEST    CL,CL
34501032  ^ 75 D9                 JNZ     SHORT 3450100D
34501034    C3                    RETN
34501035    90                    NOP
34501036    90                    NOP
34501037    90                    NOP
34501038    90                    NOP
34501039    90                    NOP
3450103A    90                    NOP
3450103B    90                    NOP
3450103C    90                    NOP
3450103D    90                    NOP
3450103E    90                    NOP
3450103F    90                    NOP
34501040    83EC 14               SUB     ESP,14
34501043    8D4424 00             LEA     EAX,DWORD PTR SS:\[ESP\]
34501047    53                    PUSH    EBX
34501048    55                    PUSH    EBP
34501049    56                    PUSH    ESI
3450104A    57                    PUSH    EDI
3450104B    50                    PUSH    EAX
3450104C    6A 40                 PUSH    40
3450104E    68 00301B00           PUSH    1B3000
34501053    68 00104000           PUSH    401000
34501058    FF15 00505034         CALL    DWORD PTR DS:\[34505000\]          ; kernel32.VirtualProtect
3450105E    8B6C24 2C             MOV     EBP,DWORD PTR SS:\[ESP+2C\]
34501062    BE 01104000           MOV     ESI,401001
34501067    B3 8B                 MOV     BL,8B
34501069    66:817E FF 8DC0       CMP     WORD PTR DS:\[ESI-1\],0C08D
3450106F    0F85 BB030000         JNZ     34501430
34501075    8D4E FF               LEA     ECX,DWORD PTR DS:\[ESI-1\]
34501078    8D5424 14             LEA     EDX,DWORD PTR SS:\[ESP+14\]
3450107C    51                    PUSH    ECX
3450107D    68 30605034           PUSH    34506030                         ; ASCII "%08X"
34501082    52                    PUSH    EDX
34501083    FF15 B4505034         CALL    DWORD PTR DS:\[345050B4\]          ; USER32.wsprintfA
34501089    8D4424 20             LEA     EAX,DWORD PTR SS:\[ESP+20\]
3450108D    50                    PUSH    EAX
3450108E    E8 6DFFFFFF           CALL    34501000
34501093    8BF8                  MOV     EDI,EAX
34501095    B8 A38B2EBA           MOV     EAX,BA2E8BA3
3450109A    F7E5                  MUL     EBP
3450109C    83C4 10               ADD     ESP,10
3450109F    33C9                  XOR     ECX,ECX
345010A1    C1EA 03               SHR     EDX,3
345010A4    0F84 86030000         JE      34501430
345010AA    8B4424 28             MOV     EAX,DWORD PTR SS:\[ESP+28\]
345010AE    3B38                  CMP     EDI,DWORD PTR DS:\[EAX\]
345010B0    74 0D                 JE      SHORT 345010BF
345010B2    41                    INC     ECX
345010B3    83C0 0B               ADD     EAX,0B
345010B6    3BCA                  CMP     ECX,EDX
345010B8  ^ 72 F4                 JB      SHORT 345010AE
345010BA    E9 71030000           JMP     34501430
345010BF    83F9 FF               CMP     ECX,-1
345010C2    0F84 68030000         JE      34501430
345010C8    8B4424 28             MOV     EAX,DWORD PTR SS:\[ESP+28\]
345010CC    8D1489                LEA     EDX,DWORD PTR DS:\[ECX+ECX\*4\]
345010CF    03C1                  ADD     EAX,ECX
345010D1    8A4C50 08             MOV     CL,BYTE PTR DS:\[EAX+EDX\*2+8\]
345010D5    84C9                  TEST    CL,CL
345010D7    8D0450                LEA     EAX,DWORD PTR DS:\[EAX+EDX\*2\]
345010DA    0F85 CB000000         JNZ     345011AB
345010E0    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345010E3    83E1 07               AND     ECX,7
345010E6    83F9 04               CMP     ECX,4
345010E9    75 64                 JNZ     SHORT 3450114F
345010EB    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
345010EE    80F9 03               CMP     CL,3
345010F1    75 14                 JNZ     SHORT 34501107
345010F3    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
345010F6    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345010F9    80E1 3F               AND     CL,3F
345010FC    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
34501100    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501102    E9 29030000           JMP     34501430
34501107    80F9 04               CMP     CL,4
3450110A    75 1D                 JNZ     SHORT 34501129
3450110C    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
3450110F    8A50 09               MOV     DL,BYTE PTR DS:\[EAX+9\]
34501112    80E2 3F               AND     DL,3F
34501115    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
34501119    80CA 40               OR      DL,40
3450111C    8816                  MOV     BYTE PTR DS:\[ESI\],DL
3450111E    8A40 04               MOV     AL,BYTE PTR DS:\[EAX+4\]
34501121    8846 02               MOV     BYTE PTR DS:\[ESI+2\],AL
34501124    E9 07030000           JMP     34501430
34501129    80F9 07               CMP     CL,7
3450112C    0F85 FE020000         JNZ     34501430
34501132    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
34501135    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501138    80E1 3F               AND     CL,3F
3450113B    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
3450113F    80C9 80               OR      CL,80
34501142    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501144    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
34501147    8956 02               MOV     DWORD PTR DS:\[ESI+2\],EDX
3450114A    E9 E1020000           JMP     34501430
3450114F    83F9 05               CMP     ECX,5
34501152    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
34501155    75 40                 JNZ     SHORT 34501197
34501157    80F9 03               CMP     CL,3
3450115A    75 19                 JNZ     SHORT 34501175
3450115C    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
3450115F    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501162    80E1 3F               AND     CL,3F
34501165    80C9 40               OR      CL,40
34501168    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450116A    8A50 04               MOV     DL,BYTE PTR DS:\[EAX+4\]
3450116D    8856 01               MOV     BYTE PTR DS:\[ESI+1\],DL
34501170    E9 BB020000           JMP     34501430
34501175    80F9 06               CMP     CL,6
34501178    0F85 B2020000         JNZ     34501430
3450117E    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
34501181    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501184    80E1 3F               AND     CL,3F
34501187    80C9 80               OR      CL,80
3450118A    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450118C    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
3450118F    8956 01               MOV     DWORD PTR DS:\[ESI+1\],EDX
34501192    E9 99020000           JMP     34501430
34501197    80F9 02               CMP     CL,2
3450119A  ^ 75 BB                 JNZ     SHORT 34501157
3450119C    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
3450119F    8A40 09               MOV     AL,BYTE PTR DS:\[EAX+9\]
345011A2    24 3F                 AND     AL,3F
345011A4    8806                  MOV     BYTE PTR DS:\[ESI\],AL
345011A6    E9 85020000           JMP     34501430
345011AB    80F9 01               CMP     CL,1
345011AE    0F85 B3000000         JNZ     34501267
345011B4    33C9                  XOR     ECX,ECX
345011B6    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345011B9    8BD1                  MOV     EDX,ECX
345011BB    C1E9 04               SHR     ECX,4
345011BE    83E2 07               AND     EDX,7
345011C1    83E1 07               AND     ECX,7
345011C4    8078 0A 02            CMP     BYTE PTR DS:\[EAX+A\],2
345011C8    0F85 62020000         JNZ     34501430
345011CE    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
345011D1    83F8 06               CMP     EAX,6
345011D4    0F83 56020000         JNB     34501430
345011DA    85C0                  TEST    EAX,EAX
345011DC    75 13                 JNZ     SHORT 345011F1
345011DE    80C9 F8               OR      CL,0F8
345011E1    C646 FF 09            MOV     BYTE PTR DS:\[ESI-1\],9
345011E5    C0E1 03               SHL     CL,3
345011E8    0ACA                  OR      CL,DL
345011EA    880E                  MOV     BYTE PTR DS:\[ESI\],CL
345011EC    E9 3F020000           JMP     34501430
345011F1    83F8 01               CMP     EAX,1
345011F4    75 13                 JNZ     SHORT 34501209
345011F6    80C9 F8               OR      CL,0F8
345011F9    C646 FF 21            MOV     BYTE PTR DS:\[ESI-1\],21
345011FD    C0E1 03               SHL     CL,3
34501200    0ACA                  OR      CL,DL
34501202    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501204    E9 27020000           JMP     34501430
34501209    83F8 02               CMP     EAX,2
3450120C    75 13                 JNZ     SHORT 34501221
3450120E    80C9 F8               OR      CL,0F8
34501211    C646 FF 33            MOV     BYTE PTR DS:\[ESI-1\],33
34501215    C0E1 03               SHL     CL,3
34501218    0ACA                  OR      CL,DL
3450121A    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450121C    E9 0F020000           JMP     34501430
34501221    83F8 03               CMP     EAX,3
34501224    75 12                 JNZ     SHORT 34501238
34501226    80C9 F8               OR      CL,0F8
34501229    8846 FF               MOV     BYTE PTR DS:\[ESI-1\],AL
3450122C    C0E1 03               SHL     CL,3
3450122F    0ACA                  OR      CL,DL
34501231    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501233    E9 F8010000           JMP     34501430
34501238    83F8 04               CMP     EAX,4
3450123B    75 13                 JNZ     SHORT 34501250
3450123D    80C9 F8               OR      CL,0F8
34501240    C646 FF 2B            MOV     BYTE PTR DS:\[ESI-1\],2B
34501244    C0E1 03               SHL     CL,3
34501247    0ACA                  OR      CL,DL
34501249    880E                  MOV     BYTE PTR DS:\[ESI\],CL
3450124B    E9 E0010000           JMP     34501430
34501250    83F8 05               CMP     EAX,5
34501253    75 03                 JNZ     SHORT 34501258
34501255    885E FF               MOV     BYTE PTR DS:\[ESI-1\],BL
34501258    80C9 F8               OR      CL,0F8
3450125B    C0E1 03               SHL     CL,3
3450125E    0ACA                  OR      CL,DL
34501260    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501262    E9 C9010000           JMP     34501430
34501267    80F9 02               CMP     CL,2
3450126A    75 6B                 JNZ     SHORT 345012D7
3450126C    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
3450126F    83E1 07               AND     ECX,7
34501272    83F9 04               CMP     ECX,4
34501275    75 20                 JNZ     SHORT 34501297
34501277    8078 0A 07            CMP     BYTE PTR DS:\[EAX+A\],7
3450127B    0F85 AF010000         JNZ     34501430
34501281    C646 FF C7            MOV     BYTE PTR DS:\[ESI-1\],0C7
34501285    C606 04               MOV     BYTE PTR DS:\[ESI\],4
34501288    C646 01 24            MOV     BYTE PTR DS:\[ESI+1\],24
3450128C    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
3450128F    8946 02               MOV     DWORD PTR DS:\[ESI+2\],EAX
34501292    E9 99010000           JMP     34501430
34501297    83F9 05               CMP     ECX,5
3450129A    75 20                 JNZ     SHORT 345012BC
3450129C    8078 0A 07            CMP     BYTE PTR DS:\[EAX+A\],7
345012A0    0F85 8A010000         JNZ     34501430
345012A6    C646 FF C7            MOV     BYTE PTR DS:\[ESI-1\],0C7
345012AA    C606 45               MOV     BYTE PTR DS:\[ESI\],45
345012AD    C646 01 00            MOV     BYTE PTR DS:\[ESI+1\],0
345012B1    8B48 04               MOV     ECX,DWORD PTR DS:\[EAX+4\]
345012B4    894E 02               MOV     DWORD PTR DS:\[ESI+2\],ECX
345012B7    E9 74010000           JMP     34501430
345012BC    8078 0A 06            CMP     BYTE PTR DS:\[EAX+A\],6
345012C0    0F85 6A010000         JNZ     34501430
345012C6    C646 FF C7            MOV     BYTE PTR DS:\[ESI-1\],0C7
345012CA    880E                  MOV     BYTE PTR DS:\[ESI\],CL
345012CC    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
345012CF    8956 01               MOV     DWORD PTR DS:\[ESI+1\],EDX
345012D2    E9 59010000           JMP     34501430
345012D7    80F9 03               CMP     CL,3
345012DA    75 5B                 JNZ     SHORT 34501337
345012DC    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
345012DF    80F9 02               CMP     CL,2
345012E2    75 23                 JNZ     SHORT 34501307
345012E4    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345012E7    84C9                  TEST    CL,CL
345012E9    75 0E                 JNZ     SHORT 345012F9
345012EB    C646 FF 74            MOV     BYTE PTR DS:\[ESI-1\],74
345012EF    8A40 04               MOV     AL,BYTE PTR DS:\[EAX+4\]
345012F2    8806                  MOV     BYTE PTR DS:\[ESI\],AL
345012F4    E9 37010000           JMP     34501430
345012F9    C646 FF 75            MOV     BYTE PTR DS:\[ESI-1\],75
345012FD    8A48 04               MOV     CL,BYTE PTR DS:\[EAX+4\]
34501300    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501302    E9 29010000           JMP     34501430
34501307    80F9 06               CMP     CL,6
3450130A    0F85 20010000         JNZ     34501430
34501310    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501313    C646 FF 0F            MOV     BYTE PTR DS:\[ESI-1\],0F
34501317    84C9                  TEST    CL,CL
34501319    75 0E                 JNZ     SHORT 34501329
3450131B    C606 84               MOV     BYTE PTR DS:\[ESI\],84
3450131E    8B50 04               MOV     EDX,DWORD PTR DS:\[EAX+4\]
34501321    8956 01               MOV     DWORD PTR DS:\[ESI+1\],EDX
34501324    E9 07010000           JMP     34501430
34501329    C606 85               MOV     BYTE PTR DS:\[ESI\],85
3450132C    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
3450132F    8946 01               MOV     DWORD PTR DS:\[ESI+1\],EAX
34501332    E9 F9000000           JMP     34501430
34501337    80F9 04               CMP     CL,4
3450133A    75 5B                 JNZ     SHORT 34501397
3450133C    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
3450133F    80F9 02               CMP     CL,2
34501342    75 22                 JNZ     SHORT 34501366
34501344    8078 09 05            CMP     BYTE PTR DS:\[EAX+9\],5
34501348    75 0E                 JNZ     SHORT 34501358
3450134A    C646 FF 72            MOV     BYTE PTR DS:\[ESI-1\],72
3450134E    8A48 04               MOV     CL,BYTE PTR DS:\[EAX+4\]
34501351    880E                  MOV     BYTE PTR DS:\[ESI\],CL
34501353    E9 D8000000           JMP     34501430
34501358    C646 FF 73            MOV     BYTE PTR DS:\[ESI-1\],73
3450135C    8A50 04               MOV     DL,BYTE PTR DS:\[EAX+4\]
3450135F    8816                  MOV     BYTE PTR DS:\[ESI\],DL
34501361    E9 CA000000           JMP     34501430
34501366    80F9 06               CMP     CL,6
34501369    0F85 C1000000         JNZ     34501430
3450136F    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
34501372    C646 FF 0F            MOV     BYTE PTR DS:\[ESI-1\],0F
34501376    80F9 05               CMP     CL,5
34501379    75 0E                 JNZ     SHORT 34501389
3450137B    C606 82               MOV     BYTE PTR DS:\[ESI\],82
3450137E    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
34501381    8946 01               MOV     DWORD PTR DS:\[ESI+1\],EAX
34501384    E9 A7000000           JMP     34501430
34501389    C606 83               MOV     BYTE PTR DS:\[ESI\],83
3450138C    8B48 04               MOV     ECX,DWORD PTR DS:\[EAX+4\]
3450138F    894E 01               MOV     DWORD PTR DS:\[ESI+1\],ECX
34501392    E9 99000000           JMP     34501430
34501397    80F9 05               CMP     CL,5
3450139A    75 27                 JNZ     SHORT 345013C3
3450139C    C646 FF FF            MOV     BYTE PTR DS:\[ESI-1\],0FF
345013A0    C606 24               MOV     BYTE PTR DS:\[ESI\],24
345013A3    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345013A6    8AD1                  MOV     DL,CL
345013A8    80E1 F1               AND     CL,0F1
345013AB    80E2 07               AND     DL,7
345013AE    D0E2                  SHL     DL,1
345013B0    0AD1                  OR      DL,CL
345013B2    C0E2 02               SHL     DL,2
345013B5    80CA 05               OR      DL,5
345013B8    8856 01               MOV     BYTE PTR DS:\[ESI+1\],DL
345013BB    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
345013BE    8946 02               MOV     DWORD PTR DS:\[ESI+2\],EAX
345013C1    EB 6D                 JMP     SHORT 34501430
345013C3    80F9 06               CMP     CL,6
345013C6    75 4B                 JNZ     SHORT 34501413
345013C8    8A48 0A               MOV     CL,BYTE PTR DS:\[EAX+A\]
345013CB    80F9 02               CMP     CL,2
345013CE    75 1C                 JNZ     SHORT 345013EC
345013D0    8078 09 07            CMP     BYTE PTR DS:\[EAX+9\],7
345013D4    75 0B                 JNZ     SHORT 345013E1
345013D6    C646 FF 76            MOV     BYTE PTR DS:\[ESI-1\],76
345013DA    8A48 04               MOV     CL,BYTE PTR DS:\[EAX+4\]
345013DD    880E                  MOV     BYTE PTR DS:\[ESI\],CL
345013DF    EB 4F                 JMP     SHORT 34501430
345013E1    C646 FF 77            MOV     BYTE PTR DS:\[ESI-1\],77
345013E5    8A50 04               MOV     DL,BYTE PTR DS:\[EAX+4\]
345013E8    8816                  MOV     BYTE PTR DS:\[ESI\],DL
345013EA    EB 44                 JMP     SHORT 34501430
345013EC    80F9 06               CMP     CL,6
345013EF    75 3F                 JNZ     SHORT 34501430
345013F1    8A48 09               MOV     CL,BYTE PTR DS:\[EAX+9\]
345013F4    C646 FF 0F            MOV     BYTE PTR DS:\[ESI-1\],0F
345013F8    80F9 07               CMP     CL,7
345013FB    75 0B                 JNZ     SHORT 34501408
345013FD    C606 86               MOV     BYTE PTR DS:\[ESI\],86
34501400    8B40 04               MOV     EAX,DWORD PTR DS:\[EAX+4\]
34501403    8946 01               MOV     DWORD PTR DS:\[ESI+1\],EAX
34501406    EB 28                 JMP     SHORT 34501430
34501408    C606 87               MOV     BYTE PTR DS:\[ESI\],87
3450140B    8B48 04               MOV     ECX,DWORD PTR DS:\[EAX+4\]
3450140E    894E 01               MOV     DWORD PTR DS:\[ESI+1\],ECX
34501411    EB 1D                 JMP     SHORT 34501430
34501413    80F9 07               CMP     CL,7
34501416    75 18                 JNZ     SHORT 34501430
34501418    8A50 0A               MOV     DL,BYTE PTR DS:\[EAX+A\]
3450141B    33C9                  XOR     ECX,ECX
3450141D    84D2                  TEST    DL,DL
3450141F    76 0F                 JBE     SHORT 34501430
34501421    33D2                  XOR     EDX,EDX
34501423    C6440E FF 90          MOV     BYTE PTR DS:\[ESI+ECX-1\],90
34501428    8A50 0A               MOV     DL,BYTE PTR DS:\[EAX+A\]
3450142B    41                    INC     ECX
3450142C    3BCA                  CMP     ECX,EDX
3450142E  ^ 7C F1                 JL      SHORT 34501421
34501430    46                    INC     ESI
34501431    8D46 FF               LEA     EAX,DWORD PTR DS:\[ESI-1\]
34501434    3D 00405B00           CMP     EAX,5B4000
34501439  ^ 0F82 2AFCFFFF         JB      34501069
3450143F    8B5424 10             MOV     EDX,DWORD PTR SS:\[ESP+10\]
34501443    8D4C24 10             LEA     ECX,DWORD PTR SS:\[ESP+10\]
34501447    51                    PUSH    ECX
34501448    52                    PUSH    EDX
34501449    68 00301B00           PUSH    1B3000
3450144E    68 00104000           PUSH    401000
34501453    FF15 00505034         CALL    DWORD PTR DS:\[34505000\]          ; kernel32.VirtualProtect
34501459    5F                    POP     EDI
3450145A    5E                    POP     ESI
3450145B    5D                    POP     EBP
3450145C    33C0                  XOR     EAX,EAX
3450145E    5B                    POP     EBX
3450145F    83C4 14               ADD     ESP,14
34501462    C3                    RETN
34501463    90                    NOP
34501464    90                    NOP
34501465    90                    NOP
34501466    90                    NOP
34501467    90                    NOP
34501468    90                    NOP
34501469    90                    NOP
3450146A    90                    NOP
3450146B    90                    NOP
3450146C    90                    NOP
3450146D    90                    NOP
3450146E    90                    NOP
3450146F    90                    NOP
34501470    68 B48A0000           PUSH    8AB4
34501475    68 898C6500           PUSH    658C89
3450147A    E8 C1FBFFFF           CALL    34501040
3450147F    83C4 08               ADD     ESP,8
34501482    C3                    RETN

```

FixGame.dll 的源码  
  
disasm.cpp        od 的汇编引擎  
FixNanomite.cpp        修复 Nanomites  
FixIat.cpp         修复 IAT  
FixPeHeader.cpp        修复偷到 PE 头里面的代码  
FixSDK.cpp        修复 SDK 部分  
GetHash.cpp        壳用的 hash 函数  
  
其中 SDK 我是手工修复的，这里只是贴代码而已，其他 3 个都是程序实现的  
  
Nanomite 我是暴力枚举 hash，还是 ccfer 的代码比较强，用特征码去判断

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || FixGame.rar  
 |