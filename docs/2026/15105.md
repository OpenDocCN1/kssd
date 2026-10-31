
# [原创]Ali 阿里聚安全加固的libdemolishdata.so格式分析及代码还原的一种方案

将apk上传到阿里云进行快速加固，下载下来可以发现最新的加固和参考资料(1)中的类似, `libdemolishdata.so` 的位置放在了 `assets\ali` 中，不过格式不变。

但很可惜该资料中并没有给出将阿里云抽取到 `libdemolishdata.so` 二进制代码还原回 `dex` 的方法，修改代价高(伸手党做不成了)，所以需要继续分析(本文在参考资料(1)的基础上分析，建议先阅读参考资料(1))。

想着动态调试太麻烦于是直接开始猜，用 `winhex` 打开 `libdemolishdata.so`，如图:

![](https://bbs.kanxue.com/upload/attach/201708/563104_5cck7bsqw7nxi1q.png)

可以发现位于 `0x10` (表示由 `0x10-0x10+3` 构成的 `DWORD`，下同), `0x20`, `0x30` 的位置的内容恰好是 `fixfunc` 调用 的内容。

那么可以推测 `0xC` 表示待修复方法个数。

用 `baksmali` 输出 `method` (`java -jar baksmali.jar list methods dex文件地址`), 可以发现 `0x18`, `0x28`, `0x38` 对应着三个被修改为 `"native"` 方法的 `methodIdx`。

学习了参考资料(2)后，可以发现 `0x1c`，`0x2c`, `0x3c` 作为 `offset` 指向的内容恰好是 `DexCode` 格式，所以关键是修复 `codeoff`。

又尝试上传了一个有 `multidex` 的 `apk`，可以猜出 `libdemolishdata.so` 其他全部信息的含义(过程省略)，如下(所有名称都是我为了方便随便命名的)

文件宏观组成如下

```
struct libDemolishData{

AliFixFuncInfo aliFixFuncInfos[dexNum];//每个AliFixFuncInfo对应一个dex的修复信息

AliCheckSumInfo aliCheckSumInfo[dexNum];//每个AliCheckSumInfo 对应一个dex的CheckSum

uint dexNum; //dex数量,如：有classes.dex和classes2.dex,则为2；只有classes.dex，则为1

};
```

`AliCheckSumInfo` 的组成比较简单，如下(可能用于检验加固后的内容是否被修改)

```
struct AliCheckSumInfo{

uint multiDexId;//dex标识，0表示classes.dex,否则表示classes{$multiDexId}.dex,如2表示classes2.dex

uint checksum;//该dex的alder32,与DexHeader.checksum相同

uint hash;//该dexSha1的一部分,与DexHeader.signature的前4字节相同

};
```

`AliFixFuncInfo` 则由三部分组成，如下

```
struct AliFixFuncInfo{

struct Header{

char magic[4];//魔数,恒为 <ali

uint multiDexId;//dex标识，0表示classes.dex,否则表示classes{$multiDexId}.dex,如2表示classes2.dex

uint dataLen;//这一AliFixFuncInfo的长度，可以借此定位下一个AliFixFuncInfo在哪

uint funcsNum;//需要修复的方法数

} header;

struct FixFunc{

uint id;//方法标志，同fixfunc调用的参数，如class调用 fixfunc(int[]{id1,id2,...}) 时，id1,id2,...就是这个id

uint multiDexId;//dex标识，0表示classes.dex,否则表示classes{$multiDexId}.dex,如2表示classes2.dex(表示似乎冗余了，为0x16对齐？)

uint methodIdx;//待修复方法的methodIdx，methodIdx含义见参考资料(2)

uint codeOffset;//指向待修复方法的真实DexCode信息，注意表示的是真实DexCode信息相对于当前AliFixFuncInfo的偏移，而并不完全相当于相对于文件头的偏移

} fixFuncs[funcsNum];

char realDexCodeInfo[dataLen-sizeof(header)-sizeof(fixFuncs)];

};
```

分析到这里可以说是很开心，以为只要在 `dex` 文件中找到 `codeOff` 改掉即可，然而 `codeOff` 是 `Uleb128` 类型的，这是一种变长类型，一旦改掉，需要修复很多偏移信息。

于是想到我们可以修改 `baksmali` 读取到的 `codeOff` 啊，再 `smali` 回去即可。

可以想象(伸手党放心，这部分附件有成品)，先增加几个类，用于读取 `libdemolishdata.so`，在读取 `dex` 文件的函数中将 `libdemolishdata.so` 的内容复制在存储 `dex` 内容的 `byte[]` 末尾,然后修正读取到的 `accessFlag` (去除 `native` 标记【`0x100`】)，修正读取到的 `codeOff` (获取到相对于当前 `AliFixFuncInfo` 的偏移后要加上 `AliFixFuncInfo` 相对于 `libdemolishdata.so` 文件头的偏移和前文中的 `byte[]` 中 `libdemolishdata.so` 信息的偏移)，这样就可以了。

附件中 `modifiedJava.zip` 是修改后的 `java` 源代码(仅修改的部分，修改前的 `baksmali` 程序的 `java` 源代码可在 `github` 下载，编译采用 `javac -cp 编译好的.jar;. \*\*\*.java`，然后将新生成的 `\*.class` 用压缩软件复制到 `jar` 内对应目录即可)；

`bakdemolish-2.2.1.jar` 是修改编译后的成品(使用说明：

1. 正常 `dex` 和修复后 `dex` 都用正常 `baksmali` 即可，不要用这个。
2. 操作命令同正常 `baksmali`，如 `java -jar bakdemolish-2.2.1.jar d classes.dex`。
3. 但须注意，将 `assets\ali` (我获得的最新加固后文件)或 `lib\\*\*` \(参考资料(1)中加固后文件,2017.01)下的 `libdemolishdata.so`、`bakdemolish-2.2.1.jar`、`classes.dex` 文件放到相同文件夹下，并保证命令提示符当前目录也 `cd` 到这一目录时才能确保正常工作【其实就是 `libdemolishdata.so` 直接是用 `FileInputStream("libdemolishdata.so")` 取读的问题，没有测试其他情况】
4. `dex` 文件名不能改【用来获取前文提到的 `multiDexId`，如 `apk` 内是 `classes2.dex`, 就是 `classes2.dex`】
5. `dex` 可以直接解压获取，也可以用 `apktool d -s apk文件` 获取)

有了这个工具，可以想象如下修复步骤(没有完全测试过)

1. `apktool d -s apk文件`
2. 获得 `libdemolishdata.so` 后用 `bakdemolish-2.2.1.jar` 获得 `smali Code`
3. 适当修复，最简单的是将 `com\ali\fixHelper.smali` 中所有方法内容都删除，然后删除 `fixfunc` 方法的 `native` 属性，补上 `.locals 0,return-void`, 如下

```
.method public static fixfunc([IZ)V

.locals 0

return-void

.end method
```

也可以用一些规则匹配删除所有 `<clinit>` 中调用 `fixfunc` 的代码

注意处理后的文件仍有一些暗桩似乎和 `a/does/not/Exists\*.smali` 有联系，可能需要处理

4. 用 `smali-*.jar` 回编译
5. 用 `apktool b <dir>` 生成新 `apk`，签名

参考资料:

1. [Caln](http://bbs.pediy.com/user-729487.htm)  ali 加固(17年1月)逆向分析 http://bbs.pediy.com/thread-217235.htm
2. 太尼玛菜了 Android Dex文件格式(一) 20bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5h3y4S2K9h3&6A6j5h3!0Q4x3V1k6H3i4K6u0r3y4U0l9K6y4e0t1%4y4q4)9J5k6h3S2@1L8h3H3`.
            Android Dex文件格式(二) 21cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5h3y4S2K9h3&6A6j5h3!0Q4x3V1k6H3i4K6u0r3y4U0l9K6y4U0R3K6y4q4)9J5k6h3S2@1L8h3H3`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [bakdemolish-2.2.1.jar](javascript:void(0);)
  （3.30MB，167次下载）
- [modifiedJava.zip](javascript:void(0);)
  （14.58kb，180次下载）
