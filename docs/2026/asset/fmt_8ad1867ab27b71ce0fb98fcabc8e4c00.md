
```
 00A3ADB8    push       ebp
 00A3ADB9    push       0A3AF37
 00A3ADBE    push       dword ptr fs:[eax]
 00A3ADC1    mov        dword ptr fs:[eax],esp
 00A3ADC4    call       Clipboard
 00A3ADC9    mov        dx,1
 00A3ADCD    call       TClipboard.HasFormat //剪切板是否是一定的格式.初始化的时候他设置的格式为字符串不为空.具体地方自己找吧.
 00A3ADD2    test       al,al
>00A3ADD4    je         00A3AF1A //剪切板没有有字符串就不用验证.
 00A3ADDA    call       Clipboard
 00A3ADDF    lea        edx,[ebp-4C]
 00A3ADE2    call       TClipboard.GetAsText //获取剪切板内容
 00A3ADE7    mov        eax,dword ptr [ebp-4C]
 00A3ADEA    lea        edx,[ebp-48]
 00A3ADED    call       Trim        //去掉空格等.
 00A3ADF2    mov        edx,dword ptr [ebp-48]
 00A3ADF5    mov        eax,dword ptr [ebp-4]
 00A3ADF8    mov        eax,dword ptr [eax+3D8]; TfrmRegister.KeyChecker:TPKVSCheckKey
 00A3ADFE    add        eax,8; TPKVSCheckKey.FKey:string
 00A3AE01    call       @UStrAsg //这是字符串复制用的函数.
 00A3AE06    mov        eax,dword ptr [ebp-4]
 00A3AE09    mov        eax,dword ptr [eax+3D8]; TfrmRegister.KeyChecker:TPKVSCheckKey
 00A3AE0F    mov        dword ptr [ebp-0C],eax
 00A3AE12    lea        eax,[ebp-10]
 00A3AE15    mov        edx,dword ptr [ebp-4]
 00A3AE18    mov        edx,dword ptr [edx+3D8]; TfrmRegister.KeyChecker:TPKVSCheckKey
 00A3AE1E    mov        edx,dword ptr [edx+8]; TPKVSCheckKey.FKey:string
 00A3AE21    call       @UStrLAsg
 00A3AE26    mov        byte ptr [ebp-11],0
 00A3AE2A    mov        al,[00A3B144]; 0x1
 00A3AE2F    push       eax
 00A3AE30    lea        eax,[ebp-50]
 00A3AE33    push       eax
 00A3AE34    xor        ecx,ecx
 00A3AE36    mov        edx,0A3B154; '-'
 00A3AE3B    mov        eax,dword ptr [ebp-10]
 00A3AE3E    call       StringReplace //去掉'-'
 00A3AE43    mov        eax,dword ptr [ebp-50]
 00A3AE46    lea        edx,[ebp-18]
 00A3AE49    call       UpperCase
 00A3AE4E    mov        eax,dword ptr [ebp-18]
 00A3AE51    mov        dword ptr [ebp-20],eax
 00A3AE54    cmp        dword ptr [ebp-20],0 //去掉'-',空格等之后,判断是否为空.
>00A3AE58    je         00A3AE65
 00A3AE5A    mov        eax,dword ptr [ebp-20]
 00A3AE5D    sub        eax,4
 00A3AE60    mov        eax,dword ptr [eax]
 00A3AE62    mov        dword ptr [ebp-20],eax
 00A3AE65    cmp        dword ptr [ebp-20],19 //判断是否是25个字符,测试的时候输入假验证码不就是25个嘛,要联想.
>00A3AE69    jne        00A3AEFE
 00A3AE6F    lea        eax,[ebp-1C]
 00A3AE72    push       eax
 00A3AE73    mov        ecx,3
 00A3AE78    mov        edx,17
 00A3AE7D    mov        eax,dword ptr [ebp-18]
 00A3AE80    call       @UStrCopy  //字符串复制到 dword ptr [ebp-18]
 00A3AE85    lea        eax,[ebp-18]
 00A3AE88    mov        ecx,3
 00A3AE8D    mov        edx,17
 00A3AE92    call       @UStrDelete //删除第23,24,25个字符.也就是删除最后三个字符.汇编中数字都是16进制的!
 00A3AE97    lea        eax,[ebp-54]
 00A3AE9A    push       eax
 00A3AE9B    lea        ecx,[ebp-58]
 00A3AE9E    mov        edx,dword ptr [ebp-18]
 00A3AEA1    mov        eax,dword ptr [ebp-0C]
 00A3AEA4    call       0091D170
 00A3AEA9    mov        edx,dword ptr [ebp-58]
 00A3AEAC    mov        ecx,3
 00A3AEB1    mov        eax,dword ptr [ebp-0C]
 00A3AEB4    call       00921B44
 00A3AEB9    mov        edx,dword ptr [ebp-54]
 00A3AEBC    mov        eax,dword ptr [ebp-1C]
 00A3AEBF    call       @UStrEqual //比较字符串
>00A3AEC4    je         00A3AEF9 //相等就跳,看下面跳的地方有TfrmRegister.btnPaste:TPngSpeedButton,TControl.SetEnabled
 00A3AEC6    lea        eax,[ebp-5C] //就知道让粘贴的图标按钮有效.
 00A3AEC9    push       eax
 00A3AECA    lea        ecx,[ebp-60]
 00A3AECD    mov        edx,dword ptr [ebp-18]
 00A3AED0    mov        eax,dword ptr [ebp-0C]
 00A3AED3    call       0091D338
 00A3AED8    mov        edx,dword ptr [ebp-60]
 00A3AEDB    mov        ecx,3
 00A3AEE0    mov        eax,dword ptr [ebp-0C]
 00A3AEE3    call       00921B44
 00A3AEE8    mov        edx,dword ptr [ebp-5C]
 00A3AEEB    mov        eax,dword ptr [ebp-1C]
 00A3AEEE    call       @UStrEqual
>00A3AEF3    je         00A3AEF9
 00A3AEF5    xor        eax,eax
>00A3AEF7    jmp        00A3AEFB
 00A3AEF9    mov        al,1
 00A3AEFB    mov        byte ptr [ebp-11],al
 00A3AEFE    mov        al,byte ptr [ebp-11]
 00A3AF01    mov        byte ptr [ebp-5],al
 00A3AF04    mov        eax,dword ptr [ebp-4]
 00A3AF07    mov        eax,dword ptr [eax+3D0]; TfrmRegister.btnPaste:TPngSpeedButton
 00A3AF0D    mov        dl,byte ptr [ebp-5]
 00A3AF10    mov        ecx,dword ptr [eax]
 00A3AF12    call       dword ptr [ecx+80]; TControl.SetEnabled
>00A3AF18    jmp        00A3AF2D
 00A3AF1A    mov        eax,dword ptr [ebp-4]
 00A3AF1D    mov        eax,dword ptr [eax+3D0]; TfrmRegister.btnPaste:TPngSpeedButton
 00A3AF23    xor        edx,edx
 00A3AF25    mov        ecx,dword ptr [eax]
 00A3AF27    call       dword ptr [ecx+80]; TControl.SetEnabled
 00A3AF2D    xor        eax,eax
 00A3AF2F    pop        edx
 00A3AF30    pop        ecx
 00A3AF31    pop        ecx
 00A3AF32    mov        dword ptr fs:[eax],edx
>00A3AF35    jmp        00A3AF54
<00A3AF37    jmp        @HandleAnyException
 00A3AF3C    mov        eax,dword ptr [ebp-4]
 00A3AF3F    mov        eax,dword ptr [eax+3D0]; TfrmRegister.btnPaste:TPngSpeedButton
 00A3AF45    xor        edx,edx
 00A3AF47    mov        ecx,dword ptr [eax]
 00A3AF49    call       dword ptr [ecx+80]; TControl.SetEnabled
 00A3AF4F    call       @DoneExcept
 00A3AF54    lea        edx,[ebp-64]
 00A3AF57    mov        eax,dword ptr [ebp-4]
 00A3AF5A    mov        eax,dword ptr [eax+390]; TfrmRegister.KeyEdit1:TKeyEdit
 00A3AF60    call       TKeyEdit.GetKey
 00A3AF65    mov        edx,dword ptr [ebp-64]
 00A3AF68    mov        eax,dword ptr [ebp-4]
 00A3AF6B    mov        eax,dword ptr [eax+3D8]; TfrmRegister.KeyChecker:TPKVSCheckKey
 00A3AF71    add        eax,8; TPKVSCheckKey.FKey:string
 00A3AF74    call       @UStrAsg
 00A3AF79    lea        edx,[ebp-6C]
 00A3AF7C    mov        eax,dword ptr [ebp-4]
 00A3AF7F    mov        eax,dword ptr [eax+3A8]; TfrmRegister.edtName:TEdit
 00A3AF85    call       TControl.GetText
 00A3AF8A    mov        eax,dword ptr [ebp-6C]
 00A3AF8D    lea        edx,[ebp-68]
 00A3AF90    call       Trim
 00A3AF95    cmp        dword ptr [ebp-68],0
>00A3AF99    je         00A3B0A5
 00A3AF9F    mov        eax,dword ptr [ebp-4]
 00A3AFA2    mov        eax,dword ptr [eax+3D8]; TfrmRegister.KeyChecker:TPKVSCheckKey
 00A3AFA8    mov        dword ptr [ebp-28],eax
 00A3AFAB    lea        eax,[ebp-2C]
 00A3AFAE    mov        edx,dword ptr [ebp-4]
 00A3AFB1    mov        edx,dword ptr [edx+3D8]; TfrmRegister.KeyChecker:TPKVSCheckKey
 00A3AFB7    mov        edx,dword ptr [edx+8]; TPKVSCheckKey.FKey:string
 00A3AFBA    call       @UStrLAsg
 00A3AFBF    mov        byte ptr [ebp-2D],0
 00A3AFC3    mov        al,[00A3B144]; 0x1
 00A3AFC8    push       eax
 00A3AFC9    lea        eax,[ebp-70]
 00A3AFCC    push       eax
 00A3AFCD    xor        ecx,ecx
 00A3AFCF    mov        edx,0A3B154; '-'
 00A3AFD4    mov        eax,dword ptr [ebp-2C]
 00A3AFD7    call       StringReplace
 00A3AFDC    mov        eax,dword ptr [ebp-70]
 00A3AFDF    lea        edx,[ebp-34]
 00A3AFE2    call       UpperCase
 00A3AFE7    mov        eax,dword ptr [ebp-34]
 00A3AFEA    mov        dword ptr [ebp-3C],eax
 00A3AFED    cmp        dword ptr [ebp-3C],0
>00A3AFF1    je         00A3AFFE
 00A3AFF3    mov        eax,dword ptr [ebp-3C]
 00A3AFF6    sub        eax,4
 00A3AFF9    mov        eax,dword ptr [eax]
 00A3AFFB    mov        dword ptr [ebp-3C],eax
 00A3AFFE    cmp        dword ptr [ebp-3C],19
>00A3B002    jne        00A3B097
 00A3B008    lea        eax,[ebp-38]
 00A3B00B    push       eax
 00A3B00C    mov        ecx,3
 00A3B011    mov        edx,17
 00A3B016    mov        eax,dword ptr [ebp-34]
 00A3B019    call       @UStrCopy  //复制最后三位到dword ptr [ebp-34]
 00A3B01E    lea        eax,[ebp-34]
 00A3B021    mov        ecx,3
 00A3B026    mov        edx,17
 00A3B02B    call       @UStrDelete
 00A3B030    lea        eax,[ebp-74]
 00A3B033    push       eax
 00A3B034    lea        ecx,[ebp-78]
 00A3B037    mov        edx,dword ptr [ebp-34] //放的是验证注册码的最后三位
 00A3B03A    mov        eax,dword ptr [ebp-28]
 00A3B03D    call       0091D170  //比较关键的call
 00A3B042    mov        edx,dword ptr [ebp-78]
 00A3B045    mov        ecx,3
 00A3B04A    mov        eax,dword ptr [ebp-28]
 00A3B04D    call       00921B44//比较关键的call
 00A3B052    mov        edx,dword ptr [ebp-74]
 00A3B055    mov        eax,dword ptr [ebp-38]
 00A3B058    call       @UStrEqual  //上面一段和一开始的分析是一样的,所以就省略了.
>00A3B05D    je         00A3B092 //同理,这里跳转的话,注册按钮就会被激活.
 00A3B05F    lea        eax,[ebp-7C]
 00A3B062    push       eax
 00A3B063    lea        ecx,[ebp-80]
 00A3B066    mov        edx,dword ptr [ebp-34]
 00A3B069    mov        eax,dword ptr [ebp-28]
 00A3B06C    call       0091D338 //比较关键的call
 00A3B071    mov        edx,dword ptr [ebp-80]
 00A3B074    mov        ecx,3
 00A3B079    mov        eax,dword ptr [ebp-28]
 00A3B07C    call       00921B44 //比较关键的call
 00A3B081    mov        edx,dword ptr [ebp-7C]
 00A3B084    mov        eax,dword ptr [ebp-38]
 00A3B087    call       @UStrEqual
>00A3B08C    je         00A3B092 //同理,这里跳转的话,注册按钮就会被激活.
 00A3B08E    xor        eax,eax
>00A3B090    jmp        00A3B094
 00A3B092    mov        al,1
 00A3B094    mov        byte ptr [ebp-2D],al
 00A3B097    mov        al,byte ptr [ebp-2D]
 00A3B09A    mov        byte ptr [ebp-21],al
 00A3B09D    mov        al,byte ptr [ebp-21]
 00A3B0A0    mov        byte ptr [ebp-3D],al
>00A3B0A3    jmp        00A3B0A9
 00A3B0A5    mov        byte ptr [ebp-3D],0
 00A3B0A9    cmp        byte ptr [ebp-3D],0
>00A3B0AD    je         00A3B0C4
 00A3B0AF    mov        eax,dword ptr [ebp-4]
 00A3B0B2    mov        eax,dword ptr [eax+3D8]; TfrmRegister.KeyChecker:TPKVSCheckKey
 00A3B0B8    call       TPKVSCheckKey.IsBlacklisted
 00A3B0BD    xor        al,1
 00A3B0BF    mov        byte ptr [ebp-3E],al
>00A3B0C2    jmp        00A3B0C8
 00A3B0C4    mov        byte ptr [ebp-3E],0
 00A3B0C8    mov        eax,dword ptr [ebp-4]
 00A3B0CB    mov        eax,dword ptr [eax+3A0]; TfrmRegister.btnRegister:TButton
 00A3B0D1    mov        dl,byte ptr [ebp-3E]
 00A3B0D4    mov        ecx,dword ptr [eax]
 00A3B0D6    call       dword ptr [ecx+80]; TControl.SetEnabled
 00A3B0DC    xor        eax,eax
 00A3B0DE    pop        edx
 00A3B0DF    pop        ecx
 00A3B0E0    pop        ecx
 00A3B0E1    mov        dword ptr fs:[eax],edx
 00A3B0E4    push       0A3B13D
 00A3B0E9    lea        eax,[ebp-80]
 00A3B0EC    mov        edx,5
 00A3B0F1    call       @UStrArrayClr
 00A3B0F6    lea        eax,[ebp-6C]
 00A3B0F9    call       @UStrClr
 00A3B0FE    lea        eax,[ebp-68]
 00A3B101    mov        edx,9
 00A3B106    call       @UStrArrayClr
 00A3B10B    lea        eax,[ebp-38]
 00A3B10E    mov        edx,2
 00A3B113    call       @UStrArrayClr
 00A3B118    lea        eax,[ebp-2C]
 00A3B11B    call       @UStrClr
 00A3B120    lea        eax,[ebp-1C]
 00A3B123    mov        edx,2
 00A3B128    call       @UStrArrayClr
 00A3B12D    lea        eax,[ebp-10]
 00A3B130    call       @UStrClr
 00A3B135    ret
<00A3B136    jmp        @HandleFinally
<00A3B13B    jmp        00A3B0E9
 00A3B13D    pop        edi
 00A3B13E    pop        esi
 00A3B13F    pop        ebx
 00A3B140    mov        esp,ebp
 00A3B142    pop        ebp
 00A3B143    ret
```
