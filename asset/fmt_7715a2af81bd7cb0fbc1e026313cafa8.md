
```
WriteFile(hFile, PByte(@AttachStart)^, integer(@AttachEnd) - integer(@AttachStart), ByteSWrite, nil);  

{**************************}  

//修改所谓的外壳处大量数据  

ChangeData := ImageSectionHeader.PointerToRawData + 3;  

SetFilePointer(hFile, ChangeData, nil, FILE_BEGIN);  

WriteFile(hFile, StartCr, 4, ByteSWrite, nil); //开始加密地址  

ChangeData := ChangeData + 5;  

SetFilePointer(hFile, ChangeData, nil, FILE_BEGIN);  

WriteFile(hFile, SizeEN, 4, ByteSWrite, nil); //大小  

ChangeData := ChangeData + 21;  

SetFilePointer(hFile, ChangeData, nil, FILE_BEGIN);  

WriteFile(hFile, OEP, 4, ByteSWrite, nil); //跳回 OEP  

{**************************}  

//没有异常,显示增加区块成功!  

ShowMessage('增加区块成功!');  

SetFilePointer(hFile, StartEN, nil, FILE_BEGIN);  

ReadFile(hFile, lpBuffer, SizeEN, BytesRead, nil);  

for i := 0 to SizeEN - 1 do  
begin  
  byte(pointer(integer(@lpBuffer) + i)^) := byte(pointer(integer(@lpBuffer) + i)^) xor $50;  
end;  

SetFilePointer(hFile, StartEN, nil, FILE_BEGIN);  

WriteFile(hFile, lpBuffer, SizeEN, ByteSWrite, nil);  

//没有异常,显示变换成功!  

ShowMessage('变换数据成功!');  

{**************************}  

SetFilePointer(hFile, (ImageDosHeader._lfanew + SizeOf(ImageNtHeaders)), nil, FILE_BEGIN);  

for i := 0 to ImageNtHeaders.FileHeader.NumberOfSections - 1 do  
begin  
  //读取块表中信息  
  ReadFile(hFile, ImageSectionHeader, SizeOf(ImageSectionHeader), BytesRead, nil);  
  if ImageSectionHeader.PointerToRawData = StartEN then  
  begin  
    ImageSectionHeader.Characteristics := $E00000E0;  
    SetFilePointer(hFile, -SizeOf(ImageSectionHeader), nil, FILE_CURRENT);  
    //保存区块信息  
    WriteFile(hFile, ImageSectionHeader, SizeOf(ImageSectionHeader), ByteSWrite, nil);  
    Break;  
  end;  
end;  

ShowMessage('修改区段属性成功!');  

{**************************}  

finally  
  {8.退出}  
  //关闭文件  
  CloseHandle(hFile);  
end;  

end;  
{**************************Func end**************************}  

procedure TForm1.Button1Click(Sender: TObject); //addshell  

begin  
  AddShell(Edit1.text, true);  
end;  

procedure TForm1.Button3Click(Sender: TObject); //browser  

begin  
  with TOpenDialog.Create(Self) do  
  try  
    Filter := '可执行文件 (\*.exe)|\*.exe';  
    if Execute then  
    begin  
      Edit1.text := FileName;  
    end;  
  finally  
    Free;  
  end;  
end;  

procedure TForm1.Button2Click(Sender: TObject); //exit  

begin  
  close;  
end;  

end.  
```
