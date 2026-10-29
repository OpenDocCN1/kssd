

# [原创] 分析一个有趣的 SO 双重壳

## 背景

去年四五月份的时候，客户反馈过一个 SO 加壳的问题。分析后发现，该问题与某 Android 应用市场 SDK 的 `libYSDK.so` 冲突有关。当时，该 SO 在解壳过程中会操作 `linker` 的 `soinfo` 结构中的部分数据，壳作者将 `dlopen` 返回的 handle 强制转换为 `soinfo` 结构体指针，这种操作必然存在兼容性问题。后来，该壳也进行了升级，其最新版本包含一些比较有趣的技术点，下面与大家分享。

## 两层壳

直接使用 IDA 打开：

![](https://bbs.kanxue.com/upload/attach/201806/652950_4BUK75UUX9W3CGW.jpg)

函数列表只有 31 个 `sub_**`（`__mprotect` 是笔者改名而来），再查看 Section 表：

![](https://bbs.kanxue.com/upload/attach/201806/652950_9EH5T7ZVX3CEC3P.jpg)

很多内容被擦掉了。`dynsym` 和 `dynstr` 在 7.0 及以上版本中是 `linker` 所需要的，必须保留；其他保留项对分析没有什么用，壳作者也进行了保留。这里直接抹掉 ELF Header 中的 Section 表偏移和大小，让 IDA 使用动态链接表解析：

![](https://bbs.kanxue.com/upload/attach/201806/652950_V8UUZFBWPPXPF44.jpg)

再次使用 IDA 打开后，函数列表发生了变化：

![](https://bbs.kanxue.com/upload/attach/201806/652950_3YQURTFV47MHA8H.jpg)

但是继续查看代码区（`JNI_OnLoad`），可以发现代码已经被加密：

![](https://bbs.kanxue.com/upload/attach/201806/652950_2ZX966R3D3XP5XY.jpg)

继续向后拖拽，发现符号表竟然都在第二个 `loadsegment` 中：

![](https://bbs.kanxue.com/upload/attach/201806/652950_Y5S5E6MXYRNM6VC.jpg)

## `init` 解密壳

首先考虑第一个 `loadsegment` 代码加密的问题。通常情况下，解密逻辑会在 `linker` 的 `init` 中执行，而 `init` 对应的那部分代码一定处于明文状态。

执行 `greadelf –d libYSDK.so`：

![](https://bbs.kanxue.com/upload/attach/201806/652950_W64DVYQKC36J8X6.jpg)

该 SO 编译时的名字是“legu”，后来重命名为“YSDK”，没有找到 `init` 段，只有 `init_array`。跳过去查看：

![](https://bbs.kanxue.com/upload/attach/201806/652950_EG8Y3SKASB2ZRQJ.jpg)

看来 `sub_838` 就是解密函数，下面贴出它的伪代码：

```c
int sub_838()

{

  unsigned int code_segment_end_segment; // ST28_4

  unsigned int data_segment_begin_page; // ST24_4

  unsigned int data_segment_end_page; // ST20_4

  unsigned __int64 v3; // kr00_8

  char v4; // ST13_1

  int result; // r0

  int v6; // [sp+4h] [bp-40h]

  int v7; // [sp+8h] [bp-3Ch]

  int j; // [sp+2Ch] [bp-18h]

  char v9; // [sp+30h] [bp-14h]

  char v10; // [sp+31h] [bp-13h]

  char v11; // [sp+32h] [bp-12h]

  char v12; // [sp+33h] [bp-11h]

  unsigned int v13; // [sp+34h] [bp-10h]

  _DWORD *e_pheaders; // [sp+38h] [bp-Ch]

  unsigned int i; // [sp+3Ch] [bp-8h]



  for ( i = (unsigned int)sub_838 & 0xFFFFF000; *(_DWORD *)i != 0x464C457F; i -= 0x1000 )

    ;

  e_pheaders = (_DWORD *)(*(_DWORD *)(i + 0x1C) + i);// i = loadbase

  v13 = 0;

  while ( *(unsigned __int16 *)(i + 0x2C) > v13 )// phnum > 0

  {

    if ( *e_pheaders != 1 || e_pheaders[6] != 5 )// type != PT_LOAD || flag != Read_Exe

    {

      if ( *e_pheaders == 1 && e_pheaders[6] == 6 )// type == PT_LOAD && flag == Read_Write

      {

        data_segment_begin_page = e_pheaders[2] & 0xFFFFF000;

        data_segment_end_page = (e_pheaders[2] + e_pheaders[4] + 4095) & 0xFFFFF000;

        break;

      }

    }

    else

    {

      code_segment_end_segment = (e_pheaders[2] + e_pheaders[4] + 4095) & 0xFFFFF000;

    }

    ++v13;

    e_pheaders += 8;

  }

  v12 = 0x2B;

  v11 = 0x99u;

  v10 = 32;

  v9 = 21;

  v3 = (unsigned __int64)(unsigned int)dword_4008 << 16;// 0x100028cc << 16

  v7 = (unsigned __int16)dword_4008;

  v6 = (unsigned __int16)dword_4008 - ((unsigned int)dword_4008 >> 16);

  _mprotect(i + ((unsigned int)dword_4008 >> 16), (v6 + 4095) & 0xFFFFF000, 3);// mprotect(loadbase + 0x1000, 0x2000, Read_Write)

  for ( j = HIDWORD(v3); j <= v7; ++j )         // 解密代码区加密部分（0x1000~0x28cc）

  {

    v4 = *(_BYTE *)(i + j);

    *(_BYTE *)(i + j) ^= (unsigned __int8)(((v11 - v10) ^ j) + v9) ^ v12;

    *(_BYTE *)(i + j) += v10 & v9 ^ v11;

    v12 += (v11 + v10 - v9) & v4 & j;

    v11 += (j + v12) ^ v4;

    v10 ^= (v4 - v12) ^ j;

    v9 += j - (v4 + v12);

  }

  _mprotect(i + HIDWORD(v3), (v6 + 4095) & 0xFFFFF000, 5);// mprotect(loadbase + 0x1000, 0x2000, Read_Exe)

  result = __cache_flush(i + HIDWORD(v3), v6);

  dword_4008 = i;            // JNI_OnLoad 中通过该变量获取so的路径

  return result;

}
```

对 `0x1000~0x28cc` 区间的解密算法很简单，不过还是偷懒直接动态调试从内存中扣取，IDA 脚本如下：

```python
import idaapi

data = idaapi.dbg_read_memory(0xEF214000, 0xEF2158CC - 0xEF214000)

fp = open("/**/ysdk.dmp", "wb")

fp.write(data)

fp.close()
```

然后将其替换到原 SO 的 `0x1000~0x28cc` 区间上，再用 IDA 打开，`JNI_OnLoad` 就可以直接进行静态分析了（还原后的内容见附件）：

![](https://bbs.kanxue.com/upload/attach/201806/652950_4XEAHT8U57M3YJK.jpg)

## `JNI_OnLoad` 加载器壳

IDA 分析的第二个加载段中的符号表是怎么回事呢？先使用 010 Editor 打开该 SO，查看其布局上的诡异之处：

![](https://bbs.kanxue.com/upload/attach/201806/652950_QVFAE95HRZQ86AY.jpg)

很明显，文件末尾放着更多内容。这些才是真正与业务逻辑相关的代码和数据。加载这些内容的代码就在 `JNI_OnLoad` 中，伪代码如下：

```c
void *__fastcall JNI_OnLoad(int a1, int a2)

{

  void *v2; // r3

  int v4; // [sp+0h] [bp-414h]

  int v5; // [sp+4h] [bp-410h]

  int cur_so_path; // [sp+8h] [bp-40Ch]

  int (__fastcall *v7)(int, int); // [sp+408h] [bp-Ch]

  int v8; // [sp+40Ch] [bp-8h]



  v5 = a1;

  v4 = a2;

  memset(&cur_so_path, 0, 0x400u);

  while ( sub_2418((int)&cur_so_path) )         // 通过 dword_4008 和 maps 比对获取当前 so 的全路径，

                                                // 在 init 解密代码段结束时把 dword_4008 赋值为了当前 so 的 base 地址

    ;

  load_true_code((int)&cur_so_path, (int)off_4004);

  v8 = dlopen(&cur_so_path, 0);

  v7 = (int (__fastcall *)(int, int))dlsym(v8, "JNI_OnLoad");// 调用真正的 JNI_OnLoad，当前这个 JNI_OnLoad 算是壳 JNI_OnLoad

  _android_log_print(6, "txtag", "load done!");

  if ( v7 )

    v2 = (void *)v7(v5, v4);

  else

    v2 = &unk_10004;

  return v2;

}
```

核心内容在 `load_true_code` 中，继续查看其伪代码（日志给出了很多提示）：

