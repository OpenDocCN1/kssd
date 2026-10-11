
# 简单 inline hook ObReferenceObjectByHandle 保护进程和屏蔽文件执行

> 原文：[`www.pediy.com/kssd/pediy10/65731.html`](https://www.pediy.com/kssd/pediy10/65731.html)

```c
// *****************************************************
//  Author:  Sysnap     
//  Link:    http://hi.baidu.com/sysnap  
//   
// *****************************************************
```

好久没弄 `inline hook` 了.最近看到还是很多人弄,而且蓝屏,就把我以前的 `inline hook` 的 code 改下,于是有此文,呵呵,给跟我一样的菜鸟科普用的,高手飘过

现在一般工具的进程保护可能是 `hook NtOpenProcess` 和 `NtTerminateProcess`...其实关于进程的话题很多,印象中 `Sudami` 同学对这方面面比较懂.下面的代码可以保护进程

`WsysCheck`-->failed

`IceSword`->failed

`Gmer`->failed

`RUK`->failed

`SnipeSword`->failde(不包括内存清 0)

其实这只是简单的进程保护,唯一的好处就是 `hook` 导出函数 `ObReferenceObjectByHandle`,稳定...而结束之的方法也有很多,比如内存清 0..`PspTerminateProcess`...`PspExitThread` 等等...其中很多很强大的办法很多 `anti-rootkit` 的工具没使用,我想可能就是为了稳定吧.的确

另外,你可以用这个 code 来屏蔽某些程序的运行...其实这个 code 的效果就是已经运行的保护其进程,没有运行的屏蔽其运行....代码写得很草/.对系统性能也有一定影响,自己根据需要再改下吧...

代码就是 `inline hook ObReferenceObjectByHandle` 使其跳到 `T_ObReferenceObjectByHandle` 中

```c
__declspec(naked)  T_ObReferenceObjectByHandle(
    IN HANDLE  Handle,
    IN ACCESS_MASK  DesiredAccess,
    IN POBJECT_TYPE  ObjectType  OPTIONAL,
    IN KPROCESSOR_MODE  AccessMode,
    OUT PVOID  *Object,
    OUT POBJECT_HANDLE_INFORMATION  HandleInformation  OPTIONAL
    )
{
  _asm
{
   mov     edi,edi------>>>执行被修改的前 5 个字节
   push    ebp
   mov     ebp,esp
   push   [ebp+0x1c]------>>>参数压栈
   push   [ebp+0x18]
   push   [ebp+0x14]
   push   [ebp+0x10]
   push   [ebp+0xc]
   push   [ebp+8]
   call   MyObReferenceObjectByHandle  -------->>>>调用我们的功能函数  
   cmp   eax,1    
   jz     end
   mov   eax,ObReferenceObjectByHandle -------->>>>正常运行原来的函数   
   add   eax,5          
   jmp   eax   
    
end:
   mov   [ebp+8],-1------>>>>如果不想让这个函数正常运行..无效句柄就可以
   mov   eax,ObReferenceObjectByHandle     
   add   eax,5          
   jmp   eax    
    
}
 
 对原来函数是否要让其正常运行的判断由 MyObReferenceObjectByHandle 完成
 
  int  MyObReferenceObjectByHandle(
    IN HANDLE  Handle,
    IN ACCESS_MASK  DesiredAccess,
    IN POBJECT_TYPE  ObjectType  OPTIONAL,
    IN KPROCESSOR_MODE  AccessMode,
    OUT PVOID  *Object,
    OUT POBJECT_HANDLE_INFORMATION  HandleInformation  OPTIONAL
    )
{
  PEPROCESS Process;
  KIRQL  oldIrql;
  int JmpOffSet;
  unsigned char Code[5]={0x8b,0xff,0x55,0x8b,0xec};
  unsigned char JmpCode[5] = { 0xe9, 0x00, 0x00, 0x00, 0x00 };
 
  if(*PsProcessType==ObjectType)//判断句柄所属对象类型是不是*PsProcessType
{
     oldIrql = KeRaiseIrqlToDpcLevel();
  __asm
  {
    CLI          
    
    MOV   eax, CR0     
    
    AND eax, NOT 10000H 
    
    MOV   CR0, eax

  }
 
  RtlCopyMemory ( ObReferenceObjectByHandle, Code, 5 );//恢复 inline hook 以便正确调用 ObReferenceObjectByHandle
 
///////////////////////////////////////////////////////////////////////////////////////////////////////////////   
 
  ObReferenceObjectByHandle(Handle,DesiredAccess,ObjectType,AccessMode,&Process,NULL);
 
  if (_stricmp((char*)((char*)Process+0x174), ProtectName) == 0 }//判断是不是我们要保护的进程
    {
      JmpOffSet= (char*)T_ObReferenceObjectByHandle - (char*)ObReferenceObjectByHandle - 5;
      
      RtlCopyMemory ( JmpCode+1, &JmpOffSet, 4 );
      
      RtlCopyMemory ( ObReferenceObjectByHandle, JmpCode, 5 );
     
  __asm
   {
      MOV   eax, CR0
     
      OR   eax, 10000H
     
      MOV   CR0, eax
     
      STI
     
   }
   
   KeLowerIrql(oldIrql);
   
   return 1;
    
    }
 
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////
 
  //否则再次 HOOK 这个函数
 
  JmpOffSet= (char*)T_ObReferenceObjectByHandle - (char*)ObReferenceObjectByHandle - 5;
 
  RtlCopyMemory ( JmpCode+1, &JmpOffSet, 4 );
 
  RtlCopyMemory ( ObReferenceObjectByHandle, JmpCode, 5 );
 
  __asm
  {
    MOV   eax, CR0
     
    OR   eax, 10000H
     
    MOV   CR0, eax
     
    STI
  }
 
 KeLowerIrql(oldIrql);
 
}
 
 return 0;
 
}
```

![](img/9dcae3b84e31b561ff8ba8d705cb515a.jpg)

![](img/3472adf402874b1c6e02bfa76173e233.jpg)

![](img/6863c741989fd9a4186b5942cd446694.jpg)

![](img/ebeed4e3aeb3f5f9a79a145c8b583165.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `sysnap.rar` |
|---|---|
