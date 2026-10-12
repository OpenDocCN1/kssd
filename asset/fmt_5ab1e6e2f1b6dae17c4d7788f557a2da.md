
```
:004011C9 68A8614000       push 004061A8
:004011CE 56              push esi
:004011CF E8FC070000       call 004019D0
:004011D4 59              pop ecx
:004011D5 85C0            test eax, eax
:004011D7 59              pop ecx
:004011D8 7511            jne 004011EB
:004011DA C605609740008B    mov byte ptr [00409760], 8B
:004011E1 C7054CA64000F3909090 mov dword ptr [0040A64C], 909090F3
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004011D8(C)`

* Possible StringData Ref from Data Obj -> `"ECX"`

```
:004011EB 68A4614000       push 004061A4
:004011F0 56              push esi
:004011F1 E8DA070000       call 004019D0
:004011F6 59              pop ecx
:004011F7 85C0            test eax, eax
:004011F9 59              pop ecx
:004011FA 7511            jne 0040120D
:004011FC C605609740008B    mov byte ptr [00409760], 8B
:00401203 C7054CA64000F1909090 mov dword ptr [0040A64C], 909090F1
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004011FA(C)`

* Possible StringData Ref from Data Obj -> `"EDX"`

```
:0040120D 68A0614000       push 004061A0
:00401212 56              push esi
:00401213 E8B8070000       call 004019D0
:00401218 59              pop ecx
:00401219 85C0            test eax, eax
:0040121B 59              pop ecx
:0040121C 7511            jne 0040122F
:0040121E C605609740008B    mov byte ptr [00409760], 8B
:00401225 C7054CA64000F2909090 mov dword ptr [0040A64C], 909090F2
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040121C(C)`

* Possible StringData Ref from Data Obj -> `"EDI"`

```
:0040122F 689C614000       push 0040619C
:00401234 56              push esi
:00401235 E896070000       call 004019D0
:0040123A 59              pop ecx
:0040123B 85C0            test eax, eax
:0040123D 59              pop ecx
:0040123E 7511            jne 00401251
:00401240 C605609740008B    mov byte ptr [00409760], 8B
:00401247 C7054CA64000F7909090 mov dword ptr [0040A64C], 909090F7
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040123E(C)`

* Possible StringData Ref from Data Obj -> `"ESI"`

```
:00401251 6898614000       push 00406198
:00401256 56              push esi
:00401257 E874070000       call 004019D0
:0040125C 59              pop ecx
:0040125D 85C0            test eax, eax
:0040125F 59              pop ecx
:00401260 7511            jne 00401273
:00401262 C6056097400090    mov byte ptr [00409760], 90
:00401269 C7054CA6400090909090 mov dword ptr [0040A64C], 90909090
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00401260(C)`

* Possible StringData Ref from Data Obj -> `"EBP"`

```
:00401273 6894614000       push 00406194
:00401278 56              push esi
:00401279 E852070000       call 004019D0
:0040127E 59              pop ecx
:0040127F 85C0            test eax, eax
:00401281 59              pop ecx
:00401282 7511            jne 00401295
:00401284 C605609740008B    mov byte ptr [00409760], 8B
:0040128B C7054CA64000F5909090 mov dword ptr [0040A64C], 909090F5
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00401282(C)`

* Possible StringData Ref from Data Obj -> `"EBP"`

＝＝＝＝以上程序是对处理 `Save_Code_Address` 不同值，做相应的处理＝＝＝＝＝＝

相当于

```
switch( Save_Code_Address )
{
    case 'EAX':
        code1=8B;
        code2=909090F0；
    case 'EBX':
        code1=8B;
        code2=909090F0；
    case 'EC':
        code1=8B;
        code2=909090F0；
    case 'EDX':
        code1=8B;
        code2=909090F0；
    case 'EDI':
        code1=8B;
        code2=909090F0；
    case 'ESI':
        code1=90;
        code2=909090F0；
    case 'EBP':
        code1=8B;
        code2=909090F0；
}
```

小结： 希望大家能帮我解答。为什么不同的寄存器会变成这样不同的数字？

它到底是代表什么？？

今天就到这里，就到这里。。。。。。（待续）豆豆虾

2001.8.11 17：17 完成

历时：1.8 个小时。

Go on..... 各位看官： （上一讲中提到的问题，不知道谁知道？能否赐教！谢了！）

