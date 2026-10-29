
```php
$is\_upload = false;
$msg = null;
if (isset($\_POST['submit'])) {
if (file\_exists($UPLOAD\_ADDR)) {
$deny\_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
$file\_name = trim($\_FILES['upload\_file']['name']);
$file\_ext = strrchr($file\_name, '.');
$file\_ext = strtolower($file\_ext); //转换为小写
$file\_ext = str\_ireplace('::$DATA', '', $file\_ext);//去除字符串::$DATA
$file\_ext = trim($file\_ext); //首尾去空
if (!in\_array($file\_ext, $deny\_ext)) {
if (move\_uploaded\_file($\_FILES['upload\_file']['tmp\_name'], $UPLOAD\_ADDR . '/' . $\_FILES['upload\_file']['name'])) {
$img\_path = $UPLOAD\_ADDR . '/' . $file\_name;
$is\_upload = true;
}
} else {
$msg = '此文件不允许上传';
}
} else {
$msg = $UPLOAD\_ADDR . '文件夹不存在,请手工创建！';
}
}
```

> - 去了空格，过滤完美，提示说文件后缀点绕过，就在php后面加个.让他无法解析，就可以绕过了

![image-20210723064834702](upload/attach/202108/922735\_DD4CU3KWAKAZJ6B.jpg)

![image-20210723064904118](upload/attach/202108/922735\_RUSKP6WHQNWQE2G.jpg)

# pass-08

```php
$is\_upload = false;
$msg = null;
if (isset($\_POST['submit'])) {
if (file\_exists($UPLOAD\_ADDR)) {
$deny\_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
$file\_name = trim($\_FILES['upload\_file']['name']);
$file\_name = deldot($file\_name);//删除文件名末尾的点
$file\_ext = strrchr($file\_name, '.');
$file\_ext = strtolower($file\_ext); //转换为小写
$file\_ext = trim($file\_ext); //首尾去空
if (!in\_array($file\_ext, $deny\_ext)) {
if (move\_uploaded\_file($\_FILES['upload\_file']['tmp\_name'], $UPLOAD\_ADDR . '/' . $\_FILES['upload\_file']['name'])) {
$img\_path = $UPLOAD\_ADDR . '/' . $file\_name;
$is\_upload = true;
}
} else {
$msg = '此文件不允许上传';
}
} else {
$msg = $UPLOAD\_ADDR . '文件夹不存在,请手工创建！';
}
}
```

> - 和前面的代码有点不同，没有去掉::$DATA字符流windows文件流绕过

![image-20210723065223780](upload/attach/202108/922735\_B5CBT74V23MYRZJ.jpg)

> - 执行的时候不带::$DATA就行了

![image-20210723065257088](upload/attach/202108/922735\_MX4HPT87YWPBTXP.jpg)

# pass-09

```php
$is\_upload = false;
$msg = null;
if (isset($\_POST['submit'])) {
if (file\_exists($UPLOAD\_ADDR)) {
$deny\_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
$file\_name = trim($\_FILES['upload\_file']['name']);
$file\_name = deldot($file\_name);//删除文件名末尾的点
$file\_ext = strrchr($file\_name, '.');
$file\_ext = strtolower($file\_ext); //转换为小写
$file\_ext = str\_ireplace('::$DATA', '', $file\_ext);//去除字符串::$DATA
$file\_ext = trim($file\_ext); //首尾去空
if (!in\_array($file\_ext, $deny\_ext)) {
if (move\_uploaded\_file($\_FILES['upload\_file']['tmp\_name'], $UPLOAD\_ADDR . '/' . $\_FILES['upload\_file']['name'])) {
$img\_path = $UPLOAD\_ADDR . '/' . $file\_name;
$is\_upload = true;
}
} else {
$msg = '此文件不允许上传';
}
} else {
$msg = $UPLOAD\_ADDR . '文件夹不存在,请手工创建！';
}
}
```

> - 黑名单机制+删除掉文件名最后一个点（若有的话），判断最后一位是不是点，字符串首尾去空。根据代码反向思考构造可以绕过的后缀为.php.空格.

> - 所以用.php.空格.就会删掉后面的点和去空格函数去掉但是还有一个.就形成绕过

![image-20210723071200203](upload/attach/202108/922735\_FUT2GB2PHPWM548.jpg)

# pass-10

