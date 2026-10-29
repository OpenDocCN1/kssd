
# [原创]Frida配合BurpSuite的Brida插件自动解密取证

## 前言

这是2W班8月份的练习题第一题。题目要求是

参考文档：https://bbs.pediy.com/thread-260422.htm

1. 使用frida来搜集图片证据，越多越好；并配置RPC自动传到电脑上保存；
2. 配置RPC，在电脑上实时显示解密的视频信息元数据；
3. 配置最新版brida（`1b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3k6h3c8W2M7X3W2U0L8$3c8G2N6s2c8S2i4K6u0r3b7Y4u0A6k6r3q4Q4c8f1k6Q4b7V1y4Q4z5o6W2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5p5q4Q4z5e0y4Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0S2Q4b7e0N6Q4z5o6k6Q4c8e0W2Q4b7e0u0Q4z5e0q4Q4c8e0c8Q4b7V1k6Q4b7e0q4Q4c8e0k6Q4z5o6q4Q4b7f1k6Q4c8e0g2Q4z5o6g2Q4z5o6y4Q4c8e0k6Q4z5e0g2Q4b7U0m8Q4c8e0k6Q4z5p5c8Q4b7f1g2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5p5y4Q4z5o6g2Q4c8e0y4Q4z5o6m8Q4z5o6q4Q4c8e0W2Q4z5o6m8Q4z5f1q4Q4c8e0S2Q4b7V1k6Q4z5o6N6T1M7X3W2V1j5g2!0q4z5q4!0n7x3q4)9^5x3#2!0q4y4#2)9&6y4q4!0m8z5r3q4H3M7q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5q4!0m8y4#2!0m8x3#2!0q4y4g2!0m8c8W2)9^5y4W2!0q4y4#2!0m8c8g2)9&6y4#2!0q4y4W2!0n7x3#2)9&6y4g2!0q4y4W2)9&6c8q4!0m8y4g2!0q4y4g2)9^5y4g2!0m8z5q4!0q4z5q4)9^5y4#2!0m8b7g2!0q4y4g2)9^5b7g2!0m8z5q4!0q4z5q4!0m8y4#2!0m8x3#2!0q4y4g2!0m8c8W2)9^5y4W2!0q4x3#2)9^5x3q4)9^5x3R3`.`）

本题加密流程已经由大佬给出，也就是在大佬的基础上进行取证。对Brida的使用花费了多一点的时间。

另外当时做这题的时候,自己对frida的操作还是停留在根据内存分布去处理数据，而10月课程学习后，其实有很多更简便兼容性也更强的方式。

这里的代码都是对应于7月份老版本的HOOK。新版本需要重新找hook点。

## 解题过程

参考文档中已经给出了APP的加密流程和解密代码。这里就不再分析APP的加密，只是打印出解密后的数据。

## 下载图片

直接定位图片解密函数，保存返回值。

### 首先尝试直接保存到手机

```js
var class_decimg = Java.use("com.ilulutv.fulao2.other.g.b");
class_decimg.b.overload('[B', '[B', 'java.lang.String').implementation = function(bArr, bArr2, str){
var result = this.b(bArr, bArr2, str);
imgResult = result;
console.log("imgResult.length->", imgResult.length);
var fileName = "/sdcard/fulao2/"+imgResult.length+".jpg";
try{
var buffer = Memory.alloc(imgResult.length);
for(var i=0;i<imgResult.length;i++){
var ival = imgResult[i];
if(ival < 0) {
ival += 256;
}
// console.log("buffer:",buffer.add(i),ival);
buffer.add(i).writeU8(ival);//imgResult[i]
}
console.log("buffer:", hexdump(buffer));
var fd = new File(fileName, "ab+");
if (fd && fd != null) {
var dex_buffer = ptr(buffer).readByteArray(imgResult.length);
fd.write(dex_buffer);
fd.flush();
fd.close();
console.log("[dump img]:", fileName);
}
}
catch(error){
console.log("write_file_log error:",error);
}
return result;
}
```

```bash
class_decimg.a-> [object Object]
class_decimg.a-> 56439
buffer: 0 1 2 3 4 5 6 7 8 9 A B C D E F 0123456789ABCDEF
95e7f148 ff d8 ff e0 00 10 4a 46 49 46 00 01 01 00 00 01 ......JFIF......
95e7f158 00 01 00 00 ff fe 00 3b 43 52 45 41 54 4f 52 3a .......;CREATOR:
95e7f168 20 67 64 2d 6a 70 65 67 20 76 31 2e 30 20 28 75 gd-jpeg v1.0 (u
95e7f178 73 69 6e 67 20 49 4a 47 20 4a 50 45 47 20 76 36 sing IJG JPEG v6
95e7f188 32 29 2c 20 71 75 61 6c 69 74 79 20 3d 20 39 30 2), quality = 90
95e7f198 0a ff db 00 43 00 03 02 02 03 02 02 03 03 03 03 ....C...........
95e7f1a8 04 03 03 04 05 08 05 05 04 04 05 0a 07 07 06 08 ................
95e7f1b8 0c 0a 0c 0c 0b 0a 0b 0b 0b 0d 0e 12 10 0d 0e 11 0e ................
95e7f1c8 0b 0b 10 16 10 11 13 14 15 15 15 0c 0f 17 18 16 ................
95e7f1d8 14 18 12 14 15 14 ff db 00 43 01 03 04 04 05 04 .........C......
95e7f1e8 05 09 05 05 09 14 0d 0b 0d 14 14 14 14 14 14 14 ................
95e7f1f8 14 14 14 14 14 14 14 14 14 14 14 14 14 14 14 14 ................
95e7f208 14 14 14 14 14 14 14 14 14 14 14 14 14 14 14 14 ................
95e7f218 14 14 14 14 14 14 14 14 14 14 14 ff c0 00 11 08 ................
95e7f228 01 ca 03 1b 03 01 22 00 02 11 01 03 11 01 ff c4 ......".........
95e7f238 00 1f 00 00 01 05 01 01 01 01 01 01 00 00 00 00 ................
[dump img]: /sdcard/fulao2/56439.jpg
```

可以保存但是速度极慢，因为不知道怎么取出result的地址，所以就新建buffer重写了一遍在保存。

![](upload/attach/202008/571058_DE6BRDBAUZ5KPSM.jpg)

### 配置rpc下载

```js
var class_decimg = Java.use("com.ilulutv.fulao2.other.g.b");
class_decimg.b.overload('[B', '[B', 'java.lang.String').implementation = function(bArr, bArr2, str){
var result = this.b(bArr, bArr2, str);
imgBase64Arr[imginx++] = gson.$new().toJson(result);
return result;
}
rpc.exports = {
getimgarr : function(i){
var result = '';
if(i in imgBase64Arr){
try{
result = imgBase64Arr[i];
console.log("imgBase64Arr[i] result->", result.length);
}
catch(error){
console.log(error);
}
}else{
console.log("i in imgBase64Arr false");
}
return result;
},
getimginx : function(){
return imginx;
},
clearimgarr: function(start,len){
for(var i = 0; i< len;i++){
// imgBase64Arr[start+i] = "clear";
delete imgBase64Arr[start+i];
}
}
}
```

```python
def downImg(imgDataStr):
try:
ms = re.findall("(-?\d+)", imgDataStr)
intArr = []
for m in ms:
ival = int(m)
if ival < 0:
ival += 256
intArr.append(ival)
bs = bytes(intArr)
m = hashlib.md5()
m.update(bs)
downname = m.hexdigest()
with open("download/"+downname+".jpg", 'ab') as f:
f.write(bs)
print("download sucess->"+ downname+".jpg")
except Exception as e:
print("downImg Error->")
imginx = 0
while True:
try:
imgArrValue = script.exports.getimginx()
while imgArrValue > imginx:
imgobj = script.exports.getimgarr(imginx)
downImg(imgobj)
script.exports.clearimgarr(imginx, 1)
imginx += 1
print(imgArrValue)
time.sleep(3)
except Exception as ex:
print("Error:", ex)
break
```

这样下载图片速度不会影响app的正常运行了，只是批量下载需要滑动触发图片加载。

![](upload/attach/202008/571058_ESENEZDZQXWP9FK.jpg)

大量读取的话，还是用原作者写好的分页查询图片，然后依次下载脱离app运行方便很多，这里就下载一点测试。

## 显示解密的视频信息元数据

由于解密的函数`com.ilulutv.fulao2.other.g.b.a`大多接口都会调用。这里我只想显示`m3u8`的数据，所以打印了堆栈，向上找了一些定位到了`c.c.a.h.a.c.a`。
