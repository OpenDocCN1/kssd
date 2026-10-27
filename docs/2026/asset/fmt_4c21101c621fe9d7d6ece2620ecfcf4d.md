
# [原创]文件映射之地址随机化原理（Win10 x64）-“缓存”

## 题记

从Win7开始，创建进程的函数变为了`nt!NtCreateUserProcess`（R0），看了科锐的**《64位Windows创建64位进程逆向分析》**受到了一些启发，由于自己研究了内存管理很久，所以我想尽自己一点薄力（我不是科锐的...hhhh），填补这个过程中文件映射的部分，同时也填补我上一篇《XX之NTDLL随机化“逆向”（XP系统）》文章中所留下的一些空白。

重要的事情说三遍：**大佬勿喷、大佬勿喷、大佬勿喷!** 自己是个业余选手..hh..，为了写这篇文章，怕嘴闲着，去买了一扎葡萄28元!!! 我家的猪都不敢这么卖....

## 正文

由于是文件映射，所以必然少不了这几位主角：`FILE_OBJECT`、`SEGMENT`、`SECTION`、`CONTROL_AREA`，外加用户层访问内存的入口`VAD`（`BITMAP`就不提了它只是`VAD`的一种“缓存”方式）。由于上述文章中没有过多的提及关于文件映射的这块内容，所以我必须自己在`nt!NtCreateUserProcess`函数流程的海洋中寻找与这些对象相关的函数影子。

## 寻找相关流程思路

研究系统：由于头铁，所以直接拿了一个Win10 x64 1909或20HX的版本研究（搭建虚拟机太麻烦），所以我就以此版本为研究对象。

寻找相关函数的思路有两种：

1. **第一种**，对特定对象中的特定字段进行下断点（这种需要知道相关对象的创建时机），然后进行栈回溯查看周围的堆栈情况。
2. **第二种**，直接从头开始从下找，人肉筛选其重点函数（参考之前一些前辈的文章）。

我选择的是第二种，但是也引用了一些技巧。首先我对`nt!NtCreateUserProcess`函数进行下断点，断下后，使用`uf /c /D 地址`，查看此函数的的下一层的函数调用关系，然后进行人肉筛选一些不重要的函数。

`uf /c /D 地址`

结果如下图：

