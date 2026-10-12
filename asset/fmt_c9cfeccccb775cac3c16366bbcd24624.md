
# LeapFTP 2.7.0

**File Version:** `2.7.0.550`

> 原文：[`www.pediy.com/kssd/pediy03/forum971.htm`](https://www.pediy.com/kssd/pediy03/forum971.htm)

## 软件信息

*   **软件名称**：`LeapFTP 2.7.0` (FileVersion: `2.7.0.550`)
*   **可执行文件**：`LeapFTP.EXE` (`966KB`)
*   **下载地点**：`http://www.leapware.com`
*   **发信人**：井 风
*   **时间**：`2000-12-08`
*   **破解工具**：`Trw20001.22`, `W32Dasm8.93`, `Hew Workshop3.0`

## 前言

此软件功能很强，十分好用，是一个极好的 FTP 软件。很久没有升级，最近才升为 `2.7` 版本。

计算和验证注册码过程比较复杂，所以用修改代码法破解。

## 过程

只要用 16 进制编辑器将下 5 个跳转改成相对应的非跳转即可（`JZ` 改成 `JNZ`、`JNZ` 改成 `JZ`）。

```text
1: 004871A0   offset   865A0
2: 004871AE   offset   865AE
3: 004871D4   offset   865D4
4: 00495844   offset   94C44
5: 0049571E   offset   94B1E
```

## 小结

将这此跳转改过来之后就不会有任何功能限制，只是窗口的 `CAPTION` 中可能存在未注册字样。

如果你觉得难看可以去掉（很容易）。

## 后记

有疑问请与我联系：`hz.cy@163.net`
`4dd466d0e1e0cc123afbe504da2462ca`
