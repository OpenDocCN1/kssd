
我总是遇到试图阻止第三方重定向的过滤器。然而，在考虑绕过过滤器之前，人们在测试使用一个开放式重定向的网站的登录流程时，碰到的一个最常见的问题就是没有正确编码这些值。例如，`71cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2P5r3q4E0M7r3I4W2i4K6u0W2j5$3!0E0i4K6u0r3L8r3!0Y4K9h3&6Q4x3@1k6J5k6i4c8#2M7X3&6Q4x3@1c8Z5N6s2c8H3M7#2)9K6b7g2)9J5c8W2)9J5c8X3#2&6M7$3W2@1k6g2)9J5k6h3y4G2L8g2)9J5c8W2!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4#2!0m8b7W2)9&6z5g2!0q4y4W2)9^5z5q4)9&6y4W2!0q4y4W2!0n7y4g2)9^5c8W2!0q4z5q4!0m8y4#2)9^5z5q4!0q4y4g2)9&6z5g2!0m8z5q4!0q4y4W2)9^5z5q4)9&6y4W2!0q4z5q4!0m8c8g2!0n7z5q4!0q4y4q4!0n7z5q4)9^5c8q4!0q4z5q4)9^5x3#2!0n7c8q4!0q4z5q4!0m8c8W2)9^5y4W2!0q4y4g2)9^5z5q4!0m8b7W2!0q4y4g2)9^5y4g2!0n7y4W2!0q4z5q4!0n7c8W2)9&6y4q4!0q4y4g2)9&6b7W2)9&6c8g2!0q4y4g2)9^5c8W2)9^5x3W2!0q4y4W2)9&6y4g2!0n7x3q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4W2!0m8x3q4!0n7b7#2!0q4y4g2!0n7b7#2)9^5c8W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9&6b7W2!0m8x3q4!0q4y4W2!0m8c8q4!0m8y4q4!0q4y4g2!0n7x3q4)9&6c8q4!0q4z5q4!0m8c8W2)9&6y4g2!0q4y4g2!0n7z5q4!0n7z5q4!0q4z5q4!0m8y4#2)9^5y4q4!0q4y4#2!0n7b7#2)9&6y4W2!0q4y4#2!0m8x3q4)9^5x3g2!0q4y4W2)9^5x3q4!0n7b7W2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4g2!0m8y4g2!0n7c8q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9&6x3q4!0m8y4W2!0q4y4g2)9^5z5q4)9&6z5g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4g2!0n7x3q4)9&6c8q4!0q4z5q4!0m8c8W2)9&6y4g2!0q4y4g2)9^5c8W2)9^5b7#2!0q4z5g2)9^5y4#2)9^5c8q4!0q4y4#2!0n7b7#2)9&6y4W2!0q4y4#2!0m8x3q4)9^5x3g2!0q4x3#2)9^5x3q4)9^5x3R3.`.

> `7dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2P5r3q4E0M7r3I4W2i4K6u0W2j5$3!0E0i4K6u0r3L8r3!0Y4K9h3&6Q4x3@1k6J5k6i4c8#2M7X3&6Q4x3@1c8Z5N6s2c8H3M7#2)9J5y4e0y4m8i4K6t1#2x3V1k6Q4x3U0f1J5c8X3#2&6M7$3W2@1k6g2)9J5k6h3y4G2L8g2)9J5y4e0u0r3`

> `7c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2P5r3q4E0M7r3I4W2i4K6u0W2j5$3!0E0i4K6u0r3L8r3!0Y4K9h3&6Q4x3@1k6J5k6i4c8#2M7X3&6Q4x3@1c8Z5N6s2c8H3M7#2)9J5y4e0y4m8i4K6t1#2x3V1k6Q4x3U0f1J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3U0f1J5c8W2)9J5y4e0y4r3M7X3g2@1N6i4u0F1N6i4u0D9i4K6t1#2x3@1c8Q4x3U0f1J5c8W2)9J5y4e0u0r3L8i4W2K6K9i4c8W2i4K6u0W2j5$3!0E0i4K6t1#2x3V1j5.`.

你是否注意到我们在一个链接中放了两个重定向？ 我们需要对最后一次重定向进行双重编码，以便浏览器最后对其进行解码并跳转。有时，我们没能正确的编码，那么，浏览器也不会正确地跳转。

#### 绕过

下面是一些我自己发现的有趣的绕过方式，你可以随时使用，也可以提供其他新的方式。

> `\/yoururl.com`
>
> `\/\/yoururl.com`
>
> `\yoururl.com`
>
> `//yoururl.com`
>
> `//theirsite@yoursite.com`
>
> `92dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2J5M7$3W2@1k6g2)9K6c8X3y4Q4x3@1c8Q4x3X3g2@1K9r3g2A6M7Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1j5`.
>
> `6d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2J5M7$3W2@1k6g2)9J5k6h3y4G2L8g2)9J5x3#2)9J5k6i4c8Z5k6h3W2J5M7$3W2@1k6g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.
>
> `c03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2J5M7$3W2@1k6g2)9J5k6h3y4G2L8g2)9J5k6i4c8Z5k6i4u0K6K9i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3`
>
> `//%2F/yoursite.com`
>
> `////yoursite.com`
>
> `06fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9r3g2A6M7Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2H3N6i4c8W2M7W2)9J5c8R3`.`.
>
> (如果他们只是检查`theirsite.com`，`.computer`是一个有效的tld
>
> `e61K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9r3g2A6M7Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3X3g2E0P5i4y4A6N6r3g2Q4x3X3g2U0L8$3@1`.
>
> 将它的域视为你的子域。
>
> `/%0D/yoursite.com`
> 也试试`%09, %00, %0a, %07`
> ``java%0d%0ascript%0d%0a:alert(0), j%0d%0aava%0d%0aas%0d%0acrip%0d%0at%0d%0a:confirm`0` ,java%07script:prompt`0` ,java%09scrip%07t:prompt`0``` `

这就是现在所有的内容，我希望你们能够喜欢，且能根据需要更新某些方式！

---

翻 译：看雪Logdty

校 对：看雪jasonk龙莲

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
