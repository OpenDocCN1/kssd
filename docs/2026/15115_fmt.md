
# [原创]百度加固逆向分析—dex还原

上回说过要再写一篇文章，这里跟大家分享一下百度壳DEX的dump与修复。

下面开始：

## 一、如何获取dex

首先，我们知道动态加载的dex必然会调用`dalvik/vm/DvmDex.cpp`中以下两个函数任意一个：

- `dvmDexFileOpenFromFd`：从文件描述符获取`DexFile`结构体
- `dvmDexFileOpenPartial`：从内存获取`DexFile`结构体

百度这里用的是`dvmDexFileOpenFromFd`，通过这个函数我们得到了`pDexFile`。

```
int dvmDexFileOpenFromFd(int fd, DvmDex** ppDvmDex)
{

    DvmDex* pDvmDex;

    DexFile* pDexFile;

    MemMapping memMap;

    int parseFlags = kDexParseDefault;

    int result = -1;

    if (gDvm.verifyDexChecksum)

        parseFlags |= kDexParseVerifyChecksum;

    if (lseek(fd, 0, SEEK_SET) < 0) {

        ALOGE("lseek rewind failed");

        goto bail;

    }

    if (sysMapFileInShmemWritableReadOnly(fd, &memMap) != 0) {

        ALOGE("Unable to map file");

        goto bail;

    }

    pDexFile = dexFileParse((u1*)memMap.addr, memMap.length, parseFlags);//这里获取了pDexFile

    if (pDexFile == NULL) {

        ALOGE("DEX parse failed");

        sysReleaseShmem(&memMap);

        goto bail;

    }

    pDvmDex = allocateAuxStructures(pDexFile);

    if (pDvmDex == NULL) {

        dexFileFree(pDexFile);

        sysReleaseShmem(&memMap);

        goto bail;

    }

    /* tuck this into the DexFile so it gets released later */

    sysCopyMap(&pDvmDex->memMap, &memMap);

    pDvmDex->isMappedReadOnly = true;

    *ppDvmDex = pDvmDex;

    result = 0;

bail:

    return result;

}
```

```
struct DexFile {

    /* directly-mapped "opt" header */

    const DexOptHeader* pOptHeader;

    /* pointers to directly-mapped structs and arrays in base DEX */

    const DexHeader*    pHeader;

    const DexStringId*  pStringIds;

    const DexTypeId*    pTypeIds;

    const DexFieldId*   pFieldIds;

    const DexMethodId*  pMethodIds;

    const DexProtoId*   pProtoIds;

    const DexClassDef*  pClassDefs;

    const DexLink*      pLinkData;

    /*

     * These are mapped out of the "auxillary" section, and may not be

     * included in the file.

     */

    const DexClassLookup* pClassLookup;

    const void*         pRegisterMapPool;       // RegisterMapClassPool

    /* points to start of DEX file data */

    const u1*           baseAddr;

    /* track memory overhead for auxillary structures */

    int                 overhead;

    /* additional app-specific data structures associated with the DEX */

    //void*               auxData;

};
```

通过`pDexFile->baseAddr`获取到dex加载的基址。

```
struct DexHeader {

    u1  magic[8];           /* includes version number */

    u4  checksum;           /* adler32 checksum */

    u1  signature[kSHA1DigestLen]; /* SHA-1 hash */

    u4  fileSize;           /* length of entire file */

    u4  headerSize;         /* offset to start of next section */

    u4  endianTag;

    u4  linkSize;

    u4  linkOff;

    u4  mapOff;

    u4  stringIdsSize;

    u4  stringIdsOff;

    u4  typeIdsSize;

    u4  typeIdsOff;

    u4  protoIdsSize;

    u4  protoIdsOff;

    u4  fieldIdsSize;

    u4  fieldIdsOff;

    u4  methodIdsSize;

    u4  methodIdsOff;

    u4  classDefsSize;

    u4  classDefsOff;

    u4  dataSize;

    u4  dataOff;

};
```

