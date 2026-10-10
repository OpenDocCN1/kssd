

# ModelMaker CodeExplorer Expert 1.05 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50001.htm`](https://www.pediy.com/kssd/pediy05/pediy50001.htm)

ModelMaker CodeExplorer Expert 1.05 Demo   
\======================================== \[Mar 17, 2002\]   
I always think the Delphi program is easy to crack, in sepcial of Delphi components or   
Experts, Plug-ins, etc. , but this time I meet trouble.   
  
1\. When expired, software will popup a messagebox and all functionalities are disable   
2\. Re-install it after expire date is useless.   
3\. Protection mechanism   
3\. Not working when openning a file whose FileAge is bigger than expire-date   
4\. if you cracked at not right place, the programm will crash delphi   
5\. adjust date too early or too late will be detected   
  
Useful functions you should set breakpoint.   
///////////////////////////////////////////////////////////////////////////////////   
        CODE:00401768 ; Sysutils::Now(void)   
        CODE:00401760 ; Sysutils::Date(void)   
        CODE:004016B8 ; Sysutils::FileAge(System::AnsiString)   
  
        CODE:00401758 ; Sysutils::SystemTimeToDateTime(\_SYSTEMTIME &)   
        CODE:004016F8 ; Sysutils::FileDateToDateTime(int)   
        CODE:00401750 ; Sysutils::DecodeDate(System::TDateTime,ushort &,ushort &,ushort &)   
        CODE:00401770 ; Sysutils::FormatDateTime(System::AnsiString,System::TDateTime)   
        CODE:00401778 ; Sysutils::StrToDate(System::AnsiString)   
///////////////////////////////////////////////////////////////////////////////////   
  
  
\=====================================================================   
\= 1. Start at MessageDlg ============================================   
\=====================================================================   
  
  
First, MessageDlg will help you to find the popup inform window code.   
CODE:004020D4 ; Dialogs::MessageDlg(...)   
  
