
# 简单方法避开断点 (3千字)

简单方法避开断点

在日常破解当中，常用的方法很多，下断点是常用方法之首x，如果程式可避开某指定断点的话，可减少被破解的机会。

所槎系，就是在程式里放上 `INT 3` 代码 `0xCC`，但楹卧诘髟过程中我们看不见 `0xCC` 呢？原来调器在下断点r把对应的字元先保全下来，放上 `0xCC`，等中断r再原，这 `0xCC` 就看不见了。H上是这的？好，下面用 `MessageBoxA` 槔，表明上面所讲，

```
#include <windows.h>

#include <stdio.h>

#include <tchar.h>

typedef void (WINAPI \*SPMESSAGEBOXA) (HWND, LPCTSTR, LPCTSTR, UINT);

SPMESSAGEBOXA SPMessageBoxA;

void OtherMessageBox(HWND, LPCTSTR, LPCTSTR,

UINT);

\_TCHAR lpBuf[40];

\_TCHAR lpBufA[40];

int

WINAPI  WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance,

              LPSTR lpszCmdLine,

int nCmdShow )

 {

     int Icon = MB\_ICONINFORMATION;

     const PSTR szCaption = "This never break even break-point

was set ! ";

     const PSTR szText = lpBuf;

     const

PSTR szTargetAPI = "MessageBoxA";

     HMODULE hUser32 = GetModuleHandle("user32");

     SPMessageBoxA = (SPMESSAGEBOXA)GetProcAddress(hUser32,

szTargetAPI);

     wsprintf(lpBufA, "%s", "This can Break

!");

     sprintf(lpBuf, "Code byte was %8X", \*(PULONG)SPMessageBoxA);

     SPMessageBoxA(0, lpBuf,lpBufA, MB\_ICONINFORMATION);

    if (\*(PUCHAR)SPMessageBoxA == 0xcc)

  wsprintf(lpBuf, "%s", "Break point was set !");

     else

         wsprintf(lpBuf, "%s", "No Break point

set !");

     OtherMessageBox(0, szText, szCaption, Icon);

     return 1;

 }

void OtherMessageBox(HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaption, UINT uType

)

 {

     \_\_asm

  {

         inc SPMessageBoxA

        push dword ptr uType

  push dword ptr lpCaption

         push

dword ptr lpText

         push 0

      call    MsgJmpBox

  jmp    end

 MsgJmpBox:

  push    ebp

  jmp    SPMessageBoxA

 end:

     }

 }
```

这程式是先取到 `MessageBoxA` 在系统中的地址（用 `GetModuleHandle` 和 `GetProcAddress`）之後把地址加一，推相对数据入stack，然后跳到加了一的地址去，因调器把 `0xCC` 放在未加一的位置上，这就避过了断点，十分简单。

在未加断点r，当第一个 `MessageBox` 出r（正常的呼叫方法），它会表示 `MessageBox` 这涵数的头四byte `51EC8B55`，第二个 `MessgaeBox` 是经过淼，不会断在 `BPX MessageBoxA` 上。

当下了断点后，\行这程式，Softice会出，你会看到下面代码。

注意头四个byte是 `558BEC51`，

```
MessageBoxA:

           55

        PUSH    EBP

    8BEC            MOV

  EBP, ESP

           51

        PUSH    ECX

    833D9894E67700  CMP    dword ptr [offset32 .data:77E69498],

00000000

           0F85C4050100    JNE/JNZ

  L77E38CBF: 6A00

PUSH    00000000

           FF7514

        PUSH    dword ptr [EBP+00000014]

          FF7510          PUSH

  dword ptr [EBP+00000010]

           FF750C

        PUSH    dword ptr [EBP+0000000C]

          FF7508          PUSH

  dword ptr [EBP+00000008]

           E804000000

    CALL    MessageBoxExA

  C9              LEAVE

      C21000          RET    0010
```

跟着按F5，`MessageBox` 出，但表示 `51EC8BCC`，这就^明了上面的f法了。

再把第一个 `MessageBox` 关闭，第二个 `MessageBox` R上弹出，没有中断！！

在这例子中，最多可加上四个byte，但在跳之前也应把跳开的位元都堆到stack去。

如加四的话：

```
push ebp
mov  ebp, esp
push ecx
jmp    SPMessageBoxA
```

要破解这方法也是十分简单，o要下 `bpx messageboxa+4` 一切就 OK 了！！
