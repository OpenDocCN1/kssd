
# bambam 壳的几个 bug

> 原文：[`www.pediy.com/kssd/pediy11/118304.html`](https://www.pediy.com/kssd/pediy11/118304.html)

两个壳的源代码参考的看了下，一个是《加密与解密》提供的源代码，还有就是 bambam 壳。不过 bambam004 壳有几个 bug。bambam004 壳是 bedrock 在 exetools 论坛提交的，可惜没什么讨论的帖子谈到其中的 bug。

`http://www.pediy.com/bbshtml/bbs8/pediy8-557.htm` 谈到以下几个 bug:

1. 在对 `stub` 进行重定位处理时，有行代码错行了。
2. 在 转换 `stub` 中 `thunk` 值时，搜索地址的方法有错漏。
3. 没有对 `pe` 文件中的多余数据(`pe` `section` 中未标明的)进行保留处理。
4. 没有对 `stub` 的 `VOffset` 和 `ROffset` 进行对齐处理
5. 对 `stub` 的输入表起始地址处理有错

我在 `vs2008` 中编译，大致修改了以下几处 bug，用 `dbgView` 测试了下，加壳之后能正常跑。

## 一、输入表通配符查找有错漏

`PatternSearch` 修改如下

```
int CCompressDlg::PatternSearch(unsigned char *SearchString, int StringLen, unsigned char *SearchBuff, int BuffSize)
{
    int i,j;
    if (BuffSize < StringLen) return -64;
    for(i=0;i<(BuffSize-StringLen);i++)
    {
        for(j=0;j<StringLen;j++)
        {
            if (
                (SearchString[j] != _XX_) &&
                (SearchString[j] != SearchBuff[i+j]) &&
                (j==2 || SearchString[j]+1 != SearchBuff[i+j])
               )
            break;       
        }
        if (j==StringLen)  
    {
      return i;
    }
    }
    return -100;
}
```

## 二、在对 stub 进行重定位处理时，有行代码错行了

`pIBRCurrent = (PIMAGE_BASE_RELOCATION)&((BYTE*)pIBRCurrent)[pIBRCurrent->SizeOfBlock];`

放在 `for` 循环外面

## 三、IMAGE_SIZEOF_BASE_RELOCATION 没有定义

这样定义 `IMAGE_SIZEOF_BASE_RELOCATION`

```
DWORD IMAGE_SIZEOF_BASE_RELOCATION = 2*sizeof(DWORD);
```

## 四、如果 PE 文件第一个 section 的尺寸为 0 的话，加密之后运行提示错误

bug 位置：加壳程序

```
for(i = 0; i < wNumSections; i++)
{
    // Reset resource flag
    bResource = false;
    pSecHdr = (PIMAGE_SECTION_HEADER)dwSecStart;
    pBackupSecHdr = (PIMAGE_SECTION_HEADER)dwBackupSecStart;
    if(strcmp((char*)pBackupSecHdr->Name, ".rsrc") == 0)
    {
      // we are at resource section
      bResource = true;     
    }
    // skip sections with raw size = 0
    if(pBackupSecHdr->SizeOfRawData == 0)
    {     
      // move to next section header
      dwSecStart += sizeof(IMAGE_SECTION_HEADER);
      pBackupSecHdr += sizeof(IMAGE_SECTION_HEADER);     //!sld! 应该改为 dwBackupSecStart +=sizeof(IMAGE_SECTION_HEADER);
      bSkipedZeroSizeSection = true;
      continue;
    }
            ......
           // move to next section header
    dwSecStart += sizeof(IMAGE_SECTION_HEADER);
    dwBackupSecStart += sizeof(IMAGE_SECTION_HEADER);
}
```

`http://www.pediy.com/bbshtml/bbs8/pediy8-557.htm` 一文所提到的第四跟第五我检查了下不存在错误。第三个 bug:没有对 `pe` 文件中的多余数据(`pe` `section` 中未标明的)进行保留处理。没注意。至少我测试 `dbgView` 能跑。

## 另外有几个值得注意的地方

`stub` 项目按照源码设置，编译的话会报 `memcpy`,`memset` 之类的函数未定义错误，这时候不能静态链接 `msvcrt.lib`，否则运行会报 `R6034` 错误。两条路：要么去掉 `/nodefaultlib` 配置项，要么自定义运行库，可以参考《Reduce EXE and DLL Size with LIBCTINY.LIB》

由于 `stub` 壳中不含有重定向，所以无法给 `DLL` `PE` 文件加壳，当然折中方案不是没有，可以在目标 `dll` 文件强制指定 `Image base`，再加壳。运行的时候祈祷指定的 `Image base` 没有被占用。当然最终极的解决方案还是像《加密与解密》一书的壳那样转储重定位表，然后壳刚开始运行重定位。

发现看雪论坛精华壳的源代码都是汇编写的，只有这个 `bambam` 是用 C/C++写的。小弟读汇编都有些吃力，更别提写汇编代码了(有学好汇编的想法)。这样的话改写壳只有在 `bambam` 基础上进行了，目前觉得 `bambam` 壳代码写的不错，还是挺有学习价值的。

ps.《加密与解密》一书加壳一章提到的壳中提供函数供源程序调用以提高壳的强度，有没有现成的例子或是更详细的参考资料？小弟冥思苦想不得其门而入。

如果使用 `bambam004` 壳支持 `DLL` 遇到困难，可以参考 `http://bbs.pediy.com/showthread.php?t=119570&highlight=`
