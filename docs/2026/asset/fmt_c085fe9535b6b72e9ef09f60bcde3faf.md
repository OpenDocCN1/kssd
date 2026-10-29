
## 三、工具封装

为了达到高效率的同时又能自主可控，我决定做一个有水平的缝合侠，简单理解就是我要把很多工具巧妙地融入到我开发的工具中来。这里需要考虑的第一个问题是：每个工具的使用方法、输入的参数、输出的结果都是不一样的，工具A的结果工具B不一定认识。

要解决这个问题，说简单也简单，说难也难，总之我是摸着石头过河成功了；原理是自己给每个工具做一个壳，外部要调用工具A需要先调用工具A的壳，然后才会传到工具A。当工具A返回了结果，工具A的壳也会最先拿到，然后将结果解析出来并按照统一的格式输出就可以了。

![](https://bbs.kanxue.com/upload/attach/202212/775297_JBMTTA2DU7J62WM.jpg)

通过这个简单的办法，我相当于把其他的安全工具变成了我的一个函数，我需要的时候调用这个函数就可以了。

按照我前面提到的需求，我梳理了一下要试用的工具有这几个：

| 序号 | 需求 | 工具 |
| --- | --- | --- |
| 1. | 爬取URL的 | `RAD` |
| 2. | 爆破URL的 | `DIRMAP` |
| 3. | 提取主机IP | 正则 |
| 4. | 快速检测热门POC | `xray` |
| 5. | 识别网站的指纹 | `dismap` |
| 6. | 对IP端口快速扫描 | `masscan` |
| 7. | 能对端口的banner识别出服务 | `nmap` |
| 8. | 能检测出SQL注入漏洞 | `sqlmap` |
| 9. | 能检测出反射性XSS漏洞 | `xsser` |

这些工具都是比较常见的工具，我第一步需要熟悉它们的使用方法，以`xray`工具为例。

### xray工具封装

`xray`的使用命令如下所示：

```bash
./xray_linux_amd64 webscan --url "da6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4x3o6m8Q4x3V1j5." --json-output /tmp/11.json
```

当`xray`执行完毕之后，它会将结果输出到指定位置，但是数据格式并不是我所期望的，我需要将它的格式读入，然后再转换成我所需要的格式。

这里我用PHP写了一个简单的脚本，它做了这几件事情：

1. 定义了参数来源位置和结果输出位置
2. 获取参数中的URL，并执行`xray`工具
3. 获取`xray`的执行结果，并解析成自定义格式
4. 将最终的结果写入到输出位置

代码示例如下所示：

```php
<?php
//获取输入的参数
$inputFile = "/data/share/input_".getenv("xflow_node_id").".json";
$outputFile = "/data/share/output_".getenv("xflow_node_id").".json";

//没有input,直接返回
if (!file_exists($inputFile)) {
 var_dump($outputFile, json_encode(['code' => 0, 'msg' => "{$inputFile}文件不存在", 'data' => []], JSON_UNESCAPED_UNICODE));
 return 0;
}
//读取上游数据
$inputData = json_decode(file_get_contents($inputFile), true);

$url = $inputData['url'];
$data = execTool($url);

//将结果写入到指定位置,供蜻蜓平台导入数据
file_put_contents($outputFile, json_encode($data, JSON_UNESCAPED_UNICODE));

//将工具执行
function execTool($url)
{

$hash = md5($url);
$resultPath = "/tmp/{$hash}/tool.json";
//清理之上一轮的结果
if (file_exists($resultPath)) unlink($resultPath);
//创建文件夹
if (!file_exists(dirname($resultPath))) {
 mkdir(dirname($resultPath), 0777, true);
}

$result = [];

$toolPath = "/data/tools/xray";
if (!file_exists($toolPath)) die("xray 工具目录不存在:{$toolPath}");

$path = "cd $toolPath && ";
// 通过系统命令执行工具
$cmd = "{$path} ./xray_linux_amd64 webscan --url \"{$url}\" --json-output {$resultPath}";
echo $cmd;
exec($cmd, $result);

$toolResult = file_exists($resultPath) ? file_get_contents($resultPath) : '[]';
$toolResult = json_decode($toolResult, true);
print_r($toolResult);
return $toolResult;
}
```

### sqlmap工具封装

再来`sqlmap`封装的例子，首先需要知道`sqlmap`的使用方法，如下所示：

```bash
sqlmap -u "958K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4x3o6m8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5x3b7." --batch --random-agent
```

当`sqlmap`执行完毕之后，我需要知道它的执行结果在什么位置，并将结果解析出来，按照规范化的格式输出到指定地址。

这里我同样用PHP写了一个脚本，做了这几件事情：

1. 定义了参数来源位置和结果输出位置
2. 获取参数中的URL，并执行`sqlmap`工具
3. 获取`sqlmap`的执行结果，并解析成自定义格式
4. 将最终的结果写入到输出位置

```php
<?php
//获取输入的参数
$inputFile = "/data/share/input_".getenv("xflow_node_id").".json";
$outputFile = "/data/share/output_".getenv("xflow_node_id").".json";

//没有input,直接返回
if (!file_exists($inputFile)) {
    file_put_contents($outputFile, json_encode([]));
    return 0;
}
//读取上游数据
$list = json_decode(file_get_contents($inputFile), true);
print_r($inputFile);
print_r($list);
$data = [];
//处理数据
foreach ($list as $val) {
    $url = $val['url'];
    $toolPath = "/data/tools/sqlmap/";

    print_r("开始扫描URL:{$url}".PHP_EOL);
    execTool($url, $toolPath);

    //录入检测结果
    $tempList = writeData($toolPath, $url);
    print_r("扫描URL:{$url}完成".PHP_EOL);
    print_r($tempList);
    $data = array_merge($data, $tempList);
}

print_r($data);
//将结果写入到指定位置,供蜻蜓平台导入数据
file_put_contents($outputFile, json_encode($data, JSON_UNESCAPED_UNICODE));

function writeData($toolPath, $url)
{
    $arr = parse_url($url);
    $file_path = $toolPath . 'result/';
    $host = $arr['host'];
    $outdir = $file_path . "{$host}/";
    $outfilename = "{$outdir}/log";

    //sqlmap输出异常
    if (!is_dir($outdir) or !file_exists($outfilename) or !filesize($outfilename)) {
        print_r("sqlmap没有找到注入点: $url");
        return [];
    }
    $ddd = file_get_contents($outfilename);
    print_r($ddd);

    exec("rm -rf $outdir");

    return [["raw" => $ddd]];
}

function execTool($v, $toolPath)
{
    $arr = parse_url($v);
    $blackExt = ['.js', '.css', '.json', '.png', '.jpg', '.jpeg', '.gif', '.mp3', '.mp4'];
    //没有可以注入的参数
    if (!isset($arr['query']) or (strpos($arr['query'], '=') === false)) {
        print_r(["URL地址不存在可以注入的参数".PHP_EOL, $v]);
        return false;
    }
    $file_path = $toolPath . 'result/';
    $cmd = "cd {$toolPath}  && python3 ./sqlmap.py -u '{$v}' --batch  --random-agent --output-dir={$file_path}";
    exec($cmd);
    return true;
}
```

通过前面`xray`和`sqlmap`两个工具封装的例子，你会发现其实每个工具封装的流程都差不多，差别只是程序的输出结果解析而已，所以到现在为止我解决了扫描器的能力问题。

## 四、动手实践

现在只需要我把几个功能连接起来就行了，这里需要考虑一个新的问题：`sqlmap`所需要的参数却是具体的多个URL地址，也就是说在调用`sqlmap`之前，我需要把URL都收集好再调用`sqlmap`，这里就有数据依赖问题。

这个问题也好办，我们需要准备三张表：目标表、功能依赖表、数据存放表。

**目标表**

| `ID` | `URL` | `create_time` |
| --- | --- | --- |

**功能表**

| `ID` | `tool_name` | `pre_tool_name` | `create_time` |
| --- | --- | --- | --- |

**数据表**

| `ID` | `tool_name` | `url` | `result` | `create_time` |
| --- | --- | --- | --- | --- |

我们可以首先从目标表中获取一个要扫描的目标，然后读取所有的功能，`for`循环功能表，只需判断当前有没有依赖问题，或者依赖问题已经解决，那么就可以得到所需的依赖数据，直接执行功能即可。

执行完成结果可以在结果页面看见，这里是我的执行结果。

![](http://oss.songboy.site/blog/20221129185302.png)

伪代码如下所示：

```php
<?php

$id = getTarget();
$toolList = getToolList();

foreach($toolList as $val){
    //判断当前工具上级依赖为空或者上级工具已执行
    if($val['pre_tool_name'] == '' or 上级工具已经执行){
        //开始使用工具对URL扫描
        scanUrl();
        //保存结果
        saveResult();
    } else {
        //上级工具还没执行完成,先跳过
        continue;
    }
}
```

这是我写好的脚本，大家可以简单改改应用，目前我写的脚本已经集成到了蜻蜓安全平台里面，你可以一键复制使用。

![](https://bbs.kanxue.com/upload/attach/202212/775297_47KZQ4HUNPG2EXG.jpg)

`92bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4q4A6L8X3N6@1K9h3&6Y4i4K6u0W2M7%4c8S2M7X3y4J5L8%4y4K6i4K6u0W2j5$3&6Q4x3V1k6K6j5$3g2F1j5i4u0A6L8#2)9J5c8X3c8W2N6r3q4A6L8q4)9K6c8X3W2V1i4K6y4p5x3e0V1K6x3b7.`

目前我已经集成了46款常见的工具，放在 GitHub 中开源，地址：`2d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0N6r3q4J5b7%4u0G2M7%4y4b7L8%4u0@1j5h3I4Q4x3V1k6c8K9h3&6Y4g2r3W2F1k6H3.`

---

作者：汤青松

日期：2022-11-29

微信：`songboy8888`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
