
# 驱动监控进程的创建

> 原文：[`www.pediy.com/kssd/pediy10/65772.html`](https://www.pediy.com/kssd/pediy10/65772.html)

这是我在[`www.codeproject.com`](http://www.codeproject.com).学习时看到的一个驱动程序,学习后对其整理的笔记

下面这个驱动程序的作用：监控准备运行的可执行文件。（由用户决定是不是让它运行）所以我们要做以下工作：

首先是修改（`NtCreateSection`）SSDT 索引号，（索引号从用户程序中得到）HOOK `NtCreateSection()` 这个函数，然后通过文件句柄获得文件名，判断它是不是可执行文件，检测其属性与判断用户是否允许它执行，如果允许就运行原来 `NtCreateSection` 这个函数，否则返回 `STATUS_ACCESS_DENIED`。

如果我们截获一个 `NtCreateSection()` 的请求，该请求要求映射可执行文件作为 `SEC_IMAGE` 属性，通过结合页保护属性，我们能够知道进程将要执行了，因此我们在这个时候作出决定：是否让其执行。如果不想让它执行，`EAX` 返回值为 `STATUS_ACCESS_DENIED`。

因为调用是从 `ntdll.dll` 以一条 `MOV EAX, ServiceIndex` 五字节指令开始的，第一个字节是 `MOV EAX`，后四字节是索引号，所以我们可以得到索引号（后四字节）然后将它进行修改成我们自己的函数索引。当然在修改之前要将原服务函数地址保存在全局变量中。

```c
if(loc->Parameters.DeviceIoControl.IoControlCode==1000)  
{  
    buff=(UCHAR*)Irp->AssociatedIrp.SystemBuffer;  
    // hook service dispatch table  
    memmove(&Index,buff,4);//所有的调用都是从 ntdll.dll 以一条五字节指令 MOV EAX, ServiceIndex 开始，四字节是索引号  
    a=4*Index+(ULONG)KeServiceDescriptorTable->ServiceTable;//从用户程序中得到索引号，a 指向服务函数地址  
    base=(ULONG)MmMapIoSpace(MmGetPhysicalAddress((void*)a),4,0);//将物理地址映射到非分页池，因此可以进行读写，减少读写服务表保护属性的麻烦  
    a=(ULONG)&Proxy;//a 指向 Proxy 函数的地址  
    _asm  
    {  
        mov eax,base  
        mov ebx,dword ptr[eax]  
        mov RealCallee,ebx//将原服务函数地址保存在全局变量中  
        mov ebx,a  
        mov dword ptr[eax],ebx//Proxy 函数地址写进服务函数表中  
    }  
    memmove(&a,&buff[4],4);  
    output=(char*)MmMapIoSpace(MmGetPhysicalAddress((void*)a),256,0);  
}  
```

下面是我们自己函数的实现：

```c
//这个函数决定是否 NtCreateSection() 被成功调用  
ULONG __stdcall check(PULONG arg)//获得指向服务参数指针  
{  
    HANDLE hand=0;PFILE_OBJECT file=0;  
    POBJECT_HANDLE_INFORMATION info=0;ULONG a;char*buff;  
    ANSI_STRING str; LARGE_INTEGER li;li.QuadPart=-10000;  
    if((arg[4]&0xf0)==0)return 1;//检测标志  
    if((arg[5]&0x01000000)==0)return 1;//检测属性  
    //通过文件句柄获得文件名  
    hand=(HANDLE)arg[6];//获得执行文件句柄  
    ObReferenceObjectByHandle(hand,0,0,KernelMode,&file,info);//&file 获得对象体指针  
    if(!file)return 1;  
    RtlUnicodeStringToAnsiString(&str,&file->FileName,1);  
    a=str.Length;buff=str.Buffer;  
    while(1)//通过循环判断是不是有". "标志  
    {  
        if(buff[a]=='.')  
        {a++;break;}  
        a--;  
    }  
    ObDereferenceObject(file);  
    if(_stricmp(&buff[a],"exe")){RtlFreeAnsiString(&str);return 1;}//判断是否为可执行文件  
    KeWaitForSingleObject(&event,Executive,KernelMode,0,0);//将当前线程置于等待状态知道信号态  
    strcpy(&output[8],buff);//将 string 复制进 buff  
    RtlFreeAnsiString(&str);  
    a=1;//用户的决定通过 a 的制来反映  
    memmove(&output[0],&a,4);  
    while(1)  
    {  
        KeDelayExecutionThread(KernelMode,0,&li);//在一个固定时间间隔内当前线程处于等待状态  
        memmove(&a,&output[0],4);  
        if(!a)break;  
    }  
    memmove(&a,&output[4],4);  
    KeSetEvent(&event,0,0);  
    return a;  
}  

//保存执行文件上下文，调用 check()函数  
_declspec(naked) Proxy()  
{  
    _asm{  
        pushfd  
        pushad  
        mov ebx,esp  
        add ebx,40  
        push ebx  
        call check  
        cmp eax,1//由 check()的返回值判断是否让可执行文件继续执行  
        jne block  
        popad  
        popfd  
        jmp RealCallee//通过，调用 ntcreatesection  
        block:popad  
        mov ebx, dword ptr[esp+8]  
        mov dword ptr[ebx],0  
        mov eax,0xC0000022L//不让其通过，返回 STATUS_ACCESS_DENIED  
        popfd  
        ret 32  
    }  
}  
```

到此，驱动程序的工作基本完成，所以在用户程序中用到一个线程来等待驱动的判断结果。

```c
char*name=(char*)&outputbuff[8];  
for(x=0;x<stringcount;x++)  
{  
    if(!stricmp(name,strings[x])){a=1;goto skip;}  
}  
strcpy(msgbuff, "Do you want to run ");  
strcat(msgbuff,&outputbuff[8]);  
if(IDYES==MessageBox(0, msgbuff,"WARNING", MB_YESNO|MB_ICONQUESTION|0x00200000L))  
{a=1; strings[stringcount]=_strdup(name);stringcount++;}  
else a=0;  
// write response to the buffer, and driver will get it  
skip:memmove(&outputbuff[4],&a,4);  
//让驱动继续  
a=0;  
memmove(&outputbuff[0],&a,4);  
```

MY BLOG: [`hi.baidu.com/hljleo`](http://hi.baidu.com/hljleo)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `protect.rar` [解压密码：pediy] |
