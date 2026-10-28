
# 建立并显示窗口

```asm
;********************************************************************
; 建立并显示窗口
;********************************************************************

                invoke           CreateWindowEx,WS\_EX\_CLIENTEDGE,\
                                 offset szClassName,offset szCaptionMain,\
                                 WS\_OVERLAPPEDWINDOW,\
                                 100,100,800,600,\
                                 NULL,NULL,hInstance,NULL
                mov              hWinMain,eax
                invoke                \_Init
                invoke           ShowWindow,hWinMain,SW\_SHOWNORMAL
                invoke           UpdateWindow,hWinMain
```

# 消息循环

```asm
;********************************************************************
; 消息循环
;********************************************************************

                 .while           TRUE
                                  invoke    GetMessage,addr @stMsg,NULL,0,0
                                  .break    .if eax  == 0
                                  invoke    TranslateMessage,addr @stMsg
                                  invoke    DispatchMessage,addr @stMsg
                 .endw

                 ret

\_WinMain         endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

start:

                 call             \_WinMain

                 invoke           ExitProcess,NULL

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

                 end              start
```

# DDRAW.INC 说明

`DDRAW.INC`这个文件主要是参考 Ewald Snel 这位牛人的.并做了些修改.

```asm
;---------------------------------------;
;        DDRAW include file                                        ;
;                                                                                ;
;        Author :  Ewald Snel                                ;
;        Copyright (C) 1999, Diamond Crew        ;
;                                                                                ;
;        a7aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2W2M7X3g2Q4x3X3g2A6M7#2)9J5c8X3c8A6j5h3#2G2L8X3c8Q4x3V1k6Q4x3U0k6F1j5Y4y4H3i4K6y4n7                               ;
;---------------------------------------;

STDMETHOD        MACRO        name, argl :VARARG
LOCAL @tmp\_a
LOCAL @tmp\_b
@tmp\_a        TYPEDEF PROTO argl
@tmp\_b        TYPEDEF PTR @tmp\_a
name                @tmp\_b                ?
ENDM

DDINVOKE        MACRO        func, this, arglist :VARARG
        mov                        eax , [this]
        mov                        eax , [eax]
        IFB <arglist>
                INVOKE [IDirectDrawVtbl. func][eax], this
        ELSE
                INVOKE [IDirectDrawVtbl. func][eax], this, arglist
        ENDIF
ENDM

DDSINVOKE        MACRO        func, this, arglist :VARARG
        mov                        eax , [this]
        mov                        eax , [eax]
        IFB <arglist>
                INVOKE [IDirectDrawSurfaceVtbl. func][eax], this
        ELSE
                INVOKE [IDirectDrawSurfaceVtbl. func][eax], this, arglist
        ENDIF
ENDM

DDCINVOKE        MACRO        func, this, arglist :VARARG
        mov                        eax , [this]
        mov                        eax , [eax]
        IFB <arglist>
                INVOKE [IDirectDrawClipperVtbl. func][eax], this
        ELSE
                INVOKE [IDirectDrawClipperVtbl. func][eax], this, arglist
        ENDIF
;这一段是本人参考着新加的.为建窗口模式的需要.
ENDM

DirectDrawCreate                PROTO :DWORD, :DWORD, :DWORD
DirectDrawCreateClipper        PROTO :DWORD, :DWORD, :DWORD
DirectDrawEnumerateA        PROTO :DWORD, :DWORD
DirectDrawEnumerate                EQU                <DirectDrawEnumerateA>
EXTERNDEF        C CLSID\_DirectDraw:GUID
EXTERNDEF        C CLSID\_DirectDrawClipper:GUID
EXTERNDEF        C IID\_IDirectDraw:GUID
EXTERNDEF        C IID\_IDirectDrawSurface:GUID
EXTERNDEF        C IID\_IDirectDrawPalette:GUID
EXTERNDEF        C IID\_IDirectDrawClipper:GUID

LPDIRECTDRAW                        TYPEDEF                PTR IDirectDraw
LPDIRECTDRAWSURFACE                TYPEDEF                PTR IDirectDrawSurface
LPDIRECTDRAWPALETTE                TYPEDEF                PTR IDirectDrawPalette
LPDIRECTDRAWCLIPPER                TYPEDEF                PTR IDirectDrawClipper
LPDDFXROP                                TYPEDEF                PTR DDFXROP
LPDDSURFACEDESC                        TYPEDEF                PTR DDSURFACEDESC
LPDDCAPS                                TYPEDEF                PTR DDCAPS
LPDDBLTFX                                TYPEDEF                PTR DDBLTFX
LPDDSCAPS                                TYPEDEF                PTR DDSCAPS
LPDDCOLORKEY                        TYPEDEF                PTR DDCOLORKEY
LPDDOVERLAYFX                        TYPEDEF                PTR DDOVERLAYFX
LPDDPIXELFORMAT                        TYPEDEF                PTR DDPIXELFORMAT
LPDDBLTBATCH                        TYPEDEF                PTR DDBLTBATCH

; windows defines
LPRECT                                        TYPEDEF                PTR RECT
LPPALETTEENTRY                        TYPEDEF                PTR PALETTEENTRY
LPRGNDATA                                TYPEDEF                PTR RGNDATA

DDCOLORKEY        STRUC
        dwColorSpaceLowValue                DWORD                ?
        dwColorSpaceHighValue                DWORD                ?
DDCOLORKEY        ENDS

DDBLTFX                STRUC
        dwSize                                                DWORD                ?
        dwDDFX                                                DWORD                ?
        dwROP                                                DWORD                ?
        dwDDROP                                                DWORD                ?
        dwRotationAngle                                DWORD                ?
        dwZBufferOpCode                                DWORD                ?
        dwZBufferLow                                DWORD                ?
        dwZBufferHigh                                DWORD                ?
        dwZBufferBaseDest                        DWORD                ?
        dwZDestConstBitDepth                DWORD                ?
UNION
        dwZDestConst                                DWORD                ?
        lpDDSZBufferDest                        LPDIRECTDRAWSURFACE                ?
ENDS
        dwZSrcConstBitDepth                        DWORD                ?
UNION
        dwZSrcConst                                        DWORD                ?
        lpDDSZBufferSrc                                LPDIRECTDRAWSURFACE                ?
ENDS
        dwAlphaEdgeBlendBitDepth        DWORD                ?
        dwAlphaEdgeBlend                        DWORD                ?
        dwReserved                                        DWORD                ?
        dwAlphaDestConstBitDepth        DWORD                ?
UNION
        dwAlphaDestConst                        DWORD                ?
        lpDDSAlphaDest                                LPDIRECTDRAWSURFACE                ?
ENDS
        dwAlphaSrcConstBitDepth                DWORD                ?
UNION
        dwAlphaSrcConst                                DWORD                ?
        lpDDSAlphaSrc                                LPDIRECTDRAWSURFACE                ?
ENDS
UNION
        dwFillColor                                        DWORD                ?
        dwFillDepth                                        DWORD                ?
        lpDDSPattern                                LPDIRECTDRAWSURFACE                ?
ENDS
```
