
# Python 获取 QQ 空间前 100 篇 blog 的地址和标题的方法

> 原文：[`www.pediy.com/kssd/pediy11/126566.html`](https://www.pediy.com/kssd/pediy11/126566.html)

相关地址均由 `firebug` 获得，本来想把 QQ 空间的日志全部下载下来，可是目前只能获取前 100 篇的文章，先发出个半成品

代码:

```python
#! /usr/bin/env python
#coding=utf-8
import urllib2,urllib, os
from HTMLParser import HTMLParser

QQnum='1026239701'
loginurl='http://g.cnc.qzone.qq.com/fcg-bin/cgi_emotion_list.fcg?uin='+QQnum+'&loginUin=0&s=414636&num=3&g_tk=5381'
blogurl='http://br.cnc.qzone.qq.com/cgi-bin/blognew/blog_output_toppage?uin='+QQnum+'&vuin=0&property=GoRE&getall=1&styledm=cnc.qzonestyle.gtimg.cn&imgdm=cnc.qzs.qq.com&bdm=b.cnc.qzone.qq.com&cate=&numperpage=100&maxlen=68&sorttype=0&pos=0&direct=1'
blogpreurl='http://user.qzone.qq.com/'+QQnum+'/blog/'

#返回页面
def visitUrl(url):
    html=''
    fd=urllib2.urlopen(url)
    html=fd.read()
    if html !='':
        fd.close()
        return html
#写入文件
def write2file(data1, data2):
    fp=open('blogtitlelist.txt', 'a')
    for eachline in data1:
        fp.write(eachline)
        fp.write('\n')
    fp.close()
        
#下载网页
def downloadurl(url, filename):
    downloadFolder = './QQspacedown' #指定保存网页的文件夹
    if not os.path.isdir( downloadFolder ):
        os.mkdir( downloadFolder )
    downfilename=downloadFolder+'/'+filename+'.html'
    op=open(downfilename, 'wb')
     
    fd=urllib2.urlopen(url)
    html=fd.read()
    if html !='':
        op.write(html)
        fd.close()
        op.close()
        return True
    fd.close()
    op.close()
    return False
#处理 html
class Parser(HTMLParser):
    def __init__(self):
        self.targets={}
        self.is_span=''
        HTMLParser.__init__(self)
    def handle_starttag(self, tag, attrs):
        if tag=='span':
            for name, value in attrs:
                if name=='id':
                    if value.startswith('blogtitle_'):
                        self.is_span=1
                        index=value.find('_')
                        self.id=value[index+1:]
    def handle_endtag(self, tag):
        if tag=='span':
            self.is_span=''
    def handle_data(self, data):
        if self.is_span:
            self.targets[self.id]=data.strip()
    def getTargets(self):
        return self.targets
if __name__=='__main__':
    map={}
    titlelist=[]
    urllist=[]
    data=visitUrl(blogurl)
    myparser=Parser()
    myparser.feed(data)
    map=myparser.getTargets()
    for key, value in map.items():
        titlelist.append(value)
        urllist.append(blogpreurl+key)
    print urllist
    write2file(titlelist)
    print 'finished'
```
