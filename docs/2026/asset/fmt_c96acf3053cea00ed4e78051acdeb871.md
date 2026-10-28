
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `foreach($method` `in` `$methods)`  `{`  `if` `(``-``not` `$method.HasBody){``continue``}`  `foreach($instr` `in` `$method.MethodBody.Instructions.ToArray())`  `{`  `if` `($instr.OpCode.Name` `-``like` `"call"` `-``and` `$instr.Operand` `-``eq $decrypt_method)`  `{`  `$index` `=` `$method.MethodBody.Instructions.IndexOf($instr)`  `$para_1_instr` `=` `$method.MethodBody.Instructions[$index` `-` `1``]`  `if` `(``-``not` `$para_1)`  `{`  `Write``-``Host` `"Someting went wrong, para was not found!"` `-``ForegroundColor Red;`  `Exit`  `}`  `$decrypted_string` `=` `$module_refl.ResolveMethod($instr.Operand.MDToken.ToInt32()).Invoke($null, $para_1_instr.Operand)`  `$method.MethodBody.Instructions[$index` `-` `1``].OpCode` `=` `[dnlib.DotNet.Emit.OpCodes]::Ldstr`  `$method.MethodBody.Instructions[$index` `-` `1``].Operand` `=` `$decrypted_string`  `$method.MethodBody.Instructions.RemoveAt($index)`  `}`  `}`  `$method.MethodBody.UpdateInstructionOffsets() | Out``-``Null`  `}` |

**成果**

随便举两个例子，可以看到，我们想要的效果已经实现了，这样分析起来就非常的流畅并且舒适了，最关键的是还可以正常进行动态调试。

![](https://bbs.kanxue.com/upload/attach/202308/944577_S4QPM75DCGYSS35.png)

![](https://bbs.kanxue.com/upload/attach/202308/944577_X8N2B3EH933FXCX.png)

![](https://bbs.kanxue.com/upload/attach/202308/944577_KRU236URJKHJ4HA.png)

但这真的完美了吗，还没有！我们可以更近一步，我们可以将"ldsfld $VAR"，即"b.R"以及"b.s"这种变量直接替换为字符串，这样可以更加方便我们阅读，也同时解决了上文中提到的"以变量形式存储"和"以内存形式存储"这两种情况，这样才是一个趋近于完美的解决方案。

![](https://bbs.kanxue.com/upload/attach/202308/944577_U67N8VQNXUA97GK.png)

![](https://bbs.kanxue.com/upload/attach/202308/944577_DG7GCVAW9BUDJWT.png)

### 找到目标赋值的所有交叉引用 & patch IL

其中原理和上一步相同，本文便不再过多赘述，直接查看下面的`ps1`代码即可。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `foreach($method` `in` `$methods)`  `{`  `if` `(``-``not` `$method.HasBody){``continue``}`  `foreach($instr` `in` `$method.MethodBody.Instructions.ToArray())`  `{`  `if` `(``-``not` `($instr.OpCode.Name` `-``like` `"ldsfld"` `-``and` `$instr.Operand.IsField)){``continue``}`  `$index` `=` `$method.MethodBody.Instructions.IndexOf($instr)`  `$field` `=` `$instr.Operand.FullName`  `if` `($field | Select``-``String` `-``Pattern` `"A\.b"``)`  `{`  `$``str` `=` `$module_refl.ResolveField($instr.Operand.MDToken.ToInt32()).GetValue($null)`  `if` `(``-``not` `$``str``){``continue``}`  `if` `(``-``not` `($``str``.GetType().Name` `-``eq` `"String"``)){``continue``}`  `$method.MethodBody.Instructions[$index].OpCode` `=` `[dnlib.DotNet.Emit.OpCodes]::Ldstr`  `$method.MethodBody.Instructions[$index].Operand` `=` `$``str`  `}`  `}`  `$method.MethodBody.UpdateInstructionOffsets() | Out``-``Null`  `}` |

**成果**

再来看效果，**Perfect！！！**，这就是我们想要的，并且还能正常调试，完美收工！

![](https://bbs.kanxue.com/upload/attach/202308/944577_YEJSEMU8F6GQ3KJ.png)

![](https://bbs.kanxue.com/upload/attach/202308/944577_CE35JJD959G54QV.png)

# 总结

本文提供了三种利用`powershell`自动化处理`.NET`样本混淆的方案，可以极大的提高分析效率，降低重复工作，希望大家有所收获，读的开心，也欢迎大家讨论和分享自己平时所用的分析方法和技巧。最后，祝大家生活愉快！！！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2023-8-4 17:41

被`g0mx`编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[# .NET平台](https://bbs.kanxue.com/forum-4-1-7.htm)
