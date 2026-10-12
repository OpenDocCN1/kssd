
# 辅助修复 Armadillo int3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6445.htm`](https://www.pediy.com/kssd/pediy06/pediy6445.htm)

一小段代码，写的比较烂，为了清晰没有优化。

`For Armadillo 3.6`

我这个其实就是一个黑盒判断函数，只要他是通过这样的函数原型：

| **代码:** |
| :--- |
| ```c<br>int CheckJumpable(int EFlags, int ecx)<br>    return 1;      // Jumpable<br>    return 0;      // otherwise<br>``` |

验证跳转情形就没问题。

使用方法是修改`***`行中 `4b2690h` 为你的程序中一个跳转情况判断的函数表的偏移地址，然后编译，用 `OD` 的 `Binary Copy & Binary Paste` 来贴到脱壳后的程序中的一个小空间里，运行获得结果，便于写代码修复 `int3`。

其实这个和那个什么 `for Armadillo` 的软件执行的功能一样，只是这个定制性更强，大家顺便看看是不是有错误。

`````text
;****************************************************************
| **代码:** |
| --- |
| ```asm<br>.386<br>.model flat,stdcall<br>option casemap : none<br><br>  CF  equ                   0001h<br>  PF  equ    0004h<br>  ZF  equ    0040h<br>  SF  equ    0080h<br>  OF  equ    0800h<br><br>.code<br><br>start proc near<br>  LOCAL  Func :DWORD      ; 跳转类型判断函数<br>  LOCAL  FunType[251] :DWORD  ; 保存跳转类型<br>  LOCAL  i :DWORD  ; 记数参数<br><br>  mov  [i], 0<br>  mov  esi, 4b2690h  ; 修改 4b2690h 为一个跳转函数表的地址  ***<br><br>GetType:<br>  lodsd                              ; 载入函数地址<br>  push  esi<br>  mov  [Func], eax       ; 保存<br>;<br>;****************************************************************<br><br>  ;JO  0F 80 dd  /  70 db      OF = 1<br>  ;JNO 0F 81 dd  /  71 db  OF = 0<br>  ;JB  0F 82 dd  /  72 db  CF = 1<br>  ;JNB 0F 83 dd  /  73 db   CF = 0<br>  ;JE  0F 84 dd  /  74 db  ZF = 1<br>  ;JNE 0F 85 dd  /  75 db  ZF = 0<br>  ;JBE 0F 86 dd  /  76 db  CF = 1 or ZF = 1<br>  ;JA  0F 87 dd  /  77 db  CF = 0 and ZF = 0<br>  ;JS  0F 88 dd  /  78 db  SF = 1<br>  ;JNS 0F 89 dd  /  79 db  SF = 0<br>  ;JPE 0F 8A dd  /  7A db  PF = 1<br>  ;JPO 0F 8B dd  /  7B db  PF = 0<br>  ;JL  0F 8C dd  /  7C db  SF <> OF<br>  ;JGE 0F 8D dd  /  7D db  SF = OF<br>  ;JLE 0F 8E dd  /  7E db  ZF = 1 or SF <> OF<br>  ;JG  0F 8F dd  /  7F db  ZF = 0 and SF = OF<br><br>  push 0  ; Ecx<br>  push 0              ; EFlags<br>  call  [Func]<br>  add  esp, 4     ; 恢复栈堆,不过保留 Ecx<br>  test  al, al<br>  je  _JOJBJEJBEJSJPEJLJLE<br><br>_JNOJNBJNEJAJNSJPOJGEJG:<br>  push  CF or PF or ZF or SF      ; EFlags<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JMPJNOJECXZ<br><br>_JNO_JNBJNEJAJNSJPOJGEJG:<br>  push  CF or ZF   <br>  call   [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JNOJNSJPOJGE<br><br>_JNBJNEJAJG:<br>  push  CF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JNEJG<br><br>_JAJNB:<br>  push  ZF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JNB<br>  jmp  _JA<br><br>_JNEJG:<br>  push  SF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JNE<br>  jmp  _JG<br><br>_JNOJNSJPOJGE:<br>  push  SF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JNOJPO<br><br>_JNSJGE:<br>  push  OF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JNS<br>  jmp  _JGE<br><br>_JNOJPO:<br>  push  OF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JPO<br>  jmp  _JNO<br><br>_JOJBJEJBEJSJPEJLJLE:<br>  push  CF or ZF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JBJEJBEJGE<br><br>_JOJSJPEJG:<br>  push  SF or PF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JSJPE<br><br>_JOJG:<br>  push  ZF or OF or SF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JO<br>  jmp  _JG<br><br>_JSJPE:<br>  push  SF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JS<br>  jmp  _JPE<br><br>_JBJEJBEJGE:   <br>  push  CF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JBJGE<br><br>_JEJBE:<br>  push  ZF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JE<br>  jmp  _JBE<br><br>_JBJGE:<br>  push  CF or SF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JB<br>  jmp  _JGE<br><br>_JMPJNOJECXZ:<br>  push  OF<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JMPJECXZ<br>  jmp  _JNO<br><br>_JMPJECXZ:<br>  add  esp, 4<br>  push  1<br>  push  0<br>  call  [Func]<br>  add  esp, 4<br>  test  al, al<br>  jne  _JMP<br>  jmp  _JECXZ<br><br>; 跳转类型返回值<br><br>_JECXZ:<br>  push  10h<br>  jmp  @F<br><br>_JMP:<br>  push  -1<br>  jmp  @F<br><br>_JO:<br>  push  0<br>  jmp  @F<br><br>_JNO:<br>  push  1<br>  jmp  @F<br><br>_JB:<br>  push  2<br>  jmp  @F<br><br>_JNB:<br>  push  3<br>  jmp  @F<br><br>_JE:<br>  push  4<br>  jmp  @F<br><br>_JNE:<br>  push  5<br>  jmp  @F<br><br>_JBE:<br>  push  6<br>  jmp  @F<br><br>_JA:<br>  push  7<br>  jmp  @F<br><br>_JS:<br>  push  8<br>  jmp  @F<br><br>_JNS:<br>  push  9<br>  jmp  @F<br><br>_JPE:<br>  push  0Ah<br>  jmp  @F<br><br>_JPO:<br>  push  0Bh<br>  jmp  @F<br><br>_JL:<br>  push  0Ch<br>  jmp  @F<br><br>_JGE:<br>  push  0Dh<br>  jmp  @F<br><br>_JLE:<br>  push  0Eh<br>  jmp  @F<br><br>_JG:   <br>  push  0Fh<br>  jmp  @F<br><br>@@:<br>  pop  eax<br>  add  esp, 4<br>  mov  ecx, [i]<br>  mov  [FunType+ecx*4], eax<br>  cmp  ecx, 0FBh<br>  jge  @F<br>  inc  ecx<br>  mov  [i], ecx<br>  pop  esi<br>  jmp  GetType<br><br>@@:<br>  ret<br><br>start endp<br><br>  end start<br>``` |

`4dd466d0e1e0cc123afbe504da2462ca`
