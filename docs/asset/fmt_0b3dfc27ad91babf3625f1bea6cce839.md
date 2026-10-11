
# MFC 程序消息映射与破解

> 原文：[`www.pediy.com/kssd/pediy11/113069.html`](https://www.pediy.com/kssd/pediy11/113069.html)

## 前言

最近在学习 MFC 程序，以前破解 MF 程序时感觉不好针对按钮下断点，所以特意熟悉与分析了 MFC 的消息映射机制。

注：这只是一个很随意的笔记，用来分析的程序只是一个 CM，所以有些观点不全面或不正确，望大家指出。

## 参考书籍

王艳平 <<Windows 程序设计>>

## 分析过程

```
00401633    .   8B7424 18          MOV ESI,DWORD PTR SS:[ESP+18]
00401637    >   6A 00              PUSH 0
00401639    .   68 34304000        PUSH Rith_Cra.00403034                ; congratulations!
0040163E    .   68 20304000        PUSH Rith_Cra.00403020                ; well done cracker!
00401643    .   8BCE              MOV ECX,ESI
00401645    .   E8 54020000       CALL <JMP.&MFC42.#4224_MessageBoxA@CWnd@@QAEHPBD0I@Z>
```

用提示字符串找到这个按钮的算法部分，也就是 MFC 程序的消息映射函数处；

一个按钮点击事件的过程如下：

```
CWinThread::PumpMessage -> CWnd::PretranslateMessage -> CWnd::WalkPreTranslateMessage -> CDialog::PreTranslateMessage -> CDialog::PreTranslateMessage -> CWnd::PreTranslateInput -> CWnd::IsDialogMessageA -> USER32 内核 -> AfxWndProcBase -> AfxWndProc -> AfxCallWndProc -> CWnd::WindowProc -> CWnd::OnWndMsg -> CWnd::OnCommand -> CDialog::OnCmdMsg -> CCmdTarget::OnCmdMsg -> _AfxDispatchCmdMsg -> CDialog::OnButton1()
```

由于我主要跟踪的是 MFC 的消息映射，所以该流程中 `USER32` 内核前的部分可以不用关心。

在刚才打开算法部分的开头下断，点击按钮，断下来，`ALT+K` 打开调用堆栈，在 `user32` 后面第一个 MFC 函数体上点显示过程，下断。

`F9` 运行断下来，看：

```
73DC8468      FF75 14              PUSH DWORD PTR SS:[EBP+14]
73DC846B      8365 FC 00           AND DWORD PTR SS:[EBP-4],0
73DC846F      FF75 10              PUSH DWORD PTR SS:[EBP+10]
73DC8472      FF75 0C              PUSH DWORD PTR SS:[EBP+C]
73DC8475      FF75 08              PUSH DWORD PTR SS:[EBP+8]
73DC8478      E8 A595F6FF          CALL MFC42.#1578_?AfxWndProc@@YGJPAUHWND__@@IIJ@Z
73DC847D      8B4D F0              MOV ECX,DWORD PTR SS:[EBP-10]
73DC8480      8B55 EC              MOV EDX,DWORD PTR SS:[EBP-14]
73DC8483      8951 04              MOV DWORD PTR DS:[ECX+4],EDX
73DC8486      8B4D F4              MOV ECX,DWORD PTR SS:[EBP-C]
73DC8489      64:890D 00000000     MOV DWORD PTR FS:[0],ECX
73DC8490      C9                   LEAVE
73DC8491      C2 1000              RETN 10
```

看到 `CALL MFC42.#1578_?AfxWndProc` 了吧，这个函数是 MFC 提供的通用消息处理函数，MFC 的消息是通过这个函数分发到每个窗口的 `CWnd` 对象，然后在 `CWnd` 对象定义的消息处理函数中处理各种消息。

然后在 `73DC8478 E8 A595F6FF CALL MFC42.#1578_?AfxWndProc@@YGJPAUHWND__@@IIJ@Z` 上下断，`F7` 单步进入：

```
73D31A36      8B75 08              MOV ESI,DWORD PTR SS:[EBP+8]
73D31A39      56                   PUSH ESI
73D31A3A      E8 C3F7FFFF         CALL MFC42.#2867_?FromHandlePermanent@CWnd@@SGPAV1@PAUHWND
73D31A3F      85C0                TEST EAX,EAX
73D31A41      74 17               JE SHORT MFC42.73D31A5A
73D31A43      3970 20             CMP DWORD PTR DS:[EAX+20],ESI
73D31A46      75 12               JNZ SHORT MFC42.73D31A5A
73D31A48      FF75 14             PUSH DWORD PTR SS:[EBP+14]
73D31A4B      FF75 10             PUSH DWORD PTR SS:[EBP+10]
73D31A4E      FF75 0C             PUSH DWORD PTR SS:[EBP+C]
73D31A51      56                   PUSH ESI
73D31A52      50                   PUSH EAX
73D31A53      E8 1C000000         CALL MFC42.#1109_?AfxCallWndProc@@YGJPAVCWnd@@PAUHWND__@@I
```

这里面有两个函数，`FromHandlePermanent@CWnd` 这个函数根据传入窗口的 `hWND` 找到其对应的 `CWnd` 对象，然后把 `(*pCwnd,hWnd,uMsg,wPram,lParam)` 传入到下个 `AfxCallWndProc` 函数中。

单步进入 `AfxCallWndProc` 函数中：

```
73D31A83      57                   PUSH EDI
73D31A84      8965 F0             MOV DWORD PTR SS:[EBP-10],ESP
73D31A87      68 DB88DC73        PUSH MFC42.#2202_?CreateObject@?$CThreadLocal@V_AFX_THREAD
73D31A8C      B9 0CE6E073        MOV ECX,MFC42.73E0E60C
73D31A91      E8 DF670900        CALL MFC42.#3030_?GetData@CThreadLocalObject@@QAEPAVCNoTra
73D31A96      8BD8                MOV EBX,EAX
73D31A98      8D43 34            LEA EAX,DWORD PTR DS:[EBX+34]
73D31A9B      8BF0                MOV ESI,EAX
73D31A9D      6A 07              PUSH 7
73D31A9F      59                   POP ECX
73D31AA0      8D7D C0            LEA EDI,DWORD PTR SS:[EBP-40]
73D31AA3      F3:A5              REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
73D31AA5      8B4D 0C            MOV ECX,DWORD PTR SS:[EBP+C]
73D31AA8      8B75 10            MOV ESI,DWORD PTR SS:[EBP+10]
73D31AAB      8B7D 08            MOV EDI,DWORD PTR SS:[EBP+8]
73D31AAE      8908                MOV DWORD PTR DS:[EAX],ECX
73D31AB0      8B45 14            MOV EAX,DWORD PTR SS:[EBP+14]
73D31AB3      8943 3C            MOV DWORD PTR DS:[EBX+3C],EAX
73D31AB6      8B45 18            MOV EAX,DWORD PTR SS:[EBP+18]
73D31AB9      8943 40            MOV DWORD PTR DS:[EBX+40],EAX
73D31ABC      33C0                XOR EAX,EAX
73D31ABE      83FE 02            CMP ESI,2
73D31AC1      895D EC            MOV DWORD PTR SS:[EBP-14],EBX
73D31AC4      8973 38            MOV DWORD PTR DS:[EBX+38],ESI
73D31AC7      8945 FC            MOV DWORD PTR SS:[EBP-4],EAX
73D31ACA      75 0E              JNZ SHORT MFC42.73D31ADA
73D31ACC      3947 38            CMP DWORD PTR DS:[EDI+38],EAX
73D31ACF      74 09              JE SHORT MFC42.73D31ADA
73D31AD1      8B4F 38            MOV ECX,DWORD PTR DS:[EDI+38]
73D31AD4      8B11                MOV EDX,DWORD PTR DS:[ECX]
73D31AD6      50                   PUSH EAX
73D31AD7      FF52 64            CALL DWORD PTR DS:[EDX+64]
73D31ADA      8365 0C 00         AND DWORD PTR SS:[EBP+C],0
73D31ADE      81FE 10010000      CMP ESI,110                                // ; 判断程序是否初始化
73D31AE4      75 0E              JNZ SHORT MFC42.73D31AF4
73D31AE6      8D45 0C            LEA EAX,DWORD PTR SS:[EBP+C]
73D31AE9      50                   PUSH EAX
73D31AEA      8D45 DC            LEA EAX,DWORD PTR SS:[EBP-24]
73D31AED      50                   PUSH EAX
73D31AEE      57                   PUSH EDI
73D31AEF      E8 51880100        CALL MFC42.73D4A345
73D31AF4      FF75 18            PUSH DWORD PTR SS:[EBP+18]
73D31AF7      8B07                MOV EAX,DWORD PTR DS:[EDI]
73D31AF9      FF75 14            PUSH DWORD PTR SS:[EBP+14]
73D31AFC      8BCF                MOV ECX,EDI
73D31AFE      56                   PUSH ESI
73D31AFF      FF90 A0000000      CALL DWORD PTR DS:[EAX+A0]
```

这里的 `GetData@CThreadLocalObject@@QAEPAVCNoTrackObject@@P6GPAV2@XZ@Z` 是为了得到一个 `AFX_MODULE_STATE` 结构，并且把当前的消息填充到 `AFX_MODULE_STATE` 结构中。

单步到 `73D31AFF FF90 A0000000 CALL DWORD PTR DS:[EAX+A0] ; <JMP.&MFC42.#6374_?WindowProc@CWnd@@MAEJIIJ@Z>`

这里，`F7` 进入：

```
73D31B8D      8BCE                MOV ECX,ESI
73D31B8F      FF75 0C             PUSH DWORD PTR SS:[EBP+C]
73D31B92      FF75 08             PUSH DWORD PTR SS:[EBP+8]
73D31B95      FF90 A4000000      CALL DWORD PTR DS:[EAX+A4]               ; <JMP.&MFC42.#5163_?OnWndMsg@CWnd@@MAEHIIJPAJ@Z>
73D31B9B      85C0                TEST EAX,EAX
73D31B9D      75 16               JNZ SHORT MFC42.73D31BB5
73D31B9F      FF75 10             PUSH DWORD PTR SS:[EBP+10]
73D31BA2      8B06                MOV EAX,DWORD PTR DS:[ESI]
73D31BA4      FF75 0C             PUSH DWORD PTR SS:[EBP+C]
73D31BA7      8BCE                MOV ECX,ESI
73D31BA9      FF75 08             PUSH DWORD PTR SS:[EBP+8]
73D31BAC      FF90 A8000000      CALL DWORD PTR DS:[EAX+A8]
73D31BB2      8945 FC             MOV DWORD PTR SS:[EBP-4],EAX
73D31BB5      8B45 FC             MOV EAX,DWORD PTR SS:[EBP-4]
```

这里面同样有两个函数调用：

`<JMP.&MFC42.#5163_?OnWndMsg@CWnd@@MAEHIIJPAJ@Z>`

这个函数是 MFC 处理消息的核心函数，在这个函数里面 MFC 确定是否对传入的消息进行处理：

如果 `OnWndMsg@CWnd` 没有处理这个消息，将由下面这个函数处理 `CALL DWORD PTR DS:[EAX+A8]`；

进入 `OnWndMsg@CWnd@@MAEHIIJPAJ@Z`。
