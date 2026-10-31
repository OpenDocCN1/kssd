
# 分享：编译 macOS 下的 LLDB

macOS 的 Xcode 中自带 LLDB。因为我写的项目需要把 LLDB 包裹到自己的子进程中，所以遇到了一个问题：当标准输入输出重定向到管道时，输出缓存会变为全缓存模式，无法及时读取 LLDB 的输出，导致我的程序无法工作。

因此，我想修改 LLDB 的源码来适应自己的项目，但改来改去也没有改好，有点弱。不过，我确实发现了用 Plink 连接 macOS 终端时，LLDB 命令行出现乱码的原因：原来 macOS 终端输出的文字可以改变颜色，那些乱码其实是颜色信息。

这里是官网的编译指引：

2d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4D9k6r3u0Q4x3X3g2D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6S2o6i4@1f1@1i4@1t1^5i4K6S2p5i4@1f1^5i4@1u0r3i4K6R3%4i4@1f1^5i4@1q4r3i4@1t1@1i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4@1q4q4i4K6R3H3i4@1f1%4i4K6W2n7i4@1t1@1i4@1f1%4i4@1q4q4i4K6R3H3i4@1f1#2i4K6S2p5i4K6V1#2i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1$3i4K6S2o6i4K6R3%4i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1#2i4@1t1&6i4@1t1$3i4@1f1@1i4@1t1^5i4K6V1@1i4@1f1$3i4K6R3^5i4K6V1I4i4@1f1$3i4K6S2o6i4K6R3&6i4@1f1%4i4K6R3#2i4@1p5%4i4@1f1#2i4@1q4q4i4K6V1^5i4@1f1%4i4@1u0p5i4K6V1I4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6V1$3i4@1t1&6i4@1f1$3i4@1t1K6i4K6V1#2i4@1f1$3i4@1t1J5i4@1p5I4i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1^5i4@1q4r3i4K6V1I4i4@1f1#2i4K6R3%4i4@1u0m8i4@1f1$3i4K6W2p5i4@1p5#2i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1$3i4K6R3^5i4K6V1I4i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1$3i4K6S2o6i4K6R3&6i4@1f1%4i4K6R3#2i4@1p5%4i4@1f1%4i4@1u0p5i4K6V1I4i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1#2i4K6R3#2i4@1t1$3i4@1f1@1i4@1u0n7i4@1K6V1$3i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4@1t1^5i4K6V1$3i4@1f1#2i4@1q4p5i4@1K6V1H3i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1^5i4@1q4r3i4@1K6V1I4i4@1f1$3i4@1K6R3^5i4@1K6V1H3i4@1f1#2i4@1K6S2m8i4@1K6W2r3i4@1f1%4i4@1K6W2m8i4@1K6R3@1i4@1f1K6i4@1K6R3H3i4@1K6R3J5

## 1. 签名

忘了这一步了，补充一下：如果不生成签名，编译出来的 LLDB 不能运行。

按照`lldb-master/docs/code-signing.txt`中的步骤操作。

1. 打开“钥匙串访问”程序，选中左侧的“登录”选项。
2. 在菜单中选择“钥匙串访问”→“证书助理”→“创建证书……”。
3. 填写以下信息：
   - 名称：`lldb_codesign`
   - 身份类型：自签名根证书
   - 证书类型：代码签名
4. 点击“创建”→“继续”→“完成”。
5. 选中左侧“种类”标签中的“我的证书”，双击刚创建的`lldb_codesign`证书。
6. 在“使用此证书时”标签中选择“始终信任”，然后关闭窗口。系统会弹出输入密码认证的窗口，输入管理员密码，再点击“更新设置”。
7. 将这个证书从“登录”选项卡用鼠标拖拽到“系统”选项卡。系统会出现两次输入管理员密码的窗口，输入密码后，在弹出的窗口中选择“始终允许”。
8. 将“系统”标签下的`lldb_codesign`证书拖拽到桌面。
9. 打开终端，执行以下命令：

```bash
sudo security add-trust -d -r trustRoot -p basic -p codeSign -k /Library/Keychains/System.keychain ~/Desktop/lldb_codesign.cer
rm -f ~/Desktop/lldb_codesign.cer
```

10. 将“系统”标签下的`lldb_codesign`证书拖拽回“登录”标签下。弹出输入管理员密码的窗口后，输入密码。
11. 重启系统，然后开始编译 LLDB。

## 2. 编译步骤

### 2.1 下载 LLDB

下载 GitHub 上的 LLDB，网址是：

44dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8s2k6E0i4K6u0V1L8h3W2J5M7X3!0J5i4K6u0r3L8r3I4V1j5R3`.`.，这个说是每 5 分钟同步一次官网数据，应该是可信的吧。

### 2.2 安装 SWIG

```bash
brew install swig
```

### 2.3 开始编译

解压完成后，在`lldb-master`文件夹中使用终端进入该文件夹，然后执行：

```bash
xcodebuild -configuration Release
```

编译完成后，`lldb-master`目录下的`build/Release`目录中就有 LLDB 了。

顺便说一下，我的 macOS 系统版本是 10.12。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
