
# 详解截取封包的原理

> 原文：[`www.pediy.com/kssd/pediy12/131386.html`](https://www.pediy.com/kssd/pediy12/131386.html)

来看雪已经有快一年了，由于技术问题一直还是临时会员（杯具）![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

最近帮人写一个截取封包改发封包的东西，由于起初什么都不懂也就开始疯狂的查资料，但是网上都千篇一律的。。。。就是这个转那个，但是那些都把基础的讲到了，但是对于像我这种什么都还不懂的人就困难了。。。。

我按照网上说的用 `hook` 技术写了一个小程序，就是 `hook` `ws2_32` 的 `send` 与 `recv` 函数实现的，但是我问题就出现了，`hook` 到他的接收和发送封包又怎么用呢？？？？？？想必刚刚开始你也有这样的疑问吧。。。。下面我就详细讲一下我所理解的

下面首先是在网上的源码，经过小小的改动

```
//////////////////////////////////////////////////////////////////////////////////
////////首先是创建 dll 文件用来 hook 程序安装钩子

library mydll;

{ Important note about DLL memory management: ShareMem must be the
  first unit in your library's USES clause AND your project's (select
  Project-View Source) USES clause if your DLL exports any procedures or
  functions that pass strings as parameters or function results. This
  applies to all strings passed to and from your DLL--even those that
  are nested in records and classes. ShareMem is the interface unit to
  the BORLNDMM.DLL shared memory manager, which must be deployed along
  with your DLL. To avoid using BORLNDMM.DLL, pass string information
  using PChar or ShortString parameters. }

uses
  SysUtils,
  Classes,
  messages,
  Variants,
  windows,
  hookapi_pas in 'hookapi_pas.pas';

type
PData = ^TData;

TData = record
Hook: THandle;
Hooked: Boolean;
end;

var
DLLData: PData;

{$R \*.res}

{------------------------------------}
{过程名:HookProc
{过程功能:HOOK 过程
{过程参数:nCode, wParam, lParam 消息的相
{ 关参数
{------------------------------------}
procedure HookProc(nCode, wParam, lParam: LongWORD);stdcall;
begin
if not DLLData^.Hooked then
begin
HookAPI;
DLLData^.Hooked := True;
end;

//调用下一个 Hook
CallNextHookEx(DLLData^.Hook, nCode, wParam, lParam);
end;

{------------------------------------}
{函数名:InstallHook
{函数功能:在指定窗口上安装 HOOK
{函数参数:sWindow:要安装 HOOK 的窗口
{返回值:成功返回 TRUE,失败返回 FALSE
{------------------------------------}
function InstallHook(SWindow: LongWORD):Boolean;stdcall;
var
ThreadID: LongWORD;
begin
Result := False;
DLLData^.Hook := 0;
ThreadID := GetWindowThreadProcessId(sWindow, nil);

//给指定窗口挂上钩子
DLLData^.Hook := SetWindowsHookEx(WH_GETMESSAGE, @HookProc, Hinstance, ThreadID);

if DLLData^.Hook > 0 then
Result := True //是否成功 HOOK
else
exit;
end;

{------------------------------------}
{过程名:UnHook
{过程功能:卸载 HOOK
{过程参数:无
{------------------------------------}
procedure UnHook;stdcall;
begin
UnHookAPI;
//卸载 Hook
UnhookWindowsHookEx(DLLData^.Hook);
end;

{------------------------------------}
{过程名:DLL 入口函数
{过程功能:进行 DLL 初始化,释放等
{过程参数:DLL 状态
{------------------------------------}
procedure MyDLLHandler(Reason: Integer);
var
FHandle: LongWORD;
begin
case Reason of
DLL_PROCESS_ATTACH:
begin //建立文件映射,以实现 DLL 中的全局变量
fhandle:=createfilemapping($ffffffff,nil,page_readwrite,0,$ffff,'mydlldata');

if FHandle = 0 then
if GetLastError = ERROR_ALREADY_EXISTS then
begin
FHandle := OpenFileMapping(FILE_MAP_ALL_ACCESS, False,'mydlldate');
if FHandle = 0 then Exit;
end else Exit;

DLLData := MapViewOfFile(FHandle, FILE_MAP_ALL_ACCESS, 0, 0, 0);
if DLLData = nil then
CloseHandle(FHandle);
end;

DLL_PROCESS_DETACH:
begin
if Assigned(DLLData) then
begin
UnmapViewOfFile(DLLData);
DLLData := nil;
end;
end;
end;
end;

{$R \*.res}
exports
InstallHook, UnHook, HookProc;
begin
DLLProc := @MyDLLHandler;
MyDLLhandler(DLL_PROCESS_ATTACH);
DLLData^.Hooked := False;
end.
```

再在 `dll` 文件创建一个 `.pas` 的文件 `<这个是用来 hook 你想 hook 的 api>`

```
unit hookapi_pas;

interface

uses
SysUtils,
Windows, WinSock, dialogs;

type

//要 HOOK 的 API 函数定义
TSockProc = function (s: TSocket; Buf:integer; len, flags: Integer): Integer; stdcall;
PJmpCode = ^TJmpCode;
TJmpCode = packed record
JmpCode: BYTE;
Address: TSockProc;
MovEAX: Array [0..2] of BYTE;
end;

procedure hookapi;
procedure unhookapi;
procedure saveinfo(ss:string);stdcall;
function recvout(var Rbuf;RLen:Integer):string;

var
ii:integer=1;
OldSend, OldRecv: TSockProc; //原来的 API 地址
JmpCode: TJmpCode;
OldProc: array [0..1] of TJmpCode;
AddSend, AddRecv: pointer; //API 地址
TmpJmp: TJmpCode;
ProcessHandle: THandle;
buf_f:array [1..100] of byte  ;

implementation

procedure saveinfo(ss:string);stdcall;
var
f:file;
filename:string;
begin
  filename:='d:\test.txt';
  assignfile(f,filename);
   closefile(f);
end ;

function recvout(var Rbuf;RLen:Integer):string;
Var
buf1:pchar;
i:integer;
ss:string;
Begin
ss:='';
buf1:=@Rbuf;
for i:=3 to 4 do
Begin
ss:=ss+inttohex(byte(buf1^),2)+' '  ;
buf1:=buf1+1;
End;
result:=ss;
End;

function MySend(s: TSocket;  Buf,len, flags: Integer): Integer; stdcall;
var
dwSize: cardinal;
tmp:string ;
i:integer;
begin
//这儿进行发送的数据处理
MessageBeep(1000); //简单的响一声
//调用直正的 Send 函数
WriteProcessMemory(ProcessHandle, AddSend, @OldProc[0], 8, dwSize);
Result := OldSend(S, Buf, len, flags);
JmpCode.Address := @MySend;
WriteProcessMemory(ProcessHandle, AddSend, @JmpCode, 8, dwSize);
end;

{---------------------------------------}
{函数功能:Recv 函数的 HOOK
{函数参数:同 Recv
{函数返回值:integer
{---------------------------------------}
function MyRecv(s: TSocket;  Buf, len, flags: Integer): Integer; stdcall;
var
dwSize: cardinal;
mc: byte;
m_buf:pchar;
i:integer;
begin
////////这里可以加入你处理封包的代码
我用接受封包来讲解：
首先用 od 加载我要的游戏，转到 recv 函数处，
MessageBeep(1000); //简单的响一声
WriteProcessMemory(ProcessHandle, AddRecv, @OldProc[1], 8, dwSize);
Result := OldRecv(S, Buf, len,flags);
JmpCode.Address := @MyRecv;
WriteProcessMemory(ProcessHandle, AddRecv, @JmpCode, 8, dwSize);
MessageBeep(1000);
end;

{------------------------------------}
{过程功能:HookAPI
{过程参数:无
{------------------------------------}
procedure HookAPI;
var
DLLModule:thandle;
dwSize: cardinal;
begin
ProcessHandle := GetCurrentProcess;
DLLModule:= LoadLibrary('ws2_32.dll');
addsend:=getprocaddress(dllmodule,'send');
addrecv:=getprocaddress(dllmodule,'recv');

JmpCode.JmpCode := $B8;
JmpCode.MovEAX[0] := $FF;
JmpCode.MovEAX[1] := $E0;
JmpCode.MovEAX[2] := 0;

ReadProcessMemory(ProcessHandle, AddSend, @OldProc[0], 8, dwSize);
JmpCode.Address := @MySend;
WriteProcessMemory(ProcessHandle, AddSend, @JmpCode, 8, dwSize); //修改 Send 入口
ReadProcessMemory(ProcessHandle, AddRecv, @OldProc[1], 8, dwSize);
JmpCode.Address := @MyRecv;
WriteProcessMemory(ProcessHandle, AddRecv, @JmpCode, 8, dwSize); //修改 Recv 入口
OldSend := AddSend;
OldRecv := AddRecv;
end;

{------------------------------------}
{过程功能:取消 HOOKAPI
{过程参数:无
{------------------------------------}
procedure UnHookAPI;
var
dwSize: Cardinal;
begin
WriteProcessMemory(ProcessHandle, AddSend, @OldProc[0], 8, dwSize);
WriteProcessMemory(ProcessHandle, AddRecv, @OldProc[1], 8, dwSize);
end;

end.
```

`///////` 通过上面得编译你便能够编译出你的 `dll` 文件了

我们都知道 `dll` 文件时不可能单独运行的，所以我们要用 `exe` 用来加载你的 `dll` 文件了

这个就自己写 `<网上一查便能查到了>`

好我们 `hook` `socket` 的程序便写好了

这个是没有处理封包的过程。

下面我来讲一下如何处理封包了

首先看一下他的原理是什么。我用一个棋牌游戏来说明

```
757B47DF >  8BFF            mov edi,edi                     ; recv 的入口点
757B47E1    55              push ebp
757B47E2    8BEC            mov ebp,esp
757B47E4    83EC 10         sub esp,0x10
757B47E7    53              push ebx
757B47E8    33DB            xor ebx,ebx
757B47EA    813D 48707D75 3>cmp dword ptr ds:[0x757D7048],WS2_32.757>
757B47F4    56              push esi
```



757B47F5 `^` `0F85` `F4EEFFFF` `jnz` `WS2_32.757B36EF`  

757B47FB `391D` `70707D75` `cmp` `dword` `ptr` `ds:[0x757D7070]` `ebx`  

757B4801 `^` `0F84` `E8EEFFFF` `je` `WS2_32.757B36EF`  

757B4807 `FF35` `44707D75` `push` `dword` `ptr` `ds:[0x757D7044]`  

757B480D `FF15` `48127B75` `call` `dword` `ptr` `ds:[<&API-MS-Win-Core-Pro>];` `kernel32.TlsGetValue`  

757B4813 `8945` `F8` `mov` `dword` `ptr` `ss:[ebp-0x8]` `eax`  

757B4816 `3BC3` `cmp` `eax` `ebx`  

757B4818 `^` `0F84` `D1EEFFFF` `je` `WS2_32.757B36EF`  

757B481E `895D` `FC` `mov` `dword` `ptr` `ss:[ebp-0x4]` `ebx`  

757B4821 `FF75` `08` `push` `dword` `ptr` `ss:[ebp+0x8]`  

757B4824 `E8` `47E8FFFF` `call` `WS2_32.757B3070`  

757B4829 `8BF0` `mov` `esi` `eax`  

757B482B `3BF3` `cmp` `esi` `ebx`  

757B482D `^` `0F84` `E1EEFFFF` `je` `WS2_32.757B3714`  

757B4833 `8B4D` `F8` `mov` `ecx` `dword` `ptr` `ss:[ebp-0x8]`  

757B4836 `8B45` `10` `mov` `eax` `dword` `ptr` `ss:[ebp+0x10]`  

757B4839 `57` `push` `edi`  

757B483A `83C1` `08` `add` `ecx` `0x8`  

757B483D `8D55` `FC` `lea` `edx` `dword` `ptr` `ss:[ebp-0x4]`**

当我注入自己的 hook 时再看一下 recv 处得代码

```
757B47DF >  B8 88C74504     mov eax,024CC788 ; recv 的入口点
757B47E4     FFE0          jmp eax                  ； 跳到我自己的代码处
757B47E6     0053 33       add byte ptr ds:[ebx+0x33],dl
757B47E9     DB81 3D48707D fild dword ptr ds:[ecx+0x7D70483D]
757B47EF     75 36         jnz short WS2_32.757B4827
757B47F1     337B 75       xor edi,dword ptr ds:[ebx+0x75]
757B47F4     56          push esi
757B47F5   ^ 0F85 F4EEFFFF  jnz WS2_32.757B36EF
757B47FB     391D 70707D75  cmp dword ptr ds:[0x757D7070],ebx
757B4801   ^ 0F84 E8EEFFFF  je WS2_32.757B36EF
757B4807     FF35 44707D75  push dword ptr ds:[0x757D7044]
757B480D     FF15 48127B75  call dword ptr ds:[<&API-MS-Win-Core-Pro>; kernel32.TlsGetValue
757B4813     8945 F8         mov dword ptr ss:[ebp-0x8],eax
757B4816     3BC3           cmp eax,ebx
757B4818   ^ 0F84 D1EEFFFF  je WS2_32.757B36EF
757B481E     895D FC         mov dword ptr ss:[ebp-0x4],ebx
757B4821     FF75 08         push dword ptr ss:[ebp+0x8]
757B4824     E8 47E8FFFF      call WS2_32.757B3070
757B4829     8BF0           mov esi,eax
757B482B     3BF3           cmp esi,ebx
757B482D   ^ 0F84 E1EEFFFF  je WS2_32.757B3714
757B4833     8B4D F8         mov ecx,dword ptr ss:[ebp-0x8]
757B4836     8B45 10         mov eax,dword ptr ss:[ebp+0x10]
757B4839     57          push edi
```

我再在 od 中转到我的代码处：

这里就是 `mov eax,024CC788`

`jmp eax` 跳到我们自己的代码：

```
024CC788  /.  55          push ebp                         ;  WS2_32.select
024CC789  |.  8BEC        mov ebp,esp
024CC78B  |.  83C4 F8     add esp,-0x8
024CC78E  |.  53          push ebx
024CC78F  |.  56          push esi
024CC790  |.  57          push edi
024CC791  |.  8B5D 10     mov ebx,[arg.3]
024CC794  |.  8B75 0C     mov esi,[arg.2]
024CC797  |.  8D45 F8     lea eax,[local.2]
024CC79A  |.  50          push eax                     ; /pBytesWritten
024CC79B  |.  6A 08        push 0x8                     ; |BytesToWrite = 8
024CC79D  |.  68 ECFB4C02  push mydll.024CFBEC           ; |Buffer = mydll.024CFBEC
024CC7A2  |.  A1 F8FB4C02  mov eax,dword ptr ds:[0x24CFBF8]         ; |
024CC7A7  |.  50          push eax                     ; |Address => 757B47DF
024CC7A8  |.  A1 FCFB4C02  mov eax,dword ptr ds:[0x24CFBFC]         ; |
024CC7AD  |.  50          push eax                     ; |hProcess => FFFFFFFF
024CC7AE  |.  E8 1999FBFF  call <jmp.&kernel32.WriteProcessMemory>  ; \WriteProcessMemory
024CC7B3  |.  8B45 14     mov eax,[arg.4]
024CC7B6  |.  50          push eax                     ; /Flags
024CC7B7  |.  53          push ebx                     ; |BufSize
024CC7B8  |.  56          push esi                     ; |Buffer
024CC7B9  |.  8B45 08     mov eax,[arg.1]                  ; |
024CC7BC  |.  50          push eax                     ; |Socket
024CC7BD  |.  FF15 D8FB4C02 call dword ptr ds:[0x24CFBD8]        ; \recv
024CC7C3  |.  8BF8         mov edi,eax
024CC7C5  |.  B8 88C74C02  mov eax,mydll.024CC788
024CC7CA  |.  A3 DDFB4C02  mov dword ptr ds:[0x24CFBDD],eax
024CC7CF  |.  8D45 F8     lea eax,[local.2]
024CC7D2  |.  50          push eax                     ; /pBytesWritten
024CC7D3  |.  6A 08        push 0x8                     ; |BytesToWrite = 8
024CC7D5  |.  68 DCFB4C02  push mydll.024CFBDC           ; |Buffer = mydll.024CFBDC
024CC7DA  |.  A1 F8FB4C02  mov eax,dword ptr ds:[0x24CFBF8]         ; |
024CC7DF  |.  50          push eax                     ; |Address => 757B47DF
024CC7E0  |.  A1 FCFB4C02  mov eax,dword ptr ds:[0x24CFBFC]         ; |
024CC7E5  |.  50          push eax                     ; |hProcess => FFFFFFFF
024CC7E6  |.  E8 E198FBFF  call <jmp.&kernel32.WriteProcessMemory>  ; \WriteProcessMemory
```

上面的就是我们下面代码的反汇编代码：

```pascal
Function MyRecv(s: TSocket; Buf, len, flags: Integer): Integer; stdcall;

var
dwSize: cardinal;
mc: byte;
m_buf: pchar;
i: integer;

begin

/简单的响一声

///showmessage(inttostr(buf)) ;

//MessageBeep(1000); //简单的响一声

WriteProcessMemory(ProcessHandle, AddRecv, @OldProc[1], 8, dwSize);

Result := OldRecv(S, Buf, len, flags);

JmpCode.Address := @MyRecv;

WriteProcessMemory(ProcessHandle, AddRecv, @JmpCode, 8, dwSize);

if (len=21) then
begin                      /////这里是我处理封包的函数 i:=buf ;

asm
push eax
mov eax,i
mov byte[eax],$c0
mov byte [eax+1],0
pop eax
end;

MessageBeep(1000);

end;

end;
```

只要你 hook 了他的 `recv` 后当封包发过来后便首先跳到你的代码，当你把封包处理后再到 `ws2_32` 的 `recv` 所有只要你在 `myrecv` 处加入你的代码便可以做自己想做的事情了

但是现在的封包都有加密，所以一般都要解密。解密的方法在这里就不再说了，因为每个程序加密方式都不太同

我希望我的理解能给你们有用。我也想通过这篇文章申请注册码。。

![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")
