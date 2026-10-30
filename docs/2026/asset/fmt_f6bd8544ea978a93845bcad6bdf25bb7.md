
| 1  2  3  4  5  6  7  8 | `bool` `__fastcall is_independ_format(``const` `char` `*a1)`  `{`  `if` `( !a1 )`  `return` `0LL;`  `if` `(` `strchr``(a1,` `` '`' ```) )`  `return` `1LL;`  `return` `strchr``(a1,` `'\''``) != 0LL;`  `}` |

若是存在单引号，且前面没有转义符，则对其`Unicode`编码，即`\\u0027`。反引号也同理。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34 | `while` `( 1 )`  `{`  `v13 = (unsigned` `__int8``)*v11;`  `if` `( !*v11 )`  `break``;`  `v15 = v11 + 1;`  `if` `( v13 ==` `'\''` `)`  `goto` `LABEL_22;`  `...`  `LABEL_22:`  `if` `( v11 != (_BYTE *)1 )`  `{`  `v13 =` `'u'``;`  `v16 = *(v11 - 1) !=` `'\\'``;`  `if` `( (unsigned` `__int64``)v11 <= v10 )`  `goto` `LABEL_24;` `// mark`  `goto` `LABEL_38;`  `}`  `goto` `LABEL_19;`  `...`  `LABEL_24:`  `if` `( !v16 )`  `goto` `LABEL_40;`  `v17 = (``__int64` `*)v22;`  `LOBYTE(v22[1]) = v13;` `// mark`  `LABEL_26:`  `if` `( v13 ==` `'u'` `)`  `{`  `(*(``void` `(__fastcall **)(``__int64``,` `const` `char` `*, _QWORD))(v3 + 3680))(` `// sprintf`  `(``__int64``)v17 + v16 + 4,`  `"%02x"``,`  `(unsigned` `__int8``)*(v15 - 1));` `// mark`  `v18 = (unsigned` `int``)(v16 + 6);`  `}` |