CODE:0044745C ; ############### S U B R O U T I N E #######################################   
CODE:0044745C   
CODE:0044745C ; show error Msg   
CODE:0044745C ; Attributes: bp-based frame   
CODE:0044745C   
CODE:0044745C sub\_0\_44745C    proc near              ; DATA XREF: ...   
CODE:0044745C   
CODE:0044745C var\_18          = dword ptr -18h   
CODE:0044745C var\_14          = dword ptr -14h   
CODE:0044745C var\_10          = dword ptr -10h   
CODE:0044745C var\_C          = dword ptr -0Ch   
CODE:0044745C var\_8          = dword ptr -8   
CODE:0044745C var\_4          = dword ptr -4   
CODE:0044745C   
CODE:0044745C                push    ebp   
CODE:0044745D                mov    ebp, esp   
CODE:0044745F                xor    ecx, ecx   
CODE:00447461                push    ecx   
CODE:00447462                push    ecx   
CODE:00447463                push    ecx   
CODE:00447464                push    ecx   
CODE:00447465                push    ecx   
CODE:00447466                push    ecx   
CODE:00447467                mov    \[ebp+var\_4\], eax   
CODE:0044746A                xor    eax, eax   
CODE:0044746C                push    ebp   
CODE:0044746D                push    offset loc\_0\_44750E   
CODE:00447472                push    dword ptr fs:\[eax\]   
CODE:00447475                mov    fs:\[eax\], esp   
CODE:00447478                mov    eax, \[ebp+var\_4\]   
CODE:0044747B                mov    ecx, \[eax\]   
CODE:0044747D                call    dword ptr \[ecx-10h\]   
CODE:00447480                cmp    ds:Magic, 0   
CODE:00447487                jnz    short loc\_0\_4474F3   
CODE:00447489                mov    ds:Magic, 1   
CODE:00447490                push    0   
CODE:00447492                push    ebp   
CODE:00447493                lea    eax, \[ebp+var\_C\]   
CODE:00447496                call    sub\_0\_4473EC   
CODE:0044749B                pop    ecx   
CODE:0044749C                push    \[ebp+var\_C\]   
CODE:0044749F                push    offset \_str\_\_\_46.Text   
CODE:004474A4                push    ebp   
CODE:004474A5                lea    eax, \[ebp+var\_10\]   
CODE:004474A8                call    sub\_0\_447408   
CODE:004474AD                pop    ecx   
CODE:004474AE                push    \[ebp+var\_10\]   
CODE:004474B1                push    offset \_str\_\_\_46.Text   
CODE:004474B6                push    ebp   
CODE:004474B7                lea    eax, \[ebp+var\_14\]   
CODE:004474BA                call    sub\_0\_447424   
CODE:004474BF                pop    ecx   
CODE:004474C0                push    \[ebp+var\_14\]   
CODE:004474C3                push    offset \_str\_\_\_46.Text   
CODE:004474C8                push    ebp   
CODE:004474C9                lea    eax, \[ebp+var\_18\]   
CODE:004474CC                call    sub\_0\_447440   
CODE:004474D1                pop    ecx   
CODE:004474D2                push    \[ebp+var\_18\]   
CODE:004474D5                lea    eax, \[ebp+var\_8\]   
CODE:004474D8                mov    edx, 7   
CODE:004474DD                call    @System@@LStrCatN$qqrv ; System \_\_linkproc\_\_ LStrCatN(void)   
CODE:004474E2                mov    eax, \[ebp+var\_8\]   
CODE:004474E5                mov    cx, ds:word\_0\_447528   
CODE:004474EC                mov    dl, 2   
CODE:004474EE                call    @Dialogs@MessageDlg$qqrx17System@AnsiString19Dialogs@TMsgDlgType47System@\_Set$t18Dialogs@TMsgDlgBtn$iuc$0$iuc$10\_i ; Dialogs::MessageDlg(System::AnsiString,Dialogs::TMsgDlgType,System::Set<Dialogs::TMsgDlgBtn,(uchar)0,(uchar)10>,int)   
CODE:004474F3   
CODE:004474F3 loc\_0\_4474F3:                          ; CODE XREF: ...   
CODE:004474F3                xor    eax, eax   
CODE:004474F5                pop    edx   
CODE:004474F6                pop    ecx   
CODE:004474F7                pop    ecx   
CODE:004474F8                mov    fs:\[eax\], edx   
CODE:004474FB                push    offset loc\_0\_447515   
CODE:00447500   
CODE:00447500 loc\_0\_447500:                          ; CODE XREF: ...   
CODE:00447500                lea    eax, \[ebp+var\_18\]   
CODE:00447503                mov    edx, 5   
CODE:00447508                call    @System@@LStrArrayClr$qqrpvi ; System \_\_linkproc\_\_ LStrArrayClr(void \*,int)   
CODE:0044750D                retn   
CODE:0044750E ; ===========================================================================   
CODE:0044750E   
CODE:0044750E loc\_0\_44750E:                          ; DATA XREF: ...   
CODE:0044750E                jmp    loc\_0\_401178   
CODE:00447513 ; ===========================================================================   
CODE:00447513                jmp    short loc\_0\_447500   
CODE:00447515 ; ===========================================================================   
CODE:00447515   
CODE:00447515 loc\_0\_447515:                          ; DATA XREF: ...   
CODE:00447515                mov    esp, ebp   
CODE:00447517                pop    ebp   
CODE:00447518                retn   
CODE:00447518 sub\_0\_44745C    endp   
  
this is a event handler, in another word, using xref function found nothing.   
before this inform window appears, the functionality window is hide so you   
cann't access it. Here is the code:   
  
  
CODE:00440ED8 sub\_0\_440ED8    proc near   
CODE:00440ED8                push    esi   
CODE:00440ED9                mov    esi, eax   
CODE:00440EDB                mov    eax, esi   
CODE:00440EDD                mov    ecx, \[eax\]   
CODE:00440EDF                call    dword ptr \[ecx-10h\]   
CODE:00440EE2                xor    edx, edx   
CODE:00440EE4                mov    eax, esi   
CODE:00440EE6                call    @Forms@TCustomForm@SetVisible$qqro ; Forms::TCustomForm::SetVisible(bool)   
CODE:00440EEB                pop    esi   
CODE:00440EEC                retn   
CODE:00440EEC sub\_0\_440ED8    endp   
  
after patching that sub into ret, the program becomes stabless and so many errers occurs. So,   
there must be CORE check code somewhere.   
  
\=====================================================================   
\= 2. How to re-install ? ============================================   
\=====================================================================   
Delete this key and adjust your machine's date to 2002/03/10, then   
run the install.exe   
  
\[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\Trapezium.InternalLexer\]   
  
\=====================================================================   
\= 3. Protection mechanism - TExpirationServer =======================   
\=====================================================================   
  
Spending some time to explore, I found this:   
  
