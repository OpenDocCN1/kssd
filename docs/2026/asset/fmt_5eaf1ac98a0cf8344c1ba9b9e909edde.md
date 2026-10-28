
我们注意到一个内存地址：`0x403000`，还记得前面的Peid的截图吗？看到`.text6`节区：`V.offset 3000`

在内存当中我们查看，目前都是一些没有特别意义的数据，当然在后面就能清楚作用了。

`Sub_401390`的作用已经标记出来：返回传入地址的前一个DWORD地址的值。

上面片段中调用的时候传入的是`0x40300+0x4`，那么返回的就是`[0x40300]`，从内存中可以看到该数据是`0x1700`。

这个值传入了`sub_4013A0`函数，该函数的作用是申请指定大小的内存空间：

```
push    ebp
mov     ebp, esp
push    40h             ; PAGE_EXECUTE_READWRITE
push    3000h           ; MEM_COMMIT | MEM_RESERVE
mov     eax, [ebp+arg_0]
push    eax
push    0
push    0FFFFFFFFh
call    VirtualAllocEx     ;之前得到的VirtualAllocEx的地址
pop     ebp
retn
```

关键要注意的是参数`0x40`：`PAGE_EXECUTE_READWRITE` 说明申请的这些空间有执行的权限，这一点需要特别留意。接着对申请的空间进行了清0操作。

根据程序运行的结果，内存空间[`0x3E0000`，`0x3E1700`)返回给程序运行，并且拥有可执行权限。

下面的内存操作，有4个循环，我们以此来看：

```
text:00401793                 mov     ecx, [ebp+BeCopyedDataNum] ; 从0x403004处开始，每0x5A数据拷贝到申请的内存中(每次间隔0xA大小的数据)
.text:00401796                 cmp     ecx, [ebp+Size] ;初始为0
.text:0040179C                 jnb     short loc_40181B ; 判断是否还有可用的空闲内存
.text:0040179E                 mov     edx, [ebp+FreeVirtualSize]
.text:004017A4                 push    edx
.text:004017A5                 mov     eax, [ebp+UsedVirtualSize]
.text:004017AB                 push    eax
.text:004017AC                 call    sub_4013C0      ; 判断还能拷贝的数据大小
.text:004017B1                 add     esp, 8
.text:004017B4                 mov     [ebp+NumberOfCopyData], eax
.text:004017BA                 mov     ecx, [ebp+NumberOfCopyData]
.text:004017C0                 push    ecx
.text:004017C1                 mov     edx, [ebp+var_280]  ;var_280为0x403004
.text:004017C7                 add     edx, [ebp+IndexSourceStr]
.text:004017CD                 push    edx
.text:004017CE                 mov     eax, [ebp+Dst]
.text:004017D4                 add     eax, [ebp+BeCopyedDataNum]
.text:004017D7                 push    eax
.text:004017D8                 call    sub_4013F0      ; 从一个内存区域拷贝指定数据至另一个内存块
.text:004017DD                 add     esp, 0Ch
.text:004017E0                 mov     ecx, [ebp+var_220] ; 源字符串的间隔(大小为10）
.text:004017E6                 add     ecx, [ebp+UsedVirtualSize]
.text:004017EC                 add     ecx, [ebp+IndexSourceStr]
.text:004017F2                 mov     [ebp+IndexSourceStr], ecx
.text:004017F8                 mov     edx, [ebp+BeCopyedDataNum]
.text:004017FB                 add     edx, [ebp+UsedVirtualSize]
.text:00401801                 mov     [ebp+BeCopyedDataNum], edx
.text:00401804                 mov     eax, [ebp+FreeVirtualSize]
.text:0040180A                 sub     eax, [ebp+NumberOfCopyData] ; 减去已经填充的数据大小
.text:00401810                 mov     [ebp+FreeVirtualSize], eax
.text:00401816                 jmp     loc_401793      ; 从0x403004处开始，每0x5A数据拷贝到申请的内存中(每次间隔0xA大小的数据)
```

整个片段的功能是比较简单的：

总体来看是进行内存拷贝操作，从`0x403004`开始，将每`0x64`字节数据(总数据量为`0x1700`)的前`0x5A`数据依次拷贝到刚刚申请的内存空间中(`0x3E0000`)中。

稍微有点意思的是`sub_4013F0`，实际上它完全等价于`memcpy`函数，"特色"如下：

```
.text:0040141E                 mov     edx, [ebp+arg_4]
.text:00401421                 add     edx, [ebp+var_8]
.text:00401424                 movzx   eax, byte ptr [edx]
.text:00401427                 mov     [ebp+var_10], eax
.text:0040142A                 mov     [ebp+var_C], 96E8FDh
.text:00401431                 mov     ecx, [ebp+var_10]
.text:00401434                 add     ecx, [ebp+var_C]
.text:00401437                 mov     edx, [ebp+arg_0]
.text:0040143A                 add     edx, [ebp+var_8]
.text:0040143D                 mov     [edx], cl
.text:0040143F                 mov     eax, [ebp+arg_0]
.text:00401442                 add     eax, [ebp+var_8]
.text:00401445                 movzx   ecx, byte ptr [eax]
.text:00401448                 sub     ecx, 961788h
.text:0040144E                 mov     edx, [ebp+arg_0]
.text:00401451                 add     edx, [ebp+var_8]
.text:00401454                 mov     [edx], cl
.text:00401456                 mov     eax, [ebp+arg_0]
.text:00401459                 add     eax, [ebp+var_8]
.text:0040145C                 mov     cl, [eax]
.text:0040145E                 sub     cl, 1
.text:00401461                 mov     edx, [ebp+arg_0]
.text:00401464                 add     edx, [ebp+var_8]
.text:00401467                 mov     [edx], cl
.text:00401469                 mov     eax, [ebp+arg_0]
.text:0040146C                 add     eax, [ebp+var_8]
.text:0040146F                 movzx   ecx, byte ptr [eax]
.text:00401472                 sub     ecx, 0D174h
.text:00401478                 mov     edx, [ebp+arg_0]
.text:0040147B                 add     edx, [ebp+var_8]
.text:0040147E                 mov     [edx], cl
```

