
# 反恐精英 1.5

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-243.htm`](https://www.pediy.com/kssd/pediy08/pediy8-243.htm)

用`金山游侠`和`Ollydbg`可以得知：

改目录`\\反恐精英\\cstrike\\dlls`下`mp.dll`文件以下值可使应的武器子弹无限

-------------------------------------------------------------------

  内存偏移值    文件偏移值    武器选号    武器名

1、  `c2cca0d`    `ca0d`    `B11`    `K&M .45`
2、  `c2c5790`    `5790`    `B12`    `9*19mm`
3、  `c2c3508`    `3508`    `B13`    `Night Hawk`
4、  `c2c9728`    `9728`    `B14`    `228`
5、  `c2c3bac`    `3bac`    `B15`    `.40 Dual`
6、  `c2c4348`    `4348`    `B16`    `Five-Seven`

1、  `c2c7951`    `7951`    `B21`    `12 Gauge`
2、  `c2ccf21`    `cf21`    `B22`    `Auto Shotgun`

1、  `c2c9012`    `9012`    `B31`    `SMG`
2、  `c2cbbd8`    `bbd8`    `B32`    `Schmidt MP`
3、  `c2c9cfd`    `9cfd`    `B33`    `ES C90`
4、  `c2c8a32`    `8a32`    `B34`    `Mac-10`
5、  `c2cc18f`    `c18f`    `B35`    `K&M UMP45`

1、  `c2c1352`    `1352`    `B41`    `CV-47`
2、  `c2cafb2`    `afb2`    `B42`    `Krieg 552`
3、  `c2c8348`    `8348`    `B43`    `M4A1`
4、  `c2c1da2`    `1da2`    `B44`    `Bullpup`
5、  `c2ca2fd`    `a2fd`    `B45`    `Schmidt`
6、  `c2c250e`    `250e`    `B46`    `Magnum`
7、  `c2c4f03`    `4f03`    `B47`    `D3AU1`
8、  `c2ca9b3`    `a9b3`    `B48`    `Krieg 550`

1、  `c2c73d2`    `73d2`    `B51`    `ES M249`

1、  `c2c4a10`    `4a10`    `B83`    `Flashbang`
2、  `c2c5fd6`    `5fd6`    `B84`    `HE Grenade`
3、  `c2cb7b8`    `b7b8`    `B85`    `Smoke Grenade`

-------------------------------------------------------------------

据此用`loader`来实现动态修改，要注意游戏会检测是否处于调试状态。。。

源代码如下：

```asm
;                                                                

    .486                       ; create 32 bit code 

    .model flat, stdcall       ; 32 bit memory model 

    option casemap :none       ; case sensitive 

    include \masm32\include\windows.inc 

    include \masm32\include\masm32.inc 

    include \masm32\include\gdi32.inc 

    include \masm32\include\user32.inc 

    include \masm32\include\kernel32.inc 

    include \masm32\include\Comctl32.inc 

    include \masm32\include\comdlg32.inc 

    include \masm32\include\shell32.inc 

    include \masm32\include\oleaut32.inc 

    includelib \masm32\lib\masm32.lib 

    includelib \masm32\lib\gdi32.lib 

    includelib \masm32\lib\user32.lib 

    includelib \masm32\lib\kernel32.lib 

    includelib \masm32\lib\Comctl32.lib 

    includelib \masm32\lib\comdlg32.lib 

    includelib \masm32\lib\shell32.lib 

    includelib \masm32\lib\oleaut32.lib 

;                                                                

    .data 

    CS_AppName1 db "cstrike.exe",0 

    CS_AppName2 db "hl.exe",0 

    CS_CommandLine db " -console -game cstrike",0 

    CS_StartupInfo db 48h dup (0) 

    BaseOfMpDll dd 0 

    IsModified dd 0 

;-------------------------------------------------

nFileNameSize=60h 

PatcherFileName db nFileNameSize dup (0) 

