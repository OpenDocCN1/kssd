
# [原创]ApkAssist(Apk一键捆绑工具)

## 功能

- `AndroidManifest.xml`反编译、编译、合并，指定`so DT_NEEDED`注入我们的so。
- Apk一键捆绑，两种姿势：
  1. `so DT_NEEDED`注入。
  2. `dex smali`注入。

## 使用说明

1. 确定系统是64位Windows，确定已安装64位版jdk 8并添加`bin`目录到`Path`环境变量。
2. 命令行：

```bash
java -jar ApkAssist.jar assit [-d -b -m -n]
```

- `-d AndroidManifest_.xml AndroidManifest.xml` 反编译`AndroidManifest.xml`到`AndroidManifest_.xml`
- `-b AndroidManifest.xml AndroidManifest_.xml` 编译`AndroidManifest_.xml`到`AndroidManifest.xml`
- `-m AndroidManifest.xml AndroidManifest_.xml` 合并`AndroidManifest_.xml`（反编译过的）到`AndroidManifest.xml`（未反编译过的）。
- `-n libjiagu.so libassist.so` 注入`libassist.so`到`libjiagu.so`。

```bash
java -jar ApkAssist.jar [-n -l]
```

- `-n libjiagu.so,libjiagu_x86.so libassist.so test.apk` 注入`libassist.so`到`libjiagu.so,libjiagu_x86.so`，并捆绑。（`so DT_NEEDED`注入）
- `-l assist test.apk` 在主dex的主类插入静态构造函数，插入加载`libassist.so`代码，并捆绑。（`dex smali`注入）

主类：`App Entry`类，没有则为`Main Activity`类。
主dex：主类所在的dex。

## 编写捆绑so

在你的so代码里插入编译器构造函数，函数里加载我的`libassist.so`。

例：

```c
void __attribute__ ((constructor)) load() {
    dlopen("libassist.so", RTLD_NOW);
    // 你的hook代码
}
```

生成Apk之后，复制`lib`目录到本工具`inject`目录。

## 目录说明

- `inject/assets` 将复制到Apk的`assets`目录。
- `inject/dex` 将目录下所有dex合并为`assist`，并复制到Apk的`assets`目录。
- `inject/lib` 将目录下与Apk的`lib`目录下同名的文件夹复制过去。

## 皮一下

发现BUG？一定是你姿势不对！打死我也不会改的！给源码自己折腾去！

源码在哪？回帖过100就有了！

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-7-19 03:31 被cdarksky编辑，原因：

上传的附件：

- [ApkAssist.7z](javascript:void(0);)
  （4.44MB，788次下载）
