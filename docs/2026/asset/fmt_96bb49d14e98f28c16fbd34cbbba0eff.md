
# Crack之亲历手迹6---破解AuthorWare5的软件狗()

其实破的是AuthorWare5的`CheckDog`函数，并没有进入狗的领空（毕竟作狗的人反pj的功力高一些）

//-----------------------------------------------

1. 先移去一个`salesagent`的保护

2. 以下为`__CheckDog`函数的反汇编代码：

```
__CheckDog      proc near
  ; CODE XREF: sub_42AE90+16p
.text:0050D680
; sub_42AF80+15p ...
.text:0050D680                test
  byte ptr dword_618918, 10h
.text:0050D687                push
  ebx
.text:0050D688                push
  esi
.text:0050D689                push
  edi
.text:0050D68A                mov
  esi, ecx
.text:0050D68C                mov
  ebx, 1
.text:0050D691                jz
    short loc_50D6D7
.text:0050D693                call
  sub_50D600
.text:0050D698                movzx
eax, ax
.text:0050D69B                mov
  edi, [esi]
.text:0050D69D                mov
  ecx, eax
.text:0050D69F                shl
  eax, 3
.text:0050D6A2                sub
  eax, ecx
.text:0050D6A4                lea
  ecx, unk_620B48[eax*2]
.text:0050D6AB                push
  ecx
.text:0050D6AC                lea
  ebx, ds:0[eax*2]
.text:0050D6B3                mov
  ecx, esi
.text:0050D6B5                call
  dword ptr [edi+8] ; 读取狗的数据
.text:0050D6B8                mov
  cx, word_620B54[ebx]
.text:0050D6BF                sub
  cx, ax
.text:0050D6C2                cmp
  cx, 1
.text:0050D6C6                sbb
  ebx, ebx
.text:0050D6C8                neg
  ebx
.text:0050D6CA                test
  ebx, ebx
.text:0050D6CC                jnz
  short loc_50D6D7
.text:0050D6CE                mov
  eax, [esp+10h]
.text:0050D6D2                mov
  ecx, esi
.text:0050D6D4                push
  eax
.text:0050D6D5                call
  dword ptr [edi] ; 显示出错对话框
.text:0050D6D7
.text:0050D6D7 loc_50D6D7:
            ; CODE XREF: __CheckDog+11j
.text:0050D6D7
; __CheckDog+4Cj
.text:0050D6D7                mov
  eax, ebx        ; eax=1表示有狗,eax=0表示无狗
.text:0050D6D9                pop
  edi
.text:0050D6DA                pop
  esi
.text:0050D6DB                pop
  ebx
.text:0050D6DC                retn
  4
.text:0050D6DC __CheckDog      endp ; sp = -8
```

Email:y97523@hotmail.com
