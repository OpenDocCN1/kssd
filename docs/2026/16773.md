
# [翻译]图像隐写之使用PHP隐藏图像中的文本

隐写术是一门研究隐藏信息的科学艺术，通过隐写术，可以只让发送者和接收者知道隐藏的信息。

图像隐写术则是研究将数据隐藏在图像中，通过该技术可以防止无关用户发现这些隐藏的信息或数据。

## 相关的原理：

图像都是由很小的点组成的，这些点就是像素。每个像素由三个字节来表示：一个代表红（Red），一个代表绿（Green），一个代表蓝（Blue）。这三种颜色的组合决定了像素的实际颜色。

```
红色：

 二进制：11001001

十进制：201



绿色：

二进制：11111000

十进制：248（原文此处为201，实际应为248）



蓝色：

二进制：00000011

十进制：3
```

这一组合呈现的像素颜色是橙色。

图像隐写的基本原理就是修改LSB（Least Significant Bit，最不重要的那一bit）并不会被人眼察觉到。所以我们修改RGB的LSB值来存储隐藏信息，同时不影响图像的颜色。

在这个例子中，我们仅仅修改蓝色的LSB，但是如果我们想的话，也可以修改所有的红，绿和蓝的成分。不过只修改蓝色的LSB，我们可以隐藏的信息更少一些。我们也可以将要隐藏信息的长度存储到图像中，不过本教程中还没有做这一点。

假设我们现在要将1101藏到图片中。

首先我们要得到图片中每个像素的RGB值，因为我们要隐藏4bit的数据，而且我们要修改蓝色的部分，所以我们需要4个像素。

假设图片前4个像素的RGB值如下：

11001100 10010001 00101011

00011000 11110000 11111110

11100010 00100101 01010101

11111101 00001010 01000011

现在，我们将这4个RGB值最后的bit替换成1101，

现在新的RGB值变成了：

11001100 10010001 00101011

00011000 11110000 11111111

11100010 00100101 01010100

11111101 00001010 01000011

加粗的最后一bit，就是我们要隐藏到图片的信息。

现在将新的RGB值应用到像素，因为人眼并不会注意到这种改变，所以图片看起来还是一样的。

加密的部分到现在就完成了，接下来我们将解密隐藏在图片的信息。

为了达到这一点，我们需要取出每个像素的RGB值，然后合并LSB来得到我们隐藏的信息。

## 用PHP实现

我们需要一张图片和3个php脚本来实现基本的图像隐写。

- `encrypt.php`
- `decrypt.php`
- `functions.php`
- `1.jpg`

### `encrypt.php`

```
<?php

include('functions.php');

$message_to_hide = 'hello';
$binary_message = toBin($message_to_hide);
$message_length = strlen($binary_message);

$src = '1.jpg';
$im = imagecreatefromjpeg($src);

for($x=0;$x<$message_length;$x++){
$y = $x;
$rgb = imagecolorat($im,$x,$y);
$r = ($rgb >>16) & 0xFF;
$g = ($rgb >>8) & 0xFF;
$b = $rgb & 0xFF;
$newR = $r;
$newG = $g;
$newB = toBin($b);
$newB[strlen($newB)-1] = $binary_message[$x];
$newB = toString($newB);
$new_color = imagecolorallocate($im,$newR,$newG,$newB);
imagesetpixel($im,$x,$y,$new_color);
}

echo $x;
imagepng($im,'simple.png');
imagedestroy($im);

?>
```

### `decrypt.php`

```
<?php

include('functions.php');

$src = 'simple.png';
$im = imagecreatefrompng($src);

$real_message = '';

for($x=0;$x<40;$x++){
$y = $x;
$rgb = imagecolorat($im,$x,$y);
$r = ($rgb >>16) & 0xFF;
$g = ($rgb >>8) & 0xFF;
$b = $rgb & 0xFF;
$blue = toBin($b);
$real_message .= $blue[strlen($blue)-1];
}
$real_message = toString($real_message);
echo $real_message;
die;

?>
```

### `functions.php`

```
<?php
function toBin($str){
$str = (string)$str;
$l = strlen($str);
$result = '';
while($l--){
$result = str_pad(decbin(ord($str[$l])),8,"0",STR_PAD_LEFT).$result;
}
return $result;
}
function toString($binary){
return pack('H*',base_convert($binary,2,16));
}
?>

```

更多的信息可以查看这个视频：

视频地址：yotube: c1aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2@1N6g2)9J5k6h3u0W2i4K6u0r3M7f1R3H3N6@1I4u0N6#2A6S2x3q4f1`. (自备梯子)

原文链接：[Image Steganography: Hiding text in images using PHP](https://bbs.kanxue.com/elink@408K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Z5k6h3c8W2j5Y4g2Y4k6$3g2J5M7#2)9J5k6h3y4G2L8g2)9J5c8X3W2E0j5h3N6W2i4K6u0V1M7%4c8W2k6$3q4F1L8$3N6J5j5i4m8Z5P5g2)9J5k6r3S2A6k6r3W2F1k6#2)9J5k6s2c8W2P5s2c8Q4x3X3c8#2M7$3W2F1k6#2)9J5k6s2m8Z5M7q4)9J5c8R3`.`.)

本文由看雪翻译小组LImp翻译

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
