
原文链接：<a href="elink@9e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6r3u0D9L8$3N6Q4x3V1k6V1k6h3!0T1k6Y4g2K6j5$3q4@1K9h3&6Y4i4K6u0V1M7r3!0%4k6i4u0K6K9r3g2D9L8q4)9J5k6s2m8#2N6s2c8A6L8X3N6Q4x3X3c8@1L8$3!0@1K9s2m8S2M7%4c8W2i4K6u0V1j5X3q4U0K9#2)9J5k6s2c8#2j5X3f1`."><mark class="encrypted">823K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6r3u0D9L8$3N6Q4x3V1k6V1k6h3!0T1k6Y4g2K6j5$3q4@1K9h3&6Y4i4K6u0V1M7r3!0%4k6i4u0K6K9r3g2D9L8q4)9J5k6s2m8#2N6s2c8A6L8X3N6Q4x3X3c8@1L8$3!0@1K9s2m8S2M7%4c8W2i4K6u0V1j5X3q4U0K9#2)9J5k6s2c8#2j5X3f1`.</mark></a>

安全专业人员很早意识到一件令他们头疼的事情，那就是攻击者喜欢让他们的工作变得更加复杂。攻击者们有一系列技术来混淆自己的地理位置，网络流量，原始代码指令。这些行为使得防御者更难以发现并阻止他们，毕竟防御者很难找到攻击痕迹，即使找到了也难以理解那些混淆过后的数据。在编程领域，混淆就是使用语言的特性和技巧来创建一个易于机器读取，但人眼很难识别的命令。

