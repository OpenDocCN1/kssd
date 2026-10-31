
# React Native App逆向

## 一. 前言

闲来无事，看到有小伙伴在交流 app 逆向的问题，于是我加入了他们。

## 二. 分析

### 1. 首先，抓包康康到底是加密了啥玩意.

![](upload/attach/202006/781457_GMV5U88TZ4U22B4.png)

### 2. APK，用查壳工具看到的腾讯加固的，dump 出 dex.

### 3. 康康 dump 出的 dex 内容

![](upload/attach/202006/781457_PGNN3TNDR6CPPHY.png)

乍一看，React 的 App，Google 搜索下答案

据说要找 `index.android.bundle` 文件

![](upload/attach/202006/781457_TS7SA33MGEW7NJU.png)

### 4. 分析 `index.android.bundle`

由于它都是 JS 代码，所以格式化下。加密猜着，应该搜索 `encry`。很容易搜到加密的关键位置

![](upload/attach/202006/781457_GW7Q4TXUHAUM3D2.png)

一路寻找，最后要找 `key` 和 `iv`.

![](upload/attach/202006/781457_K8M3VG38AEZRPMV.png)

### 5. ReactContextBaseJavaModule

康到文章的解释。那么我们直接去 dump 出的 dex 里寻找 `"RNExitApp"` 的字符串

![](upload/attach/202006/781457_D4NN7DDM59ND9EW.png)

找到了，解决。

![](upload/attach/202006/781457_JRWHUME8H826Z4X.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
