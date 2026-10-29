
```
signed int __fastcall sub_43A8(MELFReader *a1)
{
  Byte *v1; // r4@1
  int size; // r0@1
  signed int result; // r0@2
  _BYTE *v4; // r5@3
  _BYTE *v5; // r0@3
  int v6; // r5@4
  void *addr; // [sp+Ch] [bp-14h]@1

  v1 = a1;
  //计算加载所需的空间并且将加载的基址保存在addr中
  size = sub_4264(*((_DWORD *)a1 + 15), *((_DWORD *)a1 + 14), (unsigned int *)&addr, 0);
  *((_DWORD *)v1 + 18) = size;
  //mmap分配空间
  if ( size && (v4 = addr, v5 = mmap(addr, size, 0, 34, -1, 0), v5 != (_BYTE *)-1) )
  {
    v6 = v5 - v4;   //计算 loadbias
    *((_DWORD *)v1 + 17) = v5;
    result = 1;
    *((_DWORD *)v1 + 19) = v6;
  }
  else
  {
    result = 0;
  }
  return result;
}
```
`sub_4264`的原理，因为篇幅原因就不贴代码了，就是循环段表，读取每个需要load的段，然后找到加载的最小虚拟地址，和最大的虚拟地址，页对齐后，取差值就是load size。

### load segment

遍历 `program header table`，找到类型为 `PT_LOAD` 的 segment:

1. 计算 segment 在内存空间中的起始地址 `segstart` 和结束地址 `seg_end`，`seg_start` 等于虚拟偏移加上基址`load_bias`，同时由于 `mmap` 的要求，都要对齐到页边界得到 `seg_page_start` 和 `seg_page_end`。

2. 计算 segment 在文件中的页对齐后的起始地址 `file_page_start` 和长度 `file_length`。

3. 使用`mrpotect`和`memcpy`将段映射到内存，指定映射地址为 `seg_page_start`，长度为 `file_length`，文件偏移为 `file_page_start`。

```
signed int __fastcall sub_4448(MELFReader *a1, EncryptSoInfo *a2)
{
  Byte *v2; // r6@1
  unsigned int v3; // r2@3
  int v4; // r4@4
  int v5; // r5@7
  int v6; // r3@7
  int v7; // r5@7
  int v8; // r12@8
  int v9; // r2@8
  int v10; // r10@8
  unsigned int v11; // r11@8
  unsigned int v12; // r12@8
  bool v13; // cf@8
  bool v14; // zf@8
  unsigned int v15; // r8@8
  void *v16; // r7@8
  int v17; // r12@8
  int v18; // r10@8
  void *v19; // r0@16
  signed int v20; // r2@19
  size_t n; // [sp+4h] [bp-34h]@10
  unsigned int v23; // [sp+8h] [bp-30h]@2
  Byte *v24; // [sp+Ch] [bp-2Ch]@1

  v24 = a2;
  v2 = a1;
  if ( a2 )
  {
    v23 = *((_DWORD *)a2 + 3);                  // 解压后的so的起始地址
    if ( *((_DWORD *)a2 + 3) )
    {
      v3 = *((_DWORD *)a1 + 14);                // segment 数量
      if ( !v3 )
        return 1;
      v4 = 0;
      while ( 1 )
      {
        while ( 1 )                             // 找到类型为LOAD的段
        {
          v5 = *((_DWORD *)v2 + 15);            // 段表起始地址
          v6 = *(_DWORD *)(v5 + 32 * v4);       // 迭代读取段表
          v7 = v5 + 32 * v4;
          if ( v6 == 1 )
            break;
          if ( v3 <= ++v4 )
            return 1;
        }
        v8 = *(_DWORD *)(v7 + 4);               // offset segment在文件的偏移
        v9 = *(_DWORD *)(v7 + 16);              // filesz segment在so中大小
        v10 = *((_DWORD *)v2 + 19) + *(_DWORD *)(v7 + 8);// segment 在虚拟内存的结束地址
        v11 = v8 & 0xFFFFF000;
        v12 = v8 + v9;
        v13 = v23 >= v12;
        v14 = v23 == v12;
        v15 = (*(_DWORD *)(v7 + 20) + 4095 + v10) & 0xFFFFF000;
        v16 = (void *)(v10 & 0xFFFFF000);
        v17 = v12 - v11;
        v18 = v10 + v9;
        if ( v14 || !v13 )
          break;
        n = v17;
        if ( mprotect(v16, v15 - (_DWORD)v16, 3) == -1 )
          break;
        if ( n )
          memcpy(v16, (const void *)(*((_DWORD *)v24 + 2) + v11), n);
        if ( *(_DWORD *)(v7 + 24) & 2 && v18 & 0xFFF )
          memset((void *)v18, 0, 4096 - (v18 & 0xFFF));
        v19 = (void *)((v18 + 4095) & 0xFFFFF000);
        if ( v15 > (unsigned int)v19 )
          memset(v19, 0, v15 - (_DWORD)v19);
        v20 = *(_DWORD *)(v7 + 24) & 1 ? 4 : 0;
        if ( mprotect(v16, v15 - (_DWORD)v16, *(_DWORD *)(v7 + 24) & 2 | (*(_DWORD *)(v7 + 24) << 29 >> 31) | v20) == -1 )
          break;
        v3 = *((_DWORD *)v2 + 14);
        if ( v3 <= ++v4 )
          return 1;
      }
    }
  }
  return 0;
  }
```

至此第二个so已经完成了装载。

## 创建soinfo

完成装载后，就需要创建soinfo结构，并利用`MELFReader`设置一些soinfo参数。