CODE:0042DE50 aTexpirationser db 17,'TExpirationServer' ; DATA XREF: ...   
CODE:0042DE62                align 4   
CODE:0042DE64 off\_0\_42DE64    dd offset off\_0\_42DEB0  ; DATA XREF: ...   
CODE:0042DE68                dd 6 dup(0)   
CODE:0042DE80                dd offset dword\_0\_42DECC   
CODE:0042DE84                dd offset aTtimeltdexpser ; "TTimeLtdExpServer"   
CODE:0042DE88                dd 28h   
CODE:0042DE8C                dd offset off\_0\_42DDE0   
CODE:0042DE90                dd offset loc\_0\_401118  ; TObject\_SafeCallException   
CODE:0042DE94                dd offset loc\_0\_401128  ; TObject\_AfterDestruction   
CODE:0042DE98                dd offset loc\_0\_401130  ; TObject\_BeforeDestruction   
CODE:0042DE9C                dd offset loc\_0\_401138  ; TObject\_Dispatch   
CODE:0042DEA0                dd offset TObject\_DefaultHandler   
CODE:0042DEA4                dd offset TObject\_NewInstance   
CODE:0042DEA8                dd offset TObject\_Free  ; Free   
CODE:0042DEAC                dd offset @System@TObject@$bdtr$qqrv ; Create   
CODE:0042DEB0 off\_0\_42DEB0    dd offset sub\_0\_42E20C  ; DATA XREF: ...   
CODE:0042DEB0                                        ; Releated to COM, StringToGUID   
CODE:0042DEB4                dd offset sub\_0\_42E2F8  ; Related with COM   
CODE:0042DEB8                dd offset sub\_0\_42E3DC  ; procedure GetExpiredDateTime   
CODE:0042DEBC                dd offset sub\_0\_42E4E8  ; Get something   
CODE:0042DEC0                dd offset sub\_0\_42E50C  ; Get something   
CODE:0042DEC4                dd offset sub\_0\_42DFB0  ; Read something from registry   
CODE:0042DEC8                dd offset sub\_0\_42E0E0  ; Write something to registry   
CODE:0042DECC dword\_0\_42DECC  dd 0FFFF0001h          ; DATA XREF: ...   
CODE:0042DED0                dd offset sub\_0\_42E3B8  ; function IsExpired() : Boolean   
CODE:0042DED4 aTtimeltdexpser db 17,'TTimeLtdExpServer' ; DATA XREF: ...   
CODE:0042DEE6                align 4   
  
This is a Delphi Object that in charge of checking and pretecting work.   
I added some comments at its last serveral functions. Here is CORE:   
  
