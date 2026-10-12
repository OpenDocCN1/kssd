
:0002.1682 `50`          `push ax`

:0002.1683 `8D46D0`     `lea ax, [bp-30]`

:0002.1686 `8CD2`       `mov dx, ss`

:0002.1688 `52`         `push dx`

:0002.1689 `50`         `push ax`

:0002.168A `9AF08C5416` `call 0004.8CF0`            ;字符串传送：`ss:bp-3E`->`ss:bp-30`

:0002.168F `83C40A`     `add sp, 000A`

:0002.1692 `C646D100`   `mov byte ptr [bp-2F], 00`

:0002.1696 `B80100`     `mov ax, 0001`

:0002.1699 `50`         `push ax`

:0002.169A `B80200`     `mov ax, 0002`

:0002.169D `BA3316`     `mov dx, SEG ADDR of Segment 0005`

:0002.16A0 `52`         `push dx`

:0002.16A1 `50`         `push ax`

:0002.16A2 `8D46F8`     `lea ax, [bp-08]`

:0002.16A5 `8CD2`       `mov dx, ss`

:0002.16A7 `52`         `push dx`

:0002.16A8 `50`         `push ax`

:0002.16A9 `9AF08C8D16` `call 0004.8CF0`            ;字符串传送：`0005.0002`->`ss:bp-08`

:0002.16AE `83C40A`     `add sp, 000A`

:0002.16B1 `C646F900`   `mov byte ptr [bp-07], 00`

:0002.16B5 `B80100`     `mov ax, 0001`

:0002.16B8 `50`         `push ax`

:0002.16B9 `B80400`     `mov ax, 0004`

:0002.16BC `BA9E16`     `mov dx, SEG ADDR of Segment 0005`

:0002.16BF `52`         `push dx`

:0002.16C0 `50`         `push ax`

:0002.16C1 `8D8614FE`   `lea ax, [bp+FE14]`

:0002.16C5 `8CD2`       `mov dx, ss`

:0002.16C7 `52`         `push dx`

:0002.16C8 `50`         `push ax`

:0002.16C9 `9AF08CAC16` `call 0004.8CF0`            ;字符串传送：`0005.0004`->`ss:bp-1EC`

:0002.16CE `83C40A`     `add sp, 000A`

:0002.16D1 `C68615FE00` `mov byte ptr [bp-01EB], 00`

:0002.16D6 `8D46D0`     `lea ax, [bp-30]`

:0002.16D9 `8CD2`       `mov dx, ss`

:0002.16DB `52`         `push dx`

:0002.16DC `50`         `push ax`

:0002.16DD `9A568DCC16` `call 0004.8D56`            ;进制转换

:0002.16E2 `83C404`     `add sp, 0004`

:0002.16E5 `8986C4FD`   `mov [bp+FDC4], ax`

:0002.16E9 `8D46F8`     `lea ax, [bp-08]`

:0002.16EC `8CD2`       `mov dx, ss`

:0002.16EE `52`         `push dx`

:0002.16EF `50`         `push ax`

:0002.16F0 `9A568DE016` `call 0004.8D56`            ;进制转换

:0002.16F5 `83C404`     `add sp, 0004`

:0002.16F8 `8986C2FD`   `mov [bp+FDC2], ax`

:0002.16FC `8D8614FE`   `lea ax, [bp+FE14]`

:0002.1700 `8CD2`       `mov dx, ss`

:0002.1702 `52`         `push dx`

:0002.1703 `50`         `push ax`

:0002.1704 `9A568DF316` `call 0004.8D56`            ;进制转换

:0002.1709 `83C404`     `add sp, 0004`

:0002.170C `8BC8`       `mov cx, ax`                 ;数字码第四位

:0002.170E `8B86C2FD`   `mov ax, [bp+FDC2]`         ;数字码第二位

:0002.1712 `F7E9`       `imul cx`

:0002.1714 `B90900`     `mov cx, 0009`

:0002.1717 `99`         `cwd`

:0002.1718 `F7F9`       `idiv cx`

