
```
                                mov eax,DWORD ptr szDesFileName

                                .if eax

                                invoke SetDlgItemText,hWnd,IDC\_EDesFile,addr szDesFileName

                                invoke InitFileMap

                                invoke  CheckPEValidity

                                .if !eax

                                        invoke MessageBox,hWnd,addr szNotPe,addr szNotPeCap,MB\_OK or MB\_ICONERROR

                                .else

                                mov IsPe,1

                                invoke  GetPeInfo,addr PeInfo

                                invoke        wsprintf,addr szBuffer,addr szFormat,PeInfo.dwEntryPointRVA

                                invoke  SetDlgItemText,hWnd,IDC\_EEntry,addr szBuffer

                                invoke        wsprintf,addr szBuffer,addr szFormat,PeInfo.dwImageBase

                                invoke  SetDlgItemText,hWnd,IDC\_EBase,addr szBuffer

                                invoke  SetDlgItemInt,hWnd,IDC\_EScnNum,PeInfo.dwNumberOfSections,FALSE

                                .endif

                                .endif

                        .elseif ax==IDC\_BData

                                invoke \_OpenFile,addr szDataFileName

                                invoke SetDlgItemText,hWnd,IDC\_EData,addr szDataFileName

                        .elseif ax==IDC\_CHKWrite

                        invoke        IsDlgButtonChecked,hWnd,IDC\_CHKWrite

                                .if        eax == BST\_CHECKED

                                        mov WriteDataFlag,TRUE

                                        mov ebx,0

                                .else

                                        mov WriteDataFlag,FALSE

                                        mov ebx,1

                                .endif

                                invoke        GetDlgItem,hWnd,IDC\_EScnSize

                                ;not WriteDataFlag

                                invoke        EnableWindow,eax, ebx

                        .elseif  ax==IDC\_CHKUpEntry

                                invoke        IsDlgButtonChecked,hWnd,IDC\_CHKUpEntry

                                .if        eax == BST\_CHECKED

                                        mov UpEntryFlag,TRUE

                                .else

                                        mov UpEntryFlag,FALSE

                                .endif

                        .elseif ax==IDC\_BMake

                                mov eax,DWORD ptr szDesFileName

                                .if !eax || !IsPe                                           invoke MessageBox,hWnd,addr szNoFile,addr szNotPeCap,MB\_OK or MB\_ICONERROR

                                .else

                                invoke  GetScnInfo,0, addr OrglastScainfo

                                invoke  GetDlgItemText,hWnd,IDC\_EScnName,addr szNewSectionName,8

                                .if !eax

                                                invoke MessageBox,hWnd,addr szScnName,addr szNotPeCap,MB\_OK or MB\_ICONERROR

                                                mov        eax,TRUE

                                                ret

                                        .endif

                                .if !WriteDataFlag

                                        invoke GetDlgItemInt,hWnd,IDC\_EScnSize,NULL,FALSE

                                        .if !eax

                                                invoke MessageBox,hWnd,addr szScnSize,addr szNotPeCap,MB\_OK or MB\_ICONERROR

                                                mov        eax,TRUE

                                                ret

                                        .endif

                                        mov dwNewScnSize,eax

                                .else

                                        mov eax,DWORD ptr szDataFileName

                                        .if !eax

                                                invoke MessageBox,hWnd,addr szData,addr szNotPeCap,MB\_OK or MB\_ICONERROR

                                                mov        eax,TRUE

                                                ret

                                        .endif

                                        invoke  InitDataFile

                                        push  dwDataFileSize

                                        pop   dwNewScnSize

                                .endif

            invoke AddSection,offset szNewSectionName,dwNewScnSize,ScnFlagsDefault

            invoke   GetExtraDataInfo,offset ExtraData

            .if UpEntryFlag

              invoke  UpdateEntryPoint,dwNewEntry

            .endif

            invoke  CreateOutFile

            invoke AppendScnCode,offset OrglastScainfo,offset ExtraData,lpDataHead,dwNewScnSize

            invoke GetScnInfo,0,addr NewScnInfo

            invoke        wsprintf,addr szBuffer,addr szFormat,NewScnInfo.dwVirtualAddress

                                invoke  SetDlgItemText,hWnd,IDC\_EVOffset,addr szBuffer

                                invoke        wsprintf,addr szBuffer,addr szFormat,NewScnInfo.dwRawAddress

                                invoke  SetDlgItemText,hWnd,IDC\_EFOffset,addr szBuffer

                                invoke        wsprintf,addr szBuffer,addr szFormat,NewScnInfo.dwVirtualSize

                                invoke  SetDlgItemText,hWnd,IDC\_ERSize,addr szBuffer

            invoke MessageBox,hWnd,addr szText,addr szCaption,MB\_OK

            .endif

                        .elseif ax==IDC\_BAbout

                                invoke MessageBox,hWnd,addr szAbout,addr szAboutCaption,MB\_OK or MB\_ICONINFORMATION

                        .endif

                .else

                        mov        eax,FALSE

                        ret

                .endif

                mov        eax,TRUE

                ret

\_ProcDlgMain        endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

start:

                invoke        GetModuleHandle,NULL

                mov        hInstance,eax

                invoke        DialogBoxParam,hInstance,IDD\_DLGMain,NULL,offset \_ProcDlgMain,NULL

                invoke        ExitProcess,NULL

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

                end        start
```

------------------------------------------------------the end-------------------------------------------------------------------------------------

ps：附件中文件有：`AddSection.asm`，`AddSection.rc`，`AddSection.exe`，`Main.ico`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`AddSection.rar`](javascript:void(0);)

  （10.04kb，187次下载）