WeaponName db 10h dup(0) 

nWeaponNameSize dd 0 

        lpBaseAddressx  dd 05790h 

        oBufferx        db 048h 

        mBufferx        db 090h 

        nSizex          = 1h 

        lpBaseAddressx1  dd 05790h 

        oBufferx1        db 048h 

        mBufferx1        db 090h 

        nSizex1          = 1h 

        lpBaseAddressx2  dd 05790h 

        oBufferx2        db 048h 

        mBufferx2        db 090h 

        nSizex2          = 1h 

;-------------------------------------------------

        hProcess       dd 0 

        tBuffer        db 40h dup(0) 

;   MpCheck

        lpBaseAddress  = 0ca0dh 

        oBuffer        db 04bh,06ah,05h 

        mBuffer        db 090h,090h,090h 

        nSize          = 3h 

        lpBaseAddress0  = 0193240ah 

        oBuffer0        db 074h 

        mBuffer0        db 0ebh 

        nSize0          = 1h 

        lpBaseAddress1  = 0ca0dh 

        oBuffer1        db 04bh 

        mBuffer1        db 090h 

        nSize1          = 1h 

        lpBaseAddress2  = 05790h 

        oBuffer2        db 048h 

        mBuffer2        db 090h 

        nSize2          = 1h 

        lpBaseAddress3  = 03508h 

        oBuffer3        db 048h 

        mBuffer3        db 090h 

        nSize3          = 1h 

        lpBaseAddress4  = 09728h 

        oBuffer4        db 048h 

        mBuffer4        db 090h 

        nSize4          = 1h 

        lpBaseAddress5  = 03bach 

        oBuffer5        db 048h 

        mBuffer5        db 090h 

        nSize5          = 1h 

        lpBaseAddress6  = 04348h 

        oBuffer6        db 048h 

        mBuffer6        db 090h 

        nSize6          = 1h 

        lpBaseAddress7  = 07951h 

        oBuffer7        db 04ah 

        mBuffer7        db 090h 

        nSize7          = 1h 

        lpBaseAddress8  = 0cf21h 

        oBuffer8        db 04ah 

        mBuffer8        db 090h 

        nSize8          = 1h 

        lpBaseAddress9  = 09012h 

        oBuffer9        db 048h 

        mBuffer9        db 090h 

        nSize9          = 1h 

        lpBaseAddress10  = 0bbd8h 

        oBuffer10        db 048h 

        mBuffer10        db 090h 

        nSize10          = 1h 

        lpBaseAddress11  = 09cfdh 

        oBuffer11        db 048h 

        mBuffer11        db 090h 

        nSize11          = 1h 

        lpBaseAddress12  = 08a32h 

        oBuffer12        db 048h 

        mBuffer12        db 090h 

        nSize12          = 1h 

        lpBaseAddress13  = 0c18fh 

        oBuffer13        db 048h 

        mBuffer13        db 090h 

        nSize13          = 1h 

        lpBaseAddress14  = 01352h 

        oBuffer14        db 048h 

        mBuffer14        db 090h 

        nSize14          = 1h 

        lpBaseAddress15  = 0afb2h 

        oBuffer15        db 048h 

        mBuffer15        db 090h 

        nSize15          = 1h 

        lpBaseAddress16  = 08348h 

        oBuffer16        db 048h 

        mBuffer16        db 090h 

        nSize16          = 1h 

        lpBaseAddress17  = 01da2h 

        oBuffer17        db 048h 

        mBuffer17        db 090h 

        nSize17          = 1h 

        lpBaseAddress18  = 0a2fdh 

        oBuffer18        db 048h 

        mBuffer18        db 090h 

        nSize18          = 1h 

        lpBaseAddress19  = 0250eh 

        oBuffer19        db 048h 

        mBuffer19        db 090h 

        nSize19          = 1h 

        lpBaseAddress20  = 04f03h 

        oBuffer20        db 048h 

        mBuffer20        db 090h 

        nSize20          = 1h 

        lpBaseAddress21  = 0a9b3h 

        oBuffer21        db 048h 

        mBuffer21        db 090h 

        nSize21          = 1h 

        lpBaseAddress22  = 073d2h 

        oBuffer22        db 048h 

        mBuffer22        db 090h 

        nSize22          = 1h 

        lpBaseAddress23  = 04a10h 

        oBuffer23        db 04ah 

        mBuffer23        db 090h 

        nSize23          = 1h 

        lpBaseAddress24  = 05fd6h 

        oBuffer24        db 04ah 

        mBuffer24        db 090h 

        nSize24          = 1h 

        lpBaseAddress25  = 0b7b8h 

        oBuffer25        db 04ah 

        mBuffer25        db 090h 

        nSize25          = 1h 

       CopyRightCaption db "反恐精英 V1.5 子弹无限作弊器",0 

       CopyRightText    db "使用方法：将本作弊器文件名改为欲作弊武器的购买按键序列后放到反恐目录下运行。",0dh,0ah 

                       db "          如要 AK47 子弹无限，文件名就改为 b41(闪弹/手雷/烟弹用 b83/b84/b85 表示)。",0dh,0ah 

                       db "          支持<单种><两种>和<全部>三种作弊模式，模式由作弊器文件名来识别。",0dh,0ah 

                       db "          <全部>为默认模式，改名时要注意格式一定要对，具体例子格式如下：",0dh,0ah,0dh,0ah 

                       db "          作弊模式        文件名格式        作弊武器",0dh,0ah 

                       db "          单种            b41               AK47",0dh,0ah 

                       db "          两种            b13+b46           Deagle 和 AWP",0dh,0ah 