CODE:0042E3DC ; ############### S U B R O U T I N E #######################################   
CODE:0042E3DC   
CODE:0042E3DC ; procedure GetExpiredDateTime   
CODE:0042E3DC ; Attributes: bp-based frame   
CODE:0042E3DC   
CODE:0042E3DC sub\_0\_42E3DC    proc near              ; DATA XREF: ...   
CODE:0042E3DC   
CODE:0042E3DC var\_C          = dword ptr -0Ch   
CODE:0042E3DC ExpiredDateTime = dword ptr -8   
CODE:0042E3DC ExpirationServer= dword ptr -4   
CODE:0042E3DC   
CODE:0042E3DC                push    ebp   
CODE:0042E3DD                mov    ebp, esp   
CODE:0042E3DF                add    esp, 0FFFFFFF4h   
CODE:0042E3E2                push    ebx   
CODE:0042E3E3                push    esi   
CODE:0042E3E4                push    edi   
CODE:0042E3E5                xor    edx, edx   
CODE:0042E3E7                mov    \[ebp+var\_C\], edx   
CODE:0042E3EA                mov    \[ebp+ExpirationServer\], eax   
CODE:0042E3ED                xor    eax, eax   
CODE:0042E3EF                push    ebp   
CODE:0042E3F0                push    offset loc\_0\_42E4D6   
CODE:0042E3F5                push    dword ptr fs:\[eax\]   
CODE:0042E3F8                mov    fs:\[eax\], esp   
CODE:0042E3FB                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E3FE                cmp    byte ptr \[eax+4\], 0 ; if CheckedAlready then Skip   
CODE:0042E402                jnz    loc\_0\_42E4C0   
CODE:0042E408                xor    eax, eax   
CODE:0042E40A                push    ebp   
CODE:0042E40B                push    offset loc\_0\_42E49C   
CODE:0042E410                push    dword ptr fs:\[eax\]   
CODE:0042E413                mov    fs:\[eax\], esp   
CODE:0042E416                call    sub\_0\_42D4D0   
CODE:0042E41B                mov    ebx, eax   
CODE:0042E41D                lea    ecx, \[ebp+var\_C\]   
CODE:0042E420                mov    edx, 1   
CODE:0042E425                mov    eax, 5   
CODE:0042E42A                call    sub\_0\_403F2C   
CODE:0042E42F                mov    edx, \[ebp+var\_C\]   
CODE:0042E432                mov    eax, ebx   
CODE:0042E434                call    sub\_0\_42D1FC   
CODE:0042E439                lea    ecx, \[ebp+ExpiredDateTime\]   
CODE:0042E43C                mov    dx, \[ebx+16h\]   
CODE:0042E440                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E443                mov    ebx, \[eax\]   
CODE:0042E445                call    dword ptr \[ebx+14h\] ; \*CORE\* <- We need crack here   
CODE:0042E445                                        ; Read Expire Data from   
CODE:0042E445                                        ; \[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\Trapezium.InternalLexer\]   
CODE:0042E445                                        ; if success, return TRUE   
CODE:0042E448                test    al, al   
CODE:0042E44A                jz      short EXIT   
CODE:0042E44C                mov    eax, \[ebp+ExpiredDateTime\]   
CODE:0042E44F                call    Sysutils::FileDateToDateTime(int)   
CODE:0042E454                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E457                fstp    qword ptr \[eax+10h\] ; Store ExpiredDateTime   
CODE:0042E45A                wait   
CODE:0042E45B                call    Sysutils::Now(void)   
CODE:0042E460                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E463                fsubr  qword ptr \[eax+10h\] ; if ExpiredDateTime - CurrentDateTime < 32   
CODE:0042E463                                        ; that means, you cannt adjust your time   
CODE:0042E463                                        ; too early than ExpiredDateTime. In another   
CODE:0042E463                                        ; word, if your ExpiredDateTime is 2009/01/31   
CODE:0042E463                                        ; but now, CurrentDataTime in only 2002/03/21,   
CODE:0042E463                                        ; then it think you are EXPIRED too.   
CODE:0042E466                fcomp  ds:dword\_0\_42E4E4 ; 42000000 REAL4 = 32   
CODE:0042E46C                fnstsw  ax   
CODE:0042E46E                sahf   
CODE:0042E46F                setbe  bl   
CODE:0042E472                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E475                mov    \[eax+8\], bl    ; set Expired flag   
CODE:0042E475                                        ; but no use at all   
CODE:0042E478                test    bl, bl   
CODE:0042E47A                jz      short loc\_0\_42E487   
CODE:0042E47C                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E47F                mov    edx, \[ebp+ExpiredDateTime\]   
CODE:0042E482                mov    \[eax+18h\], edx  ; Store FileExpiredDateTime   
CODE:0042E485                jmp    short EXIT   
CODE:0042E487 ; ===========================================================================   
CODE:0042E487   
CODE:0042E487 loc\_0\_42E487:                          ; CODE XREF: ...   
CODE:0042E487                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E48A                xor    edx, edx   
CODE:0042E48C                mov    \[eax+10h\], edx   
CODE:0042E48F                mov    \[eax+14h\], edx   
CODE:0042E492   
CODE:0042E492 EXIT:                                  ; CODE XREF: ...   
CODE:0042E492                xor    eax, eax   
CODE:0042E494                pop    edx   
CODE:0042E495                pop    ecx   
CODE:0042E496                pop    ecx   
CODE:0042E497                mov    fs:\[eax\], edx   
CODE:0042E49A                jmp    short loc\_0\_42E4B8   
CODE:0042E49C ; ===========================================================================   
CODE:0042E49C   
CODE:0042E49C loc\_0\_42E49C:                          ; DATA XREF: ...   
CODE:0042E49C                jmp    loc\_0\_401168   
CODE:0042E4A1 ; ===========================================================================   
CODE:0042E4A1                mov    eax, \[ebp-4\]   
CODE:0042E4A4                xor    edx, edx   
CODE:0042E4A6                mov    \[eax+10h\], edx   
CODE:0042E4A9                mov    \[eax+14h\], edx   
CODE:0042E4AC                call    System \_\_linkproc\_\_ DoneExcept(void)   
CODE:0042E4B1                jmp    short loc\_0\_42E4C0   
CODE:0042E4B3 ; ===========================================================================   
CODE:0042E4B3                call    System \_\_linkproc\_\_ DoneExcept(void)   
CODE:0042E4B8   
CODE:0042E4B8 loc\_0\_42E4B8:                          ; CODE XREF: ...   
CODE:0042E4B8                mov    eax, \[ebp+ExpirationServer\]   
CODE:0042E4BB                call    SetCheckedAlready   
CODE:0042E4C0   
CODE:0042E4C0 loc\_0\_42E4C0:                          ; CODE XREF: ...   
CODE:0042E4C0                xor    eax, eax   
CODE:0042E4C2                pop    edx   
CODE:0042E4C3                pop    ecx   
CODE:0042E4C4                pop    ecx   
CODE:0042E4C5                mov    fs:\[eax\], edx   
CODE:0042E4C8                push    offset loc\_0\_42E4DD   
CODE:0042E4CD   
CODE:0042E4CD loc\_0\_42E4CD:                          ; CODE XREF: ...   
CODE:0042E4CD                lea    eax, \[ebp+var\_C\]   
CODE:0042E4D0                call    System \_\_linkproc\_\_ LStrClr(void \*)   
CODE:0042E4D5                retn   
CODE:0042E4D6 ; ===========================================================================   
CODE:0042E4D6   
CODE:0042E4D6 loc\_0\_42E4D6:                          ; DATA XREF: ...   
CODE:0042E4D6                jmp    loc\_0\_401178   
CODE:0042E4DB ; ===========================================================================   
CODE:0042E4DB                jmp    short loc\_0\_42E4CD   
CODE:0042E4DD ; ===========================================================================   
CODE:0042E4DD   
CODE:0042E4DD loc\_0\_42E4DD:                          ; DATA XREF: ...   
CODE:0042E4DD                pop    edi   
CODE:0042E4DE                pop    esi   
CODE:0042E4DF                pop    ebx   
CODE:0042E4E0                mov    esp, ebp   
CODE:0042E4E2                pop    ebp   
CODE:0042E4E3                retn   
CODE:0042E4E3 sub\_0\_42E3DC    endp   
CODE:0042E4E3   
CODE:0042E4E3 ; ===========================================================================   
CODE:0042E4E4 dword\_0\_42E4E4  dd 42000000h            ; DATA XREF: ...   
CODE:0042E4E4                                        ; 42000000 REAL4 = 32   
  
  
It is called here:   
  
