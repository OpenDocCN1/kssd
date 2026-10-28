
# [旧帖][原创]程序模拟用户登陆腾讯微博0.00雪花

因为需要针对腾讯微博编写爬虫,所以第一步肯定要登陆到微博上.腾讯提供的`api`接口.无奈接口的显示条件太多.所以就想模拟用户的登陆来进入页面.过程还是蛮好玩的.腾讯在登陆过程中用了多次随机加密,,多次跳转.不过最终还是成功的进入了.

写下模拟登陆的过程…

第一次发帖.希望大家多多指教啊…

打开抓包工具,进入登陆页面.

`768K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2A6i4K6u0W2M7s2c8D9L8$3N6A6L8U0u0Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5c8X3I4G2k6$3W2F1i4K6g2X3N6$3g2A6j5X3!0Q4x3X3g2Z5N6r3#2D9i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1@1i4@1u0m8i4K6R3$3`

`136K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2A6i4K6u0W2M7s2c8D9L8$3N6A6L8U0u0Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5c8X3I4G2k6$3W2F1i4K6g2X3N6q4)9J5k6h3A6K6`

`1caK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2#2K9g2)9J5k6i4m8@1L8r3!0Y4K9h3^5J5i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6%4j5W2)9J5c8Y4q4D9L8$3N6A6L8W2)9#2k6Y4c8Q4x3X3g2Z5N6r3#2D9i4K6y4r3N6W2)9K6c8o6p5H3x3o6j5H3x3U0l9I4`

`61eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2A6i4K6u0W2M7s2c8D9L8$3N6A6L8U0u0Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5c8X3A6K6i4K6u0r3N6W2)9J5k6h3A6K6i4K6y4r3M7W2)9K6c8o6m8Q4x3X3f1@1x3U0p5J5x3o6t1%4z5e0M7I4y4U0M7^5x3U0b7#2y4b7`.`.

`f5bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2A6i4K6u0W2M7s2c8D9L8$3N6A6L8U0u0Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5c8X3y4D9k6h3q4J5j5$3q4U0K9r3g2Q4x3X3g2Z5N6r3#2D9`

`e9bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2A6i4K6u0W2M7s2c8D9L8$3N6A6L8U0u0Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5c8X3I4G2k6$3W2F1i4K6g2X3N6$3g2A6j5X3!0Q4x3X3g2Z5N6r3#2D9`

`6d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2E0k6$3y4S2j5$3S2W2i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6r3I4G2k6$3W2F1i4K6u0r3j5h3y4Q4x3V1k6$3y4W2)9J5c8X3A6K6i4K6u0r3M7h3I4G2k6$3W2F1i4K6g2X3N6q4)9J5k6h3A6K6i4K6y4r3N6W2)9K6c8o6q4Q4x3X3f1I4`

2秒后又请求了

`12eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8@1L8r3!0Y4K9h3^5J5i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3g2U0K9#2)9K6c8Y4g2A6L8W2)9K6c8q4)9J5y4X3N6@1i4K6y4n7i4@1f1^5i4@1t1@1i4@1p5$3i4@1f1#2i4K6S2r3i4@1t1%4i4K6t1$3L8s2c8Q4x3@1u0Q4x3U0k6S2M7s2m8A6k6q4)9K6c8o6b7$3x3o6l9H3x3e0l9I4i4K6t1$3M7W2)9K6c8o6m8Q4x3X3f1#2x3o6p5I4y4o6M7@1z5o6l9J5z5e0b7I4y4K6l9#2`

不管`html`页面 只看`js`的话.

`3e3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2A6i4K6u0W2M7s2c8D9L8$3N6A6L8U0u0Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5c8X3I4G2k6$3W2F1i4K6g2X3N6q4)9J5k6h3A6K6` 返回的是一个`js`

`cffK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2E0k6$3y4S2j5$3S2W2i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6r3I4G2k6$3W2F1i4K6u0r3j5h3y4Q4x3V1k6$3y4W2)9J5c8X3A6K6i4K6u0r3M7h3I4G2k6$3W2F1i4K6g2X3N6q4)9J5k6h3A6K6i4K6y4r3N6W2)9K6c8o6q4Q4x3X3f1I4` 返回的是 `ptuiV(10062901);`

`00cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8@1L8r3!0Y4K9h3^5J5i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3g2U0K9#2)9K6c8Y4g2A6L8W2)9K6c8q4)9J5y4X3N6@1i4K6y4n7i4@1f1^5i4@1t1@1i4@1p5$3i4@1f1#2i4K6S2r3i4@1t1%4i4K6t1$3L8s2c8Q4x3@1u0Q4x3U0k6S2M7s2m8A6k6q4)9K6c8o6b7$3x3o6l9H3x3e0l9I4i4K6t1$3M7W2)9K6c8o6m8Q4x3X3f1#2x3o6p5I4y4o6M7@1z5o6l9J5z5e0b7I4y4K6l9#2`

