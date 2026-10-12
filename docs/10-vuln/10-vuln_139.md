
# MS11-092 补丁分析

> 原文：[`www.pediy.com/kssd/pediy12/144631.html`](https://www.pediy.com/kssd/pediy12/144631.html)

这个补丁出来有几天了，以前没 DIFF 比较过，所以这次很感兴趣，欢迎指出错误。

由于 IDA 版本问题。用不起来 BINDIFF,只好用 DarunGrim 了。也蛮强的，结果如下

![](img/a0cc29fc5422a078fce305827c1d37cf.jpg)

改动较大的是这个：`sub_147CF40A`      `sub_147CF412`     6

![](img/17e485106df85503023ca5866a847b35.jpg)

又是添加又是修改，我们近 IDA 看，整理后，如下

代码:

```
signed int \_\_thiscall sub\_147CF40A(void \*this, int temp2)
{
......
  v19 = this;
  result\_1 = 0;
  if ( temp2 )
  {
    src\_copy = temp2 + 16;
    if ( \*temp2 == 'ilaV' && (parm1 = \*(temp2 + 4), parm1\_ = \*(temp2 + 4), parm2 = \*(temp2 + 8), parm2 <= 512) )
    {                                           // Vali
      offset = 0;
      v21 = 0;
      if ( parm1 > 0 )
      {
        while ( v21 < 128 )
        {
          total\_size\_ = \*(src\_copy + 16);
          v5 = total\_size\_ + offset;
          total\_size = \*(src\_copy + 16);        // 拷贝大小
          offset += total\_size\_;
          if ( total\_size\_ < 0 || v5 < 0 || v5 > parm2 )
          {
            result\_1 = -2147024809;
            break;
          }
          mem\_address = alloc\_bstr\_0x20u();
          mem\_address\_ = mem\_address;
          mem\_address\_\_ = mem\_address;
          if ( !mem\_address )
          {
            result\_1 = -2147024882;
            goto error;
          }
          result\_1 = adapt\_new\_mem(mem\_address, \*src\_copy, \*(src\_copy + 4), 0);
          if ( result\_1 < 0 )
            goto error;
          data\_size = total\_size\_ - 32;
          if ( total\_size\_ - 32 > 0 )
          {
            length = (data\_size + 1) >> 1;      // BSTR 长度
            BSTR\_address = SysAllocStringLen\_(&bstrString, (data\_size + 1) >> 1);
            parm3 = (mem\_address\_ + 24);
            sub\_147CE7E8(parm3, BSTR\_address);
            SysFreeString(bstrString);          // if ( !\*v12 || !SysStringLen(\*v12) ){error!}
            (\*parm3)\[length - 1\] = 0;           // V11 指针？
            memcpy(\*parm3, (src\_copy + 32), total\_size - 32);
            SysStringByteLen(\*parm3);
            SysStringByteLen(\*parm3);
            total\_size\_ = total\_size;
            mem\_address\_ = mem\_address\_\_;
          }
          sub\_147CF06A(mem\_address\_);
          src\_copy += total\_size\_;
          ++v21;
          if ( v21 >= parm1\_ )
            break;
        }
        if ( result\_1 >= 0 )
          goto LABEL\_21;
error:
        sub\_147CF3EE(v19);
      }
LABEL\_21:
      result = result\_1;
    }
    else
    {
      result = -2147024809;
    }
  }
  else
  {
    result = -2147024809;
  }
  return result;
}
```

修改后

代码:

```
signed int \_\_thiscall sub\_147CF412(void \*this, int temp2)
{
 .....
  v19 = this;
  result\_1 = 0;
  if ( temp2 )
  {
    check\_file = \*temp2 == 'ilaV';
    src\_copy\_ = temp2 + 16;
    src\_copy = temp2 + 16;
    if ( check\_file && (parm1 = \*(temp2 + 4), parm1\_ = \*(temp2 + 4), parm2 = \*(temp2 + 8), parm2 <= 512) )
    {
      offset = 0;
      v22 = 0;
      if ( parm1 > 0 )
      {
        while ( v22 < 128 )
        {
          total\_size = \*(src\_copy\_ + 16);
          v6 = total\_size + offset;
          v14 = \*(src\_copy\_ + 16);
          offset += total\_size;
          if ( total\_size < 0 || v6 < 0 || v6 > parm2 )
          {
            result\_1 = -2147024809;
            break;
          }
          mem\_address = alloc\_bstr\_0x20u();
          mem\_address\_ = mem\_address;
          mem\_address\_\_ = mem\_address;
          if ( !mem\_address )
            goto error;
          result\_1 = adapt\_new\_mem(mem\_address, \*src\_copy\_, \*(src\_copy\_ + 4), 0);//传递地址
          if ( result\_1 < 0 )
            goto LABEL\_21;
          data\_size = total\_size - 32;          // 拷贝大小
          if ( data\_size > 0 )
          {
            BSTR\_address = SysAllocStringLen\_(&bstrString, (data\_size + 1) >> 1);
            parm3 = (mem\_address\_ + 24);//可控参数，直接读文件
            sub\_147CE7F0(parm3, BSTR\_address);//进去！！！，此函数修改了
            SysFreeString(bstrString);
            if ( !\*parm3 || !SysStringLen(\*parm3) )//这里！！补丁添加的校验
            {
error:
              result\_1 = -2147024882;
              goto LABEL\_21;
            }
            (\*parm3)\[((data\_size + 1) >> 1) - 1\] = 0;
            memcpy(\*parm3, (src\_copy + 32), data\_size);
            SysStringByteLen(\*parm3);
            SysStringByteLen(\*parm3);
            src\_copy\_ = src\_copy;
            mem\_address\_ = mem\_address\_\_;
          }
          sub\_147CF072(mem\_address\_);
          src\_copy\_ += v14;
          ++v22;
          src\_copy = src\_copy\_;
          if ( v22 >= parm1\_ )
            break;
        }
```

补丁对 `parm3` 进行了校验，看来错误就在这里了。

这个 `parm3` 怎么来的呢。我们网上看到如此字样：

`check_file = *temp2 == 'ilaV';`

`src_copy_ = temp2 + 16;`

同时随便找一个样本文件打开，这里用的是 `http://samples.mplayerhq.hu/dvr_ms/microsoft-new-way-to-shove-mpeg2-in-asf.dvr-ms`

成功查找到如下字段

![](img/14b2d3fcebe23cc7229145e08a5652bf.jpg)

我们在进

`sub_147CE7F0(parm3, BSTR_address);`看一看

![](img/e30e8a9249e77c03fdcd3271b3424521.jpg)

改动影响不大。于是，

`memcpy(*parm3, (src_copy + 32), total_size - 32);` 3 个参数都可控，任意地址写任意数据最终造成漏洞

附件为 `encdec.dll` 补丁前后

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `EncDec.zip` |

