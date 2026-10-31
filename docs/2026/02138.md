
# freeceltool 空当接龙工具破解手记 (1千字)

## freeceltool 空当接龙工具破解手记

作者：孙锋

Email：`sffs@263.net`

主页：`http://sffs.6to23.com`

下载地址是：`http://www.win2000.com.cn/down/fceltool.exe`

用`W32DASM8.93`（中文版）将它反汇编，在串试数据参考里找到注册失败的对话，到达它的位置，看看它是从那里来得，代码如下：

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00409AB8(C)

|

:00409B29 `6A00`

  push `00000000`

:00409B2B `6A00`

  push `00000000`

* Possible StringData Ref from Data Obj ->"注册码错误,注册失败!"

            |

:00409B2D `6808CD4F00`              push

`004FCD08`

:00409B32 `E85A6E0800`              call

`00490991`

到`00409AB8`看看

:00409AB8 `756F`

  jne `00409B29`

用16进制工具把`75`改`74`。

再次查找到“未注册版”这个字符串。代码如下：

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403950(C)

|

* Possible StringData Ref from Data Obj ->"未注册版"

            |

:004039B7 `6880B44F00`              push

`004FB480`

:004039BC `8B8D74FFFFFF`            mov ecx, dword

ptr `[ebp+FFFFFF74]`

:004039C2 `81C104010000`            add ecx, `00000104`

:004039C8 `E8D0BD0600`              call

`0046F79D`

好了，到`00403950`处，代码如下：

:00403950 `7465`

  je `004039B7`

将`74`改为`75`。

再查找“在未注册版本中，批量求解的局数不能超过3局”。找到位置如下：

* Possible StringData Ref form Data obj ->"(中间是找到的字符串,三行)"

:004051E3 `6864C0F00`  PUSH `OO4FC064`

:...

:...

* Referenced by a (U)nconditional or (C)onditional Jump at Addreeses:

|:004051D9(C),:004051DD(C)

就找这两个地址：

:0051C9 `7539` jne `00405204`

改 `75` 为 `74`

:004051DD `7E25` jle `00405304`

改 `7E` 为 `74`

完成注册破解过程。

成为任意注册版。

<------完------>