```php
$is\_upload = false;
$msg = null;
if (isset($\_POST['submit'])) {
if (file\_exists($UPLOAD\_ADDR)) {
$deny\_ext = array("php","php5","php4","php3","php2","html","htm","phtml","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");
$file\_name = trim($\_FILES['upload\_file']['name']);
$file\_name = str\_ireplace($deny\_ext,"", $file\_name);
if (move\_uploaded\_file($\_FILES['upload\_file']['tmp\_name'], $UPLOAD\_ADDR . '/' . $file\_name)) {
$img\_path = $UPLOAD\_ADDR . '/' .$file\_name;
$is\_upload = true;
}
} else {
$msg = $UPLOAD\_ADDR . '文件夹不存在,请手工创建！';
}
}
```

![image-20210723071532566](upload/attach/202108/922735\_CMF9497RSGF2MSF.jpg)

> - 解析：只是单纯的对第一次发现php进行删除，但是构造一个双写php被删了之后还是可以绕过，例如pphphp=php，检测到php就删了但是又构成了一个php

![image-20210723071709332](upload/attach/202108/922735\_93TEX5ZAF4PM863.jpg)

![image-20210723071723356](upload/attach/202108/922735\_NF7M3K6V57C966M.jpg)

# pass-11

```php
$is\_upload = false;
$msg = null;
if(isset($\_POST['submit'])){
$ext\_arr = array('jpg','png','gif');
$file\_ext = substr($\_FILES['upload\_file']['name'],strrpos($\_FILES['upload\_file']['name'],".")+1);
if(in\_array($file\_ext,$ext\_arr)){
$temp\_file = $\_FILES['upload\_file']['tmp\_name'];
$img\_path = $\_GET['save\_path']."/".rand(10, 99).date("YmdHis").".".$file\_ext;
if(move\_uploaded\_file($temp\_file,$img\_path)){
$is\_upload = true;
}
else{
$msg = '上传失败！';
}
}
else{
$msg = "只允许上传.jpg|.png|.gif类型文件！";
}
}
```

> - 对文件名进行了随机，提示%00截断

![image-20210723072504799](upload/attach/202108/922735\_4C9CTDP73NYZ8U8.jpg)

![image-20210723072514895](upload/attach/202108/922735\_PJZWYFP8MJ2GZGA.jpg)

# pass-12

```php
$is\_upload = false;
$msg = null;
if(isset($\_POST['submit'])){
$ext\_arr = array('jpg','png','gif');
$file\_ext = substr($\_FILES['upload\_file']['name'],strrpos($\_FILES['upload\_file']['name'],".")+1);
if(in\_array($file\_ext,$ext\_arr)){
$temp\_file = $\_FILES['upload\_file']['tmp\_name'];
$img\_path = $\_POST['save\_path']."/".rand(10, 99).date("YmdHis").".".$file\_ext;
if(move\_uploaded\_file($temp\_file,$img\_path)){
$is\_upload = true;
}
else{
$msg = "上传失败";
}
}
else{
$msg = "只允许上传.jpg|.png|.gif类型文件！";
}
}
```

![image-20210723073123544](upload/attach/202108/922735\_QZ5JBPV6G5NZD29.jpg)

![image-20210723073233835](upload/attach/202108/922735\_M3XHKCXHTCFRCQW.jpg)

# pass-13

```php
function getReailFileType($filename){
$file = fopen($filename, "rb");
$bin = fread($file, 2); //只读2字节
fclose($file);
$strInfo = @unpack("C2chars", $bin);
$typeCode = intval($strInfo['chars1'].$strInfo['chars2']);
$fileType = '';
switch($typeCode){
case 255216:
$fileType = 'jpg';
break;
case 13780:
$fileType = 'png';
break;
case 7173:
$fileType = 'gif';
break;
default:
$fileType = 'unknown';
}
return $fileType;
}
$is\_upload = false;
$msg = null;
if(isset($\_POST['submit'])){
$temp\_file = $\_FILES['upload\_file']['tmp\_name'];
$file\_type = getReailFileType($temp\_file);
if($file\_type == 'unknown'){
$msg = "文件未知，上传失败！";
}else{
$img\_path = $UPLOAD\_ADDR."/".rand(10, 99).date("YmdHis").".".$file\_type;
if(move\_uploaded\_file($temp\_file,$img\_path)){
$is\_upload = true;
}
else{
$msg = "上传失败";
}
}
}
```

> - 检查图片前两个字节直接合一个图片马解析漏洞，多试几个图片吧，有的图片过大或者过小里面包含着特殊内容的，都可以pass

![image-20210723075037447](upload/attach/202108/922735\_G9QRH268P7B62ZR.jpg)

![image-20210723075210279](upload/attach/202108/922735\_CRA4RKFG73RMSUN.jpg)

# pass-14