![](https://bbs.kanxue.com/upload/attach/202108/847490_DFKVY4V36RNCZWB.jpg)

筛选规则很简单，就是对一些解引用、内存操作、参数检查的函数一律PASS掉，因为我们的核心是**文件映射**相关（这里参考《64位Windows创建64位进程逆向分析》系列）。

经过筛选最终如下图（此时还没有动态调试哦~）：

![](https://bbs.kanxue.com/upload/attach/202108/847490_4WTZA8CEBG45CKG.jpg)

开始调试验证流程和探究其细节：

### 提出问题和猜想

由于在《XX之NTDLL随机化“逆向”（XP系统）》文章中我提到了，在Win7乃至Win10中，有3种情况：

1. 将一个exe程序重复启动，查看其基址

   第一次启动：

   ![](https://bbs.kanxue.com/upload/attach/202108/847490_YHHETTZANX9PQ53.jpg)

   在桌面移动一下坐标再次启动：

   ![](https://bbs.kanxue.com/upload/attach/202108/847490_CNJ88BXVAKEAMNH.jpg)

2. 将一个exe移动一下再回到原来位置启动，查看其基址。

   第一次启动：

   ![](https://bbs.kanxue.com/upload/attach/202108/847490_73QMNC5TR95PXT5.jpg)

   此时，我将这个exe移动（剪切的方式）到某个盘符下，再移动回来。

   ![](https://bbs.kanxue.com/upload/attach/202108/847490_HQBKQEKUR7534AS.jpg)

   很明显，此时，对于同一个软件这个基址就发生了变化。

3. 通过修改exe一些字节，进行重新运行

   没改之前运行：

   ![](https://bbs.kanxue.com/upload/attach/202108/847490_3SNYN9DTSJXYFCG.jpg)

   修改一些字节：

   ![](https://bbs.kanxue.com/upload/attach/202108/847490_Y9P4ZP92SXGUTC2.jpg)

   ![](https://bbs.kanxue.com/upload/attach/202108/847490_8AVHH6DTTRXNVM9.jpg)

   很明显，此时，exe在一个位置，但是其内容发生了变化，也会导致其基址改变。

通过这三个实验我想提出的观点是：对于**文件加载是存在“缓存”机制**的。而问题是如何进行缓存呢？这是下文开始探讨的问题。

## 调试相关流程分析

首先我给出各个对象的框架关系图：

![](https://bbs.kanxue.com/upload/attach/202108/847490_QU3WNZXVC2NX3WN.jpg)

接下来按照所过滤的函数流程开始逐层分析，验证上面图片中对象的关系和寻找如何进行“缓存”？

第一个分析的便是：`IoCreateFileEx`函数，该函数会生成一个`FILE_OBJECT`对象。通过分析可以得出`IopCreateFile`的第一个参数是`file_object`的`Handle`，所以只需跟踪即可。

![](https://bbs.kanxue.com/upload/attach/202108/847490_YRZCYGC4VP5K5ZW.jpg)

经过分析会得出以下结果：

![](https://bbs.kanxue.com/upload/attach/202108/847490_ZK6REWMQ2CK624C.jpg)

![](https://bbs.kanxue.com/upload/attach/202108/847490_W6K3GHBDQQQAT3R.jpg)

此时已经产生`FILE_OBEJCT`、`SEGMENT`、`CONTROL_AREA`对象。此时图形更新为：

![](https://bbs.kanxue.com/upload/attach/202108/847490_Q7U83Y86TDF2859.jpg)

由于此时还没有创建`EPROCESS`和`SECTION`对象，所以用户层是无法看到映射的内容的。所以重点就在于`nt!MmCreateSpecialImageSection`函数的身上。

我先简单的划分一下，可以看的更清楚一些，最终图如下：

![](https://bbs.kanxue.com/upload/attach/202108/847490_TPJMJY2P7H8VUUU.jpg)

由于我是直接运行了一个exe程序，所以必然流程走的是`MiCreateImageOrDataSection`函数。

```
struct CREATE_SECTION_PACKET

{

    ULONG Flags;

    DWORD Unknown04;

    POBJECT_ATTRIBUTES InputObjectAttributes;

    ULONG AllocateAttributes;

    ULONG InputAllocationAttributes;

    UCHAR InputSectionSignatureLevel;

    BYTE Unknown19;

    WORD Unknown1A;

    ULONG InputSectionPageProtection;

    ULONG PageProtectionMask;

    DWORD Unknown24;

    HANDLE InputFileHandle;

    PFILE_OBJECT InputFileObject;

    PFILE_OBJECT FileObject;

    CONTROL_AREA* SectionControlArea;

    KPROCESSOR_MODE InputPreviousMode;

    BYTE Unknown49[67];

    DWORD Unknown8C;

    SECTION* SectionObject;

    PLARGE_INTEGER MaximumSize;

    PACCESS_TOKEN InputToken;

    DWORD InputSessionId;

    DWORD UnknownAC;

    MI_PARTITION* Partition;

    PIRP TopLevelIrp;

    BYTE UnknownC0;

    BYTE UnknownC1[3];

    DWORD UnknownC4;

};



NTSTATUS __fastcall MiReferenceControlArea(

    CREATE_SECTION_PACKET* CreateSectionPacket,

    CONTROL_AREA* ControlArea,

    CONTROL_AREA** ControlAreaOut)

{

    CONTROL_AREA* controlArea;

//...

    fileObject = CreateSectionPacket->FileObject;

	/*

		检索Section Object指针。 如果 SEC_IMAGE 使用 ImageSectionObject 否则使用 DataSectionObject

	*/

    controlArea = fileObject->SectionObjectPointer->DataSectionObject;

    if ((CreateSectionPacket->AllocateAttributes & SEC_IMAGE) != 0)

    {

        controlArea = fileObject->SectionObjectPointer->ImageSectionObject;

    }

//...

//

// 一些非常丑陋的锁循环和验证。

//

//...

            *ControlAreaOut = controlArea;

            return STATUS_SUCCESS;

//...

}





NTSTATUS __fastcall MiCreateImageOrDataSection(

    CREATE_SECTION_PACKET* CreateSectionPacket)

{

    NTSTATUS status;

    PFILE_OBJECT fileObject;

    CONTROL_AREA controlArea;

    CONTROL_AREA* newControlArea;

//...

    fileObject = CreateSectionPacket->InputFileObject;

    if (fileObject)

    {

        //

		// 已经提供了文件对象，请使用它。

        //

        goto HaveFileObject;

    }

    if ((allocationAttributes & SEC_LARGE_PAGES) == 0)

    {

        //

		// 从输入文件句柄获取文件对象。  

        //

        status = ObReferenceObjectByHandle(

                     CreateSectionPacket->InputFileHandle,

                     MmMakeFileAccess[CreateSectionPacket->PageProtectionMask & 7],

                     IoFileObjectType,

                     CreateSectionPacket->InputPreviousMode,

                     &fileObject,

                     NULL);

        if (!NT_SUCCESS(status))

        {

            goto Exit;

        }

        if (!fileObject->SectionObjectPointer)

        {

            //

			// 如果使用了文件句柄并且没有为它创建的节，这是一个失败条件。

            //

            status = STATUS_INVALID_FILE_FOR_SECTION;

            goto Exit;

        }



:HaveFileObject

//...

        //

		// 在数据包和本地 CONTROL_AREA 中存储一些信息以维护状态以供进一步调用。

        //

        ObfReferenceObject(fileObject);

        CreateSectionPacket->FileObject = fileObject;

        controlArea.u.LongFlags = 2;

        controlArea.FilePointer.Value = fileObject;

        newControlArea = NULL;

//...

        while (1)

        {

//...

            //

            // Go reference the correct control area.

			// 去参考正确的控制区域。

            //

            status = MiReferenceControlArea(CreateSectionPacket, 

                                            &controlArea, 

                                            &sectionControlArea);

            if (NT_SUCCESS(status))

            {

                break;

            }

            if ((status == 0xC000060B) || (status == 0xC0000476))

            {

                //

                // The control area is not charged or is invalid.

				// 这个控制区域已经无效

                //

                goto Exit;

            }

        }

        CreateSectionPacket->SectionControlArea = sectionControlArea;

        if ((sectionControlArea->u.LongFlags & 2) != 0)

        {

            //

			// 我们有section控制区域，其中将包含参考部分。 现在，去创建一个新的。

            //

            status = MiCreateNewSection(CreateSectionPacket,

                                        &newControlArea);

            if (NT_SUCCESS(status)))

            {

//...

                CreateSectionPacket->SectionControlArea = newControlArea;

                goto Exit;

//...

Exit:

//...

    return status;

}
```

伪代码参考：https[:]//github.com/jxy-s/herpaderping/blob/main/res/DivingDeeper.md

![](https://bbs.kanxue.com/upload/attach/202108/847490_P3D49Q2J38B2V3W.jpg)

首先会判断是否存在`FILE_OBEJCT`对象，如果有的话直接跳转，不需要再通过输入的**句柄**获取其`FILE_OBJECT`对象（没有的话，需要走这一步）。

![](https://bbs.kanxue.com/upload/attach/202108/847490_EXAVDJF9Q8WJG2H.jpg)

其次默认设置一些标志已经字段，但是最重要的是`BeingCreated`标志位。

![](https://bbs.kanxue.com/upload/attach/202108/847490_HWEZ683YSYV4HN3.jpg)

后续会根据此标志位(`BeingCreated = 1`)来判断是否创建新的`CONTROL_AREA`。此时也会修改掉`_SEGMENT.BaseAddress`字段，所以这里就不使用缓存。

![](https://bbs.kanxue.com/upload/attach/202108/847490_JPJTG5U9HCF9FGK.jpg)