```



```assembly
            db "          全部            其它              所有武器",0dh,0ah,0dh,0ah  

            db "编制说明：win32 汇编语言编写，由 rgbsky(俞凯) 完成，谢谢朋友们的使用！",0dh,0ah  

            db "编制时间：2005 年 6 月 18 日      版本：Ver1.2",0dh,0ah  

            db "作者联系：rgbsky@sohu.com  rgbsky@hotmail.com     (QQ：175583470)",0  

    SecceedCaption db "成功",0  

    SecceedText    db "修改器顺利工作。",0          

    Error1Caption  db "失败",0  

    Error1Text     db "反恐程序未找到，请确保该修改器与 cstrike.exe 或 hl.exe 在同一目录下。",0          

    Error2Caption  db "失败",0  

    Error2Text     db "'模块文件名获取失败。",0  

    Error3Caption  db "失败",0  

    Error3Text     db "'读取内存时产生错误。",0  

    Error4Caption  db "失败",0  

    Error4Text     db "文件不匹配，请检查反恐精英的版本是否为 Ver1.5。",0  

    Error5Caption  db "失败",0  

    Error5Text     db "写入内存时产生错误。",0  

    .data?  

    DBEvent DEBUG\_EVENT <>  

    CS\_ProcessInfo PROCESS\_INFORMATION <>  

    .code  

; ??????????????????????????????

start:  

; ??????????????????????????????

invoke GetModuleFileName,NULL,addr PatcherFileName,nFileNameSize  

test eax,eax  

jz BeginPatcher  

mov nWeaponNameSize,eax  

mov ecx,eax  

sub ecx,4  

obtainlength:  

        dec ecx  

        mov ebx,offset PatcherFileName  

        mov al,byte ptr [ebx+ecx]  

        cmp al,''  

        jz getlength  

        cmp al,60h  

        jb uppercase  

        sub al,20h  

uppercase:  

        mov byte ptr [ebx+ecx],al  

        jmp obtainlength  

getlength:  

        sub nWeaponNameSize,5  

        sub nWeaponNameSize,ecx  

        xor esi,esi  

