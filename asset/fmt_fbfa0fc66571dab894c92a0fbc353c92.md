
```
;@echo  off
;goto  make
;功能:驱动实现模拟键盘按键
;说明:模拟键盘按键就需要访问 IO 端口 但是系统默认是不允许访问的，所以我们利用驱动修改 TSS 中的 IOPM(IO 许可位图),允许我们的主程序访问 IO 端口(直接用 in、out 访问).
;by: 看雪--ericzw  QQ--巴丫丫 37261550
    .586
    .model  flat,stdcall
    option  casemap:none
include  \\masm32\\include\\windows.inc
include  \\masm32\\include\\user32.inc
include  \\masm32\\include\\kernel32.inc
includelib  \\masm32\\lib\\user32.lib
includelib  \\masm32\\lib\\kernel32.lib

include  \\masm32\\include\\advapi32.inc;操作注册表使用
includelib  \\masm32\\lib\\advapi32.lib
;字符宏 声明字符串
$s MACRO txt:REQ
  local d,sn
  sn TEXTEQU @CurSeg
   .const
    d db txt,0
  @CurSeg ENDS
  sn SEGMENT
  EXITM <offset d>
ENDM
    .data
hScm  dword  ?;SCM 句柄
hService  dword  ?;服务句柄
szFullName  byte  MAX\_PATH  DUP  (?);驱动全路径
hKey  dword  ?;注册表句柄
dwProcessId  dword  ?;当前进程 ID
szProcessId  byte  'ProcessId',0
f\_flag  dword  0;标志 成功启动了驱动程序后为非 0 
    .code
;访问端口实现模拟键盘按键 
SetKey    proc    \_Lparamkey;
        @Lp1:;无论向 0x60,还是 0x64 写东西前都要等状态寄存器 OBF 变 0
        in    al,64h
        and  al,10b
        jnz  @Lp1
        ;向$64 端口写命令
        mov  al,0D2h;写键盘输出缓存命令
        out  64h,al
         
        @Lp2:;无论向 0x60,还是 0x64 写东西前都要等状态寄存器 OBF 变 0
        in    al,64h
        and   al,10b
        jnz  @Lp2
        ;向$60 端口写参数
        mov  ebx,\_Lparamkey
        mov  al,bl
        out  60h,al
        ret
SetKey    endp
;程序入口
Start  proc
  invoke  OpenSCManager,NULL,NULL,SC\_MANAGER\_ALL\_ACCESS;建立到 SCM 的连接
  .if  eax
    mov  hScm,eax
    push  eax
    invoke  GetFullPathName,$s('myDriver.sys'),MAX\_PATH,addr  szFullName,esp;取得当前文件夹下的驱动完整路径
    pop  eax
    invoke  CreateService,hScm,$s('myDriverReg'),$s('myDriverNameReg'),SERVICE\_ALL\_ACCESS,SERVICE\_KERNEL\_DRIVER,\
        SERVICE\_DEMAND\_START,SERVICE\_ERROR\_IGNORE,addr  szFullName,NULL,NULL,NULL,NULL,NULL;将驱动添加到服务数据库中并创建对应的注册表键
    .if  eax
      mov  hService,eax
      invoke  RegOpenKeyEx,HKEY\_LOCAL\_MACHINE,$s("SYSTEM\\\\CurrentControlSet\\\\Services\\\\myDriverReg"),0,KEY\_ALL\_ACCESS,addr  hKey;打开注册表 注意:每次把驱动添加到服务数据中时,程序会同时在注册表"SYSTEM\\\\CurrentControlSet\\\\Services\\\\"下面创建一个以 CreateService 第二个参数为键名的新项.
      ;我们本次的工作就是往上面的注册表路径下面添加一个新的名为"ProcessId"的键值，里面的数据是当前进程的进程 ID，供我们的驱动读取,来实现我们的操作。
      .if  eax==ERROR\_SUCCESS
        invoke  GetCurrentProcessId;取得当前进程 ID
        mov  dwProcessId,eax
        invoke  RegSetValueEx,hKey,addr  szProcessId,NULL,REG\_DWORD,addr  dwProcessId,sizeof  DWORD;向注册表中写入键值 写入我们的进程 ID
        .if  eax==ERROR\_SUCCESS
          invoke  StartService,hService,NULL,NULL;启动驱动程序,这个函数运行后直到驱动程序入口函数运行完后才返回
          inc  f\_flag;设置为非 0 下面将访问端口
          invoke  RegDeleteKey,hKey,addr  szProcessId;清除我们刚才清加的进程 ID 项  (可有可无 运行 DeleteService 后整个驱动键都会被清除
        .else
          invoke  MessageBox,NULL,$s('写入进程 ID 失败'),NULL,MB\_OK   
        .endif
        invoke  RegCloseKey,hKey;释放句柄
      .else
        invoke  MessageBox,NULL,$s('打开注册表失败'),NULL,MB\_OK   
      .endif
      invoke  DeleteService,hService;将驱动从 SCM 数据库中删除 严格的说这个函数并不真正将服务删除，它将服务做了一个删除标志，只有服务已经停止，并且服务句柄被关闭后 SCM 才真正将服务删除     
      invoke  CloseServiceHandle,hService;关闭句柄
    .else
      invoke  MessageBox,NULL,$s('添加驱动到服务数据库失败'),NULL,MB\_OK
    .endif
    invoke  CloseServiceHandle,hScm;关闭句柄
  .else
    invoke  MessageBox,NULL,$s('打开 scm 失败'),NULL,MB\_OK
  .endif
  ;如果驱动正常加载,则我们连续按 5 次'A'键
  .if  f\_flag
    invoke  MessageBox,NULL,$s("安装驱动成功,点击'确定'将模拟键盘按 5 次'A'键"),$s("成功"),MB\_OK
    mov  ecx,5
    @@:   
    push  ecx
    invoke  MapVirtualKey,VK\_A,0;把 A 键的虚拟键码翻译为扫描码
    invoke  SetKey,eax;访问 IO 实现模拟键盘按 A 键
    invoke  Sleep,1000
    pop  ecx
    loop  @B 
  .endif
  invoke  ExitProcess,-1
  ret
Start  endp
end  Start
:make
set  exe=StartD
\masm32\bin\ml /nologo /c /coff %exe%.bat
\masm32\bin\link /nologo /subsystem:windows %exe%.obj
del  %exe%.obj
echo.
pause

```

解释已经尽量写的很清楚，主程序的解释少一些，不懂的函数可以查 `msdn`。

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` | `模拟按键.rar` |
| :--- | :--- |
