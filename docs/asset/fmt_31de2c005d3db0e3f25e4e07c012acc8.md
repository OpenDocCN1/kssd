
# 商务奇兵 2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum494.htm`](https://www.pediy.com/kssd/pediy03/forum494.htm)

## 软件信息

- **软件名**：商务奇兵
- **软件版本**：2.0
- **软件大小**：971KB
- **软件授权**：共享软件
- **使用平台**：Win95/98/NT
- **发布公司**：[http://www.worldfax.net/](http://www.worldfax.net/)

## 软件简介

软件将您的进出口广告信息在一小时内同时自动刊登到世界知名的 3000 多个从事国际贸易的电子公告版和 300 个新闻组上。可以无限次发布，无需提前在各公告版注册，具备刊登情况统计。电子公告版是使用最频繁也是信息沟通最快捷和最有效的一种方案。如果有客户在找寻某个项目和服务而正是你所能提供的，而其中有某个客户在电子公告版看到你发布的消息，那么你就抓到了机会。

## 破译者后记

空中飘着鹅毛大雪,(只有北方下这么大的雪)在雪中深吸两口凉气,倍感凉爽清新.脚下踩着厚厚的雪发出咯吱咯吱的声音,觉得那样亲切,仿佛世界凝固啦!

...啪!..谁拿雪球打我!..对不起!

好久没见到这样的大雪啦,...是!马上开工.

好大的雪呀!....别打.

其实这个软件破解很简单，简单的不敢相信,这里只做学习交流之用，为了国产软件的发展，请各位大虾破费注册。

**所需工具**：`w32dsm`、`aspack` 脱壳工具、编辑器

## 破解过程

1. `[商务奇兵]`是中文软件，压缩之后在`w32dsm`中找不到我们所需要的东西。所以先对软件脱壳，过程略。
2. 打开`[商务奇兵]`注册，随便输入姓名和注册码，肯定是错误的，点击确定，弹出提示框，记住提示内容`invalid key...`。
3. 打开`w32dsm`装入`[商务奇兵]`查找错误提示`invalid key..`
4. 哇噻!...找到了耶!!!

```
:004C3B6C B9903D4C00  mov ecx, 004C3D90
:004C3B71 5A  pop edx
:004C3B72 E84DFDFFFF  call 004C38C4
:004C3B77 84C0  test al, al
:004C3B79 0F8487010000  je 004C3D06  //修改处
```

------------------------------------------------------------
略：
------------------------------------------------------------

```
* Possible StringData Ref from Code Obj ->"Thanks for registered"//出钱就微笑!
                                |
:004C3CE9 B8403E4C00  mov eax, 004C3E40
:004C3CEE E87D14F9FF  call 00455170
:004C3CF3 C783E002000001000000  mov dword ptr [ebx+000002E0], 00000001
:004C3CFD 8BC3  mov eax, ebx
:004C3CFF E84082F8FF  call 0044BF44
:004C3D04 EB0A  jmp 004C3D10

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C3B79(C)
|
* Possible StringData Ref from Code Obj ->"Invalid key. Please re enter"//错误提示!*
                                |
:004C3D06 B8603E4C00  mov eax, 004C3E60
:004C3D0B E86014F9FF  call 00455170
```

## 修改方法

在错误提示处向上走,看到`004C3B79(C)`啦吧.

去吧!在`004C3B79`处进行修改,别说不会修改.

OK

退出,进入`[商务奇兵]`注册时随便输入.爽!!

---

《阿郎居》
`langlirong@163.net`
[http://alang.shangdu.net](http://alang.shangdu.net)
2000 年 12 月 25 日
`4dd466d0e1e0cc123afbe504da2462ca`
