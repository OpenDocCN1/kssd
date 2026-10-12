
# CloneHideAdmin 开源

> 原文：[`www.pediy.com/kssd/pediy10/95649.html`](https://www.pediy.com/kssd/pediy10/95649.html)

`CloneHideAdmin` 开源

以前写的一个建立隐藏管理员的命令行工具，类属黑客工具吧，在这里发最主要是想和大家一起共同探讨汇编中命令行参数的处理方法，如果违规，管理员删帖便罢了。

大家看我的处理方法：

```asm
;***************************************************************  程序从这里开始执行   **********************************************************************

start:

  ;  invoke  StdOut,addr szHello   ;include masm32.lib and masm32.inc

    invoke  GetStdHandle,STD_OUTPUT_HANDLE
    mov     hStdOut,eax
    invoke  SetConsoleTitle,addr szTitle
    invoke  _argc
    .if     eax < 2
      jmp   ErrorShow
    .endif

    invoke  RtlZeroMemory,addr BuffTmp,sizeof BuffTmp
    invoke  _argv,1,addr BuffTmp,128
    lea     edi,BuffTmp
    .if     byte ptr [edi] == '-'
      inc   edi
      xor   ecx,ecx
      mov   cl,byte ptr [edi]
      .if   (cl == '?') || (cl == 'h') || (cl == 'H')
        invoke  ShowUsage
      .elseif (cl == 'l') || (cl == 'L')
        invoke  _argc
        .if     eax != 2    ;列帐户只要 2 个参数，加上程序名
          invoke  StdOut,CTXT("     列帐户格式不对。再来！",13,10)
          jmp     ErrorShow
        .endif
        invoke  ClearScreen
        invoke  RtlZeroMemory,addr BuffEnumSID,sizeof BuffEnumSID  ;先清空
        invoke  SetRegSecurity
        invoke  ListUserSID,CTXT("SAM\\SAM\\Domains\\Account\\Users\\Names"),addr BuffEnumSID
        invoke  StdOut,CTXT("当前机器上帐户对应 SID:",13,10,13,10)
        invoke  StdOut,addr BuffEnumSID    ;显示出来
        invoke  StdOut,CTXT(13,10,"请注意：克隆时只要输入最后 3 位",13,10)
      .elseif (cl == 'c') || (cl == 'C')
        invoke  _argc    ;克隆的话要 7 的参数，加上程序名
;        PrintDec eax
        .if     eax < 7
          PrintDec eax,"参数太少"
          invoke  StdOut,CTXT("     克隆帐户格式不对。再来！",13,10)
          jmp     ErrorShow
        .endif

        invoke  RtlZeroMemory,addr BuffSID,sizeof BuffSID
        invoke  _argv,2,addr BuffSID,sizeof BuffSID    ;取 SID 了
        invoke  lstrlen,addr BuffSID
        .if     eax > 3   
          invoke  StdOut,CTXT("        SID 只要最后三位。再来！",13,10)
          jmp     ErrorShow
        .endif

        PrintStringByAddr offset BuffSID
        invoke  RtlZeroMemory,addr BuffTmp,sizeof BuffTmp
        invoke  _argv,3,addr BuffTmp,128
        lea     edi,BuffTmp
        .if     byte ptr [edi] == '-'
          inc   edi
          mov   cl,byte ptr [edi]
          .if   (cl == 'u') || (cl == 'U')  
            invoke  RtlZeroMemory,addr BuffUser,sizeof BuffUser
            invoke  _argv,4,addr BuffUser,sizeof BuffUser  ;取用户名了
            PrintStringByAddr offset BuffUser
            invoke  RtlZeroMemory,addr BuffTmp,sizeof BuffTmp
            invoke  _argv,5,addr BuffTmp,sizeof BuffTmp
            lea     edi,BuffTmp
            .if     byte ptr [edi] == '-'
              inc   edi
              xor   ecx,ecx
              mov   cl,byte ptr [edi]
              .if   (cl == 'p') || (cl == 'P')
                invoke  RtlZeroMemory,addr BuffPass,sizeof BuffPass
                invoke  _argv,6,addr BuffPass,sizeof BuffPass  ;取密码了
                PrintStringByAddr offset BuffPass
                invoke  SetRegSecurity
                invoke  CreateHideAdmin,addr BuffUser,addr BuffPass,addr BuffSID
                invoke  StdOut,CTXT("恭喜您！！克隆帐户成功  ")
                invoke  wsprintf,addr BuffTmp,CTXT("帐户名： %s, 密码： %s"),addr BuffUser,addr BuffPass
                invoke  StdOut,addr BuffTmp
              .else
                invoke  StdOut,CTXT("      输入密码格式不对。再来！",13,10)
                jmp     ErrorShow
              .endif
            .else
              invoke  StdOut,CTXT("     输入用户名格式不对。再来！",13,10)
              jmp     ErrorShow
            .endif
          .else
            jmp   ErrorShow
          .endif
        .else          
          jmp   ErrorShow
        .endif
      .endif
    .else
ErrorShow:
      invoke  ClearScreen
      invoke  ShowUsage       
    .endif

    invoke  ExitProcess,NULL
end start
```

完整的工程大家下载了看吧！

注意：程序在 `VISTA` 运行有问题。

![](img/d91f146ed0ba673e93662243a7e72966.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `CloneHideAdmin.rar`  |