CODE:0042E52A loc\_0\_42E52A:                          ; CODE XREF: ...   
CODE:0042E52A                mov    ebx, edx   
CODE:0042E52C                mov    esi, eax   
CODE:0042E52E                xor    edx, edx   
CODE:0042E530                mov    eax, esi   
CODE:0042E532                call    System::TObject::TObject(void)   
CODE:0042E537                call    Sysutils::Now(void)   
CODE:0042E53C                fstp    qword ptr \[esi+20h\] <--- Store now date and time!!!   
CODE:0042E53F                wait   
CODE:0042E540                mov    eax, esi   
CODE:0042E542                mov    edx, \[eax\]   
CODE:0042E544                call    dword ptr \[edx+8\]   
CODE:0042E547                mov    eax, esi   
CODE:0042E549                test    bl, bl   
CODE:0042E54B                jz      short loc\_0\_42E55C   
CODE:0042E54D                call    System \_\_linkproc\_\_ AfterConstruction(System::TObject \*)   
CODE:0042E552                pop    large dword ptr fs:0   
CODE:0042E559                add    esp, 0Ch   
CODE:0042E55C   
CODE:0042E55C loc\_0\_42E55C:                          ; CODE XREF: ...   
CODE:0042E55C                mov    eax, esi   
CODE:0042E55E                pop    esi   
CODE:0042E55F                pop    ebx   
CODE:0042E560                retn   
CODE:0042E560 sub\_0\_42E51C    endp   
  
From these two pieces code, we found three \*IMPORTANT\* variables:   
CurrentDateTime   
ExpiredDateTime   
FileExpiredDateTime   
  
so, BPM these two address and you can find more check code:   
  
  
CODE:0042E3B8 ; ############### S U B R O U T I N E #######################################   
CODE:0042E3B8   
CODE:0042E3B8 ; function IsExpired() : Boolean   
CODE:0042E3B8   
CODE:0042E3B8 sub\_0\_42E3B8    proc near              ; DATA XREF: ...   
CODE:0042E3B8                push    ebx   
CODE:0042E3B9                mov    ebx, eax   
CODE:0042E3BB                mov    eax, ebx   
CODE:0042E3BD                mov    edx, \[eax\]   
CODE:0042E3BF                call    dword ptr \[edx+8\]   
CODE:0042E3C2                cmp    byte ptr \[ebx+8\], 0   
CODE:0042E3C6                jz      short loc\_0\_42E3D7   
CODE:0042E3C8                fld    qword ptr \[ebx+20h\]   
CODE:0042E3CB                fcomp  qword ptr \[ebx+10h\] ; return (ExpiredDateTime > CurrentDateTime)   
CODE:0042E3CE                fnstsw  ax   
CODE:0042E3D0                sahf   
CODE:0042E3D1                ja      short loc\_0\_42E3D7   
CODE:0042E3D3                xor    eax, eax   
CODE:0042E3D5                pop    ebx   
CODE:0042E3D6                retn   
CODE:0042E3D7 ; ===========================================================================   
CODE:0042E3D7   
CODE:0042E3D7 loc\_0\_42E3D7:                          ; CODE XREF: ...   
CODE:0042E3D7                mov    al, 1   
CODE:0042E3D9                pop    ebx   
CODE:0042E3DA                retn   
CODE:0042E3DA sub\_0\_42E3B8    endp   
  