:0002.171A `8B86C4FD`   `mov ax, [bp+FDC4]`         ;序列号第七位

:0002.171E `3BC2`       `cmp ax, dx`

:0002.1720 `7403`       `je 1725`

:0002.1722 `E92804`     `jmp 1B4D`

序列号第七位等于数字码第二位和第四位的乘积除以九的余数

:0002.1725 `B80100`     `mov ax, 0001`

:0002.1728 `50`         `push ax`

:0002.1729 `8D46C3`     `lea ax, [bp-3D]`

:0002.172C `8CD2`       `mov dx, ss`

:0002.172E `52`         `push dx`

:0002.172F `50`         `push ax`

:0002.1730 `8D46D0`     `lea ax, [bp-30]`

:0002.1733 `8CD2`       `mov dx, ss`

:0002.1735 `52`         `push dx`

:0002.1736 `50`         `push ax`

:0002.1737 `9AF08C0717` `call 0004.8CF0`            ;字符串传送：`ss:bp-3D`->`ss:bp-30`

:0002.173C `83C40A`     `add sp, 000A`

:0002.173F `C646D100`   `mov byte ptr [bp-2F], 00`

:0002.1743 `B80100`     `mov ax, 0001`

:0002.1746 `50`         `push ax`

:0002.1747 `B80400`     `mov ax, 0004`

:0002.174A `BABD16`     `mov dx, SEG ADDR of Segment 0005`

:0002.174D `52`         `push dx`

:0002.174E `50`         `push ax`

:0002.174F `8D46F8`     `lea ax, [bp-08]`

:0002.1752 `8CD2`       `mov dx, ss`

:0002.1754 `52`         `push dx`

:0002.1755 `50`         `push ax`

:0002.1756 `9AF08C3A17` `call 0004.8CF0`            ;字符串传送：`0005.0004`->`ss:bp-08`

:0002.175B `83C40A`     `add sp, 000A`

:0002.175E `C646F900`   `mov byte ptr [bp-07], 00`

:0002.1762 `B80100`     `mov ax, 0001`

:0002.1765 `50`         `push ax`

:0002.1766 `B80500`     `mov ax, 0005`

:0002.1769 `BA4B17`     `mov dx, SEG ADDR of Segment 0005`

:0002.176C `52`         `push dx`

:0002.176D `50`         `push ax`

:0002.176E `8D8614FE`   `lea ax, [bp+FE14]`

:0002.1772 `8CD2`       `mov dx, ss`

:0002.1774 `52`         `push dx`

:0002.1775 `50`         `push ax`

:0002.1776 `9AF08C5917` `call 0004.8CF0`            ;字符串传送：`0005.0005`->`ss:bp-1EC`

:0002.177B `83C40A`     `add sp, 000A`

:0002.177E `C68615FE00` `mov byte ptr [bp-01EB], 00`

:0002.1783 `8D46D0`     `lea ax, [bp-30]`

:0002.1786 `8CD2`       `mov dx, ss`

:0002.1788 `52`         `push dx`

:0002.1789 `50`         `push ax`

:0002.178A `9A568D7917` `call 0004.8D56`            ;进制转换

:0002.178F `83C404`     `add sp, 0004`

:0002.1792 `8986C4FD`   `mov [bp+FDC4], ax`

:0002.1796 `8D46F8`     `lea ax, [bp-08]`

:0002.1799 `8CD2`       `mov dx, ss`

:0002.179B `52`         `push dx`

:0002.179C `50`         `push ax`

:0002.179D `9A568D8D17` `call 0004.8D56`            ;进制转换

:0002.17A2 `83C404`     `add sp, 0004`

:0002.17A5 `8986C2FD`   `mov [bp+FDC2], ax`

:0002.17A9 `8D8614FE`   `lea ax, [bp+FE14]`

:0002.17AD `8CD2`       `mov dx, ss`

:0002.17AF `52`         `push dx`

:0002.17B0 `50`         `push ax`

:0002.17B1 `9A568DA017` `call 0004.8D56`            ;进制转换

