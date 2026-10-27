
### 5.原始资源目录表

### 6.原始加载基址

### 7.原始入口点

## 区段的融合

按照上述方法压缩后的文件，区段会比原始文件多。那么我们可以将已经压缩的区段统一的融合到一起，将区段按照，可压缩区段、不能压缩区段，外壳代码区段分开来。这里的融合，不能够移动已经压缩好区段了，因为之前的所有地址已经保存，后期外壳的装载都要通过这些地址进行定位，移动会引起偏移的错误。这里的融合仅仅是告诉系统，压缩后的pe区段数要减少而已。表面上区段数确实“减少了”，实际上，对于装载程序定位地址来说，并没有实质的改变。

可以将第一个压缩区段的虚拟地址和磁盘地址作为所可压缩区段的虚拟地址和磁盘地址，并将所有的可压缩区段的虚拟大小相加，磁盘大小相加，分别作为可压缩区段的虚拟大小和磁盘大小。修改节表头的相关节名，删除多余的节表头即可。加密与解密三中15章处理区段有详细代码，可以参考。至此，所有在内存加载的数据已经全部填充完毕了，现在只要依次将载入内存中的数据，保存到磁盘中即可。

## 外壳loader处理

外壳`loader`要面对的就是已经压缩好的pe文件，它的任务就是一边解压pe文件，同时模拟系统laoder加载PE到内存中去，完成任务之后将保存的原始PE信息写回到已经加载内存中的pe文件中去，跳到原始`oep`执行。

这里外壳代码段可以用`c`来实现的，然后通过反汇编二进制拷贝到外壳代码区段中。这里外壳代码主要是处理2个地方：

**1. 恢复原始导入表数据，传递资源目录`RESOURCE_VA`，linxer版主给出过详细源码：**

```
IMAGE_IMPORT_DESCRIPTOR *pIID =（PIMAGE_IMPORT_DESCRIPTOR）RESOURCE_VA;

for(; pIID->Name != NULL; pIID++)
    {
      //直接定位FirstThunk即可
      IMAGE_THUNK_DATA *pITD = (IMAGE_THUNK_DATA *)(pMemPointer + pIID->FirstThunk);
      //调用LoadLibraryA载入DLL
      HINSTANCE hInstance = LoadLibraryA(pMemPointer + pIID->Name);

      for(; pITD->u1.Ordinal != 0; pITD++)
      {
        FARPROC fpFun;
        if(pITD->u1.Ordinal & IMAGE_ORDINAL_FLAG32)
        {
          ///函数是以序号的方式导入的
          fpFun = GetProcAddress(hInstance, (LPCSTR)(pITD->u1.Ordinal & 0x0000ffff));
        }
        else
        { //函数是以名称方式导入的
          IMAGE_IMPORT_BY_NAME * pIIBN = (IMAGE_IMPORT_BY_NAME *)(pITD->u1.Ordinal);
          fpFun = GetProcAddress(hInstance, (LPCSTR)pIIBN->Name);
        }

        if(fpFun == NULL)
        {
          return false;
        }
        pITD->u1.Ordinal = (long)fpFun;
      }
    }
  }
```

**2. 重定位代码的修复，**

要修复重定位，我们应该知道实际载入地址，期待载入地址，和修复数据，后两者都定位在PE中了，那么我们进行修复工作，还需要知道实际载入地址。获得实际载入地址，通过栈的机制。

```
pushad    //保存环境  

		call Getaddr 

Getaddr:

		pop eax    //当前代码地址就保持在eax中了，以后的寻址可以以此为标杆地址
```

通过`eax`作为的标杆地址，又知道dll加载地址为入口代码最顶层。例如执行了`pushad`，`esp`增长32字节，未增长前，模块地址是`[esp+4]`, 现在应该是`[esp+24]`。以此可以获得代码运行的实际地址保存在`[esp+24]`，将其通过`dwInfactBase`存储，知道了实际载入地址，传递重定位目录参数`pRelocateTable`下面就可以进行修复工作了。

```
while(pRelocateTable->VirtualAddress != NULL)

 	{      //定位到数据处

 		parrOffset = (WORD*)((PBYTE)pRelocateTable + sizeof(IMAGE_BASE_RELOCATION));

              //重定位数据个数，每一个数据是word类型

 		dwRvaCount = (pRelocateTable->SizeOfBlock - sizeof(IMAGE_BASE_RELOCATION)) / 2;

 		for(DWORD i=0; i<dwRvaCount; i++)

 		{

 			dwRva = parrOffset[i];

                        //最高位nType != 3,不作处理，因为PE常见3或者0的类型

 			if((dwRva & 0xf000) !=  0x3000)

 			{

 				continue;

 			}

 			dwRva &= 0x0fff;

 			dwRva = (DWORD)(dwInfactBase + pRelocateTable->VirtualAddress + dwRva);

 			*(DWORD*)dwRva += dwRelocOffset;

 		}

                //指向下一个重定位块

 		pRelocateTable = (PIMAGE_BASE_RELOCATION)((PBYTE)pRelocateTable + pRelocateTable->SizeOfBlock);

 	}
```

当所有的`loader`模拟工作完成之后，在内存呈现的PE布局应该就如下图所示：

![](https://bbs.kanxue.com/upload/attach/201202/374528_6j2gltt825ii1nm.png)

处理完之后恢复堆栈平衡，通过之前保存的原始文件的`OEP`值，通过一个`jmp` `OEP`即跳到原始文件入口点。至此，一个简单压缩壳就完成了它的使命：）

后记：本文形成参考了很多书籍、大牛的代码和实例以及很多开源壳的代码，感谢前辈无私的奉献。上述成文都是自己学习压缩壳的一些心得笔记和体会。小弟水平有限，很多细节没有涉及到，而这也恰恰是完成一个压缩壳重要的因素，所有推荐想了解的朋友多读几遍开源壳源码，可能会收获更多：）上文肯定有不足和错误之处，没有什么技术含量，希望各位能多多的指点。最后，很感谢大家耐心的看完：）

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [无标题.png](javascript:void(0);)

  （22.34kb，34次下载）

- [无标题1.png](javascript:void(0);)

  （19.71kb，24次下载）
