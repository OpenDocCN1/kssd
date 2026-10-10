
# 【看雪读书月】软件漏洞挖掘 Fuzz 工具之五-autodafe

> 原文：[`www.pediy.com/kssd/pediy10/70319.html`](https://www.pediy.com/kssd/pediy10/70319.html)

庆祝奥运，决定今天把这个整理完。。。。断断续续写了好多天了。。。。

## 安装

1. `make sure you have libxml2`
2. `make sure you have gdb`
3. `make sure you have gcc >= 3.3.4`
4. `./configure`
5. `make`
6. `make install`

安装时候可以看到 产生 fuzz 变异数据的过程-\_-

```
Fuzz:/usr/local/condor/Fuzz/autodafe-0.1 # make install
cd ./etc/generator; ./generator.sh /usr/local/etc[*] creating directory: ./autodafe[*] creating file: ./autodafe/string.fuzz[*] creating file: ./autodafe/hex.fuzz[*] processing: "a" 0%[............................ transição]100%[*] processing: "d" 0%[............................ transição]100%[*] processing: "A" 0%[............................ transição]100%[*] processing: "D" 0%[............................ transição]100%[*] processing: "%s" 0%[............................ transição]100%[*] processing: "%n" 0%[............................ transição]100%[*] processing: ">" 0%[............................ transição]100%[*] processing: "<" 0%[............................ transição]100%[*] processing: "(" 0%[............................ transição]100%[*] processing: ")" 0%[............................ transição]100%[*] processing: "/" 0%[............................ transição]100%[*] processing: "&" 0%[............................ transição]100%[*] processing: ";" 0%[............................ transição]100%[*] processing: "|" 0%[............................ transição]100%[*] processing: "\\" 0%[............................ transição]100%
```

```
make[1]: Entering directory `/data/Fuzz/autodafe-0.1/src'
make[2]: Entering directory `/data/Fuzz/autodafe-0.1/src/adbg'
cp ./adbg /usr/local/bin
make[2]: Leaving directory `/data/Fuzz/autodafe-0.1/src/adbg'
make[2]: Entering directory `/data/Fuzz/autodafe-0.1/src/adc'
cp ./adc /usr/local/bin
make[2]: Leaving directory `/data/Fuzz/autodafe-0.1/src/adc'
make[2]: Entering directory `/data/Fuzz/autodafe-0.1/src/autodafe'
cp ./autodafe /usr/local/bin
cp -r ../../etc/generator/autodafe /usr/local/etc
make[2]: Leaving directory `/data/Fuzz/autodafe-0.1/src/autodafe'
make[2]: Entering directory `/data/Fuzz/autodafe-0.1/src/pdml2ad'
cp ./pdml2ad /usr/local/bin
make[2]: Leaving directory `/data/Fuzz/autodafe-0.1/src/pdml2ad'
make[1]: Leaving directory `/data/Fuzz/autodafe-0.1/src'
```

简单介绍：特点是支持 sniffer 包 自动生成测试数据，基于 block，自动计算 block 的大小，带调试器，能自动监控 危险函数和测试用例间的情况。包括四大组件：`adbg` `adc` `autodafe` `pdml2ad`。下面开始逐个介绍：

### 第一步，Pdml2ad :转化 PDML 文件为 Autodafe 支持的脚本语言 (*.ad) 文件。

Tips: `PDML` 可以 wireshark 抓包获得到: `file->export` 选择保持的文件类型 `pdml` 。

首先，使用 wiresshark 抓包，过滤 msn 的消息，导出 pdml 格式的 msnms.pdml

![](img/9ac1927ddc2ebaace319cf1be8941973.jpg)

```
root@SEC39_89_sles10:/usr/local/condor/fuzz/autodafe-0.1/src/pdml2ad# ./pdml2ad -d -v -p msnms.pdml msnms.ad
```

注意这里的要加个 `-p` 参数，可以解析协议格式。现在就来看看转化好的这个 msn 包的格式 autodafe 脚本的描述：

```
root@SEC39_89_sles10:/usr/local/condor/fuzz/autodafe-0.1/src/pdml2ad# cat msnms.ad
```

```
/*--------------------------------------------------------------------------*
 * xml autodafe's parser v.0.1 (c) Martin Vuagnoux - 2004-2006            *
 * auto-generated script using PDML (Packet Details Markup Language) source *
 *--------------------------------------------------------------------------*/
block_begin("packet_1");
   block_begin("packet_1.6.54.msnms");
      // name    :
      // showname: (null)
      // show    : UUN 42 smant1286@hotmail.com 3 573\r\n
      // size: 0x36 (54)
      hex(
      55 55 4e 20 34 32 20 73    6d 61 6e 74 31 32 38 36
      40 68 6f 74 6d 61 69 6c    2e 63 6f 6d 20 33 20 35
      37 33 0d 0a
      );

      // name    :
      // showname: (null)
      // show    : INVITE MSNMSGR:smant1286@hotmail.com MSNSLP/1.0\r\n
      // size: 0x49 (73)
      hex(
      49 4e 56 49 54 45 20 4d    53 4e 4d 53 47 52 3a 73
      6d 61 6e 74 31 32 38 36    40 68 6f 74 6d 61 69 6c
      2e 63 6f 6d 20 4d 53 4e    53 4c 50 2f 31 2e 30 0d
      0a
      );

      // name    :
      // showname: (null)
      // show    : To: <msnmsgr:smant1286@hotmail.com>\r\n
      // size: 0x37 (55)
      hex(
      54 6f 3a 20 3c 6d 73 6e    6d 73 67 72 3a 73 6d 61
      6e 74 31 32 38 36 40 68    6f 74 6d 61 69 6c 2e 63
      6f 6d 3e 0d 0a
      );

      // name    :
      // showname: (null)
      // show    : From: <msnmsgr:linshifei@live.cn>\r\n
      // size: 0x35 (53)
      hex(
      46 72 6f 6d 3a 20 3c 6d    73 6e 6d 73 67 72 3a 6c
      69 6e 73 68 69 66 65 69    40 6c 69 76 65 2e 63 6e
      3e 0d 0a
      );

      // name    :
      // showname: (null)
      // show    : Via: MSNSLP/1.0/TLP ;branch={F56C917C-AA07-4D3E-A2E9-FE701B079FD0}\r\n
      // size: 0x68 (104)
      hex(
      56 69 61 3a 20 4d 53 4e    53 4c 50 2f 31 2e 30 2f
      54 4c 50 20 3b 62 72 61    6e 63 68 3d 7b 46 35 36
      43 39 31 37 43 2d 41 41    30 37 2d 34 44 33 45 2d
      41 32 45 39 2d 46 45 37    30 31 42 30 37 39 46 44
      30 7d 0d 0a
      );

      // name    :
      // showname: (null)
      // show    : CSeq: 0  \r\n
      // size: 0x10 (16)
      hex(
      43 53 65 71 3a 20 30 20    0d 0a
      );

      // name    :
      // showname: (null)
      // show    : Call-ID: {F13B5C79-0126-458F-A29D-747C79C56530}\r\n
      // size: 0x49 (73)
      hex(
      43 61 6c 6c 2d 49 44 3a    20 7b 46 31 33 42 35 43
      37 39 2d 30 31 32 36 2d    34 35 38 46 2d 41 32 39
      44 2d 37 34 37 43 37 39    43 35 36 35 33 30 7d 0d
      0a
      );

      // name    :
      // showname: (null)
      // show    : Max-Forwards: 0\r\n
      // size: 0x17 (23)
      hex(
      4d 61 78 2d 46 6f 72 77    61 72 64 73 3a 20 30 0d
      0a
      );

      // name    :
      // showname: (null)
      // show    : Content-Type: application/x-msnmsgr-transreqbody\r\n
      // size: 0x50 (80)
      hex(
      43 6f 6e 74 65 6e 74 2d    54 79 70 65 3a 20 61 70
      70 6c 69 63 61 74 69 6f    6e 2f 78 2d 6d 73 6e 6d
      73 67 72 2d 74 72 61 6e    73 72 65 71 62 6f 64 79
      0d 0a
      );

      // name    :
      // showname: (null)
      // show    : Content-Length: 236\r\n
      // size: 0x21 (33)
      hex(
      43 6f 6e 74 65 6e 74 2d    4c 65 6e 67 74 68 3a 20
      32 33 36 0d 0a
      );

      // name    :
      // showname: (null)
      // show    : \r\n
      // size: 0x2 (2)
      hex(
      0d 0a
      );

      // name    :
      // showname: (null)
      // show    : Bridges: TRUDPv1 TCPv1 SBBridge TURNv1\r\n
      // size: 0x40 (64)
      hex(
      42 72 69 64 67 65 73 3a    20 54 52 55 44 50 76 31
      20 54 43 50 76 31 20 53    42 42 72 69 64 67 65 20
      54 55 52 4e 76 31 0d 0a
      );

      // name    :
      // showname: (null)
      // show    : NetID: 0\r\n
      // size: 0x10 (16)
      hex(
      4e 65 74 49 44 3a 20 30    0d 0a
      );

      // name    :
      // showname: (null)
      // show    : Conn-Type: Firewall\r\n
      // size: 0x21 (33)
      hex(
      43 6f 6e 6e 2d 54 79 70    65 3a 20 46 69 72 65 77
      61 6c 6c 0d 0a
      );

      // name    :
      // showname: (null)
      // show    : TCP-Conn-Type: Firewall\r\n
      // size: 0x25 (37)
      hex(
      54 43 50 2d 43 6f 6e 6e    2d 54 79 70 65 3a 20 46
      69 72 65 77 61 6c 6c 0d    0a
      );
```
