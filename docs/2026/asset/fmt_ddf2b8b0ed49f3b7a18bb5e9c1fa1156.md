
# [分享]学习时写的壳代码

学习时写的壳代码，太简陋了。而且还是个半成品。

高手飘过~~~~~

很多参考过看雪的内容，回馈给看雪。

希望能对像我这样的初学者有所帮助。

可以提出您友善的意见。

千万别打击小弟幼小的心灵。![](https://bbs.kanxue.com/view/img/face/41.gif)

## 加密代码段

```
	file_opt.hFile=CreateFile(szFileName,GENERIC_READ | GENERIC_WRITE,

		FILE_SHARE_READ,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0);



	FileSize=GetFileSize(file_opt.hFile,NULL);



	ReadFile(file_opt.hFile,&DosHdr,sizeof(IMAGE_DOS_HEADER),&NumRW,0);



	SetFilePointer(file_opt.hFile,DosHdr.e_lfanew,0,FILE_BEGIN);

	ReadFile(file_opt.hFile,&NtHdr,sizeof(IMAGE_NT_HEADERS),&NumRW,0);



	SetFilePointer(file_opt.hFile,

		(DosHdr.e_lfanew+sizeof(IMAGE_NT_HEADERS)),

		0,

		FILE_BEGIN);

	ReadFile(file_opt.hFile,&SecHdr,sizeof(IMAGE_SECTION_HEADER),&NumRW,0);



	//::MessageBox(NULL,(const char *)SecHdr.Name,NULL,MB_OK);

	if(!lstrcmp((const char *)SecHdr.Name,".text"))

	{

		SecHdr.Characteristics|=IMAGE_SCN_MEM_READ | IMAGE_SCN_MEM_WRITE;	//修改.text的属性

		SetFilePointer(file_opt.hFile,

			(DosHdr.e_lfanew+sizeof(IMAGE_NT_HEADERS)),

			0,

			FILE_BEGIN);

		WriteFile(file_opt.hFile,&SecHdr,sizeof(IMAGE_SECTION_HEADER),&NumRW,0);

		SetFilePointer(file_opt.hFile,SecHdr.PointerToRawData,0,FILE_BEGIN);

		TextSize=SecHdr.SizeOfRawData;

		pc=new char[TextSize];

		memset((void *)pc,0,TextSize);

		ReadFile(file_opt.hFile,pc,TextSize,&NumRW,0);

		p=pc;

		

		//加密部分与0x13进行异或

		for(i=0;i<TextSize;i++)

		{

			*p^=0x21;

			SetFilePointer(file_opt.hFile,SecHdr.PointerToRawData+i,0,FILE_BEGIN);

			WriteFile(file_opt.hFile,p,1,&NumRW,0);

			p++;

		}

		InsertMsg("成功！代码加密成功!");



		delete [] pc;

	}

	else

	{

		InsertMsg("失败！加壳失败!");

	}



	CloseHandle(file_opt.hFile);
```

## 生成不同的节的名称

```
NewSectionName	db	'.aspack',0

		db	'.vmp   ',0

		db	'.upx0  ',0

		db	'INIT   ',0

		db	'.mackt ',0

		db	'.upx1  ',0

		db	'       ',0

		db	'snyped ',0

		db	'.Katja ',0

		db	'.jtsnew',0

		db	'.SuCop0',0



        invoke	GetTickCount   

	xor	edx,edx   

	mov	ecx,0Bh

	div	ecx

	mov	eax,edx

	mov	ebx,08h

	mul	ebx

	mov	ebx,SectionName

	add	ebx,eax

	mov	SectionName,ebx
```

## anti OD

```
                push	0ABDBFFDEh

	pop	[esi].OptionalHeader.LoaderFlags

	

	push	0DFFFDDDEh

	pop	[esi].OptionalHeader.NumberOfRvaAndSizes
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [简单壳代码.rar](javascript:void(0);)

  （43.03kb，194次下载）
