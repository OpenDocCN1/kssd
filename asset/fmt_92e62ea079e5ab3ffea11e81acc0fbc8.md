
# 软件破解与注册机制作记录

## 加密狗检测逻辑分析

在 `00AA5B5C` 处 `retn` 返回至主程序。以下为关键判断代码段：

```
00AA5BD5    83F8 02           cmp     eax, 2
00AA5BD8    75 1E            jnz     short 00AA5BF8              ; 有加密狗//已注册
00AA5BDA    68 00000500      push    50000
00AA5BDF    68 405EAB00      push    00AB5E40                    ; ASCII "Engineering Tool"
00AA5BE4    8D4D EC          lea     ecx, [ebp-14]
00AA5BE7    FF15 A463AC00    call    [<MFC71.#876_ATL::CSimpleStrin>; MFC71.7C158BCD
00AA5BED    50               push    eax
00AA5BEE    6A 00            push    0                           ; 提示你已注册
00AA5BF0    FF15 E866AC00    call    [<USER32.MessageBoxA>]      ; USER32.MessageBoxA
00AA5BF6    EB 47            jmp     short 00AA5C3F

00AA5BF8    8D4D F0          lea     ecx, [ebp-10]
00AA5BFB    E8 55B4FDFF      call    CBravoMode::GetSubMode
00AA5C00    83F8 02          cmp     eax, 2
00AA5C03    75 1E            jnz     short 00AA5C23              ; 非专业版//已注册
00AA5C05    68 00000500      push    50000
00AA5C0A    68 545EAB00      push    00AB5E54                    ; ASCII "InPower ONAN"
00AA5C0F    8D4D EC          lea     ecx, [ebp-14]
00AA5C12    FF15 A463AC00    call    [<MFC71.#876_ATL::CSimpleStrin>; MFC71.7C158BCD
00AA5C18    50               push    eax
00AA5C19    6A 00            push    0                           ; 提示你已注册
00AA5C1B    FF15 E866AC00    call    [<USER32.MessageBoxA>]      ; USER32.MessageBoxA
00AA5C21    EB 1C            jmp     short 00AA5C3F

00AA5C23    68 00000500      push    50000
00AA5C28    68 645EAB00      push    00AB5E64                    ; ASCII "InPower"
00AA5C2D    8D4D EC          lea     ecx, [ebp-14]
00AA5C30    FF15 A463AC00    call    [<MFC71.#876_ATL::CSimpleStrin>; MFC71.7C158BCD
00AA5C36    50               push    eax
00AA5C37    6A 00            push    0                           ; 提示你已注册
00AA5C39    FF15 E866AC00    call    [<USER32.MessageBoxA>]      ; USER32.MessageBoxA
```

## 算法注册机制作思路

分析已完成，软件中存在多处检测机制。虽然看到了非专业版的注册码验证逻辑，但由于时间原因未深入分析完整算法。现尝试基于 PEDIY 技术制作算法注册机。

### 定位 MessageBox 调用地址

为了显示正确的注册信息，需要找到包含注册结果的 `MESSAGEBOX` 调用。通过 OD（OllyDbg）载入程序并运行，在 `MESSAGEBOX` 处下断点 `BP MESSAGEBOX`，触发弹窗后获取调用地址：

```
call [0AC66E8]
```

### 获取计算后的注册码

在注册 DLL 中定位关键调用，计算后的注册码存储在 `EDX` 寄存器指向的内存区域。相关代码如下：

```
00A890BE    68 642CAB00     push    00AB2C64                  ; ASCII "%X%X%X%X"
00A890C3    8D55 C8         lea     edx, [ebp-38]
00A890C6    52             push    edx
00A890C7    FF15 9C63AC00   call    [<MFC71.#2322_ATL::CStringT<char>]; 计算注册的 CALL，算法在此处
```

此时寄存器状态如下：
- `ECX`: `01055B08` (ASCII `"7C40603C308B45D8"`)
- `EDX`: `01055B08` (ASCII `"7C40603C308B45D8"`)

在 `00A890CD` 处 `add esp, 18` 指令后，可获取正确的注册码用于显示。