返回的是类似`ptui_checkVC('0','!2HV');`的信息

估计这三个地方比较重要..,而`d0bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8@1L8r3!0Y4K9h3^5J5i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3g2U0K9#2)9K6c8Y4g2A6L8W2)9K6c8q4)9J5y4X3N6@1i4K6y4n7i4@1f1^5i4@1t1@1i4@1p5$3i4@1f1#2i4K6S2r3i4@1t1%4i4K6t1$3L8s2c8Q4x3@1u0Q4x3U0k6S2M7s2m8A6k6q4)9K6c8o6b7$3x3o6l9H3x3e0l9I4i4K6t1$3M7W2)9K6c8o6m8Q4x3X3f1#2x3o6p5I4y4o6M7@1z5o6l9J5z5e0b7I4y4K6l9#2i4@1f1@1i4@1t1^5i4@1u0m8i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1^5i4@1q4r3i4@1t1%4i4@1f1$3i4@1t1I4i4K6R3J5` 应该在`js`中有调用

搜索下`js`的源码.

```
var B="http://ptlogin2."+g_domain+"/check?uin="+C+"&appid="+D+"&r="+Math.random();
```

可得知.`r`为随机数.而`appid`为`46000101`,固定值.`uin`为账号.而返回的`ptui_checkVC('0','!2HV');`则是调用`ptui_checkVC`函数.进行分析.得知.

是将返回值复制给页面的一个`id`为`verifyinput`隐藏域.用来在提交登陆的时候传值.

登陆.抓包可得.请求方式为`GET`.完整的`URL`如下

`f01K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8@1L8r3!0Y4K9h3^5J5i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3N6A6L8W2)9K6c8Y4g2Q4x3@1c8Q4x3U0k6Y4N6q4)9K6b7W2!0q4z5q4!0n7y4q4!0m8y4W2!0q4y4g2)9^5c8W2!0n7y4#2)9J5y4X3I4@1i4K6y4n7i4K6t1$3M7q4)9K6c8p5j5@1y4o6c8q4y4U0y4n7x3K6g2q4y4U0q4m8c8o6V1#2c8f1p5#2z5e0m8q4x3p5y4q4x3e0p5J5x3K6c8n7i4K6t1$3N6X3g2J5K9h3k6&6j5$3!0V1k6g2)9K6c8q4)9J5x3f1I4f1d9#2)9J5y4X3q4A6k6q4)9K6c8o6b7$3x3o6l9H3x3e0l9I4i4K6t1$3N6e0q4Q4x3@1c8Z5N6s2c8H3i4K6t1#2x3@1q4Q4x3U0f1J5c8W2)9J5y4e0u0r3N6q4)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6t1$3K9q4)9K6c8o6q4Q4x3U0k6X3M7X3!0E0i4K6g2X3N6h3W2Q4x3@1b7I4i4K6t1$3k6Y4m8Q4x3@1c8D9L8$3N6A6L8X3g2J5M7X3!0J5j5h3I4W2M7Y4b7.`

通过页面源码可知.

`U`为用户名

`P`为密码.加密后

`Verifycode`为刚才获取的值

`Aid`为固定值

`u1=http%3A%2F%2Ft.qq.com&h=1&from_ui=1&fp=loginerroralert`未知,

再次抓包

`da9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8@1L8r3!0Y4K9h3^5J5i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3N6A6L8W2)9K6c8Y4g2Q4x3@1c8Q4x3U0k6Y4N6q4)9K6b7W2!0q4z5q4!0n7y4q4!0m8y4W2!0q4y4g2)9^5c8W2!0n7y4#2)9J5y4X3I4@1i4K6y4n7i4K6t1$3M7q4)9K6c8o6u0m8x3K6x3H3c8e0M7J5z5o6M7J5x3p5f1J5x3e0M7K6x3e0V1%4x3K6x3H3c8e0t1H3x3V1p5$3y4U0b7J5i4K6t1$3N6X3g2J5K9h3k6&6j5$3!0V1k6g2)9K6c8q4)9J5x3f1D9J5e0#2)9J5y4X3q4A6k6q4)9K6c8o6b7$3x3o6l9H3x3e0l9I4i4K6t1$3N6e0q4Q4x3@1c8Z5N6s2c8H3i4K6t1#2x3@1q4Q4x3U0f1J5c8W2)9J5y4e0u0r3N6q4)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6t1$3K9q4)9K6c8o6q4Q4x3U0k6X3M7X3!0E0i4K6g2X3N6h3W2Q4x3@1b7I4i4K6t1$3k6Y4m8Q4x3@1c8D9L8$3N6A6L8X3g2J5M7X3!0J5j5h3I4W2M7Y4b7.`

