
# [原创]土X视频播放器的swf解密过程

没太多技术含量，主要是普通的分析与实践，大牛莫笑

某视频网站土X的播放器`Portalxxx_116.swf`下载下来以后用`asv`打开，发现关键的 swf 被加密了，当前的 swf 只是一个`loader`。没办法看到内部的内容，这让我这个一直用修改播放器去广告的小菜来说，简直伤心。

相比于优X、iqiX 的没有加密的 swf 播放器，土X的加密强度虽然不高，但是绝对能够阻挡很大一部分试图通过修改播放器去广告的行为。

当然作为小菜的我还是要试一试的。

## 定位播放器 swf 并下载分析

任意打开一个视频播放页面，右键查看源代码

看到最主要的播放器 swf 文件为

`c2bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3A6K6i4K6u0W2N6s2g2^5P5s2S2^5P5s2S2Q4x3X3g2U0L8$3#2Q4x3V1k6T1K9h3&6Q4x3V1k6D9K9h3&6^5P5s2S2^5P5q4)9J5c8W2m8G2M7Y4c8S2L8s2S2^5P5q4)9#2k6U0p5I4y4W2)9J5k6i4y4%4k6R3.`.

用下载工具下载下来。打开 swf 看里面的资源以及 as 文件，发现完全不符合这个 500K 的文件大小。