通过`pDexFile->pHeader->fileSize`获取到dex文件大小。

```
int fd = open("/sdcard/dump.dex", O_CREAT | O_WRONLY, 0666);

write(fd, pDexFile->baseAddr, pDexFile->pHeader->fileSize);

close(fd);
```

这时我们已经得到dex了。

## 二、百度壳对dex做了什么？

1、修改`DexClassDef`中的`classDataOff`字段保存的偏移为负偏移

![](https://bbs.kanxue.com/upload/attach/201706/638330_6njuqe21ttztlgc.png)

2、将`classdata`数据清空

![](https://bbs.kanxue.com/upload/attach/201706/638330_b3o4p06rixw66j2.png)

## 三、这么做如何让系统正常解析？

百度的把`classdata`的数据保存在`/data/data/xxxx/.1/1.jar`包中，会在加载dex之前先分配空间给jar包，所以他的偏移为负值，内存结构如下图：

![](https://bbs.kanxue.com/upload/attach/201706/638330_jmfiberpn858bbe.png)

## 四、还原DEX

1、获取到`pDexFile`后，我们遍历`pDexFile->pClassDefs`调用`dexGetClassData`获取到`ClassData`。将`ClassData`经过`writeLeb128`函数编码，写入到文件`classdata`并记录每个class的大小（此处参考Dexhunter做法）

```
   #define log(...) \

        {FILE *fp = fopen("/sdcard/dumpdex.log", "a+"); if (fp) {\

        fprintf(fp, __VA_ARGS__);\

        fclose(fp);}}

   //因为log被hook了所以用写文件的形式保存log

   void dump()      

   {

        const DexClassDef* pClassDefs = pDexFile->pClassDefs;

	u4 classDefsSize = pDexFile->pHeader->classDefsSize;

	DexMapList* pMaps = (DexMapList*)(g_pDexFile->baseAddr + pDexFile->pHeader->mapOff);

	u4 cls_dat_off = 0;

	for (u4 i = 0; i < pMaps->size; i++)

	{

		if (pMaps->list[i].type == 0x2000)   //0x2000代表classdata

		{

			cls_dat_off = pMaps->list[i].offset;  //获取classdata起始偏移

			break;

		}

	}

	log("classdata_offset:0x%x\n", cls_dat_off);

	log("0,");     //记录classdata的偏移用的log

	for (u4 i = 0; i<classDefsSize; i++)

	{

		const u1* data = dexGetClassData(g_pDexFile, &pClassDefs[i]);

		DexClassData *pData = ReadClassData(&data);

		if (!pData) {

			continue;

		}

		int fd = open("/sdcard/classdata", O_APPEND | O_CREAT | O_WRONLY, 0666);

		int class_data_len = 0;

		uint8_t *out = EncodeClassData(pData, class_data_len);

		log("%d,", class_data_len);//记录classdata的偏移用的log

		cls_dat_off += class_data_len;

		write(fd, out, class_data_len);

		close(fd);

	}

	log("\n");

    }
```

以上`ReadClassData`、`EncodeClassData`函数用的Dexhunter的。感谢Dexhunter作者。

2、此时我们已经有了`dump.dex`、`classdata`、`dumpdex.log`中保存的偏移。

![](https://bbs.kanxue.com/upload/attach/201706/638330_s1s5qqymoulwu4v.png)

3、将`classdata`写入到`classdata`偏移处。

4、通过程序修复`classdef`中的偏移。

![](https://bbs.kanxue.com/upload/attach/201706/638330_9l448ucc3ucqs5w.png)

5、运行完后我们已经获取到正确的dex，我们将`out.dex`拖入JEB已经可以正常解析。

![](https://bbs.kanxue.com/upload/attach/202002/638330_w7oj8mzehzuo3q1.png)

## 六、附件说明

额 附件不让上传了，大家看下思路就好

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