fillweaponname:  

        inc ecx  

        mov ebx,offset PatcherFileName  

        mov al,byte ptr [ebx+ecx]  

        mov edx,offset WeaponName  

        mov byte ptr [edx+esi],al  

        inc esi  

        cmp esi,nWeaponNameSize  

        jnz fillweaponname  

        .IF esi==3h  

        jmp Init1Weapon  

        .ELSEIF esi==7h  

        jmp Init2Weapon  

        .ELSE  

        jmp BeginPatcher  

        .ENDIF  

Init1Weapon:  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='1'  

        mov lpBaseAddressx,0ca0dh  

        mov oBufferx,04bh  

        jmp BeginPatcher  

        .ENDIF          

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='2'  

        mov lpBaseAddressx,05790h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='3'  

        mov lpBaseAddressx,03508h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='4'  

        mov lpBaseAddressx,09728h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='5'  

        mov lpBaseAddressx,03bach  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='6'  

        mov lpBaseAddressx,04348h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='2' && WeaponName[2]=='1'  

        mov lpBaseAddressx,07951h  

        mov oBufferx,04ah  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='2' && WeaponName[2]=='2'  

        mov lpBaseAddressx,0cf21h  

        mov oBufferx,04ah  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='1'  

        mov lpBaseAddressx,09012h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='2'  

        mov lpBaseAddressx,0bbd8h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='3'  

        mov lpBaseAddressx,09cfdh  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='4'  

        mov lpBaseAddressx,08a32h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='5'  

        mov lpBaseAddressx,0c18fh  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='1'  

        mov lpBaseAddressx,01352h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='2'  

        mov lpBaseAddressx,0afb2h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='3'  

        mov lpBaseAddressx,08348h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='4'  

        mov lpBaseAddressx,01da2h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='5'  

        mov lpBaseAddressx,0a2fdh  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='6'  

        mov lpBaseAddressx,0250eh  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='7'  

        mov lpBaseAddressx,04f03h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='8'  

        mov lpBaseAddressx,0a9b3h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='5' && WeaponName[2]=='1'  

        mov lpBaseAddressx,073d2h  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='8' && WeaponName[2]=='3'  

        mov lpBaseAddressx,04a10h  

        mov oBufferx,04ah  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='8' && WeaponName[2]=='4'  

        mov lpBaseAddressx,05fd6h  

        mov oBufferx,04ah  

        jmp BeginPatcher  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='8' && WeaponName[2]=='5'  

        mov lpBaseAddressx,0b7b8h  

        mov oBufferx,04ah  

        jmp BeginPatcher  

        .ENDIF  

        mov nWeaponNameSize,0  

        jmp BeginPatcher  

Init2Weapon:  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='1'  

        mov lpBaseAddressx1,0ca0dh  

        mov oBufferx1,04bh  

        jmp testb2  

        .ENDIF          

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='2'  

        mov lpBaseAddressx1,05790h  

        jmp testb2  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='3'  

        mov lpBaseAddressx1,03508h  

        jmp testb2  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='4'  

        mov lpBaseAddressx1,09728h  

        jmp testb2  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='5'  

        mov lpBaseAddressx1,03bach  

        jmp testb2  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='1' && WeaponName[2]=='6'  

        mov lpBaseAddressx1,04348h  

        jmp testb2  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='2' && WeaponName[2]=='1'  

        mov lpBaseAddressx1,07951h  

        mov oBufferx1,04ah  

        jmp testb2  

        .ENDIF  

        .IF WeaponName[0]=='B' && WeaponName[1]=='2' && WeaponName[2]=='2'  

        mov lpBaseAddressx1,0cf21h  

        mov oBufferx1,04ah  

        jmp testb2  

        .ENDIF  
