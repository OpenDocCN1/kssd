
```
0049C18A   EB 5C                 JMP SHORT 0_unpack.0049C1E8               ; 解除密码保护  
0049C18C   8B15 1C034A00         MOV EDX,DWORD PTR DS:[4A031C]            ; 0.004A1D68  
0049C192   A1 D4004A00           MOV EAX,DWORD PTR DS:[4A00D4]  
0049C197   E8 3CCBFFFF           CALL 0_unpack.00498CD8  
0049C19C   8A8B 7C030000         MOV CL,BYTE PTR DS:[EBX+37C]  
```

表述可能有些不清楚，下面是对原代码的修改，可以对照程序跟以下，就很容易理解了。

## Patches

| 地址 | 大小 | 状态 | 旧 | 新 | 注释 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `00408CED` | `6` | 激活 | `PUSH EAX` | `JMP 004C8010` | |
| `00494D4C` | `6` | 激活 | `PUSH EBP` | `JMP 004C8210` | |
| `00499EA2` | `2` | 激活 | `JNZ SHORT 0_unpack.00499EE9` | `JMP SHORT 0_unpack.00499EE9` | |
| `0049B32E` | `5` | 激活 | `JNZ SHORT 0_unpack.0049B34D` | `JMP 004C81E0` | |
| `0049B76C` | `2` | 激活 | `JNZ SHORT 0_unpack.0049B7EA` | `JMP SHORT 0_unpack.0049B7EA` | |
| `0049C183` | `9` | 激活 | `CMP BYTE PTR DS:[EBX+375],0` | `MOV BYTE PTR DS:[EBX+375],0` | |

---

## 【经验总结】

上次预告的`ebookedit`本身的一个技巧就是超链接可以使用关键字调用程序功能。

代码是 5 月写的，所以现在有点忘了。表述不是很清晰，希望大家谅解，下一篇教程是关于`ebookworshop`的（由于相关工具支持新版的反编译，所以文章只谈界面风格的输出），敬请期待。

**预告**：简单的读取内存中的`ini`配置文件，创建文件夹时过滤非法字符，移动文件，删除目录。

感谢看到这里。

---

2007 年 06 月 27 日 16:00:00

**原版下载**

`http://www.hanzify.org/index.php?Go=Show::List&ID=4170`

附件中为改后的文件，方便大家对比代码。

`4dd466d0e1e0cc123afbe504da2462ca`