You cann't simply patch this function to let me renturn \*FALSE\* directly because   
many TExpirationServer use CurrentDateTime and ExpiredDateTime extensively internal.   
  
\=====================================================================   
\= 3. FileAge Checking ===============================================   
\=====================================================================   
  
Lets now set breakpoint in Sysutils::FileAge(System::AnsiString)   
  
First time:   
  
CODE:00446FA2                mov    eax, \[ebp+var\_C\]   
CODE:00446FA5                call    Sysutils::FileAge(System::AnsiString) ; FileAgeCheck2   
CODE:00446FAA                mov    esi, eax   
CODE:00446FAC                call    sub\_0\_42DF84   
CODE:00446FB1                mov    edx, \[eax\]   
CODE:00446FB3                call    dword ptr \[edx+10h\] ; GetExpiredDateTime   
                                                          ; Return FileExpiredDateTime   
CODE:00446FB6                cmp    esi, eax   
CODE:00446FB8                jle    short loc\_0\_446FDE   
CODE:00446FBA                mov    eax, \[ebp+var\_4\]   
CODE:00446FBD                add    eax, 394h   
CODE:00446FC2                call    System \_\_linkproc\_\_ IntfClear(System::DelphiInterface<System::IInterface> &)   
CODE:00446FC7                mov    eax, \[ebp+var\_4\]   
CODE:00446FCA                add    eax, 398h   
CODE:00446FCF                call    System \_\_linkproc\_\_ IntfClear(System::DelphiInterface<System::IInterface> &)   
CODE:00446FD4                xor    eax, eax   
CODE:00446FD6                pop    edx   
CODE:00446FD7                pop    ecx   
CODE:00446FD8                pop    ecx   
CODE:00446FD9                mov    fs:\[eax\], edx   
CODE:00446FDC                jmp    short loc\_0\_447038   
CODE:00446FDE ; ===========================================================================   
CODE:00446FDE   
CODE:00446FDE loc\_0\_446FDE:                          ; CODE XREF: ...   
CODE:00446FDE                mov    eax, \[ebp+var\_4\]   
CODE:00446FE1                add    eax, 394h   
CODE:00446FE6                mov    edx, ebx   
CODE:00446FE8                call    System \_\_linkproc\_\_ IntfCopy(System::DelphiInterface<System::IInterface> &,System::DelphiInterface<System::IInterface>)   
CODE:00446FED                lea    edx, \[ebp+var\_14\]   
CODE:00446FF0                mov    eax, \[ebp+var\_4\]   
CODE:00446FF3                call    sub\_0\_447D64   
CODE:00446FF8                mov    edx, \[ebp+var\_14\]   
CODE:00446FFB                mov    eax, \[ebp+var\_4\]   
CODE:00446FFE                add    eax, 398h   
CODE:00447003                call    System \_\_linkproc\_\_ IntfCopy(System::DelphiInterface<System::IInterface> &,System::DelphiInterface<System::IInterface>)   
CODE:00447008                mov    eax, \[ebp+var\_4\]   
CODE:0044700B                call    sub\_0\_4470C0   
  
  
Second time:   
  
