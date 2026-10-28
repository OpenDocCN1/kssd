
```
                        db "          如要AK47子弹无限，文件名就改为b41(闪弹/手雷/烟弹用b83/b84/b85表示)。",0dh,0ah

                        db "          支持<单种><两种>和<全部>三种作弊模式，模式由作弊器文件名来识别。",0dh,0ah

                        db "          <全部>为默认模式，改名时要注意格式一定要对，具体例子格式如下：",0dh,0ah,0dh,0ah

                        db "          作弊模式        文件名格式        作弊武器",0dh,0ah

                        db "          单种            b41               AK47",0dh,0ah

                        db "          两种            b13+b46           Deagle和AWP",0dh,0ah

                        db "          全部            其它              所有武器",0dh,0ah,0dh,0ah

                        db "编制说明：win32汇编语言编写，由 rgbsky(俞凯) 完成，谢谢朋友们的使用！",0dh,0ah

                        db "编制时间：2005年6月18日      版本：Ver1.2",0dh,0ah

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

        Error4Text     db "文件不匹配，请检查反恐精英的版本是否为Ver1.5。",0

        Error5Caption  db "失败",0

        Error5Text     db "写入内存时产生错误。",0

    .data?

    DBEvent DEBUG_EVENT <>

    CS_ProcessInfo PROCESS_INFORMATION <>

    .code

; ?????????????????????????????????????

start:

; ?????????????????????????????????????

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

        cmp al,'\'

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

        .IF WeaponName[0]=='B' && WeaponName[1]=='3' && WeaponName[2]=='1'

        mov lpBaseAddressx1,09012h

        jmp testb2
```