```



```asm
        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='1'  
        mov lpBaseAddressx1,09012h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='2'  
        mov lpBaseAddressx1,0bbd8h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='3'  
        mov lpBaseAddressx1,09cfdh  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='4'  
        mov lpBaseAddressx1,08a32h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='5'  
        mov lpBaseAddressx1,0c18fh  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='1'  
        mov lpBaseAddressx1,01352h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='2'  
        mov lpBaseAddressx1,0afb2h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='3'  
        mov lpBaseAddressx1,08348h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='4'  
        mov lpBaseAddressx1,01da2h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='5'  
        mov lpBaseAddressx1,0a2fdh  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='6'  
        mov lpBaseAddressx1,0250eh  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='7'  
        mov lpBaseAddressx1,04f03h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='4' && WeaponName[2]=='8'  
        mov lpBaseAddressx1,0a9b3h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='5' && WeaponName[2]=='1'  
        mov lpBaseAddressx1,073d2h  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='8' && WeaponName[2]=='3'  
        mov lpBaseAddressx1,04a10h  
        mov oBufferx1,04ah  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='8' && WeaponName[2]=='4'  
        mov lpBaseAddressx1,05fd6h  
        mov oBufferx1,04ah  
        jmp testb2  
        .ENDIF  
        .IF WeaponName[0]=='B' && WeaponName[1]=='8' && WeaponName[2]=='5'  
        mov lpBaseAddressx1,0b7b8h  
        mov oBufferx1,04ah  
        jmp testb2  
        .ENDIF        
        mov nWeaponNameSize,0  
        jmp BeginPatcher  

testb2:        
        .IF WeaponName[4]=='B' && WeaponName[5]=='1' && WeaponName[6]=='1'  
        mov lpBaseAddressx2,0ca0dh  
        mov oBufferx2,04bh  
        jmp BeginPatcher  
        .ENDIF        
        .IF WeaponName[4]=='B' && WeaponName[5]=='1' && WeaponName[6]=='2'  
        mov lpBaseAddressx2,05790h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='1' && WeaponName[6]=='3'  
        mov lpBaseAddressx2,03508h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='1' && WeaponName[6]=='4'  
        mov lpBaseAddressx2,09728h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='1' && WeaponName[6]=='5'  
        mov lpBaseAddressx2,03bach  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='1' && WeaponName[6]=='6'  
        mov lpBaseAddressx2,04348h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='2' && WeaponName[6]=='1'  
        mov lpBaseAddressx2,07951h  
        mov oBufferx2,04ah  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='2' && WeaponName[6]=='2'  
        mov lpBaseAddressx2,0cf21h  
        mov oBufferx2,04ah  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='3' && WeaponName[6]=='1'  
        mov lpBaseAddressx2,09012h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='3' && WeaponName[6]=='2'  
        mov lpBaseAddressx2,0bbd8h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='3' && WeaponName[6]=='3'  
        mov lpBaseAddressx2,09cfdh  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='3' && WeaponName[6]=='4'  
        mov lpBaseAddressx2,08a32h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='3' && WeaponName[6]=='5'  
        mov lpBaseAddressx2,0c18fh  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='1'  
        mov lpBaseAddressx2,01352h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='2'  
        mov lpBaseAddressx2,0afb2h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='3'  
        mov lpBaseAddressx2,08348h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='4'  
        mov lpBaseAddressx2,01da2h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='5'  
        mov lpBaseAddressx2,0a2fdh  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='6'  
        mov lpBaseAddressx2,0250eh  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='7'  
        mov lpBaseAddressx2,04f03h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='4' && WeaponName[6]=='8'  
        mov lpBaseAddressx2,0a9b3h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='5' && WeaponName[6]=='1'  
        mov lpBaseAddressx2,073d2h  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='8' && WeaponName[6]=='3'  
        mov lpBaseAddressx2,04a10h  
        mov oBufferx2,04ah  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='8' && WeaponName[6]=='4'  
        mov lpBaseAddressx2,05fd6h  
        mov oBufferx2,04ah  
        jmp BeginPatcher  
        .ENDIF  
        .IF WeaponName[4]=='B' && WeaponName[5]=='8' && WeaponName[6]=='5'  
        mov lpBaseAddressx2,0b7b8h  
        mov oBufferx2,04ah  
        jmp BeginPatcher  
        .ENDIF  
        mov nWeaponNameSize,0  
        jmp BeginPatcher  

