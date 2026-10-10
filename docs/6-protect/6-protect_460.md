

# 一种另类的对抗 debugport 清零的方法(有点标题党了= 。=)

> 原文：[`www.pediy.com/kssd/pediy12/129429.html`](https://www.pediy.com/kssd/pediy12/129429.html)

  
前几天晚上看 intel 手册突然想到了一种另类的对抗 debug port 的方法，理论上蛮通用的，于是写了几百行代码实验了下  
但是结果很悲催，IDT HOOK 死活蓝屏，群里几个人帮我查那段 IDT HOOK 的代码的错误，结果一致认为代码没错，说我 RP 不行  
哎……好吧，我认了，我很久没扶老奶奶过马路了  
  
我简单说下思路，如果你能搞定，请联系我 habo.amy@gmail.com，让我把这些代码 close，好睡个安稳觉，谢了  
前提：你必须知道目标驱动保护（简称 213.sys）的哪个进程（简称 213.exe)，从而得到此进程的 debugport 地址  
      你必须确认 213.sys 是否创建了一个线程来不断清零这个 debugport  
  
好了，不卖关子了，嘿嘿，进入正题  
1：先 hook 掉 NtGetContextThread,NtSetContextThread，如果目标驱动没对这两个函数做手脚，此步略过  
2：IDT HOOK 一号向量  
3：枚举系统线程，对每个线程下硬件写入断点，地址就是 213.sys 要保护的进程的 debugport  
理论流程是这样的，保护线程对 213.exe 的 debugport 清零，触发断点-->1 号向量接管-->进入我们的函数->简单的处理是把 debugport 改回来，复杂的处理是自动寻找 213.sys 清零 213.exe 的 debugport 的函数体内可以 modify 的位置，改掉它，使清零操作无效，我用 XDE(一个反汇编引擎）试过，后来把这块代码去掉了，太浪费生命了……因为我觉得硬件断点速度很快，所以也就直接使用简单的办法，我承认我很懒，因为使用后者，又要多百多行代码，几千个字母，我手抽筋了![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")  
  
你们在后面的代码会看到我是针对 TP 做的，也许你们会说干掉 TP 有很多种方法，但是我要的不是结果，我要的过程……所以才有了这段尝试  
  
现在思路有了，写代码就简单了。但是我很悲催的没有完成……我承认代码写的很烂很烂很烂，但是我第一次发帖，大家还是低调点给我找找错误吧，因为我的确不喜欢太傲的人，我会说他很 213，怎么说呢，有兴趣就读完我的垃圾代码吧，没兴趣，飘过吧~  
  
贴代码会不会太多了……而且我写代码不习惯写很多注释。呃……将就着看吧  
  

代码:

```
#include <Ntddk.h>
#include <winDef.h>
#include <strsafe.h>


#define MAKELONG(a,b) ((ULONG)(((USHORT)(a))|((ULONG)((USHORT)(b))) << 16)) 

void Recover\_NtGetContextThread();  //用 SDT HOOK 来防止驱动对硬件断点的检测
void IDT\_HOOK();   
void Modify\_Code();  //修改 DEBUGPORT 为非 0


ULONG NtGetContextThread\_Addr; //SDT 中，原来的 NtGetContextThread 地址，下同
ULONG NtSetContextThread\_Addr;
ULONG Old\_Addr;    //IDT 1 号向量原来的 ISR 的地址，
ULONG DebugPort\_Addr;  //驱动保护的进程的 debugport 的 地址

typedef struct \_ServiceDescriptorTable 
{ 
    PVOID      ServiceTable;  
    PULONG   Counter;  
    ULONG    LimitNum;
    PBYTE       ArgsTable; 
} 
ServiceDescriptorTable, 
\*PServiceDescriptorTable, 
\*\*PPServiceDescriptorTable;
  
extern PServiceDescriptorTable KeServiceDescriptorTable;

#pragma pack(1)
typedef struct \_IDTENTRY{
USHORT offsetLow;
USHORT selector;
UCHAR  NoUseful;
UCHAR  segType:4;
UCHAR  segFlag:1;
UCHAR  DPL:2;
UCHAR  present:1;
USHORT offsetHigh;
}IDTENTRY,\*PIDTENTRY;   //定义 IDT 中的每个向量的格式，每个向量占 8 字节，我们只关心高位偏移和低位偏移就可以了
                                         //加起来就是这个 ISR 的地址        

typedef struct \_IDT{
USHORT limit;
USHORT baseLow;
USHORT baseHigh;
}IDT,\*PIDT;
#pragma pack()  //定义 IDT 的格式，指出基址，大小


void PPON(){
\_asm{
  cli
  mov eax,cr0
  or eax,10000h
  mov cr0,eax
  sti
}
}

void PPOFF(){
\_asm{
 cli
 mov eax,cr0
 and eax,not 10000h
 mov cr0,eax
 sti
}
}  //这两个函数不解释，你懂得

ULONG GetFuncAddrFromSDT(ULONG index){
ULONG Addr;
\_asm{
  push eax
  mov eax,KeServiceDescriptorTable
  mov eax,\[eax\]
  shr eax,2                       //-->eax/4
  add eax,index  //-->eax/4+index
  shl eax,2   //-->(eax/4+index)\*4=eax+index\*4
  mov eax,\[eax\]  
  mov Addr,eax
  pop eax
  }
return Addr;  
}   //从 SDT 中取得函数地址

ULONG GetAddr(){   //Get the list link start address
#define DEBUG\_PORT 0xBC
#define IMAGE\_NAME 0x174
#define LIST\_ADDR  0x88
ULONG cur=(ULONG)PsGetCurrentProcess();  //Current process must be SYSTEM
ULONG curAddr,startAddr;
curAddr=startAddr=(ULONG)(cur+LIST\_ADDR);  //save the start address
do{
if(strcmp((char\*)(curAddr-LIST\_ADDR+IMAGE\_NAME),"DNF.EXE")) curAddr=\*(ULONG\*)curAddr;  //如果是 DNF 进程，就搞到它的 debugport 地址，大家蛋定，DNF 是垃圾
   else{
    DebugPort\_Addr=(ULONG)(curAddr-LIST\_ADDR+DEBUG\_PORT);  
break;
}   
}while(curAddr!=startAddr);
return \*(ULONG\*)(cur+0x114)+0x22C;  //return the thread list head   //返回 system 的线程当前线程链表首地址，供遍历系统线程使用
}

ULONG SetAllSystemThreads(ULONG ListAddr){
#define THREADLIST\_ADDR 0x22C
#define CID\_OFFSET 0x1EC
ULONG curAddr,startAddr;
OBJECT\_ATTRIBUTES ObjAttr;
PCLIENT\_ID cid;
HANDLE h;
CONTEXT con;
NTSTATUS status;
InitializeObjectAttributes(&ObjAttr,0,OBJ\_CASE\_INSENSITIVE|OBJ\_KERNEL\_HANDLE,0,0);
curAddr=startAddr=ListAddr;   //你懂得，开始遍历系统线程，没得到一个，都设置对其下硬件断点
do{
cid=(PCLIENT\_ID)(ULONG)(curAddr-THREADLIST\_ADDR+CID\_OFFSET);
//h=NtOpenThread(h,THREAD\_ADLL\_ACCESS,&ObjAttr,cid);
\_asm{
    push eax
    push edx
push 80h
call GetFuncAddrFromSDT  //得到 NtOpenThread 的函数地址，0x80 是它的 index
mov edx,eax
push cid
lea eax,ObjAttr
push eax
push THREAD\_ALL\_ACCESS
push h
call edx  //这几句代码就是 NtOpenThread(h,THEAD\_ALL\_SUCCESS,&ObjAttr,cid);
             //这里主要起作用的是 CLIENT\_ID，它能标识出正确的线程
pop edx
pop eax
    }
if(h){   //如果成功打开
   \_asm{
    push eax
lea eax,con
push eax
push h
call NtGetContextThread\_Addr //等价于 NtGetContextThread(h,&con);
mov status,eax
pop eax
    }
   if(NT\_SUCCESS(status)){ //如果成功取得上下文
    con.Dr3=DebugPort\_Addr; 
con.Dr7=0xD0002080;  //至于为什么这样设置，可以去翻 Intel 手册，因为实在不好解释
//NtSetContextThread(h,&con);
\_asm{
push eax
lea eax,con
push eax
push h
call NtSetContextThread\_Addr
pop eax
}
}
ZwClose(h); //关闭句柄
   }
}while(curAddr!=startAddr);
}

NTSTATUS Jmp\_NtGetContextThread(
IN HANDLE ThreadHandle,
OUT PCONTEXT Context
){
if(strcmp((char\*)((ULONG)PsGetCurrentProcess()+0x174),"DNF.EXE")){  //不要让 DNF 得到自身的上下文，否则，否则我也不知道会怎样
\_asm jmp Jmp\_NtGetContextThread
}
else return STATUS\_UNSUCCESSFUL;
}

NTSTATUS Jmp\_NtSetContextThread(
IN HANDLE ThreadHandle,
IN PCONTEXT Context
){
if(strcmp((char\*)((ULONG)PsGetCurrentProcess()+0x174),"DNF.EXE")){  //同上
\_asm jmp Jmp\_NtSetContextThread
}
else return STATUS\_UNSUCCESSFUL;
}
    
void Rcover\_NtGetAndSetContextThread(){   //简单的 SDT HOOK，至于那段汇编，和前面 GetFuncAddrFromSDT 雷同
//SSDT HOOK
KIRQL irq;
irq=KeRaiseIrqlToDpcLevel();
PPOFF();
\_asm{
sti
push eax
push ebx
mov  eax,KeServiceDescriptorTable
mov eax,\[eax\]
shr eax,2
add eax,0x55
shl eax,2
mov ebx,\[eax\]
mov NtGetContextThread\_Addr,ebx
mov ebx,Jmp\_NtGetContextThread
mov \[eax\],ebx
shr eax,2
add eax,0x90
shl eax,2
mov ebx,\[eax\]
mov NtSetContextThread\_Addr,ebx
mov ebx,Jmp\_NtSetContextThread
mov \[eax\],ebx
pop ebx
pop eax
sti
}
PPON();
KeLowerIrql(irq);
}

void Unhook\_NtGetAndSetContextThread(){
KIRQL irq=KeRaiseIrqlToDpcLevel();
PPOFF();
\_asm{
push eax
push ebx
mov eax,KeServiceDescriptorTable
mov eax,\[eax\]
shr eax,2
add eax,55h
shl eax,2
mov ebx,NtGetContextThread\_Addr
mov \[eax\],ebx
shr eax,2
add eax,90h
shl eax,2
mov ebx,NtSetContextThread\_Addr
mov \[eax\],ebx
pop ebx
pop eax
}
PPON();
KeLowerIrql(irq);
}

void \_stdcall Modify\_Code(){  //这个驱动核心的功能，把 debugport 改回来，是不是有点杀鸡用牛刀的感觉了 = =
\_asm mov dword ptr\[DebugPort\_Addr\],0x44  //set the value of debugport to nozero
}

\_declspec(naked) IDT\_ProxyFunc(){  //代理函数，用它替换 IDT 一号向量
\_asm{
pushad
pushfd
call Modify\_Code
popfd
popad
jmp Old\_Addr
}
}

void IDT\_HOOK(){  //IDT HOOK，简单解释下
IDT idt;
PIDTENTRY entry;
\_asm sidt idt   //储存 IDT 信息到变量 idt
entry=(PIDTENTRY)MAKELONG(idt.baseLow,idt.baseHigh); //得到基址
KdPrint(("the base address of IDT is %X",(ULONG)entry));
Old\_Addr=(ULONG)MAKELONG(entry\[1\].offsetLow,entry\[1\].offsetHigh);  //保存一号向量的原 ISR 地址
KdPrint(("the NO.1 ISR address is %X",Old\_Addr));
PPOFF();
\_asm{
//entry\[1\].offsetLow=(USHORT)(ULONG)&IDT\_ProxyFunc;
//entry\[1\].offsetHigh=(USHORT)((ULONG)&IDT\_ProxyFunc>>16);  //分别修改高位和低位偏移，地址就是那个啥，呃，代理函数 IDT\_ProxyFunc
cli
push eax
push edx
mov eax,entry
add eax,8 //move to next ISR
mov edx,IDT\_ProxyFunc
mov word ptr\[eax\],dx           //\*\*\*\*注意了哈\*\*\*\*就是这个鸟指令，蓝屏的关键，肯定是 eax 问题
                                            //但是，运行到此时 eax 的地址是正确的，为什么？我用 windbg 看的呗
add eax,6 //move to offsetHigh
shr edx,16
mov word ptr\[eax\],dx
pop edx
pop eax
sti
}
PPON();  //上面的就是导致蓝屏的代码，我实在郁闷~咋就蓝了呢，我用汇编改写了上面注释的两句 C 代码
//虽然 ASM 写的很累赘，但是我只是为了找出哪里导致了蓝屏，大家不要鄙视
//我用 notepad++写了几小时代码通宵也不容易诶，一个字一个字母敲，呜呜
}

void IDT\_UNHOOK(){  //不解释了，对应 IDT\_HOOK
IDT idt;
PIDTENTRY entry;
\_asm sidt idt
entry=(PIDTENTRY)(ULONG)MAKELONG(idt.baseLow,idt.baseHigh);
\_asm cli
entry\[1\].offsetLow=(USHORT)Old\_Addr;
entry\[1\].offsetHigh=(USHORT)((ULONG)Old\_Addr>>16);
\_asm sti
}

NTSTATUS DriverUnload(PDRIVER\_OBJECT driver){
 Unhook\_NtGetAndSetContextThread();
 IDT\_UNHOOK();
}

NTSTATUS DriverEntry(PDRIVER\_OBJECT driver,PUNICODE\_STRING reg){
ULONG ListAddr;
#if DBG
\_asm int 3
#endif
driver->DriverUnload=DriverUnload;
Rcover\_NtGetAndSetContextThread();
IDT\_HOOK();
ListAddr=GetAddr();
if(DebugPort\_Addr) SetAllSystemThreads(ListAddr);
}

```