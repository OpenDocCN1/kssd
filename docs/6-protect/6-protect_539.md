

# 加壳原理与简单实现加壳（delphi 源码）

> 原文：[`www.pediy.com/kssd/pediy10/57671.html`](https://www.pediy.com/kssd/pediy10/57671.html)

\[code\]{\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AddShell（）源自于前一段时间有写的 addsection（）新增区段代码，  
在增加区段代码的基础上，追加了  
1.修改启动入口点位置  
2.增加一段壳头 xor $50 的代码 function AttachStart-function AttachEnd  
 这一段代码是先填充，再被修改成合适原 EXE 的壳头  
3.修改原启动代码入口点所在区段的段属性可写并进行 xor $50 运算加密  
  
  
不支持 addshell（）处理已经过 addshell 的 exe  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
  
unit Unit1;  
  
interface  
  
uses  
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,  
  Dialogs, StdCtrls;  
  
type  
  TForm1 = class(TForm)  
    Button1: TButton;  
    Edit1: TEdit;  
    Button2: TButton;  
    Button3: TButton;  
    procedure Button1Click(Sender: TObject);  
    procedure Button3Click(Sender: TObject);  
    procedure Button2Click(Sender: TObject);  
  private  
    { Private declarations }  
  public  
    { Public declarations }  
  end;  
  
var  
  Form1             : TForm1;  
  
implementation  
  
{$R \*.dfm}  
function AttachStart: dword; stdcall;   //我们定义的待填充数据  
asm  
       pushfd  
       pushad  
       mov eax,$12345678      //将会被自动计算并修改为加密初始地址  
       mov ebx,$1234          //将会被自动计算并修改为加密大小  
       mov ecx,0  
       @AA:  
       xor byte ptr\[eax\],$50  
       inc eax  
       inc ecx  
       cmp ecx,ebx  
       jbe @aa  
       popad  
       popfd  
       push $12345678        //将会被自动计算并修改为原 OEP  
       ret  
end;  
  
function AttachEnd: dword; stdcall;  
begin  
end;  
  
{-------------------------增加区块并实现简易加壳--------------------------------}  
procedure AddShell(lFileName: string; lBackup: boolean); //打开 exe 文件,是否备份  
var  
  hFile             : THandle;          //文件句柄  
  ImageDosHeader    : IMAGE\_DOS\_HEADER; //DOS 部首  
  ImageNtHeaders    : IMAGE\_NT\_HEADERS; //映象头  
  ImageSectionHeader: IMAGE\_SECTION\_HEADER; //块表  
  lPointerToRawData : dword;            //指向文件中的偏移  
  lVirtualAddress   : dword;            //指向内存中的偏移  
  i                 : integer;          //循环变量  
  BytesRead, ByteSWrite: Cardinal;      //读写用参数  
  AttachSize        : dword;            //附加段大小  
  AttachData, ChangeData: integer;      //附加段填充数据  
  OEP               : integer;          //使用过程中用到的 EP  
  lpBuffer          : array\[0..1024 \* 400\] of byte; {待加密数据存储缓冲区}  
  nNumberOfBytesToRead, lpNumberOfBytesRead: dword; //加密时用到的一些数据  
  StartEN, SizeEN, StartCr: dword;      //加密用的开始物理地址和大小  
begin  
  
  //定义附加段填充数据  
  AttachData := 0;  
  
  //打开文件  
  hFile := CreateFile(PChar(lFileName), GENERIC\_READ or GENERIC\_WRITE, FILE\_SHARE\_READ or FILE\_SHARE\_WRITE, nil, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, 0);  
  
  //校验  
  if hFile = INVALID\_HANDLE\_VALUE then  
  begin  
    ShowMessage('打开文件失败');  
    exit;  
  end;  
  
  //确认备份  
  if lBackup then CopyFile(PChar(lFileName), PChar(lFileName + '.bak'), False);  
  try  
  
    //读取 DOS 部首到 ImageDosHeader  
    ReadFile(hFile, ImageDosHeader, SizeOf(ImageDosHeader), BytesRead, nil);  
  
    //校验  
    if ImageDosHeader.e\_magic <> IMAGE\_DOS\_SIGNATURE then  
    begin  
      ShowMessage('不是有效的 PE 文件!');  
      exit;  
    end;  
  
    //指向映象头  
    SetFilePointer(hFile, ImageDosHeader.\_lfanew, nil, FILE\_BEGIN);  
  
    //读取映向头到 ImageNtHeaders  
    ReadFile(hFile, ImageNtHeaders, SizeOf(ImageNtHeaders), BytesRead, nil);  
  
    //校验  
    if ImageNtHeaders.Signature <> IMAGE\_NT\_SIGNATURE then  
    begin  
      ShowMessage('不是有效的 PE 文件');  
      exit;  
    end;  
    {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
    {OEP=基址+原 EP}  
    OEP := ImageNtHeaders.OptionalHeader.ImageBase + ImageNtHeaders.OptionalHeader.AddressOfEntryPoint;  
  
    {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
  
    //计算加入块对齐后大小  
    AttachSize := ((integer(@AttachEnd) - integer(@AttachStart)) div ImageNtHeaders.OptionalHeader.FileAlignment + 1) \* ImageNtHeaders.OptionalHeader.FileAlignment;  
  
    //初始化文件中偏移和映象中偏移  
    lPointerToRawData := 0;  
    lVirtualAddress := 0;  
  
    for i := 0 to ImageNtHeaders.FileHeader.NumberOfSections - 1 do  
    begin  
  
      //读取块表中信息  
      ReadFile(hFile, ImageSectionHeader, SizeOf(ImageSectionHeader), BytesRead, nil);  
  
      {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
       {查找原 EP 所在区段(原 EP 所在区段)，记录物理偏移（加密用初始地址），物理大小（加密用长度）}  
      if LPCSTR(@ImageSectionHeader.Name\[0\]) = '.EN' then  
      begin  
        ShowMessage('已经过本 addshell 处理！');  
        exit;  
      end;  
  
      if ImageNtHeaders.OptionalHeader.AddressOfEntryPoint > ImageSectionHeader.VirtualAddress then  
      begin  
        StartEN := ImageSectionHeader.PointerToRawData;  
        SizeEN := ImageSectionHeader.SizeOfRawData;  
        StartCr := ImageNtHeaders.OptionalHeader.ImageBase + ImageSectionHeader.VirtualAddress;  
      end;  
  
      {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
  
      //计算文件中偏移  
      if lPointerToRawData < ImageSectionHeader.PointerToRawData + ImageSectionHeader.SizeOfRawData then  
        lPointerToRawData := ImageSectionHeader.PointerToRawData + ImageSectionHeader.SizeOfRawData;  
  
      //计算映象中偏移  
      if lVirtualAddress < ImageSectionHeader.VirtualAddress + ImageSectionHeader.Misc.VirtualSize then  
        lVirtualAddress := ImageSectionHeader.VirtualAddress + ImageSectionHeader.Misc.VirtualSize;  
    end;  
  
    {增加块,定义块各项属性}  
  
    Move('.EN'#0, ImageSectionHeader.Name\[0\], 5);  
  
    //设置初始属性  
    ImageSectionHeader.Misc.VirtualSize := AttachSize;  
    ImageSectionHeader.VirtualAddress := lVirtualAddress;  
    ImageSectionHeader.SizeOfRawData := AttachSize;  
    ImageSectionHeader.PointerToRawData := lPointerToRawData;  
    ImageSectionHeader.PointerToRelocations := 0;  
    ImageSectionHeader.PointerToLinenumbers := 0;  
    ImageSectionHeader.NumberOfRelocations := 0;  
  
    //校正新节物理偏移(物理区块对齐)  
    if ImageSectionHeader.VirtualAddress mod ImageNtHeaders.OptionalHeader.SectionAlignment > 0 then  
      ImageSectionHeader.VirtualAddress := (ImageSectionHeader.VirtualAddress div ImageNtHeaders.OptionalHeader.SectionAlignment + 1) \* ImageNtHeaders.OptionalHeader.SectionAlignment;  
  
    //校正新节映象偏移(映象中区块对齐)  
    if ImageSectionHeader.Misc.VirtualSize mod ImageNtHeaders.OptionalHeader.SectionAlignment > 0 then  
      ImageSectionHeader.Misc.VirtualSize := (ImageSectionHeader.Misc.VirtualSize div ImageNtHeaders.OptionalHeader.SectionAlignment + 1) \* ImageNtHeaders.OptionalHeader.SectionAlignment;  
  
    //设置区块属性  
    ImageSectionHeader.Characteristics := $E00000E0;  
  
    //保存区块信息  
    WriteFile(hFile, ImageSectionHeader, SizeOf(ImageSectionHeader), ByteSWrite, nil);  
  
    //校正内存映象大小  
    ImageNtHeaders.OptionalHeader.SizeOfImage := ImageNtHeaders.OptionalHeader.SizeOfImage + ImageSectionHeader.Misc.VirtualSize;  
    //更新 OEP  
    ImageNtHeaders.OptionalHeader.AddressOfEntryPoint := ImageSectionHeader.VirtualAddress;  
  
    //校正块数目  
    Inc(ImageNtHeaders.FileHeader.NumberOfSections);  
  
    //定位到映象头  
    SetFilePointer(hFile, ImageDosHeader.\_lfanew, nil, FILE\_BEGIN);  
  
    //保存校正过的映象头  
    WriteFile(hFile, ImageNtHeaders, SizeOf(ImageNtHeaders), ByteSWrite, nil);  
  
    //定位到新节开始处  
    SetFilePointer(hFile, ImageSectionHeader.PointerToRawData, nil, FILE\_BEGIN);  
  
    //用 00 数据填充满新节  
    for i := 1 to AttachSize do  
    begin  
      WriteFile(hFile, PByte(@AttachData)^, 1, ByteSWrite, nil);  
    end;  
  
    {填充自定义数据}  
      //指向新节开始处  
    SetFilePointer(hFile, ImageSectionHeader.PointerToRawData, nil, FILE\_BEGIN);  
  
    //填充我们定义的数据  
    WriteFile(hFile, PByte(@AttachStart)^, integer(@AttachEnd) - integer(@AttachStart), ByteSWrite, nil);  
    {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
    //修改所谓的外壳处大量数据  
    ChangeData := ImageSectionHeader.PointerToRawData + 3;  
    SetFilePointer(hFile, ChangeData, nil, FILE\_BEGIN);  
    WriteFile(hFile, StartCr, 4, ByteSWrite, nil); //开始加密地址  
    ChangeData := ChangeData + 5;  
    SetFilePointer(hFile, ChangeData, nil, FILE\_BEGIN);  
    WriteFile(hFile, SizeEN, 4, ByteSWrite, nil); //大小  
    ChangeData := ChangeData + 21;  
    SetFilePointer(hFile, ChangeData, nil, FILE\_BEGIN);  
    WriteFile(hFile, OEP, 4, ByteSWrite, nil); //跳回 OEP  
  
    {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
    //没有异常,显示增加区块成功!  
    ShowMessage('增加区块成功!');  
  
  
    SetFilePointer(hFile, StartEN, nil, FILE\_BEGIN);  
    ReadFile(hFile, lpBuffer, SizeEN, BytesRead, nil);  
    for i := 0 to SizeEN - 1 do  
    begin  
      byte(pointer(integer(@lpBuffer) + i)^) := byte(pointer(integer(@lpBuffer) + i)^) xor $50;  
    end;  
    SetFilePointer(hFile, StartEN, nil, FILE\_BEGIN);  
    WriteFile(hFile, lpBuffer, SizeEN, ByteSWrite, nil);  
    //没有异常,显示变换成功!  
    ShowMessage('变换数据成功!');  
  
  
  
    {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
    SetFilePointer(hFile, (ImageDosHeader.\_lfanew + SizeOf(ImageNtHeaders)), nil, FILE\_BEGIN);  
  
    for i := 0 to ImageNtHeaders.FileHeader.NumberOfSections - 1 do  
    begin  
  
      //读取块表中信息  
      ReadFile(hFile, ImageSectionHeader, SizeOf(ImageSectionHeader), BytesRead, nil);  
      if ImageSectionHeader.PointerToRawData = StartEN then  
      begin  
        ImageSectionHeader.Characteristics := $E00000E0;  
        SetFilePointer(hFile, -SizeOf(ImageSectionHeader), nil, FILE\_CURRENT);  
        //保存区块信息  
        WriteFile(hFile, ImageSectionHeader, SizeOf(ImageSectionHeader), ByteSWrite, nil);  
        Break;  
      end;  
    end;  
    ShowMessage('修改区段属性成功!');  
    {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
  
  
  finally  
  
    {8.退出}  
        //关闭文件  
    CloseHandle(hFile);  
  end;  
  
end;  
{\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*Func end\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}  
  
procedure TForm1.Button1Click(Sender: TObject);//addshell  
begin  
  
  AddShell(Edit1.text, true);  
end;  
  
procedure TForm1.Button3Click(Sender: TObject);//browser  
begin  
  with TOpenDialog.Create(Self) do  
  try  
    Filter := '可执行文件 (\*.exe)|\*.exe';  
    if Execute then  
    begin  
      Edit1.text := FileName;  
    end;  
  finally  
    Free;  
  end;  
end;  
  
procedure TForm1.Button2Click(Sender: TObject);//exit  
begin  
  close;  
end;  
  
end.  
  
\[code\]