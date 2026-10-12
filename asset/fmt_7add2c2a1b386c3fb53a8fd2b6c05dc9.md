
after patching that sub into `ret`, the program becomes stabless and so many errers occurs. So,

there must be CORE check code somewhere.

## 2. How to re-install ?

Delete this key and adjust your machine's date to 2002/03/10, then

run the `install.exe`

`[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Trapezium.InternalLexer]`

## 3. Protection mechanism - `TExpirationServer`

Spending some time to explore, I found this:

```
CODE:0042DE50 aTexpirationser db 17,'TExpirationServer' ; DATA XREF: ...
CODE:0042DE62                 align 4
CODE:0042DE64 off_0_42DE64    dd offset off_0_42DEB0  ; DATA XREF: ...
CODE:0042DE68                 dd 6 dup(0)
CODE:0042DE80                 dd offset dword_0_42DECC
CODE:0042DE84                 dd offset aTtimeltdexpser ; "TTimeLtdExpServer"
CODE:0042DE88                 dd 28h
CODE:0042DE8C                 dd offset off_0_42DDE0
CODE:0042DE90                 dd offset loc_0_401118  ; TObject_SafeCallException
CODE:0042DE94                 dd offset loc_0_401128  ; TObject_AfterDestruction
CODE:0042DE98                 dd offset loc_0_401130  ; TObject_BeforeDestruction
CODE:0042DE9C                 dd offset loc_0_401138  ; TObject_Dispatch
CODE:0042DEA0                 dd offset TObject_DefaultHandler
CODE:0042DEA4                 dd offset TObject_NewInstance
CODE:0042DEA8                 dd offset TObject_Free  ; Free
CODE:0042DEAC                 dd offset @System@TObject@$bdtr$qqrv ; Create
CODE:0042DEB0 off_0_42DEB0    dd offset sub_0_42E20C  ; DATA XREF: ...
CODE:0042DEB0                                          ; Releated to COM, StringToGUID
CODE:0042DEB4                 dd offset sub_0_42E2F8  ; Related with COM
CODE:0042DEB8                 dd offset sub_0_42E3DC  ; procedure GetExpiredDateTime
CODE:0042DEBC                 dd offset sub_0_42E4E8  ; Get something
CODE:0042DEC0                 dd offset sub_0_42E50C  ; Get something
CODE:0042DEC4                 dd offset sub_0_42DFB0  ; Read something from registry
CODE:0042DEC8                 dd offset sub_0_42E0E0  ; Write something to registry
CODE:0042DECC dword_0_42DECC  dd 0FFFF0001h          ; DATA XREF: ...
CODE:0042DED0                 dd offset sub_0_42E3B8  ; function IsExpired() : Boolean
CODE:0042DED4 aTtimeltdexpser db 17,'TTimeLtdExpServer' ; DATA XREF: ...
CODE:0042DEE6                 align 4
```

This is a Delphi Object that in charge of checking and pretecting work.

I added some comments at its last serveral functions. Here is CORE:

