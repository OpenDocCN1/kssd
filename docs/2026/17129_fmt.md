
# [原创]多线程WEB安全日志分析脚本

为了方便平时定期web服务器安全日志分析，闲暇时写了个小脚本，支持自动`scp`远程web服务器日志到本地，然后多线程安全分析，分析策略是软waf提取到正则，所以基本上目前主流的web攻击行为,sql注入、xss、扫描器扫描、webshell、目录遍历之类都可以配合分析，软waf策略参考：4d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8%4k6W2M7$3S2W2L8r3I4Q4x3V1k6F1k6%4S2Q4y4h3k6D9N6h3q4Q4y4h3k6%4j5h3k6Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3N6$3q4X3j5$3!0F1k6R3`.`.

脚本如下：

```
#!/usr/bin/python

# -\*- coding: utf-8 -\*-

import os

import re

from multiprocessing.dummy import Pool as ThreadPool

import sys

import time

import pexpect

# 规则列表

rulelist = ['\.\./', 'select.+(from|limit)', '(?:(union(.\*?)select))', 'having|rongjitest', 'sleep\((\s\*)(\d\*)(\s\*)\)',

            'benchmark\((.\*)\,(.\*)\)', 'base64\_decode\(', '(?:from\W+information\_schema\W)',

            '(?:(?:current\_)user|database|schema|connection\_id)\s\*\(', '(?:etc\/\W\*passwd)',

            'into(\s+)+(?:dump|out)file\s\*', 'group\s+by.+\(', 'xwork.MethodAccessor',

            '(?:define|eval|file\_get\_contents|include|require|require\_once|shell\_exec|phpinfo|system|passthru|preg\_\w+|execute|echo|print|print\_r|var\_dump|(fp)open|alert|showmodaldialog)\(',

            'xwork\.MethodAccessor', '(gopher|doc|php|glob|file|phar|zlib|ftp|ldap|dict|ogg|data)\:\/',

            'java\.lang', '\$\_(GET|post|cookie|files|session|env|phplib|GLOBALS|SERVER)\[',

            '\<(iframe|script|body|img|layer|div|meta|style|base|object|input)', '(onmouseover|onerror|onload)\=',

            '.(bak|inc|old|mdb|sql|backup|java|class)$', '\.(svn|htaccess|bash\_history)',

            '(vhost|bbs|host|wwwroot|www|site|root|hytop|flashfxp).\*\.rar',

            '(phpmyadmin|jmx-console|jmxinvokerservlet)', 'java\.lang',

            '/(attachments|upimg|images|css|uploadfiles|html|uploads|templets|static|template|data|inc|forumdata|upload|includes|cache|avatar)/(\\w+).(php|jsp)']

SSH\_PASSWD = 'toor' #webserver密码

def Auto\_scp():

    cmd = ['scp -r root@192.168.188.131:/var/log/snort/\* log-2016-07-28/40/',

           'scp -r root@192.168.188.131:/var/log/\*.log log-2016-07-28/39/'

           ]

    for line in cmd:

        child = pexpect.spawn(line,timeout=300)

        child.expect('password:')

        child.sendline(SSH\_PASSWD)

        child.expect(pexpect.EOF)

    return True

def File\_Search(filepath):

    filelist = []

    for lists in os.listdir(filepath):

        path = os.path.join(filepath, lists)

        if os.path.isfile(path):

            filelist.append(path)

        if os.path.isdir(path):

            File\_Search(path)

    pool = ThreadPool(50)

    results = pool.map(Log\_Analysis, filelist)

    pool.close()

    pool.join()

def Log\_Analysis(filename):

    content = open(filename).read()

    r = open('result.txt', 'a') #需要本地先新建个result.txt文件

    r.write('\n' + '=================== web\_log\_secAnalysis ===================' + '\n' + filename + '\n')

    for regex in rulelist:

        result\_tmp = re.compile(regex,re.IGNORECASE).findall(content)

        if result\_tmp:

            r.write(str(result\_tmp) + '\n' )

    return 'True'

if \_\_name\_\_ == '\_\_main\_\_':

    if len(sys.argv) < 2:

        print "Usage: log\_SecAnalysis.py filepath"

        sys.exit(0)

    else:

        if Auto\_scp():

            start = time.clock()

            print '====> Log is analyzing, please wait for a moment <==== '

            File\_Search(sys.argv[1])

            end = time.clock()

            print '分析完毕,共运行时长:' + str(end - start)

            sys.exit(0)

        else:

            print '文件scp传输异常...'
```