:0002.17B6 `83C404`     `add sp, 0004`

:0002.17B9 `8BC8`       `mov cx, ax`                 ;数字码第五位

:0002.17BB `8B86C2FD`   `mov ax, [bp+FDC2]`         ;数字码第四位

:0002.17BF `F7E9`       `imul cx`

:0002.17C1 `B90800`     `mov cx, 0008`

:0002.17C4 `99`         `cwd`

:0002.17C5 `F7F9`       `idiv cx`

:0002.17C7 `8B86C4FD`   `mov ax, [bp+FDC4]`         ;序列号第八位

:0002.17CB `3BC2`       `cmp ax, dx`

:0002.17CD `7403`       `je 17D2`

:0002.17CF `E97B03`     `jmp 1B4D`

序列号第八位等于数字码第四位和第五位的乘积除以八的余数

:0002.17D2 `B80100`     `mov ax, 0001`

:0002.17D5 `50`         `push ax`

:0002.17D6 `8D46C4`     `lea ax, [bp-3C]`

:0002.17D9 `8CD2`       `mov dx, ss`

:0002.17DB `52`         `push dx`

:0002.17DC `50`         `push ax`

:0002.17DD `8D46D0`     `lea ax, [bp-30]`

:0002.17E0 `8CD2`       `mov dx, ss`

:0002.17E2 `52`         `push dx`

:0002.17E3 `50`         `push ax`

:0002.17E4 `9AF08CB417` `call 0004.8CF0`            ;字符串传送：`ss:bp-3C`->`ss:bp-30`

:0002.17E9 `83C40A`     `add sp, 000A`

:0002.17EC `C646D100`   `mov byte ptr [bp-2F], 00`

:0002.17F0 `B80100`     `mov ax, 0001`

:0002.17F3 `50`         `push ax`

:0002.17F4 `B80200`     `mov ax, 0002`

:0002.17F7 `BA6A17`     `mov dx, SEG ADDR of Segment 0005`

:0002.17FA `52`         `push dx`

:0002.17FB `50`         `push ax`

:0002.17FC `8D46F8`     `lea ax, [bp-08]`

:0002.17FF `8CD2`       `mov dx, ss`

:0002.1801 `52`         `push dx`

:0002.1802 `50`         `push ax`

:0002.1803 `9AF08CE717` `call 0004.8CF0`            ;字符串传送：`0005.0002`->`ss:bp-08`

:0002.1808 `83C40A`     `add sp, 000A`

:0002.180B `C646F900`   `mov byte ptr [bp-07], 00`

:0002.180F `8D46D0`     `lea ax, [bp-30]`

:0002.1812 `8CD2`       `mov dx, ss`

:0002.1814 `52`         `push dx`

:0002.1815 `50`         `push ax`

:0002.1816 `9A568D0618` `call 0004.8D56`            ;进制转换

:0002.181B `83C404`     `add sp, 0004`

:0002.181E `8986C4FD`   `mov [bp+FDC4], ax`

:0002.1822 `8D46F8`     `lea ax, [bp-08]`

:0002.1825 `8CD2`       `mov dx, ss`

:0002.1827 `52`         `push dx`

:0002.1828 `50`         `push ax`

:0002.1829 `9A568D1918` `call 0004.8D56`            ;进制转换

:0002.182E `83C404`     `add sp, 0004`

:0002.1831 `05FDFF`     `add ax, FFFF`

:0002.1834 `50`         `push ax`

:0002.1835 `9A78992C18` `call 0004.9978`            ;取绝对值

:0002.183A `83C402`     `add sp, 0002`

:0002.183D `8B8EC4FD`   `mov cx, [bp+FDC4]`

:0002.1841 `3BC8`       `cmp cx, ax`

:0002.1843 `7403`       `je 1848`

:0002.1845 `E90503`     `jmp 1B4D`

序列号第九位等于数字码第二位与三的差的绝对值

:0002.1848 `B80100`     `mov ax, 0001`

:0002.184B `50`         `push ax`