```
:00401295 833D4CA6400000      cmp dword ptr [0040A64C], 00000000 －》40A64C 的值看上一讲
:0040129C 7513              jne 004012B1 －－》比较是否设定需要读取的寄存器；
                            如果设定了值，转向 4012B1.否则继续。
:0040129E 56              push esi
:0040129F C60560974000BE    mov byte ptr [00409760], BE
:004012A6 E8E3050000       call 0040188E
:004012AB 59              pop ecx
:004012AC A34CA64000       mov dword ptr [0040A64C], eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040129C(C)`

```
:004012B1 6824A54000       push 0040A524  －－》指向被跟踪的程序名
:004012B6 E813000000       call 004012CE  －－》创建该程序进程
:004012BB 59              pop ecx
:004012BC 5F              pop edi
:004012BD 5E              pop esi
:004012BE 5D              pop ebp
:004012BF 85C0            test eax, eax
:004012C1 5B              pop ebx
:004012C2 7405            je 004012C9
:004012C4 E841000000       call 0040130A
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004012C2(C)`

```
:004012C9 33C0            xor eax, eax
:004012CB C21000          ret 0010
```

下面看 `call 004012CE` 子程序，前我们来研究一个 API〔`CreatProcess`〕它的参数说明

可是一大摞。

```c
BOOL CreateProcess(
    LPCTSTR lpApplicationName,                 // pointer to name of executable module
    LPTSTR lpCommandLine,                      // pointer to command line string
    LPSECURITY_ATTRIBUTES lpProcessAttributes, // process security attributes
    LPSECURITY_ATTRIBUTES lpThreadAttributes,  // thread security attributes
    BOOL bInheritHandles,                      // handle inheritance flag
    DWORD dwCreationFlags,                     // creation flags
    LPVOID lpEnvironment,                      // pointer to new environment block
    LPCTSTR lpCurrentDirectory,                // pointer to current directory name
    LPSTARTUPINFO lpStartupInfo,               // pointer to STARTUPINFO
    LPPROCESS_INFORMATION lpProcessInformation // pointer to PROCESS_INFORMATION
);
```

说明

创建一个新进程

返回值

`Long`，非零表示成功，零表示失败。会设置 `GetLastError`

参数表

| 参数 | 类型及说明 |
| :--- | :--- |
| `lpApplicationName` | `String`，要执行的应用程序的名字。可设为 `vbNullString`；在这种情况下，应用程序的名字应在 `lpCommandLine` 参数的起始处出现 |
| `lpCommandLine` | `String`，要执行的命令行。可用 `GetCommandLine` 函数取得一个进程使用的命令行。windows 会尽可能地根据下述搜索顺序来查找执行文件：<br>（1）包含了父进程执行文件的目录<br>（2）父进程当前的目录<br>（3）由 `GetSystemDirectory` 返回的系统目录<br>（4）仅适于 windows NT：16 位系统目录<br>（5）由 `GetSystemDirectory` 返回的 windows 目录<br>（6）由 `PATH` 环境变量指定的目录 |
| `lpProcessAttributes` | `SECURITY_ATTRIBUTES`，指定一个 `SECURITY_ATTRIBUTES` 结构，或传递零值表示采用不允许继承的默认描述符。该参数定义进程的安全特性 |
| `lpThreadAttributes` | `SECURITY_ATTRIBUTES`，指定一个 `SECURITY_ATTRIBUTES` 结构，或传递零值表示采用不允许继承的默认描述符。该参数定义进程之主进程的安全特性 |
| `bInheritHandles` | `Long`，`TRUE` 表示允许当前进程中的所有句柄都由新建的子进程继承 |
| `dwCreationFlags` | `Long`，来自 `API32.TXT` 文件的一个或多个下述常数之一，它们都带有前缀 `CREATE_`。下面这些用于 VB 程序员：<br>`CREATE_SEPARATE_WOW_VDM`（仅适用于 NT）：启动一个 16 位的 windows 应用程序时，强迫它在自己的内存空间运行<br>`CREATE_SHARED_WOW_VDM`（仅适用于 NT）：启动一个 16 位的 windows 应用程序时，强迫它在共享的 16 位虚拟机内运行<br>`CREATE_SUSPENDED`：立即挂起新进程。除非调用 `ResumeThread` 函数。<br>`IDLE_PRIORITY_CLASS`：新进程应该有非常低的优先级，只有在系统空闲的时候才能运行。基本值是 4<br>`HIGH_PRIORITY_CLASS`：新进程应该有非常高的优先级，基本值是 13 |