;------------------------------------------------------  
BeginPatcher:  
    invoke MessageBox,NULL, addr CopyRightText, addr CopyRightCaption,MB_OK or MB_ICONINFORMATION  
    invoke CreateProcess,addr CS_AppName1,addr CS_CommandLine,0,0,0,DEBUG_PROCESS+DEBUG_ONLY_THIS_PROCESS,0,0,addr CS_StartupInfo,addr CS_ProcessInfo  
    test eax,eax  
    jnz createsucceed  
    invoke CreateProcess,addr CS_AppName2,addr CS_CommandLine,0,0,0,DEBUG_PROCESS+DEBUG_ONLY_THIS_PROCESS,0,0,addr CS_StartupInfo,addr CS_ProcessInfo  
    test eax,eax  
    jz error1  
createsucceed:  
    .while TRUE  
```



```asm
      invoke WaitForDebugEvent, addr DBEvent, INFINITE

      .if DBEvent.dwDebugEventCode==EXIT_PROCESS_DEBUG_EVENT

         jmp exit

      .elseif DBEvent.dwDebugEventCode==CREATE_PROCESS_DEBUG_EVENT

         mov eax,DBEvent.u.CreateProcessInfo.hProcess

         mov hProcess,eax

      .elseif DBEvent.dwDebugEventCode==LOAD_DLL_DEBUG_EVENT

         mov eax,DBEvent.u.LoadDll.lpBaseOfDll

         mov hProcess,eax

         invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize,NULL

;          test eax,eax

;          jz error3

;------------------------------------------------------------------------------------------------

         xor esi,esi

         mov esi,nSize

goon:     dec esi

         mov ebx,offset oBuffer

         mov al,byte ptr [ebx+esi]

         mov ebx,offset tBuffer

         mov dl,byte ptr [ebx+esi]

         cmp al,dl

         jnz cdevent

         cmp esi,0

         jnz goon

cmp IsModified,1

jz cdevent

;----------------------------------------------------

        mov eax,lpBaseAddress0

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize0,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSize0

goon0:  dec esi

        mov ebx,offset oBuffer0

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goon0

        mov eax,lpBaseAddress0

        invoke WriteProcessMemory,hProcess,eax,addr mBuffer0,nSize0,NULL

        test eax,eax

        jz error5

;------------------------------------------------------------------------------------------------

        mov esi,nWeaponNameSize

        .IF esi==3h

        jmp Patcher1Weapon

        .ELSEIF esi==7h

        jmp Patcher2Weapon

        .ELSE

        jmp PatcherAllWeapon

        .ENDIF

Patcher1Weapon:

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddressx

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSizex,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSizex

goonx:  dec esi

        mov ebx,offset oBufferx

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goonx

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddressx

        invoke WriteProcessMemory,hProcess,eax,addr mBufferx,nSizex,NULL

        test eax,eax

        jz error5

        mov IsModified,1

        jmp cdevent

Patcher2Weapon:

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddressx1

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSizex1,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSizex1

goonx1:  dec esi

        mov ebx,offset oBufferx1

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goonx1

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddressx1

        invoke WriteProcessMemory,hProcess,eax,addr mBufferx1,nSizex1,NULL

        test eax,eax

        jz error5

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddressx2

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSizex2,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSizex2

goonx2:  dec esi

        mov ebx,offset oBufferx2

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goonx2

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddressx2

        invoke WriteProcessMemory,hProcess,eax,addr mBufferx2,nSizex2,NULL

        test eax,eax

        jz error5

        mov IsModified,1

        jmp cdevent

PatcherAllWeapon:

;------------------------------------------------------------------------------------------------

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress1

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize1,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSize1

