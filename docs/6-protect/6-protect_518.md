

# 学习时写的壳代码

> 原文：[`www.pediy.com/kssd/pediy10/77260.html`](https://www.pediy.com/kssd/pediy10/77260.html)

学习时写的壳代码，太简陋了。而且还是个半成品。  
高手飘过~~~~~  
很多参考过看雪的内容，回馈给看雪。  
希望能对像我这样的初学者有所帮助。  
可以提出您友善的意见。  
千万别打击小弟幼小的心灵。![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色")  
  
加密代码段：  

代码:

```
  file\_opt.hFile=CreateFile(szFileName,GENERIC\_READ | GENERIC\_WRITE,
    FILE\_SHARE\_READ,NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,0);

  FileSize=GetFileSize(file\_opt.hFile,NULL);

  ReadFile(file\_opt.hFile,&DosHdr,sizeof(IMAGE\_DOS\_HEADER),&NumRW,0);

  SetFilePointer(file\_opt.hFile,DosHdr.e\_lfanew,0,FILE\_BEGIN);
  ReadFile(file\_opt.hFile,&NtHdr,sizeof(IMAGE\_NT\_HEADERS),&NumRW,0);

  SetFilePointer(file\_opt.hFile,
    (DosHdr.e\_lfanew+sizeof(IMAGE\_NT\_HEADERS)),
    0,
    FILE\_BEGIN);
  ReadFile(file\_opt.hFile,&SecHdr,sizeof(IMAGE\_SECTION\_HEADER),&NumRW,0);

  //::MessageBox(NULL,(const char \*)SecHdr.Name,NULL,MB\_OK);
  if(!lstrcmp((const char \*)SecHdr.Name,".text"))
  {
    SecHdr.Characteristics|=IMAGE\_SCN\_MEM\_READ | IMAGE\_SCN\_MEM\_WRITE;  //修改.text 的属性
    SetFilePointer(file\_opt.hFile,
      (DosHdr.e\_lfanew+sizeof(IMAGE\_NT\_HEADERS)),
      0,
      FILE\_BEGIN);
    WriteFile(file\_opt.hFile,&SecHdr,sizeof(IMAGE\_SECTION\_HEADER),&NumRW,0);
    SetFilePointer(file\_opt.hFile,SecHdr.PointerToRawData,0,FILE\_BEGIN);
    TextSize=SecHdr.SizeOfRawData;
    pc=new char\[TextSize\];
    memset((void \*)pc,0,TextSize);
    ReadFile(file\_opt.hFile,pc,TextSize,&NumRW,0);
    p=pc;
    
    //加密部分与 0x13 进行异或
    for(i=0;i<TextSize;i++)
    {
      \*p^=0x21;
      SetFilePointer(file\_opt.hFile,SecHdr.PointerToRawData+i,0,FILE\_BEGIN);
      WriteFile(file\_opt.hFile,p,1,&NumRW,0);
      p++;
    }
    InsertMsg("成功！代码加密成功!");

    delete \[\] pc;
  }
  else
  {
    InsertMsg("失败！加壳失败!");
  }

  CloseHandle(file\_opt.hFile);

```

生成不同的节的名称  
代码:

```
NewSectionName  db  '.aspack',0
    db  '.vmp   ',0
    db  '.upx0  ',0
    db  'INIT   ',0
    db  '.mackt ',0
    db  '.upx1  ',0
    db  '       ',0
    db  'snyped ',0
    db  '.Katja ',0
    db  '.jtsnew',0
    db  '.SuCop0',0

        invoke  GetTickCount   
  xor  edx,edx   
  mov  ecx,0Bh
  div  ecx
  mov  eax,edx
  mov  ebx,08h
  mul  ebx
  mov  ebx,SectionName
  add  ebx,eax
  mov  SectionName,ebx

```

anti OD:  
代码:

```
                push  0ABDBFFDEh
  pop  \[esi\].OptionalHeader.LoaderFlags
  
  push  0DFFFDDDEh
  pop  \[esi\].OptionalHeader.NumberOfRvaAndSizes

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 简单壳代码.rar  
 |