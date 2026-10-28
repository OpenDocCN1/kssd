
```delphi
  inc(dwServices);

  end;

  FreeLibrary(hKernel);

   GetMem(nowaddress,sizeof(DWORD)\* dwServices);

   QueryBuff.Address:=  dwKernelBase+dwKiServiceTable;

   QueryBuff.Data:= nowaddress;

   QueryBuff.Length:=sizeof(DWORD)\* dwServices;

  ZwSystemDebugControl(SysDbgReadVirtualMemory,

                       @QueryBuff,

                       sizeof(QueryBuff),

                       nil,

                       0,

                       @ReturnLength);

   // messagebox(0,pchar(inttohex(Ulong(QueryBuff),4)),'lk',0);

//把结果输出来

form1.ListView1.Clear;

i:=0;

while(i<dwServices) do

begin

  form1.ListView1.Items.Add;

  form1.ListView1.Items.Item[i].Caption:='0x'+inttohex(i,2);

  form1.ListView1.Items.Item[i].SubItems.add( ssdt\_list[i].fname);

  form1.ListView1.Items.Item[i].SubItems.add('0x'+inttohex(ssdt\_list[i].address1,4));

  ssdt\_list[i].address2:= nowaddress^;

  form1.ListView1.Items.Item[i].SubItems.add('0x'+inttohex(ssdt\_list[i].address2,4));

  nowaddress:=PDWORD(DWORD(nowaddress)+sizeof(DWORD));

  s:=module;

   //搜索模块

   for j:=0 to p^-1 do

   begin

    if(ssdt\_list[i].address2>DWORD(s^.Base)) and (ssdt\_list[i].address2<DWORD(s^.Base+ s^.Size) )  then

     begin

       form1.Listview1.Items.Item[i].SubItems.add(s^.ImageName);

       break;

    end;

    s:=PModuleInformation(DWORD(module)+sizeof(TModuleInformation)\*J);

  end;

  inc(i);

end;

    FreeMem(p);

end;

//使用 ZwSystemDebugControl前必须提权

function DebugPrivilege(PName:PCHAR;bEnable:BOOL):BOOL;

var

Ok:BOOL;

hToken:THANDLE;

tp,tkpo:        TTokenPrivileges;

lBufferNeeded:DWORD;

begin

if (OpenProcessToken(GetCurrentProcess,TOKEN\_QUERY or TOKEN\_ADJUST\_PRIVILEGES,hToken)) then

begin

  tp.PrivilegeCount:=1;

  if  bEnable then

  tp.Privileges[0].Attributes:=SE\_PRIVILEGE\_ENABLED;

  LookupPrivilegeValue(nil,PName,tp.Privileges[0].Luid);

  AdjustTokenPrivileges(hToken, False, tp, Sizeof(TOKEN\_PRIVILEGES), TKPO, lBufferNeeded);

  if  GetLastError()=ERROR\_SUCCESS then

  begin

      ok:=false;

      Exit;

  end ;

  CloseHandle(hToken);

  result:=ok;

end;

end;

end.
```

![](https://bbs.kanxue.com/upload/attach/201004/172841_4hlkh9x92w4wzgt.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- `ssdt.JPG`（156.43kb，344次下载）
