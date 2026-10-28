
`image_optional_header`中在偏移`0x34`处有`kernel32.dll`的地址。以下是`image_optional_header`的结构

![](https://bbs.kanxue.com/upload/attach/202010/830337_FEGT3U4HV29S658.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_46U5FEWNTGHUPTZ.jpg)

### 实现代码

```
start:   mov  edx,[esp]
          dec   edx
         mov  ebx,[edx+0x3c]
          mov  ebx,[ebx+edx+0x34]
         cmp  edx,ebx
         jnz    start
         ret                           ;此时在edx中的值为kernel32.dll地址
```

### 方法三:使用SEH的链表来查找

原理：在SEH中默认的`unhandled exception hander` 是利用`kernel32.DLL`中的一个函数设置的。因此可以历遍所有的`exception hander`， 找到最后一个成员，该成员的前4个字节是`0ffffffffh`,后4个字节是`kernel32.dll`中的函数

实现代码:

```
      mov esi,fs[0]
@@1:  lodsd
   cmp [eax],0xffffffff
   jne   @@1
   mov  eax,[eax+0x04]
@@2:  dec   eax
     xor    ax,ax
      cmp  eax word ptr[eax]  'MZ'
     jne    @@2
      ret
```

### 解析PE文件导出数据

导出的数据存放在`IMAGE_EXPORT_DIRECTORY`中，一个相对`IMAGE_EXPORT_DIRECTORY`的`RVA`被保存在`IMAGR_OPTIONAL_HEADER`末尾部分的`IMAGE_DATA_DIRECTORY`结构体数组中，`IMAGE_DATA_DIRECTORY`数组具体位置取决于这PE文件是32位的还是64位的，一般情况下，我们认为`shellcode`在32位环境下运行，所以编译时候从PE特征值域到这个目录数组的正确偏移,计算方法如下:

`sizeof(PE_Signature)+sizeof(IMAGE_FILE_HEADER)+sizeof(IMAGE_OPTIONAL_HEADER)=120`字节

我们用一个图来表示`IMAGE_EXPORT_DIRECTORY`结构体中的相对域：

![](https://bbs.kanxue.com/upload/attach/202010/830337_B7Q65CB43YRC5B6.jpg)

上面这个图中，`AddressOfFunctions`是一个`RVA`的数组，我们为了使用这个数组，这段`shellcode`必须得映射导出函数名到这个序号，并且`shellcode`的确是使用`AddressOfNames`和`AddressOfNameOrdinals`来这么干的，这两个数组同时存在，很轻松的就能看到这两个数组是直接相关的，`AddressOfNames`是一个32位的`RVA`数组，指向富豪命的字符串，`AddressOfNameOrdinals`是一个16位序号的数组，当一个给定的数组索引`idx`，`AddressOfNames[idx]`处的符号对应的导出序号就在`AddressOfNames[idx]`处，`AddressOfNames`数组是按照字母顺序排列的，这样的话写一个二分搜索就能够很快的找到一个给定的字符串，虽然这么说，大多数`shellcode`还是简单的线性查找。其实我们按照以下的步骤就可以找到一个符号的导出地址：

1. 迭代`AddressOfNames`数组查看每个`char *`项目，然后把需要的符号的字符串做一个比较，直到找到一个匹配的项目，我们把这个`AddressOfNames`的索引称之为`AddressOfNames iName`。
2. 在`AddressOfNameOrdinals`数组中使用`IName`索引，得到的值就是`IOrdinal`值。
3. 使用`IOrdinal`索引到`AddressOfFunctions`数组，获得到的值就是被导出符号的`RVA`然后把这个值返回给请求者。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-10-27 12:14

被Golden_Boy编辑，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
