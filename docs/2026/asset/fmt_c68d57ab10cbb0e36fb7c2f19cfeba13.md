
`PDWORD` `pEAT`=`(` `PDWORD` `)` `(` `(DWORD)` `lpImage`+`RVAToOffset(` `lpImage` , `pExprot`->`AddressOfFunctions` ) `)` ;

`PDWORD` `pENT`=`(` `PDWORD` `)` `(` `(DWORD)` `lpImage`+`RVAToOffset(` `lpImage` , `pExprot`->`AddressOfNames` ) `)` ;

`PWORD` `pEIT`=`(` `PWORD` `)` `(` `(DWORD)` `lpImage`+`RVAToOffset(` `lpImage` , `pExprot`->`AddressOfNameOrdinals` ) `)` ;

// `dwOrdinal`的值可以认为是序号的索引,由于是`EAT`是按照序号大小先后导出的,如果`EAT`有值,说明存在此函数,然后从`EIT`中寻找是否有此序号,如果没有说明没有导出此函数的函数名,如果有说明有序号有地址依次遍历,代码如下

```
for (DWORD dwOrdinal = 0; dwOrdinal < pExprot->NumberOfFunctions; dwOrdinal++)
{
  if (!pEAT[dwOrdinal]) //如果EAT的值为空则跳过
  {
    continue;
  }
  bool bFlag=FALSE; //设置标记位
  for (DWORD dwIndex = 0; dwIndex < pExprot->NumberOfNames; dwIndex++)
  {
    if (dwOrdinal==pEIT[dwIndex])
    {
      PCHAR pszFunName=(PCHAR)((DWORD)lpImage+RVAToOffset(lpImage,pENT[dwIndex]));
      printf("%04X 0x%p %s\r\n",pExprot->Base+dwOrdinal,pEAT[dwOrdinal],pszFunName);
      bFlag=TRUE;
      break;
    }
  }
  if (!bFlag)
  {
    printf("%04X 0x%p %s\r\n",pExprot->Base+dwOrdinal,pEAT[dwOrdinal],"null");
  }
}
```

下图是遍历结果

