
# 水芯 Flash 播放器 v1.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue408.htm`](https://www.pediy.com/kssd/pediy04/kanxue408.htm)

简介：

1. 支持播放列表功能
2. 支持音量的调节，有静音功能键
3. 支持播放质量的调节
4. 支持大窗口播放
5. 可以存储和读取播放列表

工具：`ollydbg` + `keymake`

级别：3 级（一共十级）

步骤：

1. 加载软件
2. `Alt+E` 切换到 `Executable modules` 模式
3. 选中 `oleaut32.dll`
4. 单击右键选择 `View Name`
5. 输入 `VarBstrCmp`
6. 单击 `F2`, 下断点
7. `F9`, 启动软件
8. 输入注册网站（好象是他给的）和注册码
9. 按下“输入”按钮
10. 被中断在 `653BB9B9 >55           PUSH EBP`
11. 在右下角的里有真的注册码

注册网站：`http:\\www.shareware.com.cn`

注册码：`789000  4dd466d0e1e0cc123afbe504da2462ca`
