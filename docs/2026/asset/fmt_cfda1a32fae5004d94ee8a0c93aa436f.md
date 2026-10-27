
```
0000:00404107                 push    esi             ; nDenominator
0000:00404108                 movsx   eax, [ebp+var_22]
0000:0040410C                 push    eax             ; nNumerator
0000:0040410D                 movsx   eax, di
0000:00404110                 push    eax             ; nNumber
0000:00404111                 call    MulDiv
0000:00404116                 add     ax, [ebp+var_1C]
0000:0040411A                 mov     [ebp+var_16], ax
0000:0040411E                 cmp     [ebp+var_12], 0FFh
0000:00404124                 jle     short loc_40412C
0000:00404126                 mov     [ebp+var_12], 0FFh
//最大变化到255
0000:0040412C
0000:0040412C loc_40412C:
0000:0040412C                 cmp     [ebp+var_14], 0FFh
0000:00404132                 jle     short loc_40413A
0000:00404134                 mov     [ebp+var_14], 0FFh
0000:0040413A
0000:0040413A loc_40413A:
0000:0040413A                 cmp     [ebp+var_16], 0FFh
0000:00404140                 jle     short loc_404148
0000:00404142                 mov     [ebp+var_16], 0FFh
0000:00404148
0000:00404148 loc_404148:
0000:00404148                 mov     cl, byte ptr [ebp+var_16]
0000:0040414B                 mov     dl, byte ptr [ebp+var_14]
0000:0040414E                 mov     al, byte ptr [ebp+var_12]
0000:00404151                 call    unknown_libname_15 ; Borland Visual Component Library & Packages
//来，拼一下RGB分量
0000:00404156                 mov     [ebp+var_40.lbColor], eax
//返回的分量
0000:00404159                 lea     eax, [ebp+var_40]
0000:0040415C                 push    eax             ; LOGBRUSH *
0000:0040415D                 call    CreateBrushIndirect
0000:00404162                 mov     [ebp+hbr], eax
0000:00404165                 mov     eax, [ebp+hbr]
0000:00404168                 push    eax             ; hbr
0000:00404169                 lea     eax, [ebp+rc]
0000:0040416C                 push    eax             ; lprc
0000:0040416D                 mov     eax, [ebp+hDC]
0000:00404170                 push    eax             ; hDC
0000:00404171                 call    FillRect
0000:00404176                 push    esi             ; nDenominator
0000:00404177                 push    esi             ; nNumerator
0000:00404178                 movsx   eax, di
0000:0040417B                 push    eax             ; nNumber
0000:0040417C                 call    MulDiv
0000:00404181                 mov     edx, [ebp+Rect.right]
0000:00404184                 sub     edx, [ebp+Rect.left]
0000:00404187                 sub     edx, eax
0000:00404189                 mov     [ebp+rc.left], edx
0000:0040418C                 push    esi             ; nDenominator
0000:0040418D                 push    esi             ; nNumerator
0000:0040418E                 movsx   eax, di
0000:00404191                 inc     eax
0000:00404192                 push    eax             ; nNumber
0000:00404193                 call    MulDiv
0000:00404198                 mov     edx, [ebp+Rect.right]
0000:0040419B                 sub     edx, [ebp+Rect.left]
0000:0040419E                 sub     edx, eax
0000:004041A0                 mov     [ebp+rc.right], edx
0000:004041A3                 mov     eax, [ebp+hbr]
0000:004041A6                 push    eax             ; hbr
0000:004041A7                 lea     eax, [ebp+rc]
0000:004041AA                 push    eax             ; lprc
0000:004041AB                 mov     eax, [ebp+hDC]
0000:004041AE                 push    eax             ; hDC
0000:004041AF                 call    FillRect
0000:004041B4                 mov     eax, [ebp+hbr]
0000:004041B7                 push    eax             ; HGDIOBJ
0000:004041B8                 call    DeleteObject
0000:004041BD                 inc     edi
0000:004041BE                 dec     [ebp+var_24]
0000:004041C2                 jnz     loc_4040B9
//循环末尾
0000:004041C8
0000:004041C8 loc_4041C8:
0000:004041C8                 mov     [ebp+var_40.lbColor], 9E6A54h
0000:004041CF                 lea     eax, [ebp+var_40]
0000:004041D2                 push    eax             ; LOGBRUSH *
0000:004041D3                 call    CreateBrushIndirect
0000:004041D8                 mov     [ebp+hbr], eax
0000:004041DB                 mov     eax, [ebp+hbr]
0000:004041DE                 push    eax             ; hbr
0000:004041DF                 lea     eax, [ebp+Rect]
0000:004041E2                 push    eax             ; lprc
0000:004041E3                 mov     eax, [ebp+hDC]
0000:004041E6                 push    eax             ; hDC
0000:004041E7                 call    FrameRect
0000:004041EC                 mov     eax, [ebp+hbr]
0000:004041EF                 push    eax             ; HGDIOBJ
0000:004041F0                 call    DeleteObject
0000:004041F5                 push    0DCDCDCh        ; COLORREF
0000:004041FA                 mov     eax, [ebp+hDC]
0000:004041FD                 push    eax             ; HDC
0000:004041FE                 call    SetTextColor
0000:00404203                 push    1               ; int
0000:00404205                 mov     eax, [ebp+hDC]
0000:00404208                 push    eax             ; HDC
0000:00404209                 call    SetBkMode
0000:0040420E                 mov     [ebp+Rect.left], 2
0000:00404215                 mov     [ebp+Rect.top], 2
0000:0040421C                 sub     [ebp+Rect.bottom], 2
0000:00404220                 push    offset locret_4042A4 ; LPCSTR
0000:00404225                 push    0               ; DWORD
0000:00404227                 push    0               ; DWORD
0000:00404229                 push    0               ; DWORD
0000:0040422B                 push    0               ; DWORD
0000:0040422D                 push    1               ; DWORD
0000:0040422F                 push    0               ; DWORD
0000:00404231                 push    0               ; DWORD
0000:00404233                 push    0               ; DWORD
0000:00404235                 push    2BCh            ; int
0000:0040423A                 push    0               ; int
0000:0040423C                 push    0               ; int
0000:0040423E                 push    0               ; int
0000:00404240                 push    0FFFFFFF4h      ; int
0000:00404242                 call    CreateFontA
0000:00404247                 mov     ebx, eax
0000:00404249                 push    ebx             ; HGDIOBJ
0000:0040424A                 mov     eax, [ebp+hDC]
0000:0040424D                 push    eax             ; HDC
0000:0040424E                 call    SelectObject
0000:00404253                 cmp     [ebp+hIcon], 0
0000:00404257                 jz      short loc_40427B
0000:00404259                 push    3               ; diFlags
0000:0040425B                 push    0               ; hbrFlickerFreeDraw
0000:0040425D                 push    0               ; istepIfAniCur
0000:0040425F                 push    10h             ; cyWidth
0000:00404261                 push    10h             ; cxWidth
0000:00404263                 mov     eax, [ebp+hIcon]
0000:00404266                 push    eax             ; hIcon
0000:00404267                 push    2               ; yTop
0000:00404269                 push    2               ; xLeft
0000:0040426B                 mov     eax, [ebp+hDC]
0000:0040426E                 push    eax             ; hdc
0000:0040426F                 call    DrawIconEx
0000:00404274                 mov     [ebp+Rect.left], 14h
0000:0040427B
0000:0040427B loc_40427B:
0000:0040427B                 push    24h             ; uFormat
0000:0040427D                 lea     eax, [ebp+Rect]
0000:00404280                 push    eax             ; lpRect
0000:00404281                 push    0FFFFFFFFh      ; nCount
0000:00404283                 mov     eax, [ebp+var_C]
```
