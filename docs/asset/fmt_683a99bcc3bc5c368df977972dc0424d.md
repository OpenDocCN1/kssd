
# XX 找茬的研究 (非截屏,非对比像素,源码)

> 原文：[`www.pediy.com/kssd/pediy11/121691.html`](https://www.pediy.com/kssd/pediy11/121691.html)

## 文章信息

- 【文章标题】: XX 找茬的研究 (非截屏,非对比像素,源码)
- 【文章作者】: Sam.com[CCG]
- 【下载地址】: 自己搜索下载
- 【使用工具】: `Delphi 2007`
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

对找茬的辅助工具并不感冒,但也接触过,先说下废话,本文文字比较多,重点在讲思路

由于经常在某论坛混,论坛每天会有找茬活动,就是版主发张像找茬那样的图片,然后圈出来回复图片就可以了,因为图片

是人手处理,有时也有很难找的时候,所以就会用工具把右半的图片复制到左边去,然后把这个图层改变下透明就能看出来不

同的地方,顺便推荐个在线处理图片的网页,感觉比较强大 `http://www.pixlr.com/editor/?loc=zh-cn`

因为每天要在这花些时间,所以想自己做个程序能帮我把图片不同处自动找出来,因为我接触的辅助工具都是像素比较的

方法,所以开始我也这样做了,原理很简单,因为论坛发布的图片是一张的,包括了左右两部分,所以只要把图片分成两半,然后

逐个像素比较颜色,颜色不同的话就把这个像素点改成红色,这样效果是不错,简单不过有点慢,但经过几次使用问题就来了,

因为原图是人手制作的,所以左右两图有时会坐标有点偏差,或者两图间有空白,导致这样简单的对比法几乎所有像素都对不

上坐标的,结果相当于整张图都标上了红色.

上面的方法行不通,也是找茬辅助的难点所在,找了些资料,好像比较难的都是在确定两图的坐标上,当然应该也有其它

方法或者优秀的算法来解决这个问题,但是我不会,其实两图的不同处都是比较大的一块区域,对比的要求并不用严格到每一

个像素,像用透明的方法,坐标偏差比较大都能用肉眼判断出来,那就换个思路,也是简单的方法,不用把不同处标记出来了,

就直接把右边的图片做为一个图层放在最左边,然后用个定时器把这个图层前置和隐藏,这样一个视觉差就能很明显的看出

不同处.这个源码在下面提供,里面有个测试图片,直接拖图片进程序窗口就可以了,双击图片选择颜色后可以用像素对比的

方法来显示.

然后,回想以前玩 XX 找茬时经常被人虐,虐人的这个看你点得快有威胁还要把你踢出去,实在是理解不了这种人的心理,

既然我的这种方法简单有效,能不能运用到 XX 找茬里去呢?于是动手研究 XX 找茬,思路是这样,找到游戏的图片源,找个地方

`Hook` 把图片取出来,按上面的其中一个方法把不同处显示出来,先不考虑效率,也不考虑截屏,只要达到目的

先找找图片是怎么来的,XX 找茬有练习模式,比较方便调试,先拦下封包,看看会不会是每次从服务器传过来,如果是的话

说不定包里还有不同处的坐标之类,但结果很失望,收到的包很少,仔细想想,如果从封包入手也挺麻烦的,还要解密封包什么

的,初步来看应该也不像是从服务器发过来的,最多可能也就只有坐标而已,也或者封包数据是从 `XXGame.exe` 这个进程传过来,

反正这个方法要处理起来要花比较多时间去处理,先暂时放弃.

图片是找茬游戏重要的保护对象,无论图片是在本地或者服务器传过来,压缩或者加密是必要的,看看游戏目录,有个

`zlib1.dll`,这个不用说了,ZIP 的模块,有个导出函数 `uncompress`,在这入手看看.启动游戏后附加然后下断,只要一加载图片就

断下来了,来看看 `uncompress` 的参数,共 4 个参数

```
004604B5   |.  52           push edx
004604B6   |.  8B51 10      mov edx, dword ptr [ecx+10]
004604B9   |.  03C2         add eax, edx
004604BB   |.  8B5424 08    mov edx, dword ptr [esp+8]
004604BF   |.  8D4C24 10    lea ecx, dword ptr [esp+10]
004604C3   |.  50           push eax
004604C4   |.  51           push ecx
004604C5   |.  52           push edx
004604C6   |.  E8 A1730000  call <jmp.&zlib1.uncompress>
004604CB   |.  83C4 10      add esp, 10
0012F044   05D8D6F8     <---输出缓冲
0012F048   0012F060     <---返回解压后的实际长度
0012F04C   00F90010     <---输入缓冲
0012F050   0000C904     <---输入长度
```

函数执行过后,到 `05D8D6F8` 看看,一看内容就知道是个 `JPG`,`JPG` 的文件头很容易看出来.堆栈往下拉就能看到文件名如

`"res\\chaBig\\xxx\\origin.JPG"`, `xxx` 是个数字,按实际长度把二进制保存为 `JPG` 文件,打开看正好就是找茬的图,那么我的猜

想就没错了,所有图片资源都用 `ZIP` 压缩的,那么只要把两个图取出来就大功告成了.

先继续中断看看,当游戏换一次图时,共会中断 7,8 次,前两次是`"res\\chaBig\\xxx\\origin.JPG"`,后面的都是类似

`"res\\chaBig\\xxx\\cha1.JPG"`的,把所有图片都保存下来看看,傻眼了,一直认为找茬每次都是两个图片,一个是原图一个是

改过的,但 XX 找茬不是,他是用两张原图,然后 5 张小图放在其中一个原图的上面,这 5 个小图就是找茬的地方,想想这种游戏

要保证最小的传输量又要保证找茬的地方每次不同,这个方法也许是最有效的,那么情况麻烦了,我无法得到修改过的图片,

只有 5 块碎图,游戏应该是根据内设的坐标把这 5 个小图重绘在原图上,或者先处理完再显示,那么如果要用之前的方法来处理

的话就必须得找到他处理完的地方才行,但是一想到处理的代码可能非常复杂,又或者根本不是我想像的那样,启不是浪费

我宝贵的时间?

本来想开始这么简单顺利,现在一下就把这个方法推翻了,我可真不想花太多时间在它身上,在放弃前再考虑下有没有

什么简单的方法来达到目的,换个思路,在 `uncompress` 这个地方 `Hook` 应该是最合适的了,既然解压后 `JPG` 就直接在内存中取得

到,那么我把 5 个小图换成其它的图片行吗?比如一个笑脸,那么显示出来后这 5 个地方就一目了然了,那光是简单的替换,就

考虑用 `LPK` 来做吧,反正我也想试试 `LPK`,代码很简单,只要判断一下文件名,把所有小图都统一换成同一个图,果然有效了.

但是,显示是显示出来了,使用中有问题,最明显的是替换的小图显示有问题,几乎所有小图的右下角都是显示灰色的,

还有就是所有不同处都显示一样大小的图片,那么有效的点击范围就出问题了,只有点击左上角比较小的范围才能 100%正确,

也许点击范围也是根据小图的 `XY` 来确定的,显示问题虽然不美观,但也不影响使用,这样的话替换的图片太大又会画面乱,太小

又会有时点不正确,真是有点郁闷了.

本人比较追求完美,虽然现在这样能用,但效果实在有些接受不了.再想办法改,首先考虑的是不要改变小图的大小,以

免影响点击的判断区域,比如动态生成一个一样大小的图片来替换,那既然要做到这份上了,不如我就直接修改小图,在小图

上做个标记好了,要做到这个效果光是 `LPK` 就不行了,因为要引用 `jpeg` 这些单元,所以生成的 `LPK` 会出错

具体请参考下面的源码 `ZhaoCha.rar`，`LPK` 借用的是 `Tee_lp_Delphi_APIHOOK.rar`，谢谢 `Tee8088` 分享，不过还是没解决

显示灰色的问题，由于时间关系就不处理了，期待高手指点，对于图像的处理不熟悉，大部分是现找现写的，代码比较粗糙，

好处是不用区分找茬的版本，美女或者卡通都通用(两者只是图片和大小不同而已)，2 个 `Dll` 复制到游戏目录下即可，下面注

释下核心代码.

### LPK 部分

```pascal
procedure uncompressCallback(out OutBuf: Pointer; out OutBytes: Integer; const InBuf: Pointer; InBytes: Integer);cdecl;
var
  TmpStr: array [0..1023] of char;
  DllHandle: THandle;
begin
  //引用 zlib1 的 uncompress 直接调用它,先解压再处理
  uncompress(OutBuf, OutBytes, InBuf, InBytes);

  //取文件名,版本更新后有可能要调整
  asm
    pushad
    push dword ptr [ebp+$48]
    lea eax, TmpStr
    push eax
    call lstrcpy
  end;

  //由于 LPK 中不能用 VCL,所以要另外写个 Dll,在这里才加载
  if not assigned(@ChangeJpg) then
  begin
    DllHandle := LoadLibrary(PChar('CJpg.dll'));
    if DllHandle = 0 then
    begin
      asm
        popad
      end;
      Exit;
    end else
      @ChangeJpg:= GetProcAddress(DllHandle, 'ChangeJpg');
  end;

  //判断文件名只处理找茬所需的图片,且只处理小图片
  if (Pos('res\\cha\\', TmpStr) > 0) or ((Pos('res\\chaBig\\', TmpStr) > 0)) then
  begin
    if Pos('origin', TmpStr) > 0 then
    else
    begin
      ChangeJpg(OutBuf, OutBytes);
    end;
  end;

  asm
    popad
  end;
end;
```

### 处理图片部分

```pascal
procedure ChangeJpg(out OutBuf: Pointer; out OutBytes: Integer);cdecl;
var
  Jpg: TJPEGImage;
  Bmp: TBitmap;
  TmpStream: TMemoryStream;
begin
  Jpg:= TJPEGImage.Create;
  Bmp := TBitmap.Create;
  //为什么要多用个 TmpStream?因为 Jpg 我找不到 Size 变量,就无法写回文件长度
  TmpStream:= TMemoryStream.Create;
  //从输出缓冲里读出 Jpg
  TmpStream.Write(OutBuf, DWORD(OutBytes));
  TmpStream.Position:= 0;
  Jpg.LoadFromStream(TmpStream);

  //因为 Jpg 的像素是只读的,只能转成 Bmp 来处理,这是网上找到的方法
  Bmp.Assign(jpg);

  //在图片四边画上红线和绿线,一目了然
  Bmp.Canvas.Pen.Width:= 3;
  Bmp.Canvas.Pen.Color:= clRed;
  Bmp.Canvas.Polyline([Point(0, 0), Point(Bmp.Width, 0), Point(Bmp.Width, Bmp.Height),
                       Point(0, Bmp.Height), Point(0, 0)]);

  Bmp.Canvas.Pen.Width:= 3;
  Bmp.Canvas.Pen.Color:= clLime;
  Bmp.Canvas.Polyline([Point(0+3, 0+3), Point(Bmp.Width-3, 0+3), Point(Bmp.Width-3, Bmp.Height-3),
                       Point(0+3, Bmp.Height-3), Point(0+3, 0+3)]);

  //处理后的图片转成 Jpg
  Jpg.Assign(Bmp);
  TmpStream.Position:= 0;
  Jpg.SaveToStream(TmpStream);
  TmpStream.Position:= 0;

  //处理后的图片放回原处
  TmpStream.Read(OutBuf, TmpStream.Size);
  //改成处理后的长度
  DWORD(OutBytes):= TmpStream.Size;

  Jpg.Free;
  Bmp.Free;
  TmpStream.Free;
end;
```

最后来个未点击的效果图:

![](img/a70ee9e4a65012a089ea23e14e7e0685.jpg)

### 源码

- `PicCmp.rar`
- `ZhaoCha.rar`

## 经验总结

- `LPK` 不能使用 `VCL`
- 换个角度,看似复杂的问题可以简单化

## 版权声明

此文及源码仅限学习研究之用，请勿用于商业用途!

2010 年 10 月 05 日 21:17:32
