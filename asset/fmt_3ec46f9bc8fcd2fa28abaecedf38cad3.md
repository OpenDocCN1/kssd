
## Data Segment

```
dd 0A7672661H,0D06016F7H,04969474DH,03E6E77DBH,0AED16A4AH,0D9D65ADCH,040DF0B66H,037D83BF0H,0A9BCAE53H,0DEBB9EC5H,047B2CF7FH,030B5FFE9H  

dd 0BDBDF21CH,0CABAC28AH,053B39330H,024B4A3A6H,0BAD03605H,0CDD70693H,054DE5729H,023D967BFH,0B3667A2EH,0C4614AB8H,05D681B02H,02A6F2B94H  

dd 0B40BBE37H,0C30C8EA1H,05A05DF1BH,02D02EF8DH  

hInstance dd 0  

szStr db 'e-Stack Room 3.68',0  

szErr db '请输入用户名！',0  

szErr1 db '对不起，用户名长了',0dh,0ah,\\  
'请更换用户名！',0  

szCaption db '错误！',0
```

## Code Segment

```
.code

;****************************************************************

_CmpNam proc  
cmp eax,251CH  
jl b10  
@@:  
sub eax,251CH  
cmp eax,251CH  
jge @B  
b10:  
cmp eax,3B6H  
jl b20  
@@:  
sub eax,3B6H  
cmp eax,3B6H  
jge @B  
b20:  
cmp eax,5FH  
jl b30  
@@:  
sub eax,5FH  
cmp eax,5FH  
jge @B  
b30:  
test eax,eax  
jge b40  
@@:  
add eax,5FH  
test eax,eax  
jl @B  
b40:  
ret  

_CmpNam endp  

;****************************************************************

_CalcNam proc  
local @Temp1,@Temp2,@Temp3  
local @Temp4:BYTE  
pushad  
mov @Temp1,0A66788H  
invoke  lstrlen,offset szStr  
mov @Temp2,eax  
xor eax,eax  
xor esi,esi  
a10:          
lea eax,szName  
mov al,[eax+esi]  
cmp al,20H  
jc a20  
cmp al,7EH  
ja a20  
mov edx,@Temp1  
and edx,1FFFFFFFH  
mov ecx,@Temp1   
shr ecx,1DH  
and ecx, 31H  
xor edx,ecx  
mov @Temp1,edx  
mov @Temp4,al  
mov eax,@Temp1  
mov ecx,5FH  
cdq  
idiv ecx  
xor edx,edx  
mov dl,@Temp4  
sub edx, 20H  
sub eax,edx  
invoke _CmpNam  
mov ebx,eax  
add bl,20H  
inc @Temp2  
cmp @Temp2,5179H  
jl @F  
xor eax,eax  
mov @Temp2,eax  
@@:          
mov al,@Temp4  
xor al,bl  
and eax,0FFH  
mov edx,@Temp1  
add edx,@Temp1  
add eax,edx  
add eax,@Temp2  
mov @Temp1,eax  
mov byte ptr [Temp+esi],bl  
a20:          
inc esi  
dec edi  
jnz a10  
invoke lstrcat,offset TempNam,offset szName  
invoke lstrcat,offset TempNam,offset szStr  
invoke lstrcat,offset TempNam,offset Temp  
popad  
ret  

_CalcNam endp  

;****************************************************************

_CalcReg proc  
local @Temp1:DWORD  
pushad  
xor ebx,ebx  
invoke lstrlen,offset TempNam  
mov esi,eax  
test esi,esi  
jng c20  
mov edi,01H  
c10:  
mov @Temp1,ebx  
lea eax,TempNam  
mov al,[eax+edi-01H]  
xor al,bl  
and eax,0FFH  
mov eax,DWORD ptr [eax*4+MyTab]  
shr ebx,08H  
and ebx,00FFFFFFH  
xor eax,ebx  
mov ebx,eax  
inc edi  
dec esi  
jnz c10  
invoke dw2hex,ebx,addr szSN  
invoke dw2hex,@Temp1,addr szSN1  
invoke lstrcat,addr szSN,addr szSN1  
c20:  
popad  
ret  

_CalcReg endp  

;****************************************************************

_ProcDlgMain proc uses ebx edi esi ecx edx,hWnd,wMsg,wParam,lParam  
mov eax,wMsg   
.if eax==WM_CLOSE   
invoke  EndDialog,hWnd,NULL  
.elseif eax==WM_COMMAND   
mov eax,wParam   
.if eax == IDOK  
invoke RtlZeroMemory,addr szName,10  
invoke RtlZeroMemory,addr szSN,16  
invoke RtlZeroMemory,addr Temp,10  
invoke RtlZeroMemory,addr TempNam,50  
invoke  GetDlgItemText,hWnd,EditName,offset szName,20  
.if eax !=  NULL  
invoke  lstrlen,offset szName  
mov edi,eax  
.if eax > 0aH  
invoke MessageBox,NULL,offset szErr1,offset szCaption,MB_OK  
mov eax,FALSE  
ret  
.endif  
invoke _CalcNam  
invoke _CalcReg  
invoke SetDlgItemText,hWnd,EditSN,offset szSN  
mov eax,FALSE  
ret  
          .else  
invoke MessageBox,NULL,offset szErr,offset szCaption,MB_OK  
mov eax,FALSE  
ret  
.endif  
.elseif eax  ==  IDCANCEL   
invoke  EndDialog,hWnd,NULL   
.endif   
      .else  
          mov    eax,FALSE   
          ret   
      .endif  
mov eax,TRUE  
ret  

_ProcDlgMain endp
```

## Start Section

```
start:  
;invoke InitCommonControls  
invoke GetModuleHandle,NULL  
mov hInstance,eax  
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlgMain,0  
invoke ExitProcess,NULL  
```

```
end start
```

## Resource File (KeyGen.rc)

```
;KeyGen.rc  
#include   
#define DLG_MAIN 1  
#define EDITName 10  
#define EDITSN 11  
DLG_MAIN DIALOG 100,150,250,60  
STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU | WS_THICKFRAME  
CAPTION "电子书库 V3.68 注册机"   
FONT 9,"宋体"   
{  
 CONTROL "Name:" ,-1,"Static",SS_LEFT,10,13,40,17  
 CONTROL "SN:" ,-2,"Static",SS_CENTER,10,40,20,17  
 CONTROL "" ,10,"Edit",ES_LEFT,30,13,150,10  
 CONTROL "" ,11,"Edit",ES_LEFT,30,40,150,10  
 DEFPUSHBUTTON "GENERATE",IDOK,200,11,40,15  
 PUSHBUTTON "EXIT",IDCANCEL,200,36,41,14  
}
```

## Makefile

```
;MakeFile  
NAME = KeyGen   
OBJS = $(NAME).obj #需要的目标文件  
RES = $(NAME).res #需要的资源文件  
LINK_FLAG = /subsystem:windows #连接选项  
ML_FLAG = /c /coff #编译选项  
$(NAME).exe: $(OBJS) $(RES)  
Link $(LINK_FLAG) $(OBJS) $(RES)  
.asm.obj:  
ml $(ML_FLAG) $<  
.rc.res:  
rc $<  
clean:  
del *.obj  
del *.res
```

`4dd466d0e1e0cc123afbe504da2462ca`