```
CODE:0042E3DC ; ############### S U B R O U T I N E #######################################
CODE:0042E3DC
CODE:0042E3DC ; procedure GetExpiredDateTime
CODE:0042E3DC ; Attributes: bp-based frame
CODE:0042E3DC
CODE:0042E3DC sub_0_42E3DC    proc near             ; DATA XREF: ...
CODE:0042E3DC
CODE:0042E3DC var_C             = dword ptr -0Ch
CODE:0042E3DC ExpiredDateTime = dword ptr -8
CODE:0042E3DC ExpirationServer= dword ptr -4
CODE:0042E3DC
CODE:0042E3DC                 push    ebp
CODE:0042E3DD                 mov     ebp, esp
CODE:0042E3DF                 add     esp, 0FFFFFFF4h
CODE:0042E3E2                 push    ebx
CODE:0042E3E3                 push    esi
CODE:0042E3E4                 push    edi
CODE:0042E3E5                 xor     edx, edx
CODE:0042E3E7                 mov     [ebp+var_C], edx
CODE:0042E3EA                 mov     [ebp+ExpirationServer], eax
CODE:0042E3ED                 xor     eax, eax
CODE:0042E3EF                 push    ebp
CODE:0042E3F0                 push    offset loc_0_42E4D6
CODE:0042E3F5                 push    dword ptr fs:[eax]
CODE:0042E3F8                 mov     fs:[eax], esp
CODE:0042E3FB                 mov     eax, [ebp+ExpirationServer]
CODE:0042E3FE                 cmp     byte ptr [eax+4], 0 ; if CheckedAlready then Skip
CODE:0042E402                 jnz     loc_0_42E4C0
CODE:0042E408                 xor     eax, eax
CODE:0042E40A                 push    ebp
CODE:0042E40B                 push    offset loc_0_42E49C
CODE:0042E410                 push    dword ptr fs:[eax]
CODE:0042E413                 mov     fs:[eax], esp
CODE:0042E416                 call    sub_0_42D4D0
CODE:0042E41B                 mov     ebx, eax
CODE:0042E41D                 lea     ecx, [ebp+var_C]
CODE:0042E420                 mov     edx, 1
CODE:0042E425                 mov     eax, 5
CODE:0042E42A                 call    sub_0_403F2C
CODE:0042E42F                 mov     edx, [ebp+var_C]
CODE:0042E432                 mov     eax, ebx
CODE:0042E434                 call    sub_0_42D1FC
CODE:0042E439                 lea     ecx, [ebp+ExpiredDateTime]
CODE:0042E43C                 mov     dx, [ebx+16h]
CODE:0042E440                 mov     eax, [ebp+ExpirationServer]
CODE:0042E443                 mov     ebx, [eax]
CODE:0042E445                 call    dword ptr [ebx+14h] ; *CORE* <- We need crack here
CODE:0042E445                                          ; Read Expire Data from
CODE:0042E445                                          ; [HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Trapezium.InternalLexer]
CODE:0042E445                                          ; if success, return TRUE
CODE:0042E448                 test    al, al
CODE:0042E44A                 jz      short EXIT
CODE:0042E44C                 mov     eax, [ebp+ExpiredDateTime]
CODE:0042E44F                 call    Sysutils::FileDateToDateTime(int)
CODE:0042E454                 mov     eax, [ebp+ExpirationServer]
CODE:0042E457                 fstp    qword ptr [eax+10h] ; Store ExpiredDateTime
CODE:0042E45A                 wait
CODE:0042E45B                 call    Sysutils::Now(void)
CODE:0042E460                 mov     eax, [ebp+ExpirationServer]
CODE:0042E463                 fsubr   qword ptr [eax+10h] ; if ExpiredDateTime - CurrentDateTime < 32
CODE:0042E463                                          ; that means, you cannt adjust your time
CODE:0042E463                                          ; too early than ExpiredDateTime. In another
CODE:0042E463                                          ; word, if your ExpiredDateTime is 2009/01/31
CODE:0042E463                                          ; but now, CurrentDataTime in only 2002/03/21,
CODE:0042E463                                          ; then it think you are EXPIRED too.
CODE:0042E466                 fcomp   ds:dword_0_42E4E4 ; 42000000 REAL4 = 32
CODE:0042E46C                 fnstsw  ax
CODE:0042E46E                 sahf
CODE:0042E46F                 setbe   bl
CODE:0042E472                 mov     eax, [ebp+ExpirationServer]
CODE:0042E475                 mov     [eax+8], bl    ; set Expired flag
CODE:0042E475                                          ; but no use at all
CODE:0042E478                 test    bl, bl
CODE:0042E47A                 jz      short loc_0_42E487
CODE:0042E47C                 mov     eax, [ebp+ExpirationServer]
CODE:0042E47F                 mov     edx, [ebp+ExpiredDateTime]
CODE:0042E482                 mov     [eax+18h], edx ; Store FileExpiredDateTime
CODE:0042E485                 jmp     short EXIT
CODE:0042E487 ; ===========================================================================
CODE:0042E487
```