goon1:  dec esi

        mov ebx,offset oBuffer1

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goon1

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress1

        invoke WriteProcessMemory,hProcess,eax,addr mBuffer1,nSize1,NULL

        test eax,eax

        jz error5

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress2

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize2,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSize2

goon2:  dec esi

        mov ebx,offset oBuffer2

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goon2

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress2

        invoke WriteProcessMemory,hProcess,eax,addr mBuffer2,nSize2,NULL

        test eax,eax

        jz error5

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress3

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize3,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSize3

goon3:  dec esi

        mov ebx,offset oBuffer3

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goon3

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress3

        invoke WriteProcessMemory,hProcess,eax,addr mBuffer3,nSize3,NULL

        test eax,eax

        jz error5

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress4

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize4,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSize4

goon4:  dec esi

        mov ebx,offset oBuffer4

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goon4

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress4

        invoke WriteProcessMemory,hProcess,eax,addr mBuffer4,nSize4,NULL

        test eax,eax

        jz error5

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress5

        invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize5,NULL

        test eax,eax

        jz error3

        xor esi,esi

        mov esi,nSize5

goon5:  dec esi

        mov ebx,offset oBuffer5

        mov al,byte ptr [ebx+esi]

        mov ebx,offset tBuffer

        mov dl,byte ptr [ebx+esi]

        cmp al,dl

        jnz error4

        cmp esi,0

        jnz goon5

        mov eax,DBEvent.u.LoadDll.lpBaseOfDll

        add eax,lpBaseAddress5

        invoke WriteProcessMemory,hProcess,eax,addr mBuffer5,nSize5,NULL

        test eax,eax

        jz error5
```



```asm
mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress6  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize6,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize6  

goon6:  dec esi  

mov ebx,offset oBuffer6  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon6  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress6  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer6,nSize6,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress7  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize7,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize7  

goon7:  dec esi  

mov ebx,offset oBuffer7  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon7  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress7  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer7,nSize7,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress8  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize8,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize8  

goon8:  dec esi  

mov ebx,offset oBuffer8  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon8  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress8  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer8,nSize8,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress9  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize9,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize9  

goon9:  dec esi  

mov ebx,offset oBuffer9  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon9  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress9  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer9,nSize9,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress10  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize10,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize10  

goon10:  dec esi  

mov ebx,offset oBuffer10  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon10  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress10  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer10,nSize10,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress11  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize11,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize11  

goon11:  dec esi  

mov ebx,offset oBuffer11  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon11  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress11  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer11,nSize11,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress12  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize12,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize12  

goon12:  dec esi  

mov ebx,offset oBuffer12  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon12  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress12  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer12,nSize12,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress13  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize13,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize13  

goon13:  dec esi  

mov ebx,offset oBuffer13  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon13  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress13  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer13,nSize13,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress14  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize14,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize14  

goon14:  dec esi  

mov ebx,offset oBuffer14  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon14  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress14  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer14,nSize14,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress15  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize15,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize15  

goon15:  dec esi  

mov ebx,offset oBuffer15  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon15  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress15  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer15,nSize15,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress16  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize16,NULL  

test eax,eax  

jz error3  

xor esi,esi  

mov esi,nSize16  

goon16:  dec esi  

mov ebx,offset oBuffer16  

mov al,byte ptr [ebx+esi]  

mov ebx,offset tBuffer  

mov dl,byte ptr [ebx+esi]  

cmp al,dl  

jnz error4  

cmp esi,0  

jnz goon16  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress16  

invoke WriteProcessMemory,hProcess,eax,addr mBuffer16,nSize16,NULL  

test eax,eax  

jz error5  

mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

add eax,lpBaseAddress17  

invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize17,NULL  
```



```asm
    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize17  