进一步交叉引用，在`sub_40DB48`函数中，对可控的数据用`stringtojson`函数进行了过滤。然而，这里的过滤并不严谨，接着往下看。
![](https://bbs.kanxue.com/upload/attach/202306/949925_9HWSGKPVEEMEBMV.png)
由上述可知，若这里的`v74`不为空，则存放着`stringtojson`函数过滤后的内容，否则说明不存在单引号或反引号，也就未通过`stringtojson`函数进行过滤。又由于`stringtojson`函数处理后会带有双引号，故若包含了单引号或反引号，该命令在新版固件中实际为`/usr/sbin/module_call set networkId_merge "{...}"`。虽然由于`JSON`数据中双引号得是`\"`才行（`stringtojson`函数也会将双引号编码为`\\u0022`），无法闭合绕过双引号，但是在双引号内是可以用反引号或`$()`执行命令的，而这里只过滤了反引号，并未过滤`$()`，也就给了攻击者可趁之机。
![](https://bbs.kanxue.com/upload/attach/202306/949925_2SXG4WZQWYAPAU5.png)
不过，在新版本固件中，也在其他方面加强了一定的安全检查和防护，例如在初始化脚本`/etc/init.d/factory`中通过`rm usr/sbin/telnetd`命令删除了`telnetd`程序，也就无法通过开启远程登录而控制设备了。但是，不难想到还可以通过反弹`shell`获取设备权限，这里笔者采用的是`telnet`反弹的方式。

**请求报文：**

|  |  |
| --- | --- |
| 1 | `暂不公开` |

**演示效果：**
![](https://bbs.kanxue.com/upload/attach/202306/949925_5GAX45XTN3TW46F.png)

### 补丁2

其实，只要在`noauth.lua`的`merge`函数中对传入的`params`加个过滤即可，如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `function merge(params)`  `local cmd` `=` `require` `"luci.modules.cmd"`  `local tool` `=` `require(``"luci.utils.tool"``)`  `local _strParams` `=` `luci.json.encode(params)`    `if` `tool.includeXxs(_strParams)` `or` `tool.includeQuote(_strParams) then` `-``-` `过滤危险字符和单引号`  `tool.eweblog(_strParams,` `"MERGE FAILED INVALID DATA"``)`  `return` `'INVALID DATA'`  `end`    `return` `cmd.devSta.``set``({device` `=` `"pc"``, module` `=` `"networkId_merge"``, data` `=` `params, async` `=` `true})`  `end` |

此处，通过`includeXxs`函数过滤了各种危险字符，以及用`includeQuote`函数过滤了单引号：

|  |  |
| --- | --- |
| 1  2  3  4 | `function includeXxs(``str``)`  `local ngstr` `=` `` "[\n`&$;|]" ``  `return` `string.match(``str``, ngstr) ~``=` `nil`  `end` |

|  |  |
| --- | --- |
| 1  2  3 | `function includeQuote(``str``)`  `return` `string.match(``str``,` `"(['])"``) ~``=` `nil`  `end` |

可见，在新版本固件中，将换行符`\n`也过滤了，提高了安全性。

## 总结

这篇文章是在挖到这个`0day`挺久之后写的了。依稀记得当时刚挖到这个漏洞的时候，有着些许兴奋，但更多的是感到不易，因为我当时觉得这条调用链还挺深的，里面也牵涉到了不少东西。但是，当我如今再梳理这个漏洞的相关细节的时候，我觉得这条调用链其实也就那样吧，整个挖掘思路和利用思路都不算难，抛开影响范围，并算不上品相多好的洞QAQ。

在挖这个洞的时候，我遇到的最大挑战就是逆向分析了，我觉得这里的逆向难度还是比较大的（当然我逆向水平也很菜）。在实际逆向分析的过程中，并没有文章中写的那么流畅，当时的挖掘思路也不可能是完全按照文章的流程来的，比如需要多考虑一些东西（例如，文章中一直都在找命令注入的洞，但其实也有可能是可控的`params`字段造成的缓冲区溢出等等，这些在初次挖掘的时候也都需要考虑），当然也走了不少弯路，但好在最终是坚持下来了。

当时，我只知道`params`字段是可控的，而`params`内也是`Json`的格式，于是猜测是其中的某个特定的字段可能会造成命令注入或缓冲区溢出等问题，因此就一路挖到底了。不过如今再看来，其实就这个洞而言，是否采用自动化的方式会更简单呢（当然就工业界来说，`IoT`的全自动化漏扫我并没有看到过实际效果很好的工具，基本都是半自动化提高效率）？

进一步地从宏观上来看二进制漏洞的挖掘思路，无非就是从危险函数出发和从交互入口出发两种方式，显然前者在筛掉明显无法利用的危险函数点之后，所涉及的支路会更少，挖起来也会更容易，而后者基本是要从交互入口一路挖到中断点甚至挖到底的。然而，该漏洞却是采用后者的思路进行挖掘的，当时主要是考虑到只有一个可能的未授权入口，因此很自然地采用了后者的思路。现在想来，这里若是采用前者的思路，可能并不会那么容易地挖到此漏洞。如何更好地结合上述两种思路，特别是对于自动化漏扫来说，我觉得仍是值得思考的问题。

说了些自己粗浅的理解和感受，就说到这里吧。希望这篇文章能给各位像我一样刚入门`IoT`漏洞挖掘的师傅带来些启发，也欢迎各位大师傅与我交流。最后，希望我在不久的将来能挖到在挖掘思路和利用手法上都有所创新的高质量`0day`吧。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2025-11-12 00:16
被winmt编辑
，原因： 修正一坤年前的表述错误

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)
[#技术分享](https://bbs.kanxue.com/forum-128-1-168.htm)
[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)
[#漏洞分析](https://bbs.kanxue.com/forum-128-1-171.htm)
[#漏洞挖掘](https://bbs.kanxue.com/forum-128-1-178.htm)
[#家用设备](https://bbs.kanxue.com/forum-128-1-173.htm)
[#工控设备](https://bbs.kanxue.com/forum-128-1-174.htm)
