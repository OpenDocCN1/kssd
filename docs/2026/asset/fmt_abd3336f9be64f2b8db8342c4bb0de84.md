
其中`ProcId`只在扫描RING3层时使用

继续,包装函数就不再继续展开了,都是比较简单的:

```
function CheckInlineHook(scanInfo:PScanHookInfo):Boolean;
var
  MapModuleBase     :DWORD;
  tempStr           :string;
  tempFile          :string;

  label lbExit1;
begin
  Result:=False;

  // 用户界面回显
  tempStr:=Format('[%s]准备初始化',[string(scanInfo.ModuleName)]);
  SendMessage(MainForm.Handle, MSG_THREAD_STATUS, Integer(@tempStr[1]), 0);

  // 在这里,我将原文件拷贝到了临时目录再进行映射
  tempFile:=CopyFileToTempDirectory(scanInfo.ModulePath);
  if not FileExists(tempFile) then Exit;

  // 映射模块
  MapModuleBase:=DWORD(MapFileWriteAble(tempFile));
  if MapModuleBase = 0 then
  begin
    tempStr:=Format('[%s]映射模块发生错误',[string(scanInfo.ModuleName)]);
    SendMessage(MainForm.Handle, MSG_THREAD_ERROR, Integer(@tempStr[1]), 0);
    Exit;
  end;

  // 用户界面回显
  tempStr:=Format('[%s]初始化完毕,重定向数据...',[string(scanInfo.ModuleName)]);
  SendMessage(MainForm.Handle, MSG_THREAD_STATUS, Integer(@tempStr[1]), 0);

  // 重定向模块
  if not RelocSpecialModule(MapModuleBase, scanInfo.ModuleBase) then
  begin
    tempStr:=Format('[%s]重定向模块时发生错误',[string(scanInfo.ModuleName)]);
    SendMessage(MainForm.Handle, MSG_THREAD_ERROR, Integer(@tempStr[1]), 0);
    goto lbExit1;
  end;

  // 填充结构
  scanInfo.MapModuleBase  :=MapModuleBase;

  // 首先检测IATHook
  case scanInfo.Mode of
    KernelMode:
    begin
      CheckSpecialModuleIATHookKernel(scanInfo);
    end;

    UserMode:
    begin
      //CheckSpecialModuleIATHookUser(scanInfo);
    end;
  end;

  // 扫描.text PAGE节区
  ScanSection('.text', scaninfo);
  ScanSection('PAGE', scaninfo);

  Result:=True;

lbExit1:
  if MapModuleBase <> 0 then UnMapFile(PBYTE(MapModuleBase));
  DeleteFile(tempFile);
end;
```

重定向也就是将PE结构中重定向目录表中的所有条目进行修复,遍历一下即可,

因为我们要使我们映射的模块看起来就像是加载到了内核中一样,所以我们利用内核的加载基址来修复重定向条目

```
function RelocSpecialModule(ModuleBase:DWORD; NewBase:DWORD; IsMapped:Boolean = True):Boolean;
var
  reloc       :PImageBaseReloc;
  relocSize   :DWORD;
  RvaBase     :DWORD;
  relocNum    :DWORD;
  i           :DWORD;
  DefaultBase :DWORD;

  w           :Word;
  dw          :DWORD;
  va          :DWORD;
begin
  Result:=False;
  if ModuleBase = 0 then Exit;
  if NewBase = 0 then Exit;

  Result:=True;
  // 取得重定向目录表,如果不存在,直接返回
  reloc:=GetModuleBaseRelocDirectory(ModuleBase, IsMapped);
  if reloc = nil then Exit;

  // 取模块默认装载基址
  DefaultBase:=GetModuleDefaultBase(ModuleBase);

  // 重定向总大小
  relocSize:=GetModuleOptionalHeader(ModuleBase).DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size;

  // 遍历所有重定向条目
  while relocSize > 0 do
  begin
    relocSize:=relocSize - reloc.SizeOfBlock;
    RvaBase:=reloc.VirtualAddress;
    relocNum:=(reloc.SizeOfBlock - SizeOf(TImageBaseReloc)) div 2;
    if relocNum = 0 then Break;

    for i := 0 to relocNum - 1 do
    begin
      w:=PWORD(Cardinal(reloc)+sizeof(TImageBaseReloc)+i*sizeof(WORD))^;
      if (w and $F000) shr 12 <> 3 then Continue;
      dw:=RvaBase + (w and $FFF);                                         //RVA
      if dw = 0 then Continue;

      if IsMapped = True then dw:=GetModuleRawFromRVA(ModuleBase, dw) - ModuleBase;

      va:=dw + ModuleBase;                                                //va
      dw:=PDWORD(va)^ - DefaultBase + NewBase;                            //重定向修正后的值
      PDWORD(va)^:=dw;                                                    //写入新值
    end;

    reloc:=PIMAGEBASERELOC(Cardinal(reloc)+reloc.SizeOfBlock);
    if reloc.VirtualAddress = 0 then Break;
    if reloc.SizeOfBlock = 0 then Break;
  end;

  Result:=True;
end;
```

具体就不多解释了,看雪的加密解密第三版中详细介绍了这些内容

重定向处理完之后需要首先检测IAT hook,检测IAT hook比较简单,我们只需要解析映射的PE文件的导入表定位到相应的偏移地址,
然后使用模块的内核基址加上这个偏移地址就能直接读取出导入表的当前值,然后使用`MmGetSystemRoutineAddress`得到真实值,
两个值比较一下就OK了,当然如果导入表中的不是`hal`或`ntoskrnl`中的函数,需要自己根据模块及函数名称在驱动中解析PE,
遍历一下对应模块的导出表内容,得到真实的地址,再进行比较

<上面这步需要驱动配合>

这部分就不列出来了,网上搜索也可以得到

检测完IAT hook开始扫描`.text`和`PAGE`两个节区

```
function ScanSection(SectionName:string; scanInfo:PScanHookInfo):Boolean;
var
  pSection    :PImageSectionHeader;
begin
  Result:=False;
  if SectionName = '' then Exit;
  if scanInfo = nil then Exit;

  // 获取指定节区信息
  pSection:=GetSpecialSectionInformation(scanInfo.MapModuleBase, PAnsiChar(AnsiString(SectionName)));
  if pSection <> nil then
  begin
    // 填充结构
    ZeroMemory(@scanInfo.SectionName, Length(scanInfo.SectionName)*sizeof(Char));
    MoveMemory(@scanInfo.SectionName, PChar(SectionName), Length(SectionName)*sizeof(Char));
    scanInfo.SectionHeader:=pSection;

    CheckInlineHook_Do(scanInfo);
  end;

  Result:=True;
end;
```
