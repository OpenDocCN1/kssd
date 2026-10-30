
![](https://bbs.kanxue.com/upload/attach/202004/817966_AQ78N23HB3BVSBE.jpg)

将返回的调用地址加5后，通过遍历`msvcrt.dll`的输入表查找`CreateFile`：

![图片82 查找CreateFile](https://bbs.kanxue.com/upload/attach/202004/817966_7NAFPGK74R6UUY3.jpg)

![](https://bbs.kanxue.com/upload/attach/202004/817966_79X4BYQSN9V7X2M.jpg)

此次是查找`VirtualAlloc`：

![图片84 查找VirtualAlloc](https://bbs.kanxue.com/upload/attach/202004/817966_SMVTZ2BBSM2SC3Y.jpg)

接下来所查找函数不一一截图，依次是`ReadFile`、`WriteFile`、`CloseHandle`、`CreateProcess`、`GetModuleFileName`、`ResumeThread`、`TerminateProcess`。

![](https://bbs.kanxue.com/upload/attach/202004/817966_7K83SEA2589A98T.jpg)

其后传递给`GetProcess`的参数不再一一截图，依次是`ReadProcessMemory`、`VirtualQueryEx`、`VirtualProtectEx`、`GetModuleHandle`、`VirtualAllocEx`、`WriteProcessMemory`、`SetThreadContext`、`ZwUnmapViewOfSection`。

调用`GetTempPath()`：

![图片86 GetTempPath](https://bbs.kanxue.com/upload/attach/202004/817966_JT4E2JSSWPQAJCZ.jpg)

之后将其于临时文件夹内释放的文件名拼接到路径后：

![图片87 拼接路径](https://bbs.kanxue.com/upload/attach/202004/817966_EG622MTWVWWS35T.jpg)

打开该文件：

![图片88 CreateFile](https://bbs.kanxue.com/upload/attach/202004/817966_HJH9QPZJKR6N3NZ.jpg)

其后行为不再一一截图，依次是`GetFileSize`(获取该大小)、`VirtualAlloc(0,0x3E000,0x3000,0x40)`(分配空间)、`ReadFile`(读取文件到分配的空间内)。

解密内存中的文件内容：

![图片89 解密](https://bbs.kanxue.com/upload/attach/202004/817966_KGZFVG4CV8ZEHX2.jpg)

遍历文件句柄，找到符合下列条件的文件：

![图片90 遍历](https://bbs.kanxue.com/upload/attach/202004/817966_EDFXJPF7A9HMX2S.jpg)

![图片91 遍历结果](https://bbs.kanxue.com/upload/attach/202004/817966_FZSUMYXP3SVC94G.jpg)

调用`VirtualAlloc`分配空间并写入内容(并非解密后的文件内容)：

![图片92 分配空间](https://bbs.kanxue.com/upload/attach/202004/817966_8WKQVTZECAJQUSC.jpg)

![图片93 WriteFile](https://bbs.kanxue.com/upload/attach/202004/817966_W6M5H5CUZ5Z2QDQ.jpg)

![图片94 CloseHandle](https://bbs.kanxue.com/upload/attach/202004/817966_52BMB4BKRJWDTVD.jpg)

复制解密后的文件内容：

![图片95 复制文件内容](https://bbs.kanxue.com/upload/attach/202004/817966_8RYYJ3KSC6XEWPA.jpg)

之后创建一同名进程：

![图片96 创建进程](https://bbs.kanxue.com/upload/attach/202004/817966_RK7ZZS8G2CQU68X.jpg)

![图片97 创建完成](https://bbs.kanxue.com/upload/attach/202004/817966_FAK8MSWUNT8DY5E.jpg)

其后的部分行为见下图：

![图片98 GetThreadContext](https://bbs.kanxue.com/upload/attach/202004/817966_THQF8GDUGJDGK78.jpg)

![图片99 ReadProcessMemory](https://bbs.kanxue.com/upload/attach/202004/817966_N3UHFVDPUYC9AKV.jpg)

![图片100 VirtualQueryEx](https://bbs.kanxue.com/upload/attach/202004/817966_FJHTWBU3TAZT72W.jpg)

将解密后的文件内容写入创建的进程：

![图片101 写入到创建的进程](https://bbs.kanxue.com/upload/attach/202004/817966_CCCYHB98F64NWME.jpg)

最终，结束原进程：

![图片102 结束原来的进程](https://bbs.kanxue.com/upload/attach/202004/817966_TBCXFPGJUT8QSHN.jpg)

解密后的文件会在临时目录释放两个文件("白加黑")并运行之：

![图片103 释放文件](https://bbs.kanxue.com/upload/attach/202004/817966_PEAUU45GF99RMTF.jpg)

## 0x05 参考链接

- [手把手教你复现office公式编辑器内的第三个漏洞](https://bbs.kanxue.com/elink@35fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0V1@1z5o6b7I4)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
