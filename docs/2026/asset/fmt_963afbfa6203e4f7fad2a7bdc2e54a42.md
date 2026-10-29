
- 管理员密码 - 使管理员点击构造的链接即可修改其密码。
- 新建单击管理员通知 - 使管理员构造的链接自动添加管理员。
- 漏洞联合(XSS+CSRF)
- 【社交节点蠕虫】 - 如果向好友发送消息的HTTP请求中没有设置`CSRF-token`请求头，可结合XSS则可构造出XSS+CSRF蠕虫影响极大。
- 【拒绝服务】 - 如果通过前台构造`xssPayload`提交成功打到了管理后台，且管理员注销动作无CSRF防御机制，可构造`payload`使管理员一登录则注销。
- 【网络后台权限维度】
- 某某通过前台布局者通过提交提交A登录管理后台查看数据管理员`xssPayload`（漏洞1利用成功但这是某项攻击事件的有效权限后cookie过渡将无法在后台登录）
- 在后台有一个配置“网站名称”的选项存在存储类型xss漏洞（但普通漏洞2仅前台用户会触发管理后台并不会触发）
- 为了 web 后台的权限维度构造 CSRF 请求并利用漏洞 2 使每个普通用户都可以提交漏洞 1 的`xssPayload`（也可以设置一定的随时性部分用户也可以提交）
- 管理员日常登录后台查看数据会像他第一次触发漏洞1那样触发。

### 网络场景测试技巧

- 请求相同重放 - 智能识别令牌是否正确满足设计：`token动态生成 token会被后端验证且只能被验证一次 经过后端验证后的token立即失效`
- 修改令牌值 - 如修改请求正文并发送请求，查看响应正文是否和正常请求的响应正文相同
- 删除令牌值 - 如修改请求正文`para1=aaa&token=`并发送请求，查看响应正文是否和正常请求的响应正文相同
- 完全删除令牌参数 - 如修改请求正文为`para1=aaa`并发送请求，查看响应正文是否和正常请求的响应正文相同
- HTTP 方法可以由 `POST` 正确转换为请求`method`，如果可以为 `GET` 方法

### 浏览器的Cookie策略

若能知道浏览器的cookie策略，就会对CSRF的原理与攻击方式会有更深的理解。

浏览器所持有的cookie分成两种，一种是`Session Cookie`，叫做`临时Cookie`，一种是`Third-party Cookie`，叫做`本地Cookie`，也叫做第三方Cookie。

他们之间的区别在于`本地Cookie`是服务器在`Set-Cookie`时设置了指定的`Expire`时间，只有到了制定的`Expire`之后，`本地Cookie`才会失效，所以这种Cookie保存在本地。

而`临时Cookie`则没用绑定`Expire`，所以当浏览器关闭后，`临时Cookie`也就会失效了，所以这种`临时Cookie`保存在浏览器中。

在浏览网站的过程中，如果一个网站设置了`临时Cookie`，那么在浏览器的运行过程中，即使浏览器打开了新的网页，`临时Cookie`也是有效的。

`第三方Cookie`是什么：如果我们访问`<mark class="encrypted">a37K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0W2Q4z5o6u0Q4b7e0y4Q4c8e0c8Q4b7U0W2Q4z5o6S2Q4y4U0m8Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2j5g2)9J5k6h3y4G2L8g2)9$3x3q4!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4#2!0m8b7#2!0m8b7#2!0q4y4q4!0n7z5q4)9^5x3q4!0q4y4W2)9&6y4W2!0n7z5g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2!0m8y4W2)9^5x3W2!0q4y4W2)9&6c8g2)9&6b7#2!0q4y4W2)9^5z5q4)9&6x3g2!0q4y4q4!0n7b7W2!0m8b7#2!0q4y4g2)9&6b7#2!0m8z5q4)9$3x3r3S2@1N6s2m8Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2i4K6u0W2j5$3!0E0i4K6j5H3i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1&6i4@1p5I4i4@1t1#2i4@1f1&6i4K6W2p5i4@1p5J5i4@1f1@1i4@1t1^5i4@1q4p5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6W2o6i4@1p5^5i4@1f1#2i4K6S2q4i4@1u0n7i4@1f1^5i4@1q4q4i4@1u0r3i4@1f1&6i4K6V1%4i4@1q4q4i4K6j5H3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0W2Q4z5o6u0Q4b7e0y4Q4c8e0c8Q4b7U0W2Q4z5o6S2Q4y4U0m8Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2j5W2)9J5k6h3y4G2L8g2)9$3x3q4!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4#2!0m8b7#2!0m8b7#2!0q4y4q4!0n7z5q4)9^5z5g2!0q4y4W2)9&6y4W2!0n7z5g2!0q4x3#2)9^5x3q4)9^5x3R3`.`.</mark>`

如果有这么一种情况，浏览器从一个域的页面中，想要加载另一个域的资源，由于安全原因，一些网络站点会停止`本地Cookie`即`第三方Cookie的`发送。就是说，我们已经获得了`<mark class="encrypted">211K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0Q4x3X3g2U0L8$3#2Q4y4U0l9`.</mark> 的cookie值，当我们在`<mark class="encrypted">83dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4y4U0l9`.</mark> 页面访问包含在`<mark class="encrypted">580K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4y4U0l9`.</mark> 页面中的`<mark class="encrypted">b75K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0Q4x3X3g2U0L8$3#2Q4y4U0l9`.</mark> 的链接时，仍然不会把之前获得的`<mark class="encrypted">0c5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0Q4x3X3g2U0L8$3#2Q4y4U0l9`.</mark> 的cookie值发送过去

