
## 汇编数据

```
seg000:0045EE7C byte_45EE7C     db 2Eh, 78h, 28h, 37h, 0B3h, 0E7h, 0F7h, 7Fh, 0Bh, 59h, 40h, 9Ch, 0B2h, 3Eh, 0DEh, 0E9h
seg000:0045EE7C                 db 65h, 0AFh, 0FDh, 0Fh, 67h, 68h, 3Bh, 0A2h, 0D5h, 0B0h, 0Eh, 97h, 9, 68h, 0DCh, 3Ah
seg000:0045EE7C                 db 24h, 0Fh, 7Fh, 3Ah, 2Ch, 0D2h, 37h, 68h, 39h, 0C7h, 0E8h, 0BAh, 0D1h, 0FBh, 4Fh, 8Dh
seg000:0045EE7C                 db 0BCh, 4Eh, 23h, 0FCh, 2, 59h, 1Dh, 0E3h, 85h, 0ADh, 2, 73h, 0BBh, 0DDh, 0F9h, 7Eh

seg000:0045EEBC byte_45EEBC     db 60h, 0B9h, 0B5h, 4Eh, 56h, 42h, 9Ch, 2Dh, 9Eh, 86h, 33h, 93h, 7, 48h, 26h, 0D0h
seg000:0045EEBC                 db 0DFh, 1Fh, 5, 32h, 12h, 54h, 0FDh, 0F3h, 58h, 0Eh, 61h, 4, 0F4h, 0C2h, 1Bh, 69h
seg000:0045EEBC                 db 91h, 3Ch, 8, 1Dh, 0A1h, 5Eh, 2Eh, 67h, 0AFh, 86h, 0ECh, 64h, 2Ah, 9Bh, 4Eh, 1Ch
seg000:0045EEBC                 db 0C6h, 66h, 30h, 72h, 59h, 62h, 7Eh, 1Bh, 0BBh, 0CDh, 0C3h, 11h, 49h, 31h, 0A3h, 80h

seg000:0045EEFC byte_45EEFC     db 8Ah, 0CEh, 0D9h, 0F5h, 0A3h, 0C1h, 2Ah, 0E7h, 0BFh, 38h, 0C3h, 34h, 0BEh, 0B0h, 25h, 82h
seg000:0045EEFC                 db 9Ah, 0E7h, 5Ch, 0F2h, 41h, 0BEh, 7Ah, 31h, 0DDh, 38h, 3Dh, 9Ch, 0E2h, 0DAh, 0E9h, 0F8h
seg000:0045EEFC                 db 9Ch, 66h, 5Ch, 8Fh, 0E7h, 0F1h, 48h, 0A5h, 84h, 2Eh, 9Eh, 0EFh, 65h, 0B1h, 9, 0D4h
seg000:0045EEFC                 db 0CAh, 39h, 1Bh, 97h, 7, 0E6h, 6Dh, 73h, 24h, 73h, 0B0h, 6Ch, 64h, 6Ch, 0B3h, 7Bh

seg000:0045EF3C byte_45EF3C     db 0B8h, 56h, 22h, 0CAh, 30h, 6Fh, 0CAh, 8
seg000:0045EF3C                                         ; DATA XREF: GenKey+292o

seg000:0045EF44 byte_45EF44     db 0Dh, 3Bh, 4, 21h, 73h, 55h, 0F6h, 44h
seg000:0045EF44                                         ; DATA XREF: GenKey+2CEo
```

我自己写了一段代码，读取并验证以上参数，然后将正确的参数输出到文件中。代码见附件。

### 针对这个keygen做一个KeyVerifier，验证所生成key的正确性

加密和解密是对立统一的两个方面，相互之间联系很紧密。如果能够知道加密算法的细节，那么寻找解密算法也容易很多，同样，如果要找加密算法，知道解密算法也会有很大的帮助。

我写`KeyVerifier`的目的也差不多，既是为了帮助理解解密算法，也是为保证逆向出来的解密算法的正确性。

以前在逆向`Microsoft.Windows.Server.2003.x64.Edition.VOL.FIXED.Keymaker.Only-ZWT`的时候曾经写过一个，考虑到算法都是一样的，直接拿来用就可以了。

### 用Delphi逆向一个能用的keygen

刚开始发此贴的时候，我还没有信心用`Delphi`写一个`keygen`。因为我对`Delphi`不熟，几乎就是从来没有用过。不过后来分析汇编代码，发现`IDA`逆向出来的东西实在是太清晰了，于是我想逆向出一个能用的`keygen`也许不会很难。然后就这样仓促上手，用了两天多的时间终于搞定。

在逆向的过程中也发现最开始发的帖子有些问题。其实这里所用的算法跟以前是完全一样的，作者并没有用什么新技术，只是我自己没有看清罢了。

`Delphi`的`keygen`见附件。代码不是很多，结构很清晰，相信学过计算机的人都能看懂。

现在回过头来看，这点玩意儿实在是没什么技术含量，挺失望的，唉。。。

事实上，这一椭圆曲线加密算法在`M$`的多个产品中用到，比如`WIN_XP_PRO`、`WIN_XP_PRO_VLK`、`OFFICE_XP`、`WIN_2003_ENT_VLK`、`WIN_2003_X64_VOL`，也是不同的产品中加密算法参数不同，算法部分改动不多。大家所见过的四合一、五合一、七合一的`keygen`都是依据这样的原理来的。可以预测，过不了多久，更多合一的`keygen`将会出现在大家面前。

附件是我自己写的验证代码，以及`Delphi`逆向的`Project`。水平有限，不足之处请各路高手多多指教！

关于Windows Server注册算法的文字描述，可参考`[3]`。文档`[3]`可在网上搜索到，在`[2]`中也能找到。所有的资料都来自于Internet。

### 参考资料

```
[1] Microsoft.Windows.Server.2003.x64.Edition.VOL.FIXED.Keymaker.Only-ZWT
[2] Microsoft.Windows.XP.2003.Enterprise.Server.Keygen-YAG
[3] MSKey4in1 Read Me.doc
[4] Microsoft的25位CDKey里有什么.TXT
[5] 恢复原版XP.TXT
[6] Microsoft.Office.2007.Enterprise.Keygen.Only-MiCROSOFT
[7] Microsoft.Office.2007.Applications.Keygen.Only-MiCROSOFT
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

### 上传的附件

- `mskeygen .rar` （755.53kb，87次下载）
