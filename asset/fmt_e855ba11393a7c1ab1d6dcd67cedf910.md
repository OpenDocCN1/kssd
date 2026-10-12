
# FixGame.dll 的源码

`disasm.cpp`        od 的汇编引擎

`FixNanomite.cpp`        修复 Nanomites

`FixIat.cpp`         修复 IAT

`FixPeHeader.cpp`        修复偷到 PE 头里面的代码

`FixSDK.cpp`        修复 SDK 部分

`GetHash.cpp`        壳用的 hash 函数

其中 SDK 我是手工修复的，这里只是贴代码而已，其他 3 个都是程序实现的

Nanomite 我是暴力枚举 hash，还是 ccfer 的代码比较强，用特征码去判断

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `FixGame.rar` |