效果如下：

![](https://bbs.kanxue.com/upload/attach/201607/508937_dh7axh616180zvp.jpg)

## 针对日志未分割存储的，大big日志文件

```
#!/usr/bin/python

# -\*- coding: utf-8 -\*-

'''

this script is to Analysis big logfile.

'''

import os

import re

import threading

import sys

from multiprocessing.dummy import Pool as ThreadPool

import time

# 规则列表

rulelist = ['\.\./','select.+(from|limit)','(?:(union(.\*?)select))','having|rongjitest','sleep\((\s\*)(\d\*)(\s\*)\)','benchmark\((.\*)\,(.\*)\)','base64\_decode\(','(?:from\W+information\_schema\W)','(?:(?:current\_)user|database|schema|connection\_id)\s\*\(','(?:etc\/\W\*passwd)','into(\s+)+(?:dump|out)file\s\*','group\s+by.+\(','xwork.MethodAccessor','(?:define|eval|file\_get\_contents|include|require|require\_once|shell\_exec|phpinfo|system|passthru|preg\_\w+|execute|echo|print|print\_r|var\_dump|(fp)open|alert|showmodaldialog)\(','xwork\.MethodAccessor','(gopher|doc|php|glob|file|phar|zlib|ftp|ldap|dict|ogg|data)\:\/','java\.lang','\$\_(GET|post|cookie|files|session|env|phplib|GLOBALS|SERVER)\[','\<(iframe|script|body|img|layer|div|meta|style|base|object|input)','(onmouseover|onerror|onload)\=','(HTTrack|harvest|audit|dirbuster|pangolin|nmap|sqln|-scan|hydra|Parser|libwww|BBBike|sqlmap|w3af|owasp|Nikto|fimap|havij|PycURL|zmeu|BabyKrokodil|netsparker|httperf|bench| SF/)','\.(svn|htaccess|bash\_history)','\.(bak|inc|old|mdb|sql|backup|java|class)$','(vhost|bbs|host|wwwroot|www|site|root|hytop|flashfxp).\*\.rar','(phpmyadmin|jmx-console|jmxinvokerservlet)','/(attachments|upimg|images|css|uploadfiles|html|uploads|templets|static|template|data|inc|forumdata|upload|includes|cache|avatar)/(\\w+).(php|jsp)']

def File\_Search(filename):

    content = open(filename).readlines()

    pool = ThreadPool(50)

    results = pool.map(Log\_Analysis, content)

    pool.close()

    pool.join()

def Log\_Analysis(content):

    for regex in rulelist:

        m = re.search(regex,content)

        if m:

            r.write('匹配特征字符:' + '[' + m.group(0) + ']' + '===>' + content + '\n')

    return 'True'

if \_\_name\_\_ == '\_\_main\_\_':

    if len(sys.argv) < 2:

        print "Usage: log\_SecAnalysis.py filename"

        sys.exit(0)

    else:

        start = time.clock()

        print '====> Log is analyzing, please wait for a moment <==== '

        r = open('result.txt', 'a')  # 需要本地先新建个result.txt文件

        r.write('\n' + '=================== web\_log\_secAnalysis ===================' + '\n' + sys.argv[1] + '\n')

        File\_Search(sys.argv[1])

        end = time.clock()

        print '分析完毕,共运行时长:' + str(end - start)

        sys.exit(0)
```

![](https://bbs.kanxue.com/upload/attach/201608/508937_g7vmh1eqmzo2ovm.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [logo.jpg](javascript:void(0);)

  （105.13kb，28次下载）

- [1.jpg](javascript:void(0);)

  （141.29kb，7次下载）
