
```
//NtOpenProcess 用到的全局变量\[为了方便堆栈平衡的处理使用全局变量\]
PEPROCESS  processEPROCESS = NULL;  //保存访问者的 EPROCESS
ANSI\_STRING  p\_str1,p\_str2;      //保存进程名称
BYTE    \*ObOpenObjectByPointerAddress  = NULL; //ObOpenObjectByPointer 的地址
BYTE    \*p\_TpHookAddress = NULL;        //TP 的 HOOK 函数地址
BYTE    \*p\_ReturnAddress = NULL;        //返回到的地址
BYTE    \*p\_MyHookAddress = NULL;        //我们的 HOOK 函数在哪写入
#define DNF\_EXE  "DNF.exe"  //要检索的进程名
//////////////////////////////////////////////////////////////////////
//  名称:  Nakd\_NtOpenProcess
//  功能:  My\_RecoveryHook\_NtOpenProcess 的中继函数
//  参数:   
//  返回:   
//////////////////////////////////////////////////////////////////////
static NAKED VOID  Nakd\_NtOpenProcess()
{
  //获得调用者的 EPROCESS
  processEPROCESS = IoGetCurrentProcess();
  //将调用者的进程名保存到 str1 中
  RtlInitAnsiString(&p\_str1,(ULONG)processEPROCESS+0x174);
  //将我们要比对的进程名放入 str2
  RtlInitAnsiString(&p\_str2,DNF\_EXE);
  if (RtlCompareString(&p\_str1,&p\_str2,TRUE) == 0)
  {
    //说明是 DNF 进程访问了这里
    \_\_asm
    {
      push    dword ptr \[ebp-38h\]
      push    dword ptr \[ebp-24h\]
      push  p\_ReturnAddress
      mov    eax,p\_TpHookAddress
      jmp    eax
    }
  }
  else
  {
    \_\_asm
    {
      push    dword ptr \[ebp-38h\]
      push    dword ptr \[ebp-24h\]
      push  p\_ReturnAddress
      mov    eax,ObOpenObjectByPointerAddress
      jmp    eax
    }
  }
}
 
//////////////////////////////////////////////////////////////////////
//  名称:  My\_RecoveryHook\_NtOpenProcess
//  功能:  解除游戏保护对 NtOpenProcess 的 HOOK
//  参数:   
//  返回:  状态
//////////////////////////////////////////////////////////////////////
NTSTATUS My\_RecoveryHook\_NtOpenProcess()
{
  BYTE    \*NtOpenProcessAddress      = NULL;  //NtOpenProcess 的地址
  BYTE    \*p = NULL;      //临时
  TOP5CODE  \*top5code = NULL;  //保存 5 字节内容
  BYTE    JmpAddress\[6\] = {0xE9,0,0,0,0,0x90};
  KIRQL    Irql;
 
    //获取 NtOpenProcess 的地址
    NtOpenProcessAddress = (BYTE\*)MyGetFunAddress(L"NtOpenProcess");
    if (NtOpenProcessAddress == NULL)
    {
      KdPrint(("NtOpenProcess 地址获取失败\\n"));
      return  FAILED\_TO\_OBTAIN\_FUNCTION\_ADDRESSES;
    }
    //获取 ObOpenObjectByPointer 的地址
    ObOpenObjectByPointerAddress = (BYTE\*)MyGetFunAddress(L"ObOpenObjectByPointer");
    if (ObOpenObjectByPointerAddress == NULL)
    {
      KdPrint(("ObOpenObjectByPointer 地址获取失败\\n"));
      return  FAILED\_TO\_OBTAIN\_FUNCTION\_ADDRESSES;
    }
 
    //将 p 指向 NtOpenProcess 函数开始处
    p = NtOpenProcessAddress;
    //用一个无限循环来判断给定的特征码来确定被 HOOK 位置
    while (1)
    {
      if ((\*(p-7)    == 0x50) && 
        (\*(p-0xE)  == 0x56) &&
        (\*(p+0xd)  == 0x50) &&
        (\*(p+0x16)  == 0x3b) &&
        (\*(p+0x17)  == 0xce) &&
        (\*p      == 0xE8) &&
        (\*(p+5)    == 0x8b) &&
        (\*(p+6)    == 0xf8))
      {
        KdPrint(("%0X \\n",(ULONG)p));
        break;
      }
      //推动指针向前走
      p++;
    }
 
    //将 top5code 指向 p 的当前处
    //用以取出 call \[地址\] 这 5 字节里面的地址
    top5code = (TOP5CODE\*)p;
    p\_TpHookAddress = (BYTE\*)((ULONG)p+5+top5code->address);
 
    //找到我们写入自定义函数的地址
    p\_MyHookAddress = p-6;
    //保存调用 ObOpenObjectByPointer 函数以后的返回地址
    p\_ReturnAddress = p+5;
 
    //将一条 JMP Nakd\_NtOpenProcess 写入到数组中
    \*(ULONG \*)(JmpAddress+1)=(ULONG)Nakd\_NtOpenProcess - ((ULONG)p\_MyHookAddress+5);
 
    WPOFF();  //清除 CR0
    //提升 IRQL 中断级
    Irql=KeRaiseIrqlToDpcLevel();
    //写入
    RtlCopyMemory(p\_MyHookAddress,JmpAddress,6);
    //恢复 Irql
    KeLowerIrql(Irql);
    WPON();    //恢复 CR0
   
  return  STATUS\_SUCCESS;
}
```

![](img/1e2e6c1b48bf369c87082c470b3e5d46.jpg)

简而言之其原理就是，任何人调用了 `NtOpenProcess` 的时候会先进入 `Nakd_NtOpenProcess` 函数，我们判断。如果是游戏进程访问的话，就有可能是验证之类的，我们转到它自己的函数里面。让它保持与 ring3 层的通信。否则的话，嘿嘿……

接下来是第 3 步处理 `debugport` 清零的这块了。

我想绝大多数人关心的都是这里了，网络上能搜多到的办法几乎都失效了，有办法的人又不肯放出来，急眼了就自己想了个土办法，虽然不那么时尚，但是绝对的奏效。

由于代码凌乱不堪，简单说下其原理。我们定位内核模块 `TxxxSxxx.sys` 的首地址，然后根据特征码遍历整个模块找到我们需要的地方，然后干掉他们。

那么我们又如何能够通过人工的判断出来到底是哪里在作怪呢？利用 `syser` 或 `Start SoftICE` 对 `EPROCESS+BC` 处设置断点。就可以一层一层的追溯上去了。

到底如何用他们，我想大家自己多花点时间在看雪和 `GOOGLE` 或者 `BAIDU` 上面是不会吃亏的。

由于 `ZwQuerySystemInformation` 函数的使用非常繁琐。而且篇幅有限。所以我只给出关键代码，至于这个函数如何使用。大家可以自己在搜索引擎找“枚举内核模块”。

代码:
