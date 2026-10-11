
```
004D05B8    8B0D DC764F00   MOV ECX,DWORD PTR DS:[4F76DC]            ; ACProtec.004FC89C

004D05BE    A1 D0794F00     MOV EAX,DWORD PTR DS:[4F79D0]

004D05C3    8B00            MOV EAX,DWORD PTR DS:[EAX]

004D05C5    8B15 6C004C00   MOV EDX,DWORD PTR DS:[4C006C]            ; ACProtec.004C00B8

004D05CB    E8 78CDF7FF     CALL ACProtec.0044D348

................................................................

004D0584    55              PUSH EBP   //补充上替换的六个字节代码，OD 插件修正入口点 D0584,重建输入表选方式 2 或 Loadpe dump full

004D0585    8BEC            MOV EBP,ESP

004D0587    83C4 F4         ADD ESP,-0C

004D058A    B8 94024D00     MOV EAX,ACProtec.004D0294

004D058F    E8 0465F3FF     CALL ACProtec.00406A98

004D0594    A1 D0794F00     MOV EAX,DWORD PTR DS:[4F79D0]

004D0599    8B00            MOV EAX,DWORD PTR DS:[EAX]

004D059B    E8 90CDF7FF     CALL ACProtec.0044D330

004D05A0    8B0D AC774F00   MOV ECX,DWORD PTR DS:[4F77AC]            ; ACProtec.004FC8A4

004D05A6    A1 D0794F00     MOV EAX,DWORD PTR DS:[4F79D0]

004D05AB    8B00            MOV EAX,DWORD PTR DS:[EAX]

004D05AD    8B15 F4024C00   MOV EDX,DWORD PTR DS:[4C02F4]            ; ACProtec.004C0340

004D05B3    E8 90CDF7FF     CALL ACProtec.0044D348

004D05B8    8B0D DC764F00   MOV ECX,DWORD PTR DS:[4F76DC]            ; ACProtec.004FC89C

004D05BE    A1 D0794F00     MOV EAX,DWORD PTR DS:[4F79D0]

004D05C3    8B00            MOV EAX,DWORD PTR DS:[EAX]

004D05C5    8B15 6C004C00   MOV EDX,DWORD PTR DS:[4C006C]            ; ACProtec.004C00B8

004D05CB    E8 78CDF7FF     CALL ACProtec.0044D348

................................................................
```

## 4. 修复 IAT

关闭 OD，打开加壳程序，`ImportREC` 用跟踪等级 3 修复程序。

## 5. 运行脱壳修复程序

出现一个写内存错误提示，确定后进入程序界面，不能加密程序。

## 6. 怎么最近总是伪破解，伪脱壳

反复回顾过程，不知哪里有问题，反复几次仍然如此，估计过不了 `RSAKEY` 嵌入解码技术。

## 7. 虽然主程序修复脱壳失败

但它加密的程序强度就没有这么高，仍然不失为一个好方法，注意只适用于早期的 `Acprotect1.09g` 一下版本加密的程序。

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
