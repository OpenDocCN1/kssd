
# IDA7.0 中文字符搜索的实现方法(17.10.13更新内容)

## 2017.10.13更新

主要更新`CLT`文件内容,

去掉了`u20..u7F`,因为会影响到`F5`高亮功能(感谢地狱怪客 反馈),另外提供一种修改方法.

## 方法1

1: 保存`Chinese.clt`文件到`IDA\CFG`目录中.

将以下内容保存为`clt`文件,
```
u2000..u206F,
u2F00..u2FDF,
u3000..u303F,
u31C0..u31EF,
u3400..u4DBF,
u4E00..u9FFF,
uF900..uFAFF,
uFE30..uFE4F,
u20000..u2A6DF,
u2A700..u2BA7F,
u2B740..u2B81F,
u2F800..u2FA1F;
```
2: 修改`IDA.CFG`文件中`ENCODING_CULTURES`项目.添加一条`GB2312:Chinese`, 注意格式别写错.这里的`Chinese`对应上面保存的`clt`文件.不需要`.clt`扩展名.
```
ENCODING_CULTURES =
1250: Central_Europe,
1251: Cyrillic,
1252: Latin_1,
1253: Greek,
1254: Turkish,
1255: Hebrew,
1256: Arabic,
1257: Baltic,
1258: Vietnam,
874: Thai,
GB2312: Chinese, //这里是添加的.
cp863: Latin_1 Greek;
```

## 方法2

1: 同样保存`clt`文件到`ida\cfg`目录下,但并不修改`ENCODING_CULTURES`项目.

2: 添加`StrlitChars`项目`Culture_Chinese`,注意:这里使用`clt`文件需要加上`Culture_`作为声明.否则`IDA`不会查找`clt`文件并使用.

两种方法,使用的都是`clt`文件.当然也可以使用内置的`Culture_CJK_Unified_Ideographs`.该项目在`NameChars`中注释.

最后.看下同一个文件`OD`的`ustrref`插件搜索与`IDA`搜索的对比.

![](https://bbs.kanxue.com/upload/attach/201710/1258_fmri1ftwagg1uhb.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.jpg](javascript:void(0);)

  （50.90kb，78次下载）
