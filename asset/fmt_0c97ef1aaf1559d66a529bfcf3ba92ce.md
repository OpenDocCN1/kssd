
```asm
StrMidBuffer proc lpszStr:DWORD,nPos:DWORD,nNum:DWORD,lpszSubStr:DWORD
  
  pushad  
  
  mov esi,lpszStr  
  mov edi,lpszSubStr  
  invoke lstrlen,esi  
  mov ebx,eax  
  mov ecx,nNum  
  mov edx,nPos  
  dec edx  
  sub ebx,edx  
  add edx,lpszStr  
  mov esi,edx  
  rep movs BYTE ptr es:[edi],BYTE ptr ds:[esi]  
  mov BYTE ptr [edi+ecx],0  
  popad  
  ret  
StrMidBuffer endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

GetRegKey proc hDlg:DWORD  
  local szRegName[32]:BYTE,szRegNum[64]:BYTE,szRegID[64]:BYTE  
  local nLen1:DWORD,nLen2:DWORD,szTmp1[80]:BYTE,szTmp2[80]:BYTE,szTmp3[32]:BYTE  
  pushad  
  invoke GetDlgItemText,hDlg,IDC_NAME,addr szRegName,sizeof szRegName    ;取用户姓名  
  .if !eax  
    invoke SetDlgItemText,hDlg,IDC_REG,addr szTextErr1  
  .else  
    invoke GetDlgItemText,hDlg,IDC_ID,addr szRegID,sizeof szRegID    ;取注册 ID 编号  
    .if !eax  
      invoke SetDlgItemText,hDlg,IDC_REG,addr szTextErr2  
    .else  
;-------------------------------------------------------------------------------------  
;    将注册 ID 编号和用户姓名连接起来、转换为大写并去掉'-'字符  
;-------------------------------------------------------------------------------------  
      invoke lstrlen,addr szRegName  
      invoke CharUpperBuff,addr szRegName,eax  
      invoke lstrcat,addr szRegID,addr szRegName  
      invoke lstrlen,addr szRegID  
      mov nLen1,eax  
      mov ecx,eax  
      xor ebx,ebx  
      xor edx,edx  
    @1:  
      mov al,BYTE ptr [szRegID+edx]  
      inc edx  
      dec ecx  
      cmp al,2dh  
      je @1  
      mov BYTE ptr [szTmp1+ebx],al  
      inc ebx  
      cmp al,0  
      jnz @1  
;---------------------------------------------------------------------------------------  
;    在给定字符串中循环取相应的字符并组成新字符串 1  
;---------------------------------------------------------------------------------------  
      invoke lstrlen,addr szStr1  
      mov nLen2,eax  
      mov ecx,nLen1  
      xor edx,edx  
      xor esi,esi  
    @2:   
      push ecx  
      mov ecx,nLen2  
      mov al,BYTE ptr [szTmp1+edx]  
      lea edi,szStr1  
      repne scas BYTE ptr es:[edi]  
      mov ebx,nLen2  
      sub ebx,ecx  
      dec ebx  
      mov al,BYTE ptr [szStr2+ebx]  
      mov BYTE ptr [szTmp2+esi],al  
      inc edx  
      inc esi  
      pop ecx  
      dec ecx  
      jnz @2  
      mov BYTE ptr [szTmp2+esi-1],0  
;--------------------------------------------------------------------------------------  
;    在新字符串 1 中取相应的子字符串并连接成新字符串 2  
;--------------------------------------------------------------------------------------  
      invoke lstrlen,addr szTmp2  
      mov nLen1,eax  
      invoke RtlZeroMemory, addr szTmp1, sizeof szTmp1  
      invoke StrMidBuffer,addr szTmp2,10h,1,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      mov ecx,nLen1  
      sub ecx,12h  
      invoke StrMidBuffer,addr szTmp2,13h,ecx,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,1,5,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,10h,1,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,6,5,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,11h,1,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      mov ecx,nLen1  
      sub ecx,12h  
      invoke StrMidBuffer,addr szTmp2,13h,ecx,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,0Bh,5,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,11h,1,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,6,5,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,6,5,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,11h,1,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      mov ecx,nLen1  
      sub ecx,12h  
      invoke StrMidBuffer,addr szTmp2,13h,ecx,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,0Bh,5,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,11h,1,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,6,5,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      invoke StrMidBuffer,addr szTmp2,12h,1,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
      mov ecx,nLen1  
      sub ecx,12h  
      invoke StrMidBuffer,addr szTmp2,13h,ecx,addr szTmp3  
      invoke lstrcat,addr szTmp1,addr szTmp3  
;------------------------------------------------------------------------------------------  
;    在新字符串 2 中循环取相应的字符组成新字符串 3  
;------------------------------------------------------------------------------------------  
      invoke RtlZeroMemory, addr szTmp2, sizeof szTmp2  
      invoke lstrlen,addr szTmp1  
      mov nLen1,eax  
      mov ecx,eax  
      xor edi,edi  
      xor edx,edx  
      xor esi,esi  
    @3:  
      movzx ebx,BYTE ptr [szTmp1+esi]  
      add edi,ebx  
      mov eax,edi  
      push ecx  
      mov ecx,3eh  
      cdq  
      idiv ecx  
      mov bl,BYTE ptr [szStr1+edx]  
      mov BYTE ptr [szTmp2+esi],bl  
      pop ecx  
      inc esi  
      dec ecx  
      jnz @3  
;--------------------------------------------------------------------------------------------  
;    在新字符串 3 中取相应的了字符串与固定字符串连接并转换成大写生成序列号  
;--------------------------------------------------------------------------------------------  
      invoke RtlZeroMemory, addr szRegNum, sizeof szRegNum  
      mov edx,nLen1  
      sub edx,13h  
      mov ecx,0Bh  
      invoke StrMidBuffer,addr szTmp2,edx,ecx,addr szTmp1  
      invoke CharUpperBuff,addr szTmp1,ecx  
      invoke StrMidBuffer,addr szTmp1,1,5,addr szTmp3  
      invoke lstrcat,addr szRegNum,addr szTmp3  
      mov BYTE ptr [szRegNum+5],2dh  
      invoke StrMidBuffer,addr szTmp1,6,5,addr szTmp3  
      invoke lstrcat,addr szRegNum,addr szTmp3  
      mov BYTE ptr [szRegNum+11],2dh  
      invoke StrMidBuffer,addr szTmp1,0Bh,1,addr szTmp3  
      invoke lstrcat,addr szRegNum,addr szTmp3  
      invoke lstrcat,addr szRegNum,addr szStrCon  
      invoke SetDlgItemText,hDlg,IDC_REG,addr szRegNum  
    .endif  
  .endif  
  popad  
  ret  
GetRegKey endp
```

**版权声明**：本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