发现.

`P` 和`Verifycode`和上次不同.其他值不变.暂时不管.而`Verifycode`我们已经知道是怎么来的了重点关注`P`

分析页面源码,发现在登录时 实际上市调用了`ptui_checkValidate`这个函数而该函数又调用了`ajax_Submit`.这个函数.

```
if(E[A].name=="p"){
var F="";F+=E.verifycode.value;
F=F.toUpperCase();
B+=md5(md5_3(E.p.value)+F)
```

简单看下 可以得知`p`是由`MD5_3`的密码加密后再和验证串联后再次加密获得..继续跟进.

可以得知腾讯的加密方式还是很复杂的..共有

`md5_3`

`md5`

`hex_md5`

`str_md5`

`core_md5`

`md5_cmn`

`md5_ff`

`md5_gg`

`md5_hh`

`md5_ii`

`safe_add`

`bit_rol`

`str2binl`

`binl2str`

`binl2hex`

等函数.不用仔细看了 直接转成`JAVA`来测试一下吧.

返回 `ptuiCB('0','0','f36K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5y4#2)9J5b7#2)9J5y4K6m8Q4x3U0N6Q4x3V1y4Q4x3U0N6Q4c8e0N6Q4z5e0W2Q4b7V1u0Q4c8e0g2Q4b7V1c8Q4z5e0g2Q4c8e0k6Q4z5o6S2Q4z5e0m8Q4c8e0g2Q4z5p5q4Q4z5f1k6Q4c8f1k6Q4b7V1y4Q4z5o6q4Q4x3U0N6Q4x3U0W2Q4x3@1t1'.`

返回登录成功.但是这时还不能进入个人的主页.而且看着还像是函数.难道还是再次执行.在`js`文件中查找 果然…

```
if(C!=""||D==0){
var A=new Date();
A.setHours(A.getHours()+24*30);
document.cookie="ptui_loginuin2="+escape(f_u.value)+";expires ="+A.toGMTString()+";domain =ui.ptlogin2.qq.com";
top.location.href=C;return
}
```

发现为设置`cookie`.既然是设置`cookie`.我们就抓包来看

请求

`登陆请求`

`Cookie: pt2gguin=o0>账号<; pgv_pvid=8173925820; pgv_flv=10.0; pgv_r_cookie=1162840646389; o_cookie=>账号<; ac=1,009,008; uin=o0>账号<; skey=@rAt7FjYfB; ptisp=ctc; pgv_info=ssid=s9544862000; ptcz=8212ca17ace4b736761ff883f0b20a6bf77dcdcc52fee257377f143723b3ce5c; ptui_qstatus=1; ptvfsession=71c028e3b65b11ea1c3be3c00c7f8e0b74d39b3e8b6ab5f9b21a6899c1bc979c54b80640181bb90170196609ba4f5382; ptuserinfo=e29982e58fb3e7bebd`

登陆返回

`Set-Cookie: pt2gguin=o0>账号<; EXPIRES=Fri, 02-Jan-2020 00:00:00 GMT; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: uin=o0XXXXXXX; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: skey=@rAt7FjYfB; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: clientuin=; EXPIRES=Fri, 02-Jan-1970 00:00:00 GMT; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: clientkey=; EXPIRES=Fri, 02-Jan-1970 00:00:00 GMT; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: zzpaneluin=; EXPIRES=Fri, 02-Jan-1970 00:00:00 GMT; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: zzpanelkey=; EXPIRES=Fri, 02-Jan-1970 00:00:00 GMT; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: ptisp=ctc; PATH=/; DOMAIN=qq.com;`

`Set-Cookie: ptuserinfo=e29982e58fb3e7bebd; PATH=/; DOMAIN=ptlogin2.qq.com;`

跳转请求

`Cookie: uin=o0>账号<; skey=@rAt7FjYfB; ptisp=ctc; pgv_info=ssid=s3593389632; pt2gguin=o0XXXXXXX; mb_reg_from=8`

有几个字段不一样.是从请求头中新返回的..设置进去..

..登陆成功,进入主页.至此 就通过程序模拟用户的登陆,并成功的绕做了腾讯的`OAUTH`验证..

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