但是这里也是有一个前提的，就是`<mark class="encrypted">c5eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4Q4x3X3g2U0L8$3#2Q4y4U0l9`.</mark> 页面包含在`<mark class="encrypted">eeeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0W2Q4b7e0q4Q4b7U0g2Q4c8e0W2Q4z5f1c8Q4b7e0u0Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4y4U0m8Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2j5W2)9J5k6h3y4G2L8g2)9$3x3q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5g2)9&6x3#2!0n7c8g2!0q4y4W2)9^5c8g2!0m8y4g2!0q4y4W2)9&6y4#2!0n7y4W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7b7W2)9^5c8q4!0q4y4#2)9^5y4q4!0n7y4W2!0q4y4q4!0n7z5q4)9^5c8q4!0q4y4q4!0n7b7#2)9&6b7g2!0q4y4W2)9^5b7g2)9^5b7g2!0q4y4q4!0n7z5g2)9^5b7W2!0q4y4g2)9^5z5g2)9^5c8q4!0q4z5q4)9^5c8g2!0n7y4#2!0q4y4g2!0n7c8g2)9&6y4#2!0q4y4#2)9&6b7g2)9^5y4q4)9$3x3r3S2@1N6s2m8Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1i4K6u0W2j5$3!0E0i4K6j5H3i4@1f1%4i4K6W2m8i4K6R3@1j5$3!0G2K9$3W2W2i4@1f1#2i4K6R3H3i4@1u0o6i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1^5i4@1u0r3i4K6R3%4i4@1f1#2i4K6S2q4i4@1u0n7i4@1f1K6i4K6R3H3i4K6R3J5</mark>

但是这里也是有一个前提的，就是<mark class="encrypted">bc3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4c8e0W2Q4b7e0q4Q4b7U0g2Q4c8e0W2Q4z5f1c8Q4b7e0u0Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0N6Q4z5f1q4Q4z5o6c8Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2j5W2)9J5k6h3y4G2L8g2!0q4y4g2)9^5b7#2)9^5y4g2!0q4y4g2)9&6x3q4!0m8b7W2!0q4y4g2)9&6b7#2!0m8z5l9`.`.</mark>`<iframe>`, `<img>`, `<script>`, `<link>` 等标签中，因为这些标签在IE中，是禁止浏览器向这些标签中发送第三方cookie的。

当然，并不是所有的浏览器都会禁止，比如火狐浏览器，就默认支持发送第三方cookie。

所以至少对于IE浏览器，攻击者需要精心构造攻击环境，比如可以先诱使用户在当前浏览器中先访问目标站点，使得session cookie有效，然后尝试CSRF。

#### P3P头的副作用

> 尽管对于有些CSRF来说，并不是很需要cookie的，但大部分的敏感信息或重要操作都躲藏在认证之后，因此浏览器拦截第三方cookie的发送，一定程度上还是降低了CSRF的威力，可是P3P头的出现，使得一切复杂起来。

P3P Header 是W3C制定的一项关于隐私的标准，全称是The Platform for Privacy Preferences.

> 如果网站返回给浏览器的HTTP头部中包含有PHP3头，则在某种程度上来说，他会允许浏览器发送第三方cookie，即使是在上文所描述的IE浏览器`<iframe>`、`<input>` 也将不在拦截第三方cookie的发送。

> 比如存在<mark class="encrypted">2a1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4c8e0g2Q4z5e0u0Q4z5p5y4%4N6%4N6Q4x3X3g2T1i4K6u0W2j5$3!0E0i4@1f1@1i4@1t1^5i4@1p5@1i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1#2i4K6W2r3i4K6W2r3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6W2o6i4@1p5^5N6%4N6%4i4K6u0W2j5W2!0q4y4q4!0n7z5q4)9^5b7g2!0q4y4g2!0m8c8q4)9&6z5q4!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4q4!0n7z5q4)9^5x3q4!0q4y4q4!0n7z5q4!0m8b7g2!0q4z5g2!0m8x3g2!0n7y4g2!0q4z5g2)9&6c8q4!0m8x3W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7b7W2)9&6y4W2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4g2)9^5b7#2)9^5y4g2!0q4y4g2)9&6x3q4!0m8b7Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4b7f1q4A6k6Y4u0S2L8h3g2Q4c8e0y4Q4z5o6m8Q4z5o6t1`.</mark>

> `<mark class="encrypted">e22K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5o6k6Q4z5o6g2Q4c8e0g2Q4b7f1g2Q4b7U0W2Q4c8e0c8Q4b7U0S2Q4b7V1q4Q4y4U0l9`.</mark><iframe width=20 height=30 src=<mark class="encrypted">338K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6i4y4@1i4K6u0W2M7r3S2H3</mark>></iframe>`

> 然后`<mark class="encrypted">f9bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6i4y4@1i4K6u0W2M7r3S2H3i4K6j5H3</mark> 是对`a.com`域设置cookie的页面，其内容为：
>
> ```
> <?php header("set-cookie:test=axis;path=/")?>
> ```