CODE:00447580                movzx  eax, word ptr \[ebx+16h\]   
CODE:00447584                call    sub\_0\_42F8B0    ; Read C:\\Program Files\\ModelMaker\\Shared\\Templates\\Index.dat   
CODE:00447589                test    al, al   
CODE:0044758B                jnz    short loc\_0\_447592   
CODE:0044758D                call    Sysutils::Abort(void)   
CODE:00447592   
CODE:00447592 loc\_0\_447592:                          ; CODE XREF: ...   
CODE:00447592                mov    eax, \[ebp+var\_4\]   
CODE:00447595                mov    ebx, \[eax+3BCh\]   
CODE:0044759B                mov    eax, \[ebp+var\_4\]   
CODE:0044759E                inc    dword ptr \[eax+3B8h\]   
CODE:004475A4                mov    eax, \[ebp+var\_4\]   
CODE:004475A7                cmp    dword ptr \[eax+3B8h\], 1   
CODE:004475AE                jnz    loc\_0\_44777A   
CODE:004475B4                xor    edx, edx   
CODE:004475B6                push    ebp   
CODE:004475B7                push    offset loc\_0\_447727   
CODE:004475BC                push    dword ptr fs:\[edx\]   
CODE:004475BF                mov    fs:\[edx\], esp   
CODE:004475C2                mov    eax, \[ebp+var\_4\]   
CODE:004475C5                cmp    dword ptr \[eax+394h\], 0   
CODE:004475CC                jz      short loc\_0\_447629   
CODE:004475CE                lea    edx, \[ebp+var\_18\]   
CODE:004475D1                mov    eax, \[ebp+var\_4\]   
CODE:004475D4                mov    eax, \[eax+394h\]   
CODE:004475DA                mov    ecx, \[eax\]   
CODE:004475DC                call    dword ptr \[ecx+18h\]   
CODE:004475DF                mov    eax, \[ebp+var\_18\]   
CODE:004475E2                lea    edx, \[ebp+var\_14\]   
CODE:004475E5                mov    ecx, \[eax\]   
CODE:004475E7                call    dword ptr \[ecx+18h\]   
CODE:004475EA                mov    eax, \[ebp+var\_14\]   
CODE:004475ED                call    Sysutils::FileAge(System::AnsiString)   
CODE:004475F2                mov    edx, \[ebp+var\_C\]   
CODE:004475F5                add    edx, 10h   
CODE:004475F8                cmp    eax, edx   
CODE:004475FA                jl      short loc\_0\_447629   
CODE:004475FC                mov    eax, 5   
CODE:00447601                call    sub\_0\_4040EC   
CODE:00447606                mov    eax, 6   
CODE:0044760B                call    sub\_0\_4040EC   
CODE:00447610                mov    eax, \[ebp+var\_4\]   
CODE:00447613                mov    eax, \[eax+3A0h\]   
CODE:00447619                test    eax, eax   
CODE:0044761B                jz      short loc\_0\_447622   
CODE:0044761D                call    Forms::TCustomForm::Release(void)   
  
Third Time:   
  
CODE:00445237                jz      short loc\_0\_445267   
CODE:00445239                mov    eax, \[ebx\]   
CODE:0044523B                call    Sysutils::FileAge(System::AnsiString)   
CODE:00445240                mov    edx, ds:off\_0\_4549D0 ; 20020417 <-- FIX ADDRESS   
CODE:00445246                mov    edx, \[edx\]   
CODE:00445248                add    edx, 2   
CODE:0044524B                cmp    eax, edx   
CODE:0044524D                jge    short loc\_0\_44526E   
CODE:0044524F                lea    edx, \[ebp+var\_18\]   
CODE:00445252                mov    eax, \[ebp+var\_8\]   
CODE:00445255                call    sub\_0\_444FA8   
CODE:0044525A                mov    edx, \[ebp+var\_18\]   
CODE:0044525D                mov    eax, \[ebp+var\_4\]   
CODE:00445260                mov    ecx, \[eax\]   
CODE:00445262                call    dword ptr \[ecx+2Ch\]   
CODE:00445265                jmp    short loc\_0\_44526E   
  
I bpm ds:off\_0\_4549D0 and found this:   
  
