
# 逆向工程与脱壳经验总结

## 1. 恢复 OEP 与初步分析

`004E6F9E` `^ E9 BD90FEFF` 指令跳转至 `wscut.004D0060`，返回 OEP。
`004E6FA3` `90` 执行 `nop`。

返回 OEP 后，撤消代码的修改，再打开 `ImpoetREC`，输入 `OEP : D0060`，点自动查找，发现很多函数正常了，但还有少数函数不正确！这是壳加密的原因！

## 2. 使用工具修正 IAT

我们用 `LOADPE DUMP` 出文件，使用 `ImpoetREC` 修正 `IAT`，此时运行程序出错，说明还有函数未修复！

下面是手工活，我们举一个为例，其他相同：

## 3. 代码修改详解

原始代码片段：

```
0042D306  |.  8BF0                mov esi,eax
0042D308  |.  8935 EC4A4D00       mov dword ptr ds:[4D4AEC],esi
0042D30E  |.  85F6                test esi,esi
0042D310  |.  75 0A               jnz short 12345_.0042D31C
0042D312  |.  B8 A0784000         mov eax,<jmp.&user32.ShowScrollBar>
0042D317  |.  A3 EC4A4D00         mov dword ptr ds:[4D4AEC],eax
0042D31C      68 80D44200         push 12345_.0042D480                ;  ASCII "FlatSB_GetScrollRange"
0042D321      53                  push ebx
0042D322      E8 599CFDFF         call 12345_.00406F80                //此处跳向壳内,解密函数!
0042D327  |.  8BF0                mov esi,eax
0042D329  |.  8935 F04A4D00       mov dword ptr ds:[4D4AF0],esi
0042D32F  |.  85F6                test esi,esi
0042D331  |.  75 0A               jnz short 12345_.0042D33D
```

上面的跳向壳内的加密也很直观，就是把 `FlatSB_GetScrollRange` 转换为相应的函数地址，就 OK 了。为此，我们将上面代码改为：

修改后的代码片段：

```
0042D306  |.  8BF0                mov esi,eax
0042D308  |.  8935 EC4A4D00       mov dword ptr ds:[4D4AEC],esi
0042D30E  |.  85F6                test esi,esi
0042D310  |.  75 0A               jnz short 12345_.0042D31C
0042D312  |.  B8 A0784000         mov eax,<jmp.&user32.ShowScrollBar>
0042D317  |.  A3 EC4A4D00         mov dword ptr ds:[4D4AEC],eax
0042D31C      B8 72011B5D         mov eax,comctl32.FlatSB_GetScrollRange
0042D321      90                  nop
0042D322      90                  nop
0042D323      90                  nop
0042D324      90                  nop
0042D325      90                  nop
0042D326      90                  nop
0042D327  |.  8BF0                mov esi,eax
0042D329  |.  8935 F04A4D00       mov dword ptr ds:[4D4AF0],esi
0042D32F  |.  85F6                test esi,esi
0042D331  |.  75 0A               jnz short 12345_.0042D33D
```

也就是直接给 `EAX` 赋值函数地址，就可以了，其余还有几处相同！

## 4. 结果验证

这样，重建 PE 后，再运行程序一切 OK，已没有 NAG 了，也没有试用期限了！

## 5. 经验总结

该程序没有 SDK，也没有自己的加密方法，全是依靠壳的加密，也是程序的失败之处。解壳后程序也就昭然于天下，这也是软件作者要注意的问题。本人也是软件加密的学习者，欢迎你和我交流！

## 6. 版权声明

谢谢观看，转载请注明作者并保持文章的完整，谢谢！

2006 年 08 月 25 日 22:14:25

`4dd466d0e1e0cc123afbe504da2462ca`
