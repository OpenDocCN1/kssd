
# 破解笔记

## 反汇编片段

```
00507A85  |. 38D9              CMP CL,BL         //比较真假。
00507A87  |. 75 41             JNZ SHORT FRMMSTR.00507ACA     //不同则OVER。
00507A89  |. 4A                DEC EDX          //计数器减量
00507A8A  |. 74 17             JE SHORT FRMMSTR.00507AA3       //为0则跳走。
00507A8C  |. 38FD              CMP CH,BH
00507A8E  |. 75 3A             JNZ SHORT FRMMSTR.00507ACA
00507A90  |. 4A                DEC EDX   //计数器减量
00507A91  |. 74 10             JE SHORT FRMMSTR.00507AA3      //为0则跳走。
00507A93  |. 81E3 0000FF00     AND EBX,0FF0000                //如果还有第三个字节，则用与运算取出。
00507A99  |. 81E1 0000FF00     AND ECX,0FF0000                //同上。
00507A9F  |. 39D9              CMP ECX,EBX                    //比较真假。
00507AA1  |. 75 27             JNZ SHORT FRMMSTR.00507ACA     //不同则OVER。
00507AA3  |> 01C0              ADD EAX,EAX                              ;  FRMMSTR.<ModuleEntryPoint>
00507AA5  |. EB 23             JMP SHORT FRMMSTR.00507ACA     //从这里跳走才算成功的。
*省去多行
00507ACA  |> 5F                POP EDI
00507ACB  |. 5E                POP ESI
00507ACC  |. 5B                POP EBX
00507ACD  \. C3                RETN
```

## 后记

经过1个小时的跟踪，1个小时写出破文来，虽然这个软件最后是明码比较，我还没有发现这个真码怎么计算出来的，好象不明显。进入前面那个比较真假的函数，你立即就可以看到真码了。当然还有10几个假码在那里列出来了，不要被它所迷惑哟。只有用它自动取的那个注册码才可以成功（废话，假码跟它也不相等啊。）

虽然没有困难找出注册码，但如何分析出注册码的算法是关键。我还需要继续分析。先把这个笔记POST出来再说吧。

浪费您时间阅读此文，十分感谢！再次向关注我的破文的各位表示感谢！！

## 结果

`UserName`:     wanggang

`RegisterCode`: D8383VMDKKAAAL-LLF4VVIII34AAAAA-LLF4VVIII34AA1-1

2006年1月17日晚完稿

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
