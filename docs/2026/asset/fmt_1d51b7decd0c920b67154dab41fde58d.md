
# [原创]小工具_通过串口上传下载文件

## 说明

有的设备只有一个串口可以看，这时候想要下个文件下来还有点费事，这里写了个小工具。有些细节可能考虑的不到位，但改改应该大体能用。

## 环境准备

```
1. 安装pyserial库，我用的3.5版本
pip install pyserial==3.5
2.可以先使用 MobaXterm、Xshell这类工具连上串口，登录一下，因为代码里没有登录功能；串口的登录状态是可以保持的
3.有的系统登录上去之后，时不时会有个dmesg消息弹出来，可以调整下日志输出级别，不要那么频繁输出
cat /proc/sys/kernel/printk
6 4 1 7
echo "1 4 1 7" >/proc/sys/kernel/printk
```

## 代码

`main.py`

```python
import serial
import re
import base64
'''
1.下载文件：使用base64、hexdump对文件编码，通过串口终端打印出来，再解码落地成文件。
2.上传文件：先对文件进行base64编码，上传后再解码
3.之所以编码后再传输，是为了减少不必要的干扰，至少可以避开一些特殊字符；随着环境的变化，有些细节可能考虑的不到位，但改改应该大体能用。
pip install pyserial==3.5
'''
class SerialTool(object):
    def __init__(self, portx="COM5", bps=115200, timeout=1):
        #端口，Linux上的 /dev/ttyUSB0 等 或 Windows上的 COM3 等
        #波特率，标准值之一：50,75,110,134,150,200,300,600,1200,1800,2400,4800,9600,19200,38400,57600,115200; 一般用9600 或者 115200
        try:
            self.p = serial.Serial(portx, bps, timeout=timeout)
            self.rn=self.getRN()
        except Exception as e:
            print(e)
            self.rn=b"\n"
    def getRN(self):
        rn=b"\n"
        self.p.write(rn)
        data=self.p.readall()
        prompt=data.split(rn)[1] # b"\r\n/ #"
        if b"\n"+prompt==data:
            rn=b"\n"
        elif b"\r\n"+prompt==data:
            rn=b"\r\n"
        return rn
    def exec(self, cmd=b"pwd"):
        if isinstance(cmd, str):
            cmd=cmd.encode()
        self.p.write(cmd+b"\n")
        data = self.p.readall()
        print(data.decode())
    def downloadFileByHexdump(self, rfile="/etc/passwd", outfile=""):
        if not outfile:
            outfile=rfile.split("/")[-1]
        file_size=self.getFileSize(rfile)
        self.p.write("hexdump {rfile}\n".format(rfile=rfile).encode())
        data = self.p.readall()
        self.writeFile(outfile+".hex", data)
        content=self.rn.join(data.split(self.rn)[1:-1])
        file_data = self.decodeHexdump(content)
        if isinstance(file_size, int):
            file_data=file_data[0:file_size]
        self.writeFile(outfile, file_data)
    def decodeHexdump(self, content, byteorder="little"):
        lst=content.split(b"\n")
        flag_star=False
        pre_addr=0
        cur_addr=0
        data=b""
        for line in lst:
            if b"*" not in line:
                r=line.strip().split()
                cur_addr=int(r[0], 16)
                if flag_star:
                    data+=int(0).to_bytes(1, byteorder) * (cur_addr-pre_addr-16)
                flag_star=False
                for h in r[1:]:
                    n=int(len(h)/2) # len("0000")/2=2
                    data+=int(h, 16).to_bytes(n, byteorder) # byteorder: little, big 有的hexdump按照小端显示，但有的会按照大端显示
                pre_addr=cur_addr
            else:
                flag_star=True
        return data
    def downloadFileByBase64(self, rfile="/etc/passwd", outfile=""):
        if not outfile:
            outfile=rfile.split("/")[-1]
        self.p.write("cat {rfile} |base64\n".format(rfile=rfile).encode())
        data = self.p.readall()
        self.writeFile(outfile+".base64", data)
        content=self.rn.join(data.split(self.rn)[1:-1])
        file_data = self.decodeBase64(content)
        self.writeFile(outfile, file_data)
    def decodeBase64(self, content):
        content=content.replace(b"\r", b"").replace(b"\n", b"")
        file_data=base64.b64decode(content)
        return file_data
    def uploadFileByBase64(self, lfile="a.txt", rfile=None):
        if not rfile:
            print("rfile=None")
            exit(1)
        data=self.readFile(lfile)
        print()
        self.p.write("cat <<EOF > {rfile}.tmpfile\n".format(rfile=rfile).encode()+base64.b64encode(data)+b"\nEOF\n")
        ignor_data = self.p.readall()
        #print(ignor_data)
        self.p.write("cat {rfile}.tmpfile | base64 -d > {rfile}\n".format(rfile=rfile).encode())
        ignor_data = self.p.readall()
        #print(ignor_data)
        self.p.write("rm -f {rfile}.tmpfile\n".format(rfile=rfile).encode())
        ignor_data = self.p.readall()
        #print(ignor_data)
    def close(self):
        self.p.close()
    def getFileSize(self, rfile):
        try:
            self.p.write("ls -l {rfile}\n".format(rfile=rfile).encode())
            data = self.p.readall()
            #print(data)
            filter=re.compile(b"[rwx-]+[\s]+\d+[\s]+[\S]+[\s]+[\S]+[\s]+(\d+)[\s]+") #-rwxr-xr-x 1 user root 63
            res=filter.findall(data)
            if res:
                size=int(res[0])
                return size
        except Exception as e:
            print(e)
    def writeFile(self, file, data):
        fout=open(file,"wb")
        fout.write(data)
        fout.close()
    def readFile(self, file):
        fout=open(file,"rb")
        data=fout.read()
        fout.close()
        return data
    def test(self):
        '''
        data=self.readFile(file="run.sh.hex")
        import pdb; pdb.set_trace()
        outfile="test.tmp"
        content=self.rn.join(data.split(self.rn)[1:-1])
        self.writeFile(outfile, self.decodeHexdump(content))
        '''
        data=self.readFile(file="cert.pem.base64")
        #import pdb; pdb.set_trace()
        outfile="test.tmp"
        content=self.rn.join(data.split(self.rn)[1:-1])
        self.writeFile(outfile, self.decodeBase64(content))

def main():
    m=SerialTool(portx="COM6", bps=115200, timeout=1)
    #m.downloadFileByBase64(rfile="run.sh")
    #m.downloadFileByHexdump(rfile="run.sh")
    m.uploadFileByBase64("a.txt", "/tmp/a.txt")
    #m.test()

if __name__ == "__main__":
    main()
```

## 说明

有的设备只有一个串口可以看，这时候想要下个文件下来还有点费事，这里写了个小工具。有些细节可能考虑的不到位，但改改应该大体能用。

## 环境准备

1. 安装 `pyserial` 库，我用的 `3.5` 版本：`pip install pyserial==3.5`
2. 可以先使用 MobaXterm、Xshell 这类工具连上串口，登录一下，因为代码里没有登录功能；串口的登录状态是可以保持的
3. 有的系统登录上去之后，时不时会有个 `dmesg` 消息弹出来，可以调整下日志输出级别，不要那么频繁输出：
   `cat /proc/sys/kernel/printk`
   `6 4 1 7`
   `echo "1 4 1 7" >/proc/sys/kernel/printk`

## 代码

`main.py`

