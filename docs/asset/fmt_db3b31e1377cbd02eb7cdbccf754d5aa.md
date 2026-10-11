
```assembly
mov dl,al
pop ecx
mov edx,edi
and edx,7
xor ecx,ecx
mov cl,byte ptr [xorBuffer+edx]
xor eax,ecx
add dword ptr [TempResult], eax
inc edi
cmp byte ptr ds:[NameBuffer+edi],0
jnz ConvertName
mov eax,dword ptr [TempResult]
mov ecx,64h
cdq
idiv ecx
mov dword ptr [ResultName],edx ;保存用户名处理结果

;-----------------------------------
;获得第 2 段注册码,我用穷举来实现
;-----------------------------------

xor edi,edi

@1:
mov al,byte ptr [CodeBase+edi]
mov dl,0Ah
mul dx
mov byte ptr [MulResult],al
xor edx,edx

@2:
mov al,byte ptr [CodeBase+edx]
mov cl,byte ptr [MulResult]
add al,cl
mov ecx,dword ptr [ResultName]
add ecx,210h
cmp al,cl
je SaveCode1
inc edx
cmp edx,9
jl @2
inc edi
cmp edi,9
jl @1

SaveCode1:
mov al,byte ptr [CodeBase+edi]
mov byte ptr [CodeBuffer+6],al ;保存第二段第一位值
mov al,byte ptr [CodeBase+edx]
mov byte ptr [CodeBuffer+7],al ;保存第二段第一位值

;--------------------------------------
;获得第 1、3 段注册码,我以变换用户名实现
;--------------------------------------

xor edi,edi

GetCode:
mov dl,byte ptr [NameBuffer+edi]
add edx,edx
and edx,7
mov al,byte ptr [CodeBase+edx] ;取单个注册码
mov byte ptr [CodeBuffer+edi],al ;填入第一段注册码
add edx,5
and edx,7
mov al,byte ptr [CodeBase+edx] ;取单个注册码
mov byte ptr [CodeBuffer+9+edi],al ;填入第三段注册码
inc edi
cmp edi,5
jl GetCode

;-----------------
;处理 1、3 段注册码
;-----------------

xor edi,edi
mov dword ptr [TempResult],0
xor eax,eax

ConvertCode1:
mov al,byte ptr [CodeBuffer+edi]
push eax
mov dl,al
pop ecx
mov edx,edi
and edx,7
xor ecx,ecx
mov cl,byte ptr [xorBuffer+edx]
xor eax,ecx
add dword ptr [TempResult], eax
inc edi
cmp byte ptr ds:[CodeBuffer+edi],0
jnz ConvertCode1

xor edi,edi
xor eax,eax

ConvertCode2:
mov al,byte ptr [CodeBuffer+9+edi]
push eax
mov dl,al
pop ecx
mov edx,edi
and edx,7
xor ecx,ecx
mov cl,byte ptr [xorBuffer+edx]
xor eax,ecx
add dword ptr [TempResult], eax
inc edi
cmp byte ptr ds:[CodeBuffer+9+edi],0
jnz ConvertCode2

mov eax,dword ptr [TempResult]
mov ecx,64h
cdq
idiv ecx
mov word ptr [ResultCode],dx

;-----------------------------------
;获得第 4 段注册码,我用穷举来实现
;-----------------------------------

xor edi,edi

@3:
mov al,byte ptr [CodeBase+edi]
mov dl,0Ah
mul dx
mov byte ptr [MulResult],al
xor edx,edx

@4:
mov al,byte ptr [CodeBase+edx]
mov cl,byte ptr [MulResult]
add al,cl
mov cx,word ptr [ResultCode]
add cx,210h
cmp al,cl
je SaveCode2
inc edx
cmp edx,9
jl @4
inc edi
cmp edi,9
jl @3

SaveCode2:
mov al,byte ptr [CodeBase+edi]
mov byte ptr [CodeBuffer+15],al ;保存第四段第一位值
mov al,byte ptr [CodeBase+edx]
mov byte ptr [CodeBuffer+16],al ;保存第四段第二位值

;-------------------
;为注册码添加分隔线
;-------------------

mov byte ptr [CodeBuffer+5],2Dh
mov byte ptr [CodeBuffer+8],2Dh
mov byte ptr [CodeBuffer+14],2Dh
ret

CalCode endp
end start
```

--------------------------------------------------------------------------------

### 资源文件

```rc
;>>>>>>>>>>>>>>>>>
;>>> rsrc.rc >>>>>
;>>>>>>>>>>>>>>>>>

#define IDDIALOG 10
#define IDREGNAME 4
#define IDREGCODE 3
#define IDC_STC1 5
#define IDC_STC2 6
#define IDOK 1
#define IDABOUT 2

IDDIALOG DIALOGEX 6,5,196,104
CAPTION "GraphicWorkshop v2.0a 注册机"
FONT 8,"MS Sans Serif"
STYLE 0x10CA0800
EXSTYLE 0x00000000
BEGIN
  CONTROL "",IDREGNAME,"Edit",0x50810000,76,38,106,13,0x00000000
  CONTROL "",IDREGCODE,"Edit",0x50810800,76,60,106,13,0x00000000
  CONTROL "Registration Name",IDC_STC1,"Static",0x50000201,12,38,60,13,0x00000000
  CONTROL "Registration Code",IDC_STC2,"Static",0x50000201,12,60,58,11,0x00000000
  CONTROL "计算",IDOK,"Button",0x50010000,40,81,43,15,0x00000000
  CONTROL "关于",IDABOUT,"Button",0x50010000,114,81,43,15,0x00000000
END
```

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
