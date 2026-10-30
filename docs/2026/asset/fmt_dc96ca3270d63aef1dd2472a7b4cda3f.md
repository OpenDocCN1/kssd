
# 一. 路径穿越

关于路径穿越比较经典的例子有`tp-link`的，`a1fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4Q4x3X3c8U0L8$3&6K6N6h3I4@1i4K6u0W2j5$3!0E0i4K6u0r3k6Y4S2V1j5i4c8S2i4K6u0r3M7$3g2U0j5$3!0F1M7#2)9J5c8Y4m8J5L8$3c8Q4x3V1k6@1k6h3#2W2k6r3W2S2i4K6u0r3j5h3c8$3K9i4y4G2M7X3W2W2M7#2)9#2k6Y4c8^5N6q4)9J5c8U0t1H3x3e0f1H3y4o6p5H3i4K6u0V1x3q4)9#2k6W2c8b7i4K6u0V1e0r3W2F1K9#2)9#2k6W2g2F1j5i4g2@1K9r3g2F1N6r3W2U0j5i4c8W2k6q4)9#2k6X3I4G2j5$3q4D9i4K6g2X3k6X3W2D9k6g2)9#2k6X3c8A6M7$3y4D9L8%4y4#2M7X3g2Q4y4h3k6$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6g2X3N6U0p5H3i4K6u0W2N6s2S2@1i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1%4i4K6W2n7i4@1t1@1i4@1f1$3i4K6S2q4i4@1p5#2i4@1f1^5i4@1q4r3i4@1u0n7i4@1f1#2i4K6S2r3i4K6V1$3i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1#2i4@1q4r3i4K6R3$3i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1K6i4K6R3H3i4K6R3J5`

# 二. 溢出漏洞

其中`GoAhead`历史漏洞就有`2a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0t1I4y4K6l9%4`

以`redirect`为例，当你访问目标机器当前不允许访问的页面时会把你重定向到登陆页面。但是其中重定向`host`是由你发的包的`host`头决定的，如果你发的越长，返回的也就越长，如果没有准备足够的空间就会造成溢出.

这么说比较模糊，可以看`d-link`的溢出漏洞，`CVE-2018-11013`，相关文章：`1baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6y4X3z5e0N6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3g2^5M7r3I4G2K9i4c8Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6g2Q4x3V1j5I4x3#2)9J5c8V1c8Q4x3X3c8x3K9h3&6C8i4K6u0V1c8p5W2d9i4K6u0V1z5o6p5$3i4K6u0V1b7e0u0Q4x3X3c8o6e0W2)9J5k6s2u0G2N6i4c8W2M7W2)9J5k6s2y4@1j5h3y4C8i4K6u0V1j5X3q4K6k6h3c8Q4x3X3c8T1N6h3k6X3k6i4u0Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5c8R3`.`.

`GoAhead`

![1552128134891](https://bbs.kanxue.com/upload/attach/201903/847726_EFB9XM7D974AAZA.png)

这里从代码分析。

```c
static int websHomePageHandler(webs_t wp, char_t *urlPrefix, char_t *webDir,

                            int arg, char_t *url, char_t *path, char_t *query)

{

/*

 *    If the empty or "/" URL is invoked, redirect default URLs to the home page

 */

    if (*url == '\0' || gstrcmp(url, T("/")) == 0) {

        websRedirect(wp, T("home.asp"));

        return 1;

    }

    return 0;

}
```

```c
void websRedirect(webs_t wp, char_t *url)

