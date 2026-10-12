
看看有没有能用到的，或者翻一下我以往的帖子。里面应该有

代码:

```c
//保存 5 字节代码的结构
#pragma pack(1)
typedef struct _TOP5CODE
{
  UCHAR  instruction;  //指令
  ULONG  address;    //地址
}TOP5CODE,*PTOP5CODE;
#pragma pack( )

//ssdt 表结构
typedef struct _ServiceDescriptorTable {
  PVOID ServiceTableBase;    //System Service Dispatch Table 的基地址   
  PVOID ServiceCounterTable;
  //包含着 SSDT 中每个服务被调用次数的计数器。这个计数器一般由 sysenter 更新。 
  unsigned int NumberOfServices;//由 ServiceTableBase 描述的服务的数目。   
  PVOID ParamTableBase; //包含每个系统服务参数字节数表的基地址-系统服务参数表 
}*PServiceDescriptorTable;
//由 SSDT 索引号获取当前函数地址
//NtOpenProcess  [[KeServiceDescriptorTable]+0x7A*4]
extern PServiceDescriptorTable KeServiceDescriptorTable;

//////////////////////////////////////////////////////////////////////
//  名称:  MyGetFunAddress
//  功能:  获取函数地址
//  参数:  函数名称字符串指针
//  返回:  函数地址
//////////////////////////////////////////////////////////////////////
ULONG MyGetFunAddress( IN PCWSTR FunctionName)
{
  UNICODE_STRING UniCodeFunctionName;
  RtlInitUnicodeString( &UniCodeFunctionName, FunctionName );
  return (ULONG)MmGetSystemRoutineAddress( &UniCodeFunctionName );    
}

//////////////////////////////////////////////////////////////////////
//  名称:  myGetCurrentAddress
//  功能:  获取 SSDT 表中指定函数的当前地址
//  参数:  index:指定函数在表中的索引号
//  返回:  地址
//////////////////////////////////////////////////////////////////////
ULONG myGetCurrentAddress(IN ULONG index)
{
  ULONG    SSDT_Cur_Addr;
  __asm
  {
    push  ebx
    push  eax
    mov   ebx,KeServiceDescriptorTable
    mov   ebx,[ebx]
    mov   eax,index
    shl   eax,2
    add   ebx,eax
    mov   ebx,[ebx]
    mov   SSDT_Cur_Addr,ebx
    pop   eax
    pop   ebx
  }

  return  SSDT_Cur_Addr;
}

VOID WPOFF()
{
  __asm
  {
    cli
    mov eax,cr0
    and eax,not 10000h
    mov cr0,eax
  }
}

VOID WPON()
{
  __asm
  {
    mov eax,cr0
    or eax,10000h
    mov cr0,eax
    sti
  }
}
```

记在最后面的话：大家要善用搜索引擎(建议学习`google hacking`技巧)，勤做笔记，最后要说的依然是感谢，感谢`GOOGLE\BAIDU\PEDIY\DEBUGMAN`。还有那些默默发帖的人~

如果有时间的话，我会将其他几个游戏保护的分析资料也放出来

什么`GPK\HP\HS`的。大家不要催不要急，一定会放出来的。等到我觉得这些东西都没有挑战性的时候那么也就不会再有资料陆续放出来了……

如果有好东西记得与我分享![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

![](img/b56ec7ac43f57e87ff7d76f3e935a064.jpg)