```
char *__fastcall sub_3D60(EncryptSoInfo *a1)
{
  const char *v1; // r4@1
  char *result; // r0@2
  char *v3; // r5@2

  v1 = (const char *)(a1 + 20);
  if ( strlen((const char *)a1 + 20) > 0x7F )
  {
    result = 0;
  }
  else
  {
    result = (char *)operator new(0x128u); //创建soinfo
    v3 = result;
    if ( result )
    {
      memset(result, 0, 0x128u);
      strncpy(v3, v1, 0x7Fu);
      result = v3;
    }
  }
  return result;
}

sub_3DBC(EncryptSoInfo *a1)
{
  //节选
  soinfo = sub_3D60(*(Byte **)v1);
  v3 = (int)soinfo;
  if ( !soinfo )
    goto LABEL_16;
  v4 = nmemb_56;
  v5 = nmemb_68;
  v6 = nmemb_72;
  v7 = nmemb_76;
  *((_DWORD *)soinfo + 33) = nmemb_56;          // 设置 soinfo->phnum;
  *((_DWORD *)soinfo + 35) = v5;                // 设置 soinfo->loadstart
  *((_DWORD *)soinfo + 36) = v6;                // 设置 soinfo->size;  
  *((_DWORD *)soinfo + 62) = v7;                // 设置 soinfo->loadbias
  //
}
```

### 链接

自定义linker最重要的一步就是链接，链接主要步骤是：

1. 定位 `dynamic segment`
2. 解析 `dynamic section`
3. 加载该so所依赖的so
4. 重定位

这一部分是由`sub_33F8`函数完成的，因为这个函数很长，所以节选主要步骤贴出来

```
signed int __fastcall sub_33F8(soinfo *a1)
{
    //遍历program header找到类型为Dynamic的program header，从而定位到Dynamic segment
    sub_45F4(*((_DWORD *)a1 + 32), *((_DWORD *)v2 + 33), *((_DWORD *)v2 + 62), (_DWORD *)v2 + 37, &v38);
    v3 = *((_DWORD *)v2 + 37); //dynamic segment

    //解析dynamic segment
    v4 = *(_DWORD *)v3; //符号类型

    if ( *(_DWORD *)v3 ) {
        v5 = (int *)(v3 + 8);
        while ( 1 ) {
            switch ( v4 ) {
                //根据符号类型做重定位
            }
        }
    }

    LABEL_7:
    if ( *(_DWORD *)v3 )
    {
      v11 = 0;
      do
      {
        if ( v10 == 1 )
        {
          v13 = (const char *)(*((_DWORD *)v2 + 39) + *(_DWORD *)(v3 + 4));
          if ( strlen(v13) > 0x80 )
            goto LABEL_21;
          if ( *((_DWORD *)v2 + 72) <= v11 )
            goto LABEL_21;
          v14 = 136 * v11++;
          strncpy((char *)(*((_DWORD *)v2 + 73) + v14 + 4), v13, 0x7Fu);
          v15 = dlopen(v13, 0);  //加载所需要的so
          if ( !v15 )
            goto LABEL_21;
          *(_DWORD *)(*((_DWORD *)v2 + 73) + v14) = v15;
          *(_DWORD *)(*((_DWORD *)v2 + 73) + v14 + 132) = 0;
        }
        v12 = *(_DWORD *)(v3 + 8);
        v3 += 8;
        v10 = v12;
      }
      while ( v12 );
    }
}
```

### dump second so

清楚第二个so的加载流程后，就可以dump出第二个so，具体做法是在`sub_3DBC`处下断点，dump内存，并且解密ELF header 和 program header table。解密算法是`libjiagu.so`中的`sub_6868`.

```
#sub_6868

def decryptso(data, size):
    result = bytearray(data)
    for i in range(size):
        result[i] ^= 0x50
    return str(result)

def dump_so(start_address, size):
    fp = open("E:\\dump.so", "wb")

    #read elf header and decrypt header
    data = idaapi.dbg_read_memory(start_address, 52)
    header = decryptso(data, 52)
    fp.write(header)

    #read elf program header tables
    phnum = 9
    data = idaapi.dbg_read_memory(start_address + 52,  phnum * 32)
    pht = decryptso(data, phnum * 32)
    fp.write(pht)

    #read other part
    data = data = idaapi.dbg_read_memory(start_address + 52 + phnum * 32,  size - phnum * 32 - 52)
    fp.write(data)

    fp.close()
```

### 进入JNI_Onload

在加载完第二个so后，`libjiagu.so`通过`sub_3F7C(soinfo *info, char* fucname)`,来找到第二个so的`JNI_Onload`入口，并在case 35中跳转到第二个so的`JNI_Onload`.

![](https://bbs.kanxue.com/upload/attach/201804/657395_54H6YW432MJSE9D.png)

## 虚拟机部分

这一部分就正式开始执行源apk中的代码了。

### dump dex

源dex要被执行，肯定需要加载，所以在`libart.so`的`OpenMemory`函数下断点就可以dump出dex，使用IDAPython dump dex

```
#add break at OpenMemory
addBrk("libart.so", OpenMemory_offset)
fn_f9()
r0 = GetRegValue("R0")
r1 = GetRegValue("R1")
print("orgin dex at: %x" % (r0))
delBrk("libart.so", OpenMemory_offset)
#dump dex
data = idaapi.dbg_read_memory(r0, r1)
fp = open('d:\\dump.dex', 'wb')
fp.write(data)
fp.close()
```

用jeb打开dump出来的dex可以发现 `oncreate`函数native化了。

![](https://bbs.kanxue.com/upload/attach/201804/657395_E52HM78D5K8KKZA.png)

### 找到onCreate函数
