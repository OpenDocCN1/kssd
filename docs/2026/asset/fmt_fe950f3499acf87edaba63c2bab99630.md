
# [原创]挖掘ruckus路由器漏洞总结的几个点

## 一.防止路径穿越

在挖掘ruckus服务器的漏洞过程中，首先就是想要登陆进去。所以要找一个路径穿越的漏洞，想到了可以用`../../../`或者是URL编码后的`%2e%2e%2f%2e%2e%2f%2e%2e%2f`，进而读取密码文件，但是实践证明不行。比如`supportinfo.txt`是`/www`下的一个文件，用来存储路由器的log。如果存在路径穿越，则`GET /images/../../../../../../../../../../../supportinfo.txt`应该读取不到文件，但最后还是能读到`supportinfo.txt`。说明将`../`以某种方式限制在了`/www`目录

ruckus使用的是`GoAhead2.1.8`，让我们看看其源码做了哪些处理吧

在这之前可以看看历史漏洞：

<mark class="encrypted">108K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0t1I4y4K6l9%4</mark>

<mark class="encrypted">cc3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0t1H3y4U0l9%4</mark>

```c
int websDefaultHandler(webs_t wp, char_t *urlPrefix, char_t *webDir, int arg,
char_t *url, char_t *path, char_t *query)
{
websStatType sbuf;
char_t *lpath, *tmp, *date;
int bytes, flags, nchars;
flags = websGetRequestFlags(wp);
// 此处处理传入路径 websValidateUrl，对其中的../和其他一些情况作了处理
if (websValidateUrl(wp, path) < 0)
{
/*
* preventing a cross-site scripting exploit -- you may restore the
* following line of code to revert to the original behavior...
*/
/*websError(wp, 500, T("Invalid URL %s"), url);*/
websError(wp, 500, T("Invalid URL"));
return 1;
}
lpath = websGetRequestLpath(wp);
nchars = gstrlen(lpath) - 1;
if (lpath[nchars] == '/' || lpath[nchars] == '\\') {
lpath[nchars] = '\0';
}
//。。。。
}
```

```c
int websValidateUrl(webs_t wp, char_t *path)
{
/*
Thanks to Dhanwa T (dhanwa@polyserve.com) for this fix -- previously,
if an URL was requested having more than (the hardcoded) 64 parts,
the webServer would experience a hard crash as it attempted to
write past the end of the array 'parts'.
*/
#define kMaxUrlParts 64 // 此处是防止溢出,当./././././层级太多时，这个用来矫正URL的函数反而会出现溢出，为什么和层级有关请看下面。https://www.exploit-db.com/exploits/21707
char_t *parts[kMaxUrlParts]; /* Array of ptr's to URL parts */ //用来存储路径的每一层的字符串
char_t *token, *dir, *lpath;
int i, len, npart;
a_assert(websValid(wp));
a_assert(path);
dir = websGetRequestDir(wp);
if (dir == NULL || *dir == '\0') {
return -1;
}
/*
* Copy the string so we don't destroy the original
*/
path = bstrdup(B_L, path);
websDecodeUrl(path, path, gstrlen(path));
//初始化 层数以及第一层的指针
len = npart = 0;
parts[0] = NULL;
/*
* 22 Jul 02 -- there were reports that a directory traversal exploit was
* possible in the WebServer running under Windows if directory paths
* outside the server's specified root web were given by URL-encoding the
* backslash character, like:
*
* GoAhead is vulnerable to a directory traversal bug. A request such as
*
* GoAhead-server/../../../../../../../ results in an error message
* 'Cannot open URL'.
* However, by encoding the '/' character, it is possible to break out of
* the
* web root and read arbitrary files from the server.
* Hence a request like:
*
* GoAhead-server/..%5C..%5C..%5C..%5C..%5C..%5C/winnt/win.ini returns the
* contents of the win.ini file.
* (Note that the description uses forward slashes (0x2F), but the example
* uses backslashes (0x5C). In my tests, forward slashes are correctly
* trapped, but backslashes are not. The code below substitutes forward
* slashes for backslashes before attempting to validate that there are no
* unauthorized paths being accessed.
*/
token = gstrchr(path, '\\');
while (token != NULL)
{
*token = '/';
token = gstrchr(token, '\\');//把\替换成/，在windows平台会出现这个问题，参考链接https://www.exploit-db.com/exploits/20607
}
token = gstrtok(path, T("/"));
/*
* Look at each directory segment and process "." and ".." segments
* Don't allow the browser to pop outside the root web.
*/
// 下面这个循环是处理的核心，函数把url按/解析成一层一层的，存储每一层名称的是char_t *parts[kMaxUrlParts];
//如果是../就把当前层数index往上移动，路径中永远不会包含../，也就解决了路径穿越的问题
while (token != NULL)
{
if (npart >= kMaxUrlParts)// 判断是否达到了最大层,防止溢出
{
/*
* malformed URL -- too many parts for us to process.
*/
bfree(B_L, path);
return -1;
}
if (gstrcmp(token, T("..")) == 0) // 判断该层是不是..，如果是就把定位层数的npart--
{
if (npart > 0)
{
npart--;
}
}
else if (gstrcmp(token, T(".")) != 0) //如果不是.,就层数+1，将该层存储
{
parts[npart] = token;
len += gstrlen(token) + 1;
npart++; //
}
token = gstrtok(NULL, T("/"));
}
#ifdef WIN32
if (isBadWindowsPath(parts, npart))
{
bfree(B_L, path);
return -1;
}
#endif
/*
* Create local path for document. Need extra space all "/" and null.
*/
if (npart || (gstrcmp(path, T("/")) == 0) || (path[0] == '\0'))
{
lpath = balloc(B_L, (gstrlen(dir) + 1 + len + 1) * sizeof(char_t));
gstrcpy(lpath, dir);
for (i = 0; i < npart; i++)
{
gstrcat(lpath, T("/"));
gstrcat(lpath, parts[i]);
}
websSetRequestLpath(wp, lpath);
bfree(B_L, path);
bfree(B_L, lpath);
}
else
{
bfree(B_L, path);
return -1;
}
return 0;
}
```