```
00A890CD    83C4 18         add     esp, 18                   ; 获取正确注册码
00A890D0    8B4D 0C         mov     ecx, [ebp+C]
00A890D3    FF15 A463AC00   call    [<MFC71.#876_ATL::CSimpleStringT<char>]; 读取密码
```

### 跳转与补丁代码

寻找空闲内存区域 `00AB1790`，在 `00A890CD` 处插入跳转指令 `JMP 00AB1790`。在该区域写入以下补丁代码以显示注册码：

```
pushad                 ; 保存寄存器堆栈
mov   eax, edx         ; 将 EDX 中的注册码传入 EAX
push  eax              ; 压栈
push  0                ; 压栈
call  [0AC66E8]        ; 调用 MessageBox 显示
popad                  ; 恢复堆栈
add   esp, 18          ; 还原 ESP
mov   ecx, [ebp+C]     ; 还原 ECX
jmp   00A890D3         ; 跳回原程序流程
```

### 资源修改

使用 `Resource Hacker` 打开 `BvLock.dll`，修改资源对话框 ID `2003` 的内容，加入自定义说明及注册界面信息：

```
2003 DIALOGEX 0, 0, 242, 115

STYLE DS_MODALFRAME | DS_SETFOREGROUND | WS_POPUP | WS_VISIBLE | WS_CAPTION

EXSTYLE WS_EX_TOPMOST

CAPTION "关于[qyc]软件破解-->流行时代  看雪学院  OCN"

LANGUAGE LANG_ENGLISH, SUBLANG_ENGLISH_US

FONT 8, "MS Sans Serif"

{
   CONTROL "&Register", 1, BUTTON, BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 62, 163, 0, 0
   CONTROL "&关闭", 2, BUTTON, BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 99, 95, 50, 14
   CONTROL "这是一个发电机调试软件的非专业版算法注册机.\n 它生成的 KEY 可以注册 2006 年发行 INPOWER 5.0 非专业版.\n 本注册机不用你手工输入机器码来计算注册码(一切都是为了你着想) BY 小 Q 2006 年 2 月 25 日.", 2012, STATIC, SS_CENTER | WS_CHILD | WS_VISIBLE | WS_GROUP, 6, 18, 230, 33
   CONTROL "", -1, BUTTON, BS_GROUPBOX | WS_CHILD | WS_VISIBLE, 3, 0, 236, 112
   CONTROL "", 2014, EDIT, ES_LEFT | ES_AUTOHSCROLL | WS_CHILD | WS_BORDER | WS_TABSTOP, 195, 131, 0, 0
   CONTROL "请问你把注册码记在纸上了吗?关闭注册进行注册吧!哈哈.", 2011, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 23, 81, 211, 12
   CONTROL "有空来来看雪学院:[www.pediy.com.]", 2013, STATIC, SS_CENTER | WS_CHILD | WS_VISIBLE | WS_GROUP, 43, 55, 157, 10
   CONTROL "", 2009, EDIT, ES_LEFT | ES_UPPERCASE | ES_AUTOHSCROLL | WS_CHILD | WS_VISIBLE | WS_BORDER | WS_TABSTOP, 138, 130, 0, 0
   CONTROL "", 2008, EDIT, ES_LEFT | ES_AUTOHSCROLL | ES_READONLY | WS_CHILD | WS_VISIBLE | WS_BORDER, 138, 106, 0, 0
   CONTROL "o○o 流行时代 o○o www.popbase.net", 2017, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 47, 68, 153, 12
   CONTROL "说明:", -1, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 110, 6, 31, 12
   CONTROL "", 2023, COMBOBOX, CBS_DROPDOWNLIST | CBS_SORT | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 118, 11, 0, 0
   CONTROL "请问你把注册码记在纸上了吗?关闭注册进行注册吧!哈哈.", -1, STATIC, SS_LEFT | WS_CHILD | WS_GROUP, 25, 116, 204, 10
}
```

## 结语

至此，无需原程序的完美算法注册机制作完成。本次分析耗时较长，感谢看雪版主提供的文件上传权限支持。

BY QYC 2006---2008//转贴请保持完整

`4dd466d0e1e0cc123afbe504da2462ca`