![](https://bbs.kanxue.com/upload/attach/201407/528829_bicbr6y6hw3wl94.png)

## 主要 AS 文件

分析以后发现主要有用的 as 文件有三个，主要是利用 AS 的`ByteArray`来进行加解密。

1. `SWZLoader` —— 主要用来解密的函数
2. `playerloader` —— 主要用来加载文件解密等处理
3. `playerloader_xxxplayer` —— 加密后的 swf 数据

### `playerloader` 加载流程

先来看第 2 个用来加载文件加解密的 AS 的主要内容

```
private function init(_arg1:Event=null):void
{
var _local2:SWZLoader;
var _local3:ByteArray;
removeEventListener(Event.ADDED_TO_STAGE, this.init);
//生成一个SWZLoader解密模块的对象
_local2 = new SWZLoader();
//将playerloader_xxxplayer这个加密后的swf数据块作为bytearray读入local3中
_local3 = ByteArray(new this.tudouPlayer());
//调用解密模块解密字节流
_local2.load(_local3);
//完成后调用onLoadedHlr
_local2.contentLoaderInfo.addEventListener(Event.COMPLETE, this.onLoadedHlr);
}

private function onLoadedHlr(_arg1:Event):void
{
       //将字节流作为一个swf加载运行
         this.mainSwf = (_arg1.target.content as Sprite);
            if (this.mainSwf != null){
                addChild(this.mainSwf);
         ......
};
```

### `SWZLoader` 解密逻辑

加载过程了解清楚以后剩下的就是查看`SWZLoader`的`load()`函数是如何解密的。

看第 1 个`SWZLoader`类的`load()`函数

```
public function load(param1) : void
{
           ..................
            if (param1 is ByteArray)
            {
//传入字节码解密，返回解密后的字节码
                _loc_2 = this.decode(param1);

//this.dataByteArray=_loc_2;这句是我自己添加的，用于存解密后的数据，dataByteArray申明一个bytearray的变量就好了

                this.loadBytes(_loc_2);
                return;
            }
            ............................
}
```

关键的解密过程就是`this.decode(param1);`

```
private function decode(param1:ByteArray) : ByteArray
{
//这里看不懂，不过肯定不是在这里返回
            if (int(Capabilities.version.split(/[ ,]+""[ ,]+/)[1]) > 10)
            {
                return param1;
            }
//验证字节流有效性
            if (param1 == null || param1.length < 32)
            {
                return null;
            }
            var _loc_2:* = new ByteArray();
/*生成一个字节流对象_loc_2。为何_loc_2的长度是这么来的呢。
由于SWF文件的开头八个字节是有意义的。
00000000h: 46 57 53 0A 2A 34 0B 00                         ; FWS.*4..
46 57 53 是SWF标记，也有可能是SWC SWZ代表是zlib加密过的 0A是版本 后面四个字节 00 0B 034 2A表示解密后的文件大小
param1[4] | param1[5] << 8 | param1[6] << 16 | param1[7]所以这里是取swf文件大小
 */
            _loc_2.length = param1[4] | param1[5] << 8 | param1[6] << 16 | param1[7] << 24;

//调用Decoder().decode从字节流的param1的位置17开始解密到_loc_2的第八个字节后面
            if (!new Decoder().decode(param1[12], param1, 17, _loc_2, 8, _loc_2.length - 8))
            {
                return null;
            }
//复制swf文件的头八个字节
            var _loc_3:* = 0;
            while (_loc_3 < 8)
            {

                _loc_2[_loc_3] = param1[_loc_3];
                _loc_3++;
            }
//70就是ascii F 也就是SWF标记里的F 代表已经被解密过了，如果是Z或者是C表示还是加密的。
            _loc_2[0] = 70;
//修正版本号为 10
            _loc_2[3] = param1[13] > 0 ? (param1[13]) : (10);
//返回解密后的swf字节流
            return _loc_2;
}// end function
```

后来才知道`decode()`用的是`LZMA`算法。这里就懒得再说了。有点麻烦。如果熟悉可以用 C 写出来。

## 还原解密过程

接下来就是还原这个过程。

首先用`avs`提取这个`playerloader_xxxplayer`加密块

![](https://bbs.kanxue.com/upload/attach/201407/528829_bh8ii9z8nljctbu.png)

用 Adobe Flash CS 5.5 新建一个工程，为`test.fla`添加关联的默认 as。也就相当于`MAIN`函数。

将`SWZloader`中有用的部分抠出来。这个解密的函数没办法完全反编译过来，我从网上扣了一个。关键词搜索`public class LZMA{ public static function decode`应该就可以了。这样`SWZLoader`基本不用改什么就是完好可用的 AS，我只是自己添加一些变量，用于获取数据。

现在自己开始写主要的调用 AS

我用我半桶水的 java 语言终于还是完成了一个艰难的任务。

```
public function aaa()
{
			 trace("Hello world!");
			var loa:loader1=new loader1();//所有的过程都放在loader1里面进行
}
class loader1
{
	public function loader1()
	{

			trace("enter");
var ul:URLLoader = new URLLoader();
ul.dataFormat = URLLoaderDataFormat.BINARY;
ul.addEventListener(Event.COMPLETE, onComplete);
//上面抠出来的bytearray重命名为data.bin作为URLRequest load进来,完成后调用onComplete

ul.load(new URLRequest("data.bin"));
trace("loaded");

	}

	function onComplete(e:Event):void
	{
          var ba:ByteArray = (e.target as URLLoader).data;
		    var _loc_2:SWZLoader = null;
            _loc_2 = new SWZLoader();
            _loc_2.load(ba);
		    var fr:FileReference = new FileReference();
//dataByteArray, 是前面我SWZLOADER添加过的变量。
            fr.save(_loc_2.dataByteArray, "player.swf")
	}
}
```

调试运行一下，就会弹出一个框让你保存`player.swf`，这个就是解密后的 swf 文件了。

最后才是真正播放器的庐山真面目啊

![](https://bbs.kanxue.com/view/img/face/1.gif)

![](https://bbs.kanxue.com/upload/attach/201407/528829_brf8480lt81izae.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`111.png`](javascript:void(0);)（28.16kb，36次下载）
- [`222.png`](javascript:void(0);)（28.60kb，6次下载）
- [`333.png`](javascript:void(0);)（27.87kb，8次下载）
- [`data.rar`](javascript:void(0);)（1.93MB，133次下载）
