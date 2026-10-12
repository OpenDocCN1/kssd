
# Easy File & Folder Protector 4.82 算法分析

> 原文：[www.pediy.com/kssd/pediy10/63305.html](https://www.pediy.com/kssd/pediy10/63305.html)

- **破文作者**：tianxj
- **作者邮箱**：tianxj_2007@126.com
- **作者主页**：www.chinapyg.com
- **破解工具**：PEiD, OD
- **破解平台**：Windows XP
- **软件名称**：Easy File & Folder Protector 4.82 (2008-4-1 版)
- **软件大小**：746KB
- **软件语言**：英文
- **原版下载**：自己找一下
- **保护方式**：注册码
- **软件简介**：
  一个非常 Easy 的文件和文件夹保护程序。只要你给 Easy File & Folder Protector 设置一个密码，你就可以保护你的任何文件和文件夹，没有这个密码任何人都无法访问经过 Easy File & Folder Protector 保护的文件和文件夹。对文件夹进行保护时文件夹进行了隐藏，这样就显得更安全。此外，它还可防止别人通过网络访问你的文件。如果你要保护自己的隐私，你就要试试 Easy File & Folder Protector 了。
  - 使用方法：
    - 启动保护："添加文件或文件夹" --> "启动"按钮 --> "应用"按钮
    - 解除保护："点选文件列表中的文件或文件夹" --> "解除"按钮
- **破解声明**：
  我是一只小菜鸟，偶得一点心得，愿与大家分享：）
  初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 一、运行程序，进行注册，输入错误的注册信息进行检测

有提示信息 `"registration key is not valid"`

## 二、用 PEiD 对这个软件查壳

为 Borland Delphi 6.0 - 7.0

## 三、用 PE Explorer 和 DeDe 查找按钮事件

```
004A2284   /.  55             PUSH EBP
```

## 四、运行 OD，打开 EFPP，输入错误的注册信息

代码:
