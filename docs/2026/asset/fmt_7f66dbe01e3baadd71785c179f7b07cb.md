
### 代码段

```asm
mov     eax,ecx              ;将 cl 存到 eax 中

and     eax,0ffh             ;eax 与 0xff 做 与运算,结果存入 eax 中

imul    byte ptr [esi]       ;al  与 当前字符 ASII 相乘,结果存入eax中

add     edx,eax              ;叠加 上次 edx 与 当前 结果 ,保存到 edx 中

inc     esi                  ;指向下一个字符

jmp     \_again               ;循环继续
```

`\_finish:`                                     ;运算结束

```asm
mov     eax,0beae0h          ;将 0xbeae0 存入eax 中

sub     eax,edx              ; 用 0xbeae0 - edx (上面循环结果)

mov     edx,eax              ; 将最后结果保存到 edx 寄存器中

ret
```

`GetReg`          endp

end start                                                                            ; 代码段结束

### 资源文件

`reg.rc`                                                                               ; 资源文件

```rc
#include                <resource.h>
```

```rc
#define        ICO\_MAIN                0x1000

#define        DLG\_MAIN                1

#define IDC\_TEXTNAME        107

#define IDC\_TEXTCODE    108
```

```rc
ICO\_MAIN        ICON                "Main.ico"
```

```rc
DLG\_MAIN DIALOG 150, 160, 180, 134

STYLE DS\_MODALFRAME | WS\_POPUP | WS\_VISIBLE | WS\_CAPTION | WS\_SYSMENU | WS\_THICKFRAME

CAPTION "超级电话通3.05注册机"

FONT 9, "宋体"

{

GROUPBOX "输入框", -1, 27, 5, 125, 100, BS\_GROUPBOX

LTEXT "用户名", -1, 35, 20, 105, 10

EDITTEXT IDC\_TEXTNAME, 35, 35, 105, 12

LTEXT "注册码", -1, 35, 60, 105, 10

EDITTEXT IDC\_TEXTCODE, 35, 75, 105, 12

DEFPUSHBUTTON "显示注册码", IDOK, 100, 115, 50, 14

}
```

后记：由此可见注册机也并不是那么难的.

在这里也感谢各位的支持...![](https://bbs.kanxue.com/view/img/face/1.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