CODE:004485DD                mov    edx, ds:off\_0\_4549D0  <-- will change its content   
CODE:004485E3                movzx  eax, word ptr \[ebx+16h\]   
CODE:004485E7                call    sub\_0\_42F8B0   
CODE:004485EC                mov    dl, 1   
CODE:004485EE                mov    eax, ds:\`\_\_vtable\_\_'\[Graphics::TBitmap\]   
CODE:004485F3                call    Graphics::TBitmap::TBitmap(void)   
  
  
Go to sub\_0\_42F8B0 :   
  
.  ////////////////////////////////////////////////////////////   
.  // Read file data into a TStringList from   
.  // C:\\Program Files\\ModelMaker\\Shared\\Templates\\Index.dat   
.  ///////////////////////////////////////////////////////////   
.   
.   
.   
CODE:0042F928                mov    edx, \[ebp+var\_14\]   
CODE:0042F92B                lea    ecx, \[ebp+var\_10\] ; string to return   
CODE:0042F92E                mov    eax, \[ebp+var\_8\]   
CODE:0042F931                call    Classes::TStrings::GetValue(System::AnsiString)   
CODE:0042F936                mov    eax, \[ebp+var\_10\] ; store EXPIRED DATE TIME   
CODE:0042F939                or      edx, 0FFFFFFFFh   
CODE:0042F93C                call    Sysutils::StrToIntDef(System::AnsiString,int)   
CODE:0042F941                mov    \[esi\], eax        ; CRACK here, put a right value to \[esi\]   
CODE:0042F943                cmp    dword ptr \[esi\], 0   
CODE:0042F946                setnl  \[ebp+var\_1\]   
CODE:0042F94A   
  
This file is encrypted and here is the decryped memory dump:   
  
0023:03661044 43 75 73 74 6F 6D 3D 31-36 34 0D 0A 54 65 6D 70  Custom=164..Temp   
0023:03661054 6C 61 74 65 73 3D 34 34-0D 0A 31 34 39 3D 37 34  lates=44..149=74   
0023:03661064 37 38 37 37 39 33 30 0D-0A 57 69 7A 61 72 64 73  7877930..Wizards   
0023:03661074 3D 37 35 32 0D 0A 50 61-74 74 65 72 6E 73 3D 31  =752..Patterns=1   
0023:03661084 30 32 34 0D 0A 00 00 00-00 67 0F 01 64 67 0F 01  024......g..dg..   
  
There are 5 lines and you can clearly see "149=747877930" that is written by   
Installer program.   
  
  
  
\=====================================================================   
\= CONCLUSION ========================================================   
\=====================================================================   
  
You have seen the all protection code I found, they can be classified   
of two catagories: 1. TExpirationServer 2. Index.dat   
In first protection, the CORE security code is read expiration date   
from the HIDDEN key of windows registry. In second part, another   
expiration date is read from encrypt text file, Index.dat. After   
the two expiration date are got, then the program will compare current   
date and the file date that you are editting with them when you use   
every function of it, such as Add Field, Add Class, etc. If expiration   
is detected some object will be release and the program will raise   
exception becuase AV error.   
  
THE ISSUES THAT I HAVEN'T GOT CLEAR   
\-----------------------------------   
1\. The encrypt/decrypt algorithm which used to process Index.dat   
2\. Why you are always expired once you're dectected expired? It must   
  store some other information on my machine. I must delete security   
  registry and uninstall, then re-install again.   
3\. Maybe there are other checkpoint I have not find :( I hope there is   
  NONE.   
  
4\. Installer will check date too and protection code is in MMXInstall.dll.   
  I have no time to make a \*Ripped\* version. So, if you want to install,   
  you must set date to a \*Right\* date. Sorry for inconvenient.   
  
\=====================================================================   
\= .. CRACK .......... 2009/01/31 -> 3A3F0000 ========================   
\=====================================================================   
  
  
CODE 1.1: TExpirationServer Read Registry   
  
.0042DFB0: 52                          push        edx   
.0042DFB1: BA00003F3A                  mov        edx,03A3F0000   
.0042DFB6: 8911                        mov        \[ecx\],edx   
.0042DFB8: 33C0                        xor        eax,eax   
.0042DFBA: B001                        mov        al,001   
.0042DFBC: 5A                          pop        edx   
.0042DFBD: C3                          retn   
  
CODE 1.2: TExpirationServer GetExpiredDateTime   
  
.0042E46F: B301                        mov        bl,001   
.0042E471: 90                          nop   
.0042E472: 8B45FC                      mov        eax,\[ebp\]\[-04\]   
.0042E475: 885808                      mov        \[eax\]\[08\],bl   
  
  
CODE 2: Index.dat   
  
.0042F936: B800003F3A                  mov        eax,03A3F0000   
.0042F93B: EB04                        jmps      .00042F941   
.0042F93D: 90                          nop   
.0042F93E: 90                          nop   
.0042F93F: 90                          nop   
.0042F940: 90                          nop   
.0042F941: 8906                        mov        \[esi\],eax   
.0042F943: 833E00                      cmp        d,\[esi\],000   

> \=====================================================================   
>   
> \>fc MIDEXD6.dll MIDEXD6.old /b   
> Comparing files MIDEXD6.dll and MIDEXD6.OLD   
> 0002D3B0: 52 55   
> 0002D3B1: BA 8B   
> 0002D3B2: 00 EC   
> 0002D3B3: 00 6A   
> 0002D3B4: 3F 00   
> 0002D3B5: 3A 6A   
> 0002D3B6: 89 00   
> 0002D3B7: 11 6A   
> 0002D3B8: 33 00   
> 0002D3B9: C0 6A   
> 0002D3BA: B0 00   
> 0002D3BB: 01 6A   
> 0002D3BC: 5A 00   
> 0002D3BD: C3 6A   
> 0002D3BE: 5A 00   
> 0002D3BF: C3 6A   
> 0002D86F: B3 0F   
> 0002D870: 01 96   
> 0002D871: 90 C3   
> 0002ED36: B8 8B   
> 0002ED37: 00 45   
> 0002ED38: 00 F0   
> 0002ED39: 3F 83   
> 0002ED3A: 3A CA   
> 0002ED3B: EB FF   
> 0002ED3C: 04 E8   
> 0002ED3D: 90 67   
> 0002ED3E: 90 1D   
> 0002ED3F: 90 FD   
> 0002ED40: 90 FF   
>   
> \>   
>   
> \===================================================================== \[THE END\]  4dd466d0e1e0cc123afbe504da2462ca