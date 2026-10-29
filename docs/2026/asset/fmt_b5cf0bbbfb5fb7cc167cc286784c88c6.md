
```
#encoding: utf-8

import os

from bs4 import BeautifulSoup

import os.path

import argparse

import sys

import base64

# SQLMap自定义选项
_options = " --technique BEST --batch --threads 10 "

def usage():
    print" "
    print"  Usage: ./burp-to-sqlmap.py [options]"
    print"  Options: -f, --file               <BurpSuit State File>"
    print"  Options: -o, --outputdirectory    <Output Directory>"
    print"  Options: -s, --sqlmappath         <SQLMap Path>"
    print"  Options: -p, --proxy              <Use Proxy>"
    print"  Example: python burp-to-sqlmap.py -f [BURP-STATE-FILE] -o [OUTPUT-DIRECTORY] -s [SQLMap-Path] -p [Proxy]"
    print" "

parser = argparse.ArgumentParser()
parser.add_argument("-f", "--file",default="burp")
parser.add_argument("-o", "--outputdirectory",default="output")
parser.add_argument("-s", "--sqlmappath")
parser.add_argument("-p", "--proxy")
args = parser.parse_args()

if not args.file or (os.path.exists("burp") == False):
    usage()
    sys.exit(0)

if os.path.exists("output") == False:
    os.mkdir("output")

if args.proxy:
    proxyvalue = "--proxy " + args.proxy
else:
    proxyvalue = ""

vulnerablefiles = []
filename = args.file
directory = args.outputdirectory
sqlmappath = args.sqlmappath
if not os.path.exists(directory):
    os.makedirs(directory)

# 提取数据包
packetnumber = 0
print " [+] Exporting Packets ..."
with open(filename, 'r') as f:
    soup = BeautifulSoup(f.read(), "html.parser")
    for i in soup.find_all("request"):
        packetnumber = packetnumber + 1
        print "   [-] Packet " + str(packetnumber) + " Exported."
        outfile = open(os.path.join(args.outputdirectory, str(packetnumber) + ".txt"), "w")
        outfile.write(base64.b64decode(i.text.strip()))
    print " "
    print str(packetnumber) + " Packets Exported Successfully."
    print " "

# SQLMap测试
print " [+] Testing SQL Injection on packets ...  (Based on your network connection Test can take up to 5 minutes.)"
for file in os.listdir(directory):
    print "   [-] Performing SQL Injection on packet number " + file[:-4] + ". Please Wait ..."
    _command = "sqlmap -r " + directory + "\\" + file + _options + proxyvalue + " > " + directory + "\\testresult" + file
    print _command
    os.system(_command)
    if 'is vulnerable' in open(directory + "\\testresult" + file).read() or "Payload:" in open(
            directory + "\\testresult" + file).read():
        print "    - URL is Vulnerable."
        vulnerablefiles.append(file)
    else:
        print "    - URL is not Vulnerable."
    print "    - Output saved in " + directory + "\\testresult" + file
print " "
print "--------------"
print "Test Done."
print "Result:"
if not vulnerablefiles:   
    print "No vulnerabilities found on your target."
else:
    for items in vulnerablefiles:
        print "Packet " + items[:-4] + " is vulnerable to SQL Injection. for more information please see " + items
print "--------------"
print " "
```

### 测试效果

![](https://bbs.kanxue.com/upload/attach/201811/214974_DQ9DG23SHE8PY9F.png)

![](https://bbs.kanxue.com/upload/attach/201811/214974_4M5J7PWR6C3SS9S.png)

## 参考

472K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3y4K6i4K6u0r3k6h3&6Y4L8r3W2K6K9q4)9J5c8U0b7#2y4o6t1^5i4K6u0V1j5Y4g2D9K9#2)9J5k6s2y4I4L8q4)9J5k6r3W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6s2g2K6K9h3&6Y4i4K6u0V1j5Y4g2J5M7q4)9J5k6s2c8G2i4K6u0V1M7%4q4D9L8h3q4H3i4K6u0W2M7r3c8X3

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
