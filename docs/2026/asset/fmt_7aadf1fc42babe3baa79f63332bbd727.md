
```pascal
//  从导入表中获取一个函数的地址

function GetImportFunAddr(lpImportAddr: Pointer): Pointer; stdcall;

begin

  Result := PPointer(PPointer(Cardinal(lpImportAddr) + 2)^)^;

end;



//  KeServiceDescriptorTable+函数名计算SSDT函数偏移

function SystemServiceName(AFunc: Pointer): PLONG; stdcall;

var

  lpKeServiceDescriptorTable: PServiceDescriptorEntry;

begin

  lpKeServiceDescriptorTable := GetImportFunAddr(@KeServiceDescriptorTable);

  Result := PLONG(Cardinal(lpKeServiceDescriptorTable^.ServiceTableBase) + (SizeOf(ULONG) * PULONG(ULONG(AFunc) + 1)^));

end;



//  KeServiceDescriptorTable+序号名计算SSDT函数偏移

function SystemServiceOrd(iOrd: ULONG): PLONG; stdcall;

var

  lpKeServiceDescriptorTable: PServiceDescriptorEntry;

begin

  lpKeServiceDescriptorTable := GetImportFunAddr(@KeServiceDescriptorTable);

  Result := PLONG(PLONG(Cardinal(lpKeServiceDescriptorTable^.ServiceTableBase) + (SizeOf(ULONG) * iOrd)));

end;



function FindShadowTable:Pointer;  //XP版本

var

 lpKeServiceDescriptorTable:ULONG;

begin

 lpKeServiceDescriptorTable := ULONG(GetImportFunAddr(@KeServiceDescriptorTable));

 Result:=Pointer(lpKeServiceDescriptorTable-$40);

end;



function FindShadowTable2:Pointer;

var

 cPtr, pOpcode:ULONG;

 I:ULONG;

begin

  Result:=nil;

  cPtr:=ULONG(GetImportFunAddr(@KeAddSystemServiceTable));

  I:=cPtr;

  While (I<(cPtr+$1000)) Do

  begin

    if MmIsAddressValid(Pointer(I)) then

    begin

      if word(pointer(I)^)=$888d then

      begin

       Result:=PPointer(I+2)^;

       break

      end;

    end;

    I:=I+1;

  end;

end;



//序号名计算Shadow SSDT函数偏移

function ShadowSystemServiceOrd(iOrd: ULONG): PLONG; stdcall;

var

  lpKeServiceDescriptorTable:PShadowSrvDescriptorEntry;

begin

  lpKeServiceDescriptorTable :=FindShadowTable2;

  Result := PLONG(Cardinal(lpKeServiceDescriptorTable^.win32kTable.ServiceTableBase) + (SizeOf(ULONG) * iOrd));

end;
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