{

    char_t    *msgbuf, *urlbuf, *redirectFmt;



    a_assert(websValid(wp));

    a_assert(url);



    websStats.redirects++;

    msgbuf = urlbuf = NULL;



/*

 *    Some browsers require a http://host qualified URL for redirection

 */

    if (gstrstr(url, T("http://")) == NULL) {

        if (*url == '/') {

            url++;

        }



        redirectFmt = T("http://%s/%s");



#ifdef WEBS_SSL_SUPPORT

        if (wp->flags & WEBS_SECURE) {

            redirectFmt = T("https://%s/%s");

        }

#endif



        fmtAlloc(&urlbuf, WEBS_MAX_URL + 80, redirectFmt,

            websGetVar(wp, T("HTTP_HOST"),     websHostUrl), url);

        url = urlbuf;

    }



/*

 *    Add human readable message for completeness. Should not be required.

 */

    fmtAlloc(&msgbuf, WEBS_MAX_URL + 80, 

        T("<html><head></head><body>\r\n\

        This document has moved to a new <a href=\"%s\">location</a>.\r\n\

        Please update your documents to reflect the new location.\r\n\

        </body></html>\r\n"), url);



    websResponse(wp, 302, msgbuf, url);



    bfreeSafe(B_L, msgbuf);

    bfreeSafe(B_L, urlbuf);

}
```

其中`WEBS_MAX_URL`，`#define WEBS_MAX_URL 4096 /*Maximum URL size for sanity*/`，尝试过超过`4096`，`burpsuit`显示`done`，但是没有响应包，不会崩溃。

用`burpsuit`测了好多其他的变量，都不能达到效果。

![1552128278785](https://bbs.kanxue.com/upload/attach/201903/847726_ZNJPA55V8KM5ZGT.png)

![1552128312667](https://bbs.kanxue.com/upload/attach/201903/847726_HNG5A8MVMYTGMH6.png)

![1552128333259](https://bbs.kanxue.com/upload/attach/201903/847726_4AN5EGPE7DDCNEQ.png)

# 三. web服务对URL的“处理”

在最开始接触路由器安全的时候对URL各种请求路径还是很疑惑的，下面就结合`ruckus` `webs`二进制文件和源码分析。

IDA反汇编`main`函数片段，开头有一大段这些`mips`汇编，是不是不知道该如何分析呢

```asm
w      $a1, (dword_10000098 - 0x10000000)($v0)

jalr    $t9 ; websOpenListen

nop

lw      $gp, 0xB0+var_98($sp)

li      $v0, 1

la      $t9, websUrlHandlerDefine

la      $a3, websSecurityHandler

addiu   $a0, $s0, (asc_44F540+4 - 0x450000)  # ""

move    $a1, $zero

move    $a2, $zero

jalr    $t9 ; websUrlHandlerDefine

sw      $v0, 0xB0+var_A0($sp)

lw      $gp, 0xB0+var_98($sp)

move    $a1, $zero

la      $a0, _rsm_net_IF_ToBridge

la      $t9, websUrlHandlerDefine

la      $a3, websFormHandler

addiu   $a0, (aForms - 0x440000)  # "/forms"

move    $a2, $zero

jalr    $t9 ; websUrlHandlerDefine

sw      $zero, 0xB0+var_A0($sp)

lw      $gp, 0xB0+var_98($sp)

move    $a1, $zero

la      $a0, _rsm_net_IF_ToBridge

la      $t9, websUrlHandlerDefine

la      $a3, websCgiHandler

addiu   $a0, (aCgiBin - 0x440000)  # "/cgi-bin"

move    $a2, $zero

jalr    $t9 ; websUrlHandlerDefine

sw      $zero, 0xB0+var_A0($sp)

lw      $gp, 0xB0+var_98($sp)

move    $a1, $zero

la      $a0, _rsm_net_IF_ToBridge

la      $t9, websUrlHandlerDefine

la      $a3, supportInfoURLHandler

addiu   $a0, (aSupportinfoTxt - 0x440000)  # "/supportinfo.txt"

move    $a2, $zero

jalr    $t9 ; websUrlHandlerDefine

sw      $zero, 0xB0+var_A0($sp)

lw      $gp, 0xB0+var_98($sp)
```

好在我们可以看源码

看`initWebs()`

```c
int main(int argc, char** argv)

{

/*

 *    Initialize the memory allocator. Allow use of malloc and start 

 *    with a 60K heap.  For each page request approx 8KB is allocated.

 *    60KB allows for several concurrent page requests.  If more space

 *    is required, malloc will be used for the overflow.

 */

    bopen(NULL, (60 * 1024), B_USE_MALLOC);

    signal(SIGPIPE, SIG_IGN);



/*

 *    Initialize the web server

 */

    if (initWebs() < 0) {// 这个是我们要关注的

        return -1;

    }



#ifdef WEBS_SSL_SUPPORT

    websSSLOpen();

#endif



/*

 *    Basic event loop. SocketReady returns true when a socket is ready for

 *    service. SocketSelect will block until an event occurs. SocketProcess

 *    will actually do the servicing.

 */

    while (!finished) {

        if (socketReady(-1) || socketSelect(-1, 1000)) {

            socketProcess(-1);

        }

        websCgiCleanup();

        emfSchedProcess();

    }



#ifdef WEBS_SSL_SUPPORT

    websSSLClose();

#endif



#ifdef USER_MANAGEMENT_SUPPORT

    umClose();

#endif



/*

 *    Close the socket module, report memory leaks and close the memory allocator

 */

    websCloseServer();

    socketClose();

#ifdef B_STATS

    memLeaks();

#endif

    bclose();

    return 0;

}
```
