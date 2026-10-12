
# vb 函数注释脚本

> 原文：[`www.pediy.com/kssd/pediy12/136862.html`](https://www.pediy.com/kssd/pediy12/136862.html)

可把类似 `CALL <xxx>` 地址注释上该函数的帮助信息，效果：

```
00482096   > \83A5 C8FDFFFF>AND DWORD PTR SS:[EBP-238],0
0048209D   >  FF75 C4       PUSH DWORD PTR SS:[EBP-3C]
004820A0   .  E8 4531F8FF   CALL <xxx.__vbaLenBstr>             ;  获得一个字符串的长度，注：VB 中一个汉字的长度也为 1
004820A5   .  8BC8          MOV ECX,EAX
004820A7   .  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]
```

上面的 `CALL <xxx.__vbaLenBstr>` 注释上该函数的帮助信息：获得一个字符串的长度，注：VB 中一个汉字的长度也为 1，自动注释一些常用的 `vb` 函数功能，帮助资源是从 `newlaos` 翻译的 `vb` 函数中获取的。

如果代码段较大的话，运行时间会比较长。

```
var var1
var var2
var var3
var var4
var var5
var var6
var var7
var var8
var var9
var var10
var var11
var var12
var var13
var var14
var var15
var var16
var var17
var var18
var var19
var var20
var var21
var var22
var var23
var var24
var var25
var var26
var var27
var var28
var var29
var var30
var var31
var var32
var var33
var var34
var var35

mov var1,"__vbaI2Str"
mov var2,"__vbaI4Str"
mov var3,"__vbar4Str"
mov var4,"__vbar8Str"
mov var5,"VarCyFromStr"
mov var6,"VarBstrFromI2"
mov var7,"__vbaStrCopy"
mov var8,"__vbaVarCopy"
mov var9,"__vbaVarMove"
mov var10,"__vbavaradd"
mov var11,"__vbavarsub"
mov var12,"__vbavarmul"
mov var13,"__vbavaridiv"
mov var14,"__vbavarxor"
mov var15,"__vbavarfornext"
mov var16,"__vbafreestr"
mov var17,"__vbafreeobj"
mov var18,"__vbastrvarval"
mov var19,"multibytetowidechar"
mov var20,"rtcMsgBox"
mov var21,"__vbavarcat"
mov var22,"__vbafreevar"
mov var23,"__vbaLenBstr"
mov var24,"rtcInputBox"
mov var25,"__vbaNew"
mov var26,"__vbaNew2"
mov var27,"rtcTrimBstr"
mov var28,"__vbastrcomp"
mov var29,"__vbastrcmp"
mov var30,"__vbavartsteq"
mov var31,"rtcMidCharVar"
mov var32,"rtcLeftCharVar"
mov var33,"rtcRightCharVar"
mov var34,"__vbaStrCat"
mov var35,"__vbaStrCmp"

var var11
var var21
var var31
var var41
var var51
var var61
var var71
var var81
var var91
var var101
var var111
var var121
var var131
var var141
var var151
var var161
var var171
var var181
var var191
var var201
var var211
var var221
var var231
var var241
var var251
var var261
var var271
var var281
var var291
var var301
var var311
var var321
var var331
var var341
var var351

mov var11,"将一个字符串转为 8 位的数值形式(范围在 0 至 255 之间) 或 2 个字节的数值形式(范围在 -32,768 到 32,767 之间)。"
mov var21,"将一个字符串转为长整型(4 个字节)的数值形式(范围从-2,147,483,648 到 2,147,483,647)"
mov var31,"将一个字符串转为单精度单精度浮点型(4 个字节)的数值形式"
mov var41,"将一个字符串转为双精度单精度浮点型(8 个字节)的数值形式"
mov var51,"(仅 VB6 库. 要调试，则在 WINICE.DAT 里必须有 OLEAUT32.DLL)字符串到变比型数据类型"
mov var61,"(仅 VB6 库. 要调试，则在 WINICE.DAT 里必须有 OLEAUT32.DLL)整型数据到字符串:"
mov var71,"将一个字符串拷贝到内存，类似于 Windows API HMEMCPY"
mov var81,"将一个变量值串拷贝到内存"
mov var91,"变量在内存中移动，或将一个变量值串拷贝到内存"
mov var101,"两个变量值相加"
mov var111,"第一个变量减去第二个变量"
mov var121,"两个变量值相乘"
mov var131,"第一个变量除以第二个变量，得到一个整数商"
mov var141,"两个变量值做异或运算"
mov var151,"这是 VB 程序里的循环结构， For... Next...                     (Loop)"
mov var161,"释放出字符串所占的内存，也就是把内存某个位置的字符串给抹掉"
mov var171,"释放出 VB 一个对象(一个窗口，一个对话框)所占的内存，也就是把内存某个位置的一个窗口，一个对话框抹掉"
mov var181,"从字符串特点位置上获取其值"
mov var191,"将数据转换为宽字符格式，VB 在处理数据之都要这样做，在 TRW2000 显示为 7.8.7.8.7.8.7.8"
mov var201,"调用一个消息框，类似于 WINDOWS 里的 messagebox/a/exa,此之前一定有个 PUSH 命令将要在消息框中显示的数据压入椎栈"
mov var211,"将两个变量值相连，如果是两个字符串，就连在一起"
mov var221,"释放出变量所占的内存，也就是把内存某个位置的变量给抹掉"
mov var231,"获得一个字符串的长度，注：VB 中一个汉字的长度也为 1"
mov var241,"显示一个 VB 标准的输入窗口，类似 window's API getwindowtext/a, GetDlgItemtext/a"
mov var251,"调用显示一个对话框，类似 Windows' API Dialogbox"
mov var261,"调用显示一个对话框，类似 Windows' API Dialogboxparam/a"
mov var271,"将字串左右两边的空格去掉"
mov var281,"比较两个字符串，类似于 Window's API lstrcmp"
mov var291,"比较两个字符串，类似于 Window's API lstrcmp"
mov var301,"比较两个变量值是否相等"
mov var311,"从字符串中取相应字符，VB 中的 MID 函数，用法 MID("字符串","开始的位置","取几个字符")"
mov var321,"从字符串左边取相应字符，VB 中的用法：left("字符串","从左边开始取几个字符")"
mov var331,"从字符串右边取相应字符，VB 中的用法：Right("字符串","从右边开始取几个字符")"
mov var341,"用字符串的操作，就是将两个字符串合起来，在 VB 中只有一个&或+"
mov var351,"字符串比较，在 VB 中只有一个=或<>"

    var cont_01
    var cont_02
    var cont_03
    var imagen_base
    var imagen_CODEBASE
    var tempaddr
    var endaddr
    var test
    gmi eip,CODEBASE
    mov tempaddr,$RESULT
    gmi eip,CODESIZE
    add tempaddr,$RESULT
    mov endaddr,tempaddr
    mov tempaddr,eip
    mov test,"asdfasdf"

ccc:

    find tempaddr,#E8??????FF#
    mov cont_02,$RESULT
    mov cont_03,cont_02
    add  $RESULT,1
    mov cont_01,[$RESULT]
    add cont_02,cont_01
    add cont_02,5
    mov tempaddr,cont_03
    gn cont_02

cmp $RESULT_2,var1 
  jne   ddd1 
    CMT cont_03, var11    
   jmp finish    
   ddd1:
cmp $RESULT_2,var2 
  jne   ddd2 
    CMT cont_03, var21    
   jmp finish    
   ddd2:
cmp $RESULT_2,var3 
  jne   ddd3 
    CMT cont_03, var31    
   jmp finish    
   ddd3:
cmp $RESULT_2,var4 
  jne   ddd4 
    CMT cont_03, var41    
   jmp finish    
   ddd4:
cmp $RESULT_2,var5 
  jne   ddd5 
    CMT cont_03, var51    
   jmp finish    
   ddd5:
cmp $RESULT_2,var6 
  jne   ddd6 
    CMT cont_03, var61    
   jmp finish    
   ddd6:
cmp $RESULT_2,var7 
  jne   ddd7 
    CMT cont_03, var71    
   jmp finish    
   ddd7:
cmp $RESULT_2,var8 
  jne   ddd8 
    CMT cont_03, var81    
   jmp finish    
   ddd8:
cmp $RESULT_2,var9 
  jne   ddd9 
    CMT cont_03, var91    
   jmp finish    
   ddd9:
cmp $RESULT_2,var10 
  jne   ddd10 
    CMT cont_03, var101    
   jmp finish    
   ddd10:
cmp $RESULT_2,var11 
  jne   ddd11 
    CMT cont_03, var111    
   jmp finish    
   ddd11:
cmp $RESULT_2,var12 
  jne   ddd12 
    CMT cont_03, var121    
   jmp finish    
   ddd12:
cmp $RESULT_2,var13 
  jne   ddd13 
    CMT cont_03, var131    
   jmp finish    
   ddd13:
cmp $RESULT_2,var14 
  jne   ddd14 
    CMT cont_03, var141    
   jmp finish    
   ddd14:
cmp $RESULT_2,var15 
  jne   ddd15 
    CMT cont_03, var151    
   jmp finish    
   ddd15:
cmp $RESULT_2,var16 
  jne   ddd16 
    CMT cont_03, var161    
   jmp finish    
   ddd16:
cmp $RESULT_2,var17 
  jne   ddd17 
    CMT cont_03, var171    
   jmp finish    
   ddd17:
cmp $RESULT_2,var18 
  jne   ddd18 
    CMT cont_03, var181    
   jmp finish    
   ddd18:
cmp $RESULT_2,var19 
  jne   ddd19 
    CMT cont_03, var191    
   jmp finish    
   ddd19:
cmp $RESULT_2,var20 
  jne   ddd20 
    CMT cont_03, var201    
   jmp finish    
   ddd20:
cmp $RESULT_2,var21 
  jne   ddd21 
    CMT cont_03, var211    
   jmp finish    
   ddd21:
cmp $RESULT_2,var22 
  jne   ddd22 
    CMT cont_03, var221    
   jmp finish    
   ddd22:
cmp $RESULT_2,var23 
  jne   ddd23 
    CMT cont_03, var231    
   jmp finish    
   ddd23:
cmp $RESULT_2,var24 
  jne   ddd24 
    CMT cont_03, var241    
   jmp finish    
   ddd24:
cmp $RESULT_2,var25 
  jne   ddd25 
    CMT cont_03, var251    
   jmp finish    
   ddd25:
cmp $RESULT_2,var26 
  jne   ddd26 
    CMT cont_03, var261    
   jmp finish    
   ddd26:
cmp $RESULT_2,var27 
  jne   ddd27 
    CMT cont_03, var271    
   jmp finish    
   ddd27:
cmp $RESULT_2,var28 
  jne   ddd28 
    CMT cont_03, var281    
   jmp finish    
   ddd28:
cmp $RESULT_2,var29 
  jne   ddd29 
    CMT cont_03, var291    
   jmp finish    
   ddd29:



```
CMT cont_03, var291
jmp finish

ddd29:
cmp $RESULT_2,var30
jne ddd30

CMT cont_03, var301
jmp finish

ddd30:
cmp $RESULT_2,var31
jne ddd31

CMT cont_03, var311
jmp finish

ddd31:
cmp $RESULT_2,var32
jne ddd32

CMT cont_03, var321
jmp finish

ddd32:
cmp $RESULT_2,var33
jne ddd33

CMT cont_03, var331
jmp finish

ddd33:
cmp $RESULT_2,var34
jne ddd34

CMT cont_03, var341
jmp finish

ddd34:
cmp $RESULT_2,var35
jne ddd35

CMT cont_03, var351
jmp finish

finish:
cmp cont_03,0
Je bbb

add tempaddr,5
jmp ccc

error:
msg "错误"

bbb:
msg "注释完成"
```
