
# armadillo 变换 IAT 的修复!

> 原文：[`www.pediy.com/kssd/pediy06/pediy6492.htm`](https://www.pediy.com/kssd/pediy06/pediy6492.htm)

**方法一:**

1. 修改跳转避开 IAT 加密(见有关介绍)
2. 用 `ImprtREC` 得到所有函数列表,并 `save tree`.
3. 把同一 dll 的所有 API 组合起来(任意顺序),建立一个正常的 IAT.然后根据同一 dll 的 API 组合顺序建立一个变换后在 IAT 中偏移表,每个 API 对应 2 字节的偏移.
4. 在程序入口根据同一 dll 的 API 组合顺序在 IAT 偏移表,重新变换处理成为一个不连续的 IAT.

**方法二:**

1. 修改跳转避开 IAT 加密(见有关介绍)
2. 在壳没有变换 IAT 前,用 `ImprtREC` 得到一个正常的 IAT.
3. 修复时在程序入口按照壳的变换规则(把壳的变换代码复制过来,稍微修改即可)处理 IAT.

**总结:**

方法一适应于所有任何变换 IAT 的壳(不仅仅是 ARMA).不管过程,只要 IAT 变换后的结果.

方法二必须要了解 Arma 壳的变换 IAT 过程.

`4dd466d0e1e0cc123afbe504da2462ca`