在多次尝试过后，没法读取在`/www`外的文件。

虽然如此但还是发现了信息泄露的漏洞，ruckus路由器请求路径以`/images/`开头可以任意读取`/www`中的文件，其中`supportinfo.txt`包含了各种调试信息，（但`supportinfo.txt`必须在登陆调用相关功能后生成，如果目标曾经使用了这个功能就可以读取到相关内容），可造成信息泄露，比如ps信息，版本信息等。

关于路径穿越比较经典的例子有tp-link的，<mark class="encrypted">c68K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4Q4x3X3c8U0L8$3&6K6N6h3I4@1i4K6u0W2j5$3!0E0i4K6u0r3k6Y4S2V1j5i4c8S2i4K6u0r3M7$3g2U0j5$3!0F1M7#2)9J5c8Y4m8J5L8$3c8Q4x3V1k6@1k6h3#2W2k6r3W2S2i4K6u0r3j5h3c8$3K9i4y4G2M7X3W2W2M7#2)9#2k6Y4c8^5N6q4)9J5c8U0t1H3x3e0f1H3y4o6p5H3i4K6u0V1x3q4)9#2k6W2c8b7i4K6u0V1e0r3W2F1K9#2)9#2k6W2g2F1j5i4g2@1K9r3g2F1N6r3W2U0j5i4c8W2k6q4)9#2k6X3I4G2j5$3q4D9i4K6g2X3k6X3W2D9k6g2)9#2k6X3c8A6M7$3y4D9L8%4y4#2M7X3g2Q4y4h3k6$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6g2X3N6U0p5H3i4K6u0W2N6s2S2@1i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1%4i4K6W2n7i4@1t1@1i4@1f1$3i4K6S2q4i4@1p5#2i4@1f1^5i4@1q4r3i4@1u0n7i4@1f1#2i4K6S2r3i4K6V1$3i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1#2i4@1q4r3i4K6R3$3i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1K6i4K6R3H3i4K6R3J5</mark>

## 二.溢出漏洞

其中GoAhead历史漏洞就有<mark class="encrypted">8e9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0t1I4y4K6l9%4</mark>

以`redirect`为例，当你访问目标机器当前不允许访问的页面时会把你重定向到登陆页面。但是其中重定向`host`是由你发的包的`host`头决定的，如果你发的越长，返回的也就越长，如果没有准备足够的空间就会造成溢出.

这么说比较模糊，可以看d-link的溢出漏洞，`CVE-2018-11013`，相关文章：<mark class="encrypted">89cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6y4X3z5e0N6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3g2^5M7r3I4G2K9i4c8Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6g2Q4x3V1j5I4x3#2)9J5c8V1c8Q4x3X3c8x3K9h3&6C8i4K6u0V1c8p5W2d9i4K6u0V1z5o6p5$3i4K6u0V1b7e0u0Q4x3X3c8o6e0W2)9J5k6s2u0G2N6i4c8W2M7W2)9J5k6s2y4@1j5h3y4C8i4K6u0V1j5X3q4K6k6h3c8Q4x3X3c8T1N6h3k6X3k6i4u0Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5c8R3`.`.</mark>

GoAhead

![1552128134891](/upload/attach/201903/847726_EFB9XM7D974AAZA.png)

这里从代码分析。

```c
static int websHomePageHandler(webs_t wp, char_t *urlPrefix, char_t *webDir,
int arg, char_t *url, char_t *path, char_t *query)
{
/*
* If the empty or "/" URL is invoked, redirect default URLs to the home page
*/
if (*url == '\0' || gstrcmp(url, T("/")) == 0) {
websRedirect(wp, T("home.asp"));
return 1;
}
return 0;
}
```