goon17:  dec esi  

    mov ebx,offset oBuffer17  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon17  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress17  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer17,nSize17,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress18  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize18,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize18  

goon18:  dec esi  

    mov ebx,offset oBuffer18  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon18  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress18  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer18,nSize18,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress19  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize19,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize19  

goon19:  dec esi  

    mov ebx,offset oBuffer19  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon19  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress19  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer19,nSize19,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress20  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize20,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize20  

goon20:  dec esi  

    mov ebx,offset oBuffer20  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon20  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress20  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer20,nSize20,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress21  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize21,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize21  

goon21:  dec esi  

    mov ebx,offset oBuffer21  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon21  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress21  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer21,nSize21,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress22  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize22,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize22  

goon22:  dec esi  

    mov ebx,offset oBuffer22  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon22  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress22  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer22,nSize22,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress23  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize23,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize23  

goon23:  dec esi  

    mov ebx,offset oBuffer23  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon23  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress23  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer23,nSize23,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress24  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize24,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize24  

goon24:  dec esi  

    mov ebx,offset oBuffer24  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon24  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress24  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer24,nSize24,NULL  

    test eax,eax  

    jz error5  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress25  

    invoke ReadProcessMemory,hProcess,eax,addr tBuffer,nSize25,NULL  

    test eax,eax  

    jz error3  

    xor esi,esi  

    mov esi,nSize25  

goon25:  dec esi  

    mov ebx,offset oBuffer25  

    mov al,byte ptr [ebx+esi]  

    mov ebx,offset tBuffer  

    mov dl,byte ptr [ebx+esi]  

    cmp al,dl  

    jnz error4  

    cmp esi,0  

    jnz goon25  

    mov eax,DBEvent.u.LoadDll.lpBaseOfDll  

    add eax,lpBaseAddress25  

    invoke WriteProcessMemory,hProcess,eax,addr mBuffer25,nSize25,NULL  

    test eax,eax  

    jz error5  

    mov IsModified,1  

    ;    invoke MessageBox,NULL, addr SecceedText, addr SecceedCaption,MB_OK or MB_ICONINFORMATION  

    .elseif DBEvent.dwDebugEventCode==EXCEPTION_DEBUG_EVENT  

    .if DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION_BREAKPOINT  

        invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_CONTINUE  

        .continue  

    .endif  

    .endif  

cdevent:  

    invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_EXCEPTION_NOT_HANDLED  

.endw  

error1:  

    invoke MessageBox,NULL, addr Error1Text, addr Error1Caption,MB_OK or MB_ICONERROR  

    invoke TerminateProcess,CS_ProcessInfo.hProcess,NULL  

    jmp exit  

error2:  

    invoke MessageBox,NULL, addr Error2Text, addr Error2Caption,MB_OK or MB_ICONERROR  

    invoke TerminateProcess,CS_ProcessInfo.hProcess,NULL  

    jmp exit  

error3:  

    invoke MessageBox,NULL, addr Error3Text, addr Error3Caption,MB_OK or MB_ICONERROR  

    invoke TerminateProcess,CS_ProcessInfo.hProcess,NULL  

    jmp exit  

error4:  

    invoke MessageBox,NULL, addr Error4Text, addr Error4Caption,MB_OK or MB_ICONERROR  

    invoke TerminateProcess,CS_ProcessInfo.hProcess,NULL  

    jmp exit  

error5:  

    invoke MessageBox,NULL, addr Error5Text, addr Error5Caption,MB_OK or MB_ICONERROR  

    invoke TerminateProcess,CS_ProcessInfo.hProcess,NULL  

    jmp exit  

exit:  

    invoke CloseHandle,CS_ProcessInfo.hProcess  

    invoke CloseHandle,CS_ProcessInfo.hThread  

    invoke ExitProcess,NULL  
```



`end` `start`

`4dd466d0e1e0cc123afbe504da2462ca`