![](https://bbs.kanxue.com/upload/attach/201312/346893_4sasy6b7hpmfb1g.png)

## 2.3.2 导入表:

还是先看一下此表的结构体

```
struct _IMAGE_IMPORT_DESCRIPTOR {
0x00 union {
0x00 DWORD Characteristics;
0x00 PIMAGE_THUNK_DATA OriginalFirstThunk;
} u;
0x04 DWORD TimeDateStamp;
0x08 DWORD ForwarderChain;
0x0c DWORD Name;
0x10 PIMAGE_THUNK_DATA FirstThunk;
};
```

`OriginalFirstThunk`:指向输入名称表的`INT`的`RVA`

`FirstThunk`:指向输入地址表的`RVA`,如果没有载入到内存,他和`OriginalFirstThunk`:指向的内容一致,他们分别指向如下结构体的数组,且这个数组以一个空的`IMAGE_THUNK_DATA`结尾

```
typedef struct _IMAGE_THUNK_DATA {
union {
0x00 LPBYTE ForwarderString;
0x00 PDWORD Function;
0x00 DWORD Ordinal;
0x00 PIMAGE_IMPORT_BY_NAME AddressOfData;
} u1;
} IMAGE_THUNK_DATA,*PIMAGE_THUNK_DATA;
```

`Ordinal`:导入函数的序号,如果此值的最高位为1,则采用序号导入的方式,则低31位将呗看作函数序号,我们可以用宏`IMAGE_SNAP_BY_ORDINAL32`来判断此项是否为序号

`AddressOfData`:指向`IMAGE_IMPORT_BY_NAME`结构

```
typedef struct _IMAGE_IMPORT_BY_NAME
{
0x00 WORD Hint;导入函数的序号
0x02 BYTE Name[1];导入函数的名称
} IMAGE_IMPORT_BY_NAME,*PIMAGE_IMPORT_BY_NAME;
```

遍历思想:我们并不知道到底导入了多少`DLL`,所以用`while`循环比较合适,如果`IMAGE_IMPORT_DESCRIPTOR`结构体的`name`字段无值,说明遍历结束,如果有值进入`IMAGE_THUNK_DATA`结构体进行判断,这是个联合体,同一时段只能有一个位段有值,前两个位段只是在载入到内存时才有值,所以排除判断,前门说我们有一个宏来进行判断序号,所以用`if(!IMAGE_SNAP_BY_ORDINAL32(pTHUNK->u1.Ordinal))else`来判断即可

下面给出伪代码,读者可以自己试着编写出具体代码

```
//假设pImport表示指向导入表的指针, pTHUNK表示指向IMAGE_THUNK_DATA32的指针
While(pImport->Name)
{
  .....
   while (pTHUNK->u1.Ordinal) //如果此值无效,第四个字段才有效,所以先判断此值
       {
         If(!IMAGE_SNAP_BY_ORDINAL32(pTHUNK->u1.Ordinal))
            {
               …
               pTHUNK++;
               continue;
}
Else
{
}
}
pImport++;
}
```

测试效果如下

![](https://bbs.kanxue.com/upload/attach/201312/346893_e6sbbhunzoi64ux.png)

## 2.3.3 资源表:

```
struct _IMAGE_RESOURCE_DIRECTORY {
0x00 DWORD Characteristics;属性
0x04 DWORD TimeDateStamp;建立时间
0x08 WORD MajorVersion;主板本
0x0a WORD MinorVersion;自版本
0x0c WORD NumberOfNamedEntries;用字符串作为资源标识的条目个数
0x0e WORD NumberOfIdEntries;用ID作为资源标识的个数
};
```

这是资源表的结构体,资源在`PE`文件中以目录的结构存在,一般情况下这个目录分三层,从根目录开始依次为资源类型,目录资源`ID`与资源代码页,这三层都是`IMAGE_RESOURCE_DIRECTORY`结构为头部,并在其后紧随着一个`IMAGE_RESOURCE_DIRECTORY_ENTRY`的结构数组, `IMAGE_RESOURCE_DIRECTORY`主要是指出后面结构体数组的成员个数,而后面这个`ENTRY`结构数组的每个成员分别指向下一层目录结构或者是资源数据

```
typedef struct _IMAGE_RESOURCE_DIRECTORY_ENTRY {
    union {
        struct {
            DWORD NameOffset:31;资源名偏移
            DWORD NameIsString:1;资源名为字符串
        } DUMMYSTRUCTNAME;
        DWORD   Name;资源类型
        WORD    Id;资源ID
    } DUMMYUNIONNAME;
    union {
        DWORD   OffsetToData;数据偏移地址
        struct {
            DWORD   OffsetToDirectory:31;子目录偏移地址
            DWORD   DataIsDirectory:1;下一层是目录
        } DUMMYSTRUCTNAME2;
    } DUMMYUNIONNAME2;
}IMAGE_RESOURCE_DIRECTORY_ENTRY, *PIMAGE_RESOURCE_DIRECTORY_ENTRY;
```

如果`NameIsString`为1, `NameOffset`指向`IMAGE_RESOURCE_DIR_STRING_U`

```
typedef struct _IMAGE_RESOURCE_DIR_STRING_U {
    WORD    Length;字符串长度
    WCHAR   NameString[ 1 ];字符串数组
} IMAGE_RESOURCE_DIR_STRING_U, *PIMAGE_RESOURCE_DIR_STRING_U;
```

经过三层遍历后,最后会到达`IMAGE_RESOURCE_DATA_ENTRY`

```
typedef struct _IMAGE_RESOURCE_DATA_ENTRY {
    DWORD   OffsetToData;资源数据的RVA
    DWORD   Size;        资源数据的长度
    DWORD   CodePage;    代码页
    DWORD   Reserved;
} IMAGE_RESOURCE_DATA_ENTRY, *PIMAGE_RESOURCE_DATA_ENTRY;
```

遍历思想:在第一层时, `IMAGE_RESOURCE_DIRECTORY_ENTRY`的`name`字段或者结构体有效,保存的是资源类型,第二层遍历时`ID`或者是联合体内的数据有效,第三层时`Name`字段有效,保存的是资源语言区域类型,第二个联合体同一时间只有一个字段有效,所以比较好判断

下面给出伪代码

```
//假设pRes1指向第一层IMAGE_RESOURCE_DIRECTORY
//第一层为目录遍历
DWORD dwCount1=pRes1->NumberOfIdEntries+pRes1->NumberOfNamedEntries;
For(DWORD i = 0; i < dwCount1; i++)
   {
      获取IMAGE_RESOURCE_DIRECTORY_ENTRY结构指针pEntry1
      If(类型是字符串)
        {
           ……………
}
      Else类型是Name字段
        {
           …………..

}
//接下来判断第二个联合体指向的是数据还是目录
If(如果为目录)
{
   获取第二层IMAGE_RESOURCE_DIRECTORY 的指针pRes2,
      If(ID是字符串)//注意这层是ID
       {
}
Else  Id字段指明
{
}
//接下来判断第二个联合体指向的是数据还是目录
If(是目录)
{
        获取第三层IMAGE_RESOURCE_DIRECTORY 的指针pRes3
            //这是最后一层
           直接获取资源代码页的数据
}
Else是数据
{
}
}
Else是数据
{
  ……..
}
}
```

效果如下

![](https://bbs.kanxue.com/upload/attach/201312/346893_68houc9kw6bf5is.png)

## 2.3.4 重定位表:

```
typedef struct _IMAGE_BASE_RELOCATION {
    DWORD   VirtualAddress;需要重定位的起始RVA
    DWORD   SizeOfBlock;   本结构与TypeOffset总大小
//  WORD    TypeOffset[1];
} IMAGE_BASE_RELOCATION;
```

这个相对比较简单,需要说明的是`VirtualAddress`字段,这个字段更像前面所说的基址,由于`SizeOfBlock`字段指明了这个重定位表的大小,所以用它的值减去`VirtualAddress`和`SizeOfBlock`这两个字段所占大小,此时的大小将是`TypeOffset`数组的大小,这个结构体的定义如下

```
struct{
WORD Offset:12;重定位的偏移,
WORD Type:4;重定位类型
}
```

注意`Offset`这个字段,这个字段的数据加上刚才的`VirtualAddress`才是真正的`RVA`

遍历思想:我们并不知道有几个重定位区块,所以用`while`循环比较适合,这个比较简单,贴出伪代码

```
While(pBase->VirtualAddress)
{
  获得需要重定位项的个数dwBaseOfNum
  获得指向TypeOffset的结构体指针pOffset
   for (DWORD i = 0; i < dwBaseOfNum; i++)
      {
        依次获取每个TypeOffset的类型,进而进行判断并修复
}
指向下个重定位位段
}
```

效果如下

![](https://bbs.kanxue.com/upload/attach/201312/346893_9wwyexv0s4t0k46.png)

到现在我们已经遍历了几个比较重要而且常用的目录表,其余的就是界面编写与数据修改部分,不在我们的讨论范围内

## 3:总结

开发过程中,以前学过的一些控件的用法有些模糊了,尤其是树控件的消息响应,都是从网上找到相关资料进行解决,开发本来就是个很细的工作,不要抱着侥幸心理,例如资源的遍历,要考虑全面才能解析各种变态`PE`.虽然名字叫`PeEdit`,但只能编辑保存少部分信息,更多的信息还不能进行保存,希望可以在后续版本中进行不断优化和增加功能,成为一个真正的`PeEdit`.

后记:

走到现在,非常感谢任老师的教学,许`Sir`和薛老师的课下指导

整个界面模仿了大名鼎鼎的`LoadPE`,书籍参考了黑客免杀攻防(`任晓珲`著).

[PeEdit.rar](https://bbs.kanxue.com/attach-download-84804.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [01.png](javascript:void(0);)
  （5.54kb，5次下载）
- [11.png](javascript:void(0);)
  （52.96kb，3次下载）
- [12.png](javascript:void(0);)
  （30.29kb，3次下载）
- [13.png](javascript:void(0);)
  （22.73kb，3次下载）
- [14.png](javascript:void(0);)
  （67.51kb，3次下载）
- [15.png](javascript:void(0);)
  （72.82kb，3次下载）
- [16.png](javascript:void(0);)
  （148.27kb，3次下载）
- [17.png](javascript:void(0);)
  （159.63kb，3次下载）
- [18.png](javascript:void(0);)
  （101.71kb，3次下载）
- [19.png](javascript:void(0);)
  （101.46kb，3次下载）
- [20.png](javascript:void(0);)
  （272.65kb，3次下载）
- [21.png](javascript:void(0);)
  （34.67kb，3次下载）
- [PeEdit.rar](javascript:void(0);)
  （166.20kb，62次下载）
