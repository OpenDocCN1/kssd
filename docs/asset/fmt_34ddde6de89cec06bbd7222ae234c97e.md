
8、 `A` 收到 `S` 回复的 `B` 的公网 `IP` 和端口号等信息以后，开始连接到 `B` 公网 `IP` 和端口号。由于在步骤 6 中 `B` 曾经尝试连接过 `A` 的公网 `IP` 地址和端口，`NAT-B` 纪录了此次连接的信息，所以当 `A` 主动连接 `B` 时，`NAT-B` 会认为是合法的 `SYN` 数据，并允许通过，从而直接的 `TCP` 连接建立起来了。

参考网址：

`http://midcom-p2p.sourceforge.net/draft-ford-midcom-p2p-01.txt`

`http://www.vckbase.com/document/viewdoc/?id=1773`