混淆技术在不断发展，但幸运的是，防御者也逐渐意识到并发展了与之对应的反混淆技术。正如我今年上半年在[BSides Charm](https://bbs.kanxue.com/elink@496K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5K9f1k6F1j5W2g2h3P5U0x3&6y4$3E0Q4x3U0k6S2L8i4m8Q4x3@1u0X3k6h3q4@1N6i4u0W2i4K6y4p5P5h3!0#2N6s2g2Q4x3X3g2T1k6b7`.`.)上所介绍的，现在有一些令人兴奋的方法可以应用机器学习（ML）来对抗PowerShell的混淆技术。但是在正式开始之前，让我们先了解一些常见的混淆技术，尤其关注攻击者对于PowerShell的混淆。

## PowerShell脚本混淆

PowerShell功能十分强大。它旨在从命令行自动执行任务并解决配置管理的问题，由此创建了许多重要工具。PowerShell有很多优点：例如易于导入模块，能够访问核心API和远程命令，也正是由于这些优点使其成为攻击者执行无文件攻击的首选工具之一。使用本地或预先安装的工具来进行攻击已经越来越受欢迎，至少部分是由于基于文件的AV系统（例如ML引擎）的进步以检测从未见过的攻击。

对分析师和防御者来说，可以通过记录PowerShell命令并捕获脚本文件以进行分析。这使安全人员有机会在系统被入侵后取证，以了解攻击者的行为以及获取的成果（是否攻击成功。不过，攻击者可不喜欢这样做（this 代指上文取证工作，是说不会让安全人员轻易进行有效取证），所以他们经常会对命令进行模糊处理和编码，以阻止和减慢分析人员的速度。

每种语言都有自己的混淆方法，其中有许多都是通用的。对于PowerShell而言，以下是一些常见的方法：

- ![](https://bbs.kanxue.com/upload/attach/201811/740325_M3UBU984SUQQAVN.jpg)

  PS（PowerShell）中一些变量/名称不区分大小写

- ![将单词进行分割。分割后的每个部分可能是也可能不是一个单词。](https://bbs.kanxue.com/upload/attach/201811/740325_4CK2UZQJF4KP4EW.jpg)

  将单词进行分割，分割后的每个部分可能是也可能不是一个单词

- ![通常用于将变量插入到命令语句中，在这种情况下，它用于打乱字符串。](https://bbs.kanxue.com/upload/attach/201811/740325_K3G4XVTVV4UJMDE.jpg)

  通常用于将变量插入到命令语句中，在这种情况下，它用于打乱字符串

- ![](https://bbs.kanxue.com/upload/attach/201811/740325_2XX4GAPFF9VFKS7.jpg)

  反引号可以用作连接字符，有时也可以表示[特殊字符](https://bbs.kanxue.com/elink@087K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8G2N6$3g2J5M7$3S2W2L8r3I4Q4x3V1k6E0L8$3c8#2L8r3g2Q4x3V1k6E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2H3L8%4N6W2M7Y4y4Z5k6h3I4D9i4K6u0W2j5$3!0J5k6g2)9J5c8X3q4T1L8%4g2@1i4K6u0r3j5h3u0G2N6i4c8Q4y4h3k6K6M7r3g2U0K9h3q4D9i4K6g2X3j5$3S2S2M7X3q4U0N6r3g2J5M7#2)9K6c8Y4k6A6k6i4N6Q4x3@1c8H3L8%4N6W2M7Y4y4Z5k6h3I4D9i4K6u0V1y4R3`.`.)。但是，如果在变量的中间使用反引号，它会连接同一行的前后连个字符

- ![](https://bbs.kanxue.com/upload/attach/201811/740325_XAWWHW22KZRTVA5.jpg)

  将字符串[转换](https://bbs.kanxue.com/elink@b80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7K6j5@1i4K6u0W2j5$3!0E0i4K6u0r3M7s2y4Q4x3V1k6U0j5h3I4D9i4K6u0W2K9s2c8E0L8l9`.`.)为命令操作

- ![](https://bbs.kanxue.com/upload/attach/201811/740325_PG43H2MT2QB8YU9.jpg)

  空格在某些操作中无关紧要，因此添加它只是为了使阅读变得更麻烦

- ![](https://bbs.kanxue.com/upload/attach/201811/740325_QW9NX396A7DC48Y.jpg)

  用代表特定字符的ascii码的值替换字符

还有更复杂的混淆方法，如变量创建和替换。混淆器将随机变量定义为字符串的全部或部分，并通过文件在该字符串的位置插入/替换它的地方。有许多方法可以实现替换。以下是几个例子：

- **[格式化操作符](https://bbs.kanxue.com/elink@d66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7K6j5@1i4K6u0W2j5$3!0E0i4K6u0r3M7s2y4Q4x3V1k6K6P5h3&6@1j5i4S2Q4x3X3c8X3i4K6u0V1L8%4m8W2M7X3q4@1L8%4u0Q4x3X3g2Z5N6r3#2D9)**

  输入：

  ```
  {1}PSScriptRoot{0}..{0}PSVersionCompare.psd1 -F  ‘\’,’$’

  ```

  输出：

  ```
  $PSScriptRoot\..\PSVersionCompare.psd1

  ```

- **[替换功能](https://bbs.kanxue.com/elink@122K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7K6j5@1i4K6u0W2j5$3!0E0i4K6u0r3M7s2y4Q4x3V1k6J5k6i4m8D9j5h3y4W2i4K6u0W2K9s2c8E0L8l9`.`.)**

  输入:

  ```
  (pZyPSScriptRoot\Add-LTUser.ps1).replace('pZy',’$’)

  ```

  输出:

  ```
  $PSScriptRoot\Add-LTUser.ps1

  ```

还有更多的例子我们没有详细列举，由于篇幅有限这里只能列举一部分，但是它们都是可以解决的。

使用Daniel Bohannon的[`Invoke-Obfuscation`](https://bbs.kanxue.com/elink@058K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5h3&6A6k6h3I4T1L8$3S2S2L8X3&6G2L8W2)9J5c8V1W2F1N6X3!0C8k6g2)9J5k6p5!0T1k6Y4g2K6j5$3q4@1K9h3!0F1)模块可以轻松使用以上列出的和更多没有列出的混淆方法。它是我们研究中所有混淆和编码工作的首选来源。

使用`Invoke-Obfuscation`，我们可以同时使用多个模糊处理方式，例如：

混淆之前：

```
$packageName = 'kvrt'

$url = 'ee7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8W2N6X3u0#2K9h3I4V1M7#2)9J5k6h3E0S2M7%4m8W2M7Y4y4C8P5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6T1N6h3W2D9k6s2y4Q4x3V1k6w2g2W2u0f1i4K6u0r3L8r3q4@1k6i4y4@1i4K6u0r3k6Y4g2D9L8q4)9J5c8V1E0h3f1W2c8Q4x3X3g2W2P5r3g2Q4x3U0M7`.

$checksum = '8f1de79beb31f1dbb8b83d14951d71d41bc10668d875531684143b04e271c362'

$checksumType = 'sha256'

$toolsPath = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)"

$installFile = Join-Path $toolsPath "kvrt.exe"

try {

  Get-ChocolateyWebFile -PackageName "$packageName" `

                        -FileFullPath "$installFile" `

                        -Url "$url" `

                        -Checksum "$checksum" `

                        -ChecksumType "$checksumType"



  # create empty sidecars so shimgen only creates one shim

  Set-Content -Path ("$installFile.ignore") `

              -Value $null



  # create batch to start executable

  $batchStart = Join-Path $toolsPath "kvrt.bat"

  'start %~dp0\kvrt.exe -accepteula' | Out-File -FilePath $batchStart -Encoding ASCII

  Install-BinFile "kvrt" "$batchStart"

} catch {

  throw $_.Exception

}

```

图表1：原始Powershell脚本示例

混淆之后：

```
${P`ACka`Ge`NAMe} = ("{0}{1}" -f 'kv','rt')

${U`RL} = ("{4}{11}{0}{6}{10}{3}{7}{2}{13}{15}{1}{16}{5}{8}{9}{14}{12}"-f'persky-','il','com/dev','bs','http:/','/','l','.','KVR','T/late','a','/devbuilds.kas','T.exe','b','st/full/KVR','u','ds')

${Check`s`UM} = ("{15}{16}{10}{3}{11}{6}{14}{9}{4}{5}{13}{1}{8}{7}{12}{2}{0}"-f 'c362','84143b0','71','79beb31f1d','1','bc','495','e','4','d71d4','e','bb8b83d1','2','10668d8755316','1','8f','1d')

${C`HE`cksu`m`TYpe} = ("{1}{0}" -f'56','sha2')

${T`Ool`s`PATH} = "$(Split-Path -parent $MyInvocation.MyCommand.Definition) "

${instALL`F`i`Le} = .("{0}{2}{1}{3}" -f'J','-Pa','oin','th') ${tOO`lSP`ATh} ("{0}{1}{2}" -f 'k','vrt.e','xe')

try {

&("{2}{5}{0}{4}{3}{1}" -f'colateyWe','e','Ge','Fil','b','t-Cho') -PackageName "$packageName" `

-FileFullPath "$installFile" `

-Url "$url" `

-Checksum "$checksum" `

-ChecksumType "$checksumType"



&("{2}{3}{0}{1}"-f '-','Content','Se','t') -Path ("$installFile.ignore") `

-Value ${nu`Ll}



${B`At`C`HSTart} = &("{0}{2}{1}{3}"-f 'J','i','o','n-Path') ${TOol`s`patH} ("{0}{2}{1}" -f 'k','.bat','vrt')

((("{1}{2}{3}{4}{5}{7}{0}{6}"-f'ce','start ','%','~dp0{0}','k','vrt','pteula','.exe -ac'))-f [CHar]92) | .("{0}{1}{2}"-f 'Out-','Fi','le') -FilePath ${BA`T`c`hstARt} -Encoding ("{0}{1}"-f 'AS','CII')

&("{1}{0}{3}{2}"-f'l-','Instal','nFile','Bi') ("{0}{1}"-f 'k','vrt') "$batchStart"

} catch {

throw ${_}."E`X`CEPtiOn"

}

```

图表2：通过`Invoke-Obfuscation`进行模糊处理后的Powershell脚本示例

## 编码技术

文本也可以转换为其他字符映射方案，进一步模糊。本文我们只关注两种方案：ascii转成hex以及ascii转成decimal。例如，'A'可以用十六进制表示为'41'，用十进制表示为'65'，'['用十六进制表示为'5B'，用十进制表示为'91'。

对PowerShell脚本进行完全编码需要一些额外的逻辑，解释器可以使用这些逻辑来解码文本。使用十进制编码的示例脚本如下所示：
