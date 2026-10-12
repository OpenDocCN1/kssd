
## 汇编代码段

```
mov ebx,dword ptr[result3]

movzx ecx,bl

inc edi

cmp edi,ecx

jl @7

mov edx,dword ptr[result1]      ;累计结果,作为中间值的低 16 位

mov eax,dword ptr[cmpResult]

shl eax,16

or eax,edx                     ;执行后得到中间值

mov dword ptr[cmpResult],eax    ;中间值存放到变量 cmpbuffer 中

;-------------------
;根据中间值反推注册码
;-------------------

xor ebp,ebp

mov eax,dword ptr[cmpResult]

mov dword ptr[cmpbuffer],eax

@8:

xor edi,edi

add edi,30h

@9:

mov eax,dword ptr[cmpbuffer+ebp*4-4]

add eax,30h

sub eax,edi

shr eax,1

cdq

mov ecx,5

div ecx

inc edi

cmp edx,0

jnz @9

mov dword ptr[cmpbuffer+ebp*4],eax   ;我把每次计算得到的累计值保存起来，留做验算时用

mov edx,edi

mov byte ptr[SerialTemp+ebp],dl

inc ebp

cmp eax,0

jnz @8

;--------------------------------
;取出计算出的注册码,验算正确后输出
;--------------------------------

xor edi,edi

xor ecx,ecx

@10:

mov al,byte ptr[SerialTemp+ebp-1]    ;取出计算出来的注册码

movzx eax,al

lea ecx,dword ptr[ecx+ecx*4]

shl ecx,1

lea ecx,dword ptr[ecx+eax-30h]

cmp ecx,dword ptr[cmpbuffer+ebp*4-8] ;和前面保存的累计值比较，看有无出入，没有出入则保存结果

je @11

mov ebx,ecx                   ;如果有出入，下面进行调整

sub ebx,dword ptr[cmpbuffer+ebp*4-8]

sub eax,ebx

sub ecx,ebx

@11:

mov byte ptr[SerialBuffer+edi],al

inc edi

dec ebp

cmp ebp,0

jg @10

invoke SetDlgItemText,hDlg,IDC_CODE,addr SerialBuffer

popad

ret

GetKey endp

end start

;------------------------------------------------------
```

## 资源文件 (keygen.rc)

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;>>>>>   keygen.rc     >>>>>
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#define IDD_DLG1 1000

#define IDC_GRP1 1007

#define IDC_NAME 1001

#define IDC_CODE 1002

#define IDC_STC1 1003

#define IDC_STC2 1004

#define IDC_OK 1005

#define IDC_ABOUT 1006

IDD_DLG1 DIALOGEX 6,5,238,52

CAPTION "Keygen For Polyview Version4.251"

FONT 8,"MS Sans Serif"

STYLE 0x10CA0880

EXSTYLE 0x00000001

BEGIN
  CONTROL "",IDC_NAME,"Edit",0x50010000,64,9,104,15,0x00000200
  CONTROL "",IDC_CODE,"Edit",0x50010800,64,29,104,15,0x00000200
  CONTROL "Licensee:",IDC_STC1,"Static",0x50000201,8,11,34,9,0x00000000
  CONTROL "License Number:",IDC_STC2,"Static",0x50000000,8,31,53,9,0x00000000
  CONTROL "GENERATE",IDC_OK,"Button",0x50010000,180,9,48,15,0x00000000
  CONTROL "ABOUT",IDC_ABOUT,"Button",0x50010000,180,29,48,15,0x00000000
  CONTROL "",IDC_GRP1,"Button",0x50000007,4,0,170,48,0x00000000
END
```

## 其他信息

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