就行了，就是两个指针按照字节大小依次复制数据。需要注意的在IDA中：

`.text.0040147E` `move [edx],cl`

而在OD中：

`0040147E  |.  880A          |MOV BYTE PTR DS:[EDX],CL`

IDA中貌似把按照字节对齐拷贝这个给省略了，分析的时候注意一下。

中间一部分比较长的，看起来是在做一些加减法：

`(((((0x96E8FD+?)%0xFF-0x961788)%0xFF)-0x1)%FF)-0xD174)%FF`

上述表达式得到的值复制给目的地址，其实上面的结果还是是`'?'`

因为：`0xFD-0x88-0x01-0x74=0`

我们可以理解成"算术花指令"，这样做也许能起到一定的免杀效果吧。

拷贝完毕之后，内存区域[`0x3E0000`，`0x3E17000`)填满了从`.text6`节区拷贝过来的数据。

下面的一个循环，对数据进行了解密：

```
text:00401827                 mov     ecx, [ebp+var_214]
.text:0040182D                 add     ecx, 4
.text:00401830                 mov     [ebp+var_214], ecx
.text:0401836                 mov     edx, [ebp+var_214]
.text:0040183C                 cmp     edx, [ebp+Size] ; 判断是否解密完毕
.text:00401842                 jnb     short loc_40188F
.text:00401844                 mov     eax, [ebp+Dst]
.text:0040184A                 add     eax, [ebp+var_214]
.text:00401850                 mov     ecx, [eax]        ;获取相应内存地址的数据
.text:00401852                 add     ecx, [ebp+var_214] ;加上其所对应的索引
.text:00401858                 mov     edx, [ebp+Dst]
.text:0040185E                 add     edx, [ebp+var_214]
.text:00401864                 mov     [edx], ecx        ;更新内存的值
.text:00401866                 mov     eax, [ebp+var_214]
.text:0040186C                 add     eax, 1405h      ; 索引值加上0x1405
.text:00401871                 mov     ecx, [ebp+Dst]
.text:00401877                 add     ecx, [ebp+var_214]
.text:0040187D                 xor     eax, [ecx]        ;与内存的值进行异或
.text:0040187F                 mov     edx, [ebp+Dst]
.text:00401885                 add     edx, [ebp+var_214]
.text:0040188B                 mov     [edx], eax       ;用异或之后的值更新内存
.text:0040188D                 jmp     short loc_401827
```

算法也比较简单，我们把这些数据看成一个`0x1700`大小的`int`数组，`arr[0x1700]`：

```
for (i = 0; i < 0x1700/4; i++)
{
    temp = i*4+0x1405;
    temp ^= (arr[i]+i);
    arr[i] = temp;
}
```

解密完毕之后我们看到了一系列熟悉的API字符串：

![](https://bbs.kanxue.com/upload/attach/201205/372322_ijdunqkx48mqxy2.png)

接下来的操作，有些前面已经分析过了，直接描述最后的效果：

申请了一个大小为`0x1C00`的内存空间：[`0x3F0000`, `0x3F1C00`)

又有类似的数据[`0x40499C`，`0x40B99C`) [`0x404998`]=`0x200`

然后申请了大小为`0x200`的内存空间：[`0x8F0000`，`0x8F0200`) 并将其初始化。

接着按照之前相同的方式从`0x40499C`拷贝`0x200`字节的数据到`0x8F0000`处(每`0x64`数据只拷贝前`0x5A`个)。用同样的算法：

```
for (i = 0; i < 0x200/4; i++)
{
    temp = i*4+0x1405;
    temp ^= (arr[i]+i);
    arr[i] = temp;
}
```

对`0x8F0000`的数据进行解密。

上述循环执行完毕之后，进行[`0x3E0000`]=`0x8F0000`。即将第三次申请的内存空间地址存放到第一次申请的内存空间区域的第一个DWORD区域。

然后将[`0x3E0000`, `0x3E1700`)的数据全部拷贝到[`0x3F0300`, `0x3F1A00`)处

### 重要数据解密

`Sub_401490`的作用是修复API字符串名称和指向这些字符串的指针的数据。

我们来仔细看看：

```
.text:00401A31                 mov     edx, [ebp+var_4] ; 第二次申请的内存区域的首地址
.text:00401A34                 sub     edx, 400000h    ; 减去exe的基地址0x400000
.text:00401A3A                 push    edx
.text:00401A3B                 mov     eax, [ebp+var_4] ; 第二次申请的内存区域的首地址
.text:00401A3E                 push    eax
.text:00401A3F                 mov     ecx, [ebp+var_224] ; 0x8F0000第三次申请的内存区域地址
.text:00401A45                 push    ecx
.text:00401A46                 call    sub_401490      ; 修复API字符串名称和指向其地址的值的关系
```

先看第二个参数`0x3F0000`的相关内存数据：

![](https://bbs.kanxue.com/upload/attach/201205/372322_4gh1xwx5rfh7y8w.png)

第一个字符串的地址为：`0x3F0308`
