
# 天书夜读上 Inline Hook ZwCreateSection

> 原文：[`www.pediy.com/kssd/pediy11/125253.html`](https://www.pediy.com/kssd/pediy11/125253.html)

新手初来乍到，老鸟飞过，菜鸟共同学习……
源码见附件
部分源码及分析贴出……

```c
#include "xde.h"
#include <ntddk.h>
#include <WinDef.h>

unsigned long  g_uCr0;
unsigned long Addr_ZwCreateSection,Addr_ZwCreateSection_Body;

//模仿天书 InlineHook，填写绝对跳转指令
unsigned long my_address;    // 我要跳转的地址
unsigned long length;
unsigned char code[12];      //使用一个比较大的空间来容纳未知指令
struct xde_instr myjmp = {0};
static unsigned char * g_new_address =NULL;
static unsigned long total_length = 0;

//关闭内存写保护 WriteProtectOff
void WriteProtectOff()
{
  unsigned long uAttr;
  __asm
  {
    push eax;
    mov eax, cr0;
    mov uAttr, eax;
    and eax, 0FFFEFFFFh; // CR0 16 BIT = 0
    mov cr0, eax;
    pop eax;
    cli
  };
  g_uCr0 = uAttr; //保存原有的 CRO 属性
}

//开启内存写保护
VOID WriteProtectOn()
{
  __asm
  {
    sti
    push eax;
    mov eax, g_uCr0; //恢原有 CR0 性
    mov cr0, eax;
    pop eax;
  };
}

//由函数名 获得 函数地址
unsigned long GetFunctionAddr( IN PCWSTR FunctionName)
{
  UNICODE_STRING UniCodeFunctionName;
  RtlInitUnicodeString( &UniCodeFunctionName, FunctionName );
  return (unsigned long)MmGetSystemRoutineAddress( &UniCodeFunctionName );
}

//Hook 函数所作的工作
NTSTATUS  DetourZwCreateSection( OUT PHANDLE SectionHandle, IN ACCESS_MASK DesiredAccess, IN POBJECT_ATTRIBUTES ObjectAttributes OPTIONAL, IN PLARGE_INTEGER MaximumSize OPTIONAL, IN ULONG SectionPageProtection, IN ULONG AllocationAttributes, IN HANDLE FileHandle OPTIONAL  )
{
  KdPrint(("\n\nSectionHandle:%X\nFileHandle:%X\n\n",SectionHandle,FileHandle));
  return STATUS_SUCCESS;
}

//中继函数
static __declspec(naked) MyZwCreateSectionRelay( OUT PHANDLE SectionHandle, IN ACCESS_MASK DesiredAccess, IN POBJECT_ATTRIBUTES ObjectAttributes OPTIONAL, IN PLARGE_INTEGER MaximumSize OPTIONAL, IN ULONG SectionPageProtection, IN ULONG AllocationAttributes, IN HANDLE FileHandle OPTIONAL  )
{//__declspec(naked)表示不希望编译器帮我们将参数压栈，所以这里需要我们手动将参数压栈
  if (g_new_address == NULL)
  {
    __asm
    {
      //这里的操作是将参数压入栈空间
      push ebp
      mov ebp,esp
      push [ebp + 1ch]
      push [ebp + 18h]
      push [ebp + 14h]
      push [ebp + 10h]
      push [ebp + 0ch]
      push [ebp + 8h]
      push [ebp + 4h]
      //上面的代码为参数压栈操作
      //下面这两句的代码是将下面的函数内指令地址(old_codes)保存到全局变量(g_new_address)中，
      //以便我们向这里拷贝我们想要执行的指令，在这里是 ZwCreateSection 开头的 total_length 个字节的指令
      push old_codes
        pop g_new_address
        //下面的代码用于参数的弹出操作
      mov edx, [ebp + 4h]
      pop edx
      mov edx, [ebp + 8h]
      pop edx
      mov edx, [ebp + 0ch]
      pop edx
      mov edx, [ebp + 10h]
      pop edx
      mov edx, [ebp + 14h]
      pop edx
      mov edx, [ebp + 18h]
      pop edx
      mov edx, [ebp + 1ch]
      pop edx
      mov esp, ebp
      pop ebp
      ret 0x1c
    }
  }
  else
  {
    __asm
    {
      //这里的操作是将参数压入栈空间
      push ebp
      mov ebp,esp
      push [ebp + 1ch]
      push [ebp + 18h]
      push [ebp + 14h]
      push [ebp + 10h]
      push [ebp + 0ch]
      push [ebp + 8h]
      push [ebp + 4h]
      //上面的代码为参数压栈操作
    }
    //这里调用我们的挂钩函数，实现我们想要的操作
    DetourZwCreateSection(SectionHandle,DesiredAccess,ObjectAttributes,MaximumSize,SectionPageProtection,AllocationAttributes,FileHandle);
    __asm
    {
      mov edx, [ebp + 4h]
      pop edx
      mov edx, [ebp + 8h]
      pop edx
      mov edx, [ebp + 0ch]
      pop edx
      mov edx, [ebp + 10h]
      pop edx
      mov edx, [ebp + 14h]
      pop edx
      mov edx, [ebp + 18h]
      pop edx
      mov edx, [ebp + 1ch]
      pop edx
      mov esp, ebp
      pop ebp
      //上面这些是为了堆栈平衡
      old_codes:
      //在这里写入足够多的 nop，形成一片空间来容纳拷贝过来的旧代码
        nop         //1
        nop         //2
        nop         //3
        nop         //4
        nop         //5
        nop         //6
        nop         //7
        nop         //8
        nop         //9
        nop         //10
        nop         //11
        nop         //12
        nop         //13
        nop         //14
        nop         //15
        //跳转到 ZwCreateSection+total_length 的位置继续执行
        jmp Addr_ZwCreateSection_Body
    };
  }
}

//安装 HOOK 的函数
NTSTATUS HookZwCreateSection()
{
  KIRQL Irql;
  unsigned long length;
  struct xde_instr code_instr = { 0 } ;
  Addr_ZwCreateSection_Body = GetFunctionAddr(L"ZwCreateSection");  //获得 ZwCreateSection 的开始地址;
  Addr_ZwCreateSection =  Addr_ZwCreateSection_Body;
  while (total_length < 7)
  {
    //反汇编一条指令
    length = xde_disasm((unsigned char*)Addr_ZwCreateSection_Body,&code_instr);
    if (length == 0)        //如果有指令解析失败，就直接返回失败
    {
      return STATUS_FILE_INVALID;
    }
    total_length += length;  //计算已经反汇编的指令的总长度
    Addr_ZwCreateSection_Body += length;//解析地址移动
  }
  //开始 InlineHook
  //关闭内存写保护，在下面要有写操作
  WriteProtectOff();
  //提升 IRQL 中断级到 DISPATCH_LEVEL,防止线程切换
  Irql = KeRaiseIrqlToDpcLevel();
  if (g_new_address != NULL)
  {
    //保存要 Hook 的内核函数前 total_length 个字节的内容
    RtlCopyMemory(g_new_address,(unsigned char *)Addr_ZwCreateSection,total_length);
  }
  else
  {
    MyZwCreateSectionRelay(NULL,0,NULL,NULL,0,0,NULL);
    //保存要 Hook 的内核函数前 total_length 个字节的内容
    RtlCopyMemory(g_new_address,(unsigned char *)Addr_ZwCreateSection,total_length);
  }
  //向要 Hook 的内核函数的开头 total_length 个字节写入 JMP 指令，使其跳转到新定义的钩子函数中
  //RtlCopyMemory((unsigned char *)Addr_ZwCreateSection,JmpAddress,total_length);
  RtlCopyMemory((unsigned char *)Addr_ZwCreateSection,code,total_length);
  //恢复 IRQL 中断级到
  KeLowerIrql(Irql);
  //开启写保护
  WriteProtectOn();
  return STATUS_SUCCESS;
}

//卸载 Hook 函数
void UnHookZwCreateSection()
{
  //把开头五个字节再写会到原函数
  KIRQL Irql;
  //关闭写保护
  WriteProtectOff();
  //提升中断级
  Irql = KeRaiseIrqlToDpcLevel();
  //还原原来的五个字节
  RtlCopyMemory((unsigned char *)Addr_ZwCreateSection,g_new_address,total_length);
  //降低中断级
  KeLowerIrql(Irql);
  //打开写保护
  WriteProtectOn();
}

//卸载函数
VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
{
  DbgPrint("My Driver Unloaded!");
  UnHookZwCreateSection();
}

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject,PUNICODE_STRING uniRegistry)
{
  pDriverObject->DriverUnload = OnUnload;
  my_address = (unsigned long)MyZwCreateSectionRelay;    //我要跳转的地址,即中继函数的地址
  myjmp.opcode = 0xea;        //填写指令码 eah ，对应的汇编码为 jmp
  myjmp.addrsize = 4;          //地址长度为 4 个字节（32 位）
  myjmp.datasize = 2;          //选择子作为立即数，长度为 2 个字节
  myjmp.addr_l[0] = my_address;//填写我要跳转的地址
  myjmp.data_s[0] = 8;          //选择子，内核代码段选择子为 8
  length = xde_asm(code,&myjmp);//将绝对跳转指令写到 code 变量中
  HookZwCreateSection();
  return STATUS_SUCCESS;
}
```

环境 `vs2008` + `ddkwizard` + `VMware 7.1` + `WinXP sp3`

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `TianShuInlineHook0.1.rar` |

我也上一个，共同进步：

```c
#include <ntddk.h>
#include "InlineHook.h"

UCHAR* PsGetProcessImageFileName( IN PEPROCESS Process );

typedef NTSTATUS (*NtCreateSection)(
              OUT PHANDLE  SectionHandle ,
              IN ACCESS_MASK  DesiredAccess ,
              IN POBJECT_ATTRIBUTES  ObjectAttributes OPTIONAL,
              IN PLARGE_INTEGER  MaximumSize OPTIONAL,
```



```
IN ULONG  SectionPageProtection ,  
IN ULONG  AllocationAttributes ,  
IN HANDLE  FileHandle OPTIONAL  
);

NtCreateSection OldNtCreateSection;

int NtCreateSectionPatchCodeLen = 0;

#pragma LOCKEDCODE

PVOID NtCreateSectionRet;

int Hooked = FALSE;

#pragma LOCKEDCODE

__declspec(naked) NTSTATUS NtCreateSectionHookZone(,...)

{

  _asm

  {

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    _emit 0x90;

    jmp [NtCreateSectionRet];

  }

}

NTSTATUS DriverUnload(  

        IN PDRIVER_OBJECT  DriverObject  

        )

{

  if (Hooked == TRUE)

  {

    UnHookApi(L"ZwCreateSection",TRUE,0,(PVOID)NtCreateSectionHookZone,NtCreateSectionPatchCodeLen);

  }

  DbgPrint("驱动已经被卸载了\n");

  return STATUS_SUCCESS;

}

#pragma LOCKEDCODE

wchar_t* __stdcall FindSubStringW(const wchar_t *str,int nLength,const wchar_t *strSearch)

{

  wchar_t   *cp=(wchar_t *)str;    

  wchar_t   *s1, *s2;    

  if(!*strSearch)    

    return ((wchar_t *)str);    

  while(nLength && *cp  )    

  {    

    s1   =   cp;    

    s2   =   (wchar_t*)strSearch;    

    while(*s1 && *s2 && !(*s1-*s2))    

      s1++,   s2++;    

    if(!*s2)    

      return(cp);    

    cp++;

    nLength--;

  }    

  return(NULL);    

}

NTSTATUS

NewNtCreateSection(

         OUT PHANDLE  SectionHandle ,  

         IN ACCESS_MASK  DesiredAccess ,  

         IN POBJECT_ATTRIBUTES  ObjectAttributes OPTIONAL,  

         IN PLARGE_INTEGER  MaximumSize OPTIONAL,  

         IN ULONG  SectionPageProtection ,  

         IN ULONG  AllocationAttributes ,  

         IN HANDLE  FileHandle OPTIONAL  

        )

{

  NTSTATUS status;

  PFILE_OBJECT    FileObject;

  PEPROCESS Eprocess;

  char *ProcessName;

  BOOL bRetOK=TRUE;

  _try

  {

    if (NT_SUCCESS(ObReferenceObjectByHandle(FileHandle,0,NULL,KernelMode,&FileObject,NULL)))

    {

      if (FindSubStringW(FileObject->FileName.Buffer,FileObject->FileName.MaximumLength/2,L"\\InstDrv.exe"))

      {

        Eprocess = PsGetCurrentProcess();

        ProcessName=(char*)PsGetProcessImageFileName(Eprocess);

        DbgPrint("[fileobject] %wZ  [caller] %s\r\n",&FileObject->FileName,ProcessName);

        ObDereferenceObject(FileObject);

        bRetOK=FALSE;

        __leave;

      }

      ObDereferenceObject(FileObject);          //放弃对 FileObject 的引用

    }        

    OldNtCreateSection = (NtCreateSection)NtCreateSectionHookZone;

    status = OldNtCreateSection(SectionHandle ,

      DesiredAccess ,  

      ObjectAttributes,  

      MaximumSize,  

      SectionPageProtection ,  

      AllocationAttributes ,  

      FileHandle);

  }

  __except(EXCEPTION_EXECUTE_HANDLER)

  {

    DbgPrint("NewNtCreateSection Error \r\n");

  }

  if (bRetOK==FALSE)

  {

    return STATUS_ACCESS_DENIED;

  }

  return status;

}

NTSTATUS

DriverEntry(

      IN PDRIVER_OBJECT  DriverObject,  

      IN PUNICODE_STRING  RegistryPath  

      )

{

  int bRet;

  DbgPrint("驱动已经加载了\n");

  DriverObject->DriverUnload = DriverUnload;

  bRet = HookApi((DWORD)NewNtCreateSection,L"ZwCreateSection",TRUE,0,(PVOID)NtCreateSectionHookZone,&NtCreateSectionPatchCodeLen,&NtCreateSectionRet);

  if(!bRet)

  {

    DbgPrint("hook NtCreateSection failed\n");

  }else{

    Hooked = TRUE;

  }

  return STATUS_SUCCESS;

}
```

### 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) |
| Inlinehook_NtCreateSection.rar |
