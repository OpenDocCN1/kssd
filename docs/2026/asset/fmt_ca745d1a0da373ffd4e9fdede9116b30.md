
**1. 预检查**：在函数的开头部分，进行了一些预检查。包括检查 PE 文件的一些属性，例如是否需要完整性检查等。对应的代码在函数的开头部分，例如：

```
if (opt->exact)
    throwCantPackExact();
```

这一段代码检查了是否需要精确打包。如果需要，则抛出异常。

**2. 处理PE文件的各个部分**：在函数的中间部分，处理了PE文件的各个部分，包括导入表，资源，TLS(线程局部存储)，重定位等等。对应的代码在函数的中间部分，例如：

```
const unsigned dllstrings = processImports();
processTls(&tlsiv); // call before processRelocs!!
processLoadConf(&loadconfiv);
processResources(&res);
processExports(&xport);
processRelocs();
```

这一段代码处理了 PE 文件的导入表，TLS，加载配置，资源，导出表和重定位。

**3. 对文件进行打包和压缩**：在处理完所有的部分后，开始对文件进行打包和压缩。在这个过程中，可能会对文件进行一些修改，例如修改PE头部，添加或删除某些段等。对应的代码在函数的后半部分，例如：

```
callCompressWithFilters(ft, filter_strategy, ih.codebase);
```

这一段代码调用了一个函数来进行压缩，并使用了过滤器。

**4. 将处理后的数据写入到输出文件，并复制覆盖层**：最后，将处理后的数据写入到输出文件，并复制文件的覆盖层（如果存在的话）。对应的代码在函数的最后部分，例如：

```
fo->write(&oh, sizeof(oh));
fo->write(osection, sizeof(osection[0]) * oobjs);
...
copyOverlay(fo, overlay, obuf);
```

这一段代码将处理后的 PE 头部和各个部分写入到输出文件，然后复制文件的覆盖层。复制覆盖层是指在原始的可执行文件压缩过程中，一些数据并没有被压缩，这部分数据通常被称为覆盖层(overlay)。这可能包括一些附加的未压缩数据，**例如数字签名，不会被压缩。**在解压缩过程中，这部分覆盖层数据需要被直接复制到解压缩的文件中，而不需要进行解压缩处理。

## `p_w64pe_amd64.cpp`

接下来是 `p_w64pe_amd64.cpp`，`p_w64pe_amd64.cpp` 实现了针对64位PE文件的压缩打包逻辑，而 `pefile.cpp` 包含了PE文件格式的通用处理逻辑。`p_w64pe_amd64.cpp`是对`pefile.cpp`模板的复用：

1. `p_w64pe_amd64.cpp` 继承自 `Packer` 和 `PeFile` 类，具体实现了64位PE文件的压缩打包操作，`p_w64pe_amd64.cpp`中的`pack0()`方法是`PackW64PeAmd64`类中的成员方法，它调用了父类`PeFile`的`pack0()`模板方法进行压缩。

2. `PeFile` 类在 `pefile.cpp` 中实现，它包含了针对PE文件格式的通用处理逻辑，如读取PE头信息、导入表处理、重定位表处理等。

3. `p_w64pe_amd64.cpp` 重用了`PeFile`类的功能，并实现了64位PE特有的压缩打包逻辑。

4. 在 `p_w64pe_amd64.cpp` 中，关键的`pack()`方法会调用`PeFile::pack0()`进行实际的压缩工作。

5. `PeFile::pack0()` 实现了压缩的通用流程，读取原始文件信息、压缩主代码段、生成新头信息等。

6. 所以 `p_w64pe_amd64.cpp` 依赖并复用了 `pefile.cpp` 中`PeFile`类的通用功能，并在此基础上实现64位PE文件的特定处理。

> 对于64位PE，`p_w64pe_amd64.cpp`中的`pack0()`会实例化`PeFile::pack0<LE64>()`，而`PeFile32`和`PeFile64`则分别实例化`PeFile::pack0<LE32>()`和`PeFile::pack0<LE64>()`

接着分析`p_w64pe_amd64.cpp`的源码：
