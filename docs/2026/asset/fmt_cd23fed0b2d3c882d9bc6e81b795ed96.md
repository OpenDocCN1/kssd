
| 1 | `http://` 24aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8J5j5h3y4@1K9h3y4S2L8r3#2S2L8s2N6S2M7X3g2S2L8X3q4D9P5i4y4A6M7#2)9J5k6h3y4G2L8b7`.` ```/shellcode/annoy_user.exe` |
5. 这段shellcode在文件系统上留下了什么迹象?

|  |  |
| --- | --- |
| 1 | `C:\Windows\system32\1.exe` |
6. 这段shellcode做了什么?

![image-20210329101521420](https://bbs.kanxue.com/upload/attach/202103/867232_CGFPSC3PWR85KGZ.jpg)

|  |  |
| --- | --- |
| 1 | `从C2下载恶意文件并执行` |

## Lab19-02

1. 这段shellcode被注入到什么进程中?

![image-20210329105220146](https://bbs.kanxue.com/upload/attach/202103/867232_TNNBES5NHF8YPA2.jpg)

|  |  |
| --- | --- |
| 1 | `通过注册表查询到的默认浏览器` |
2. 这段shellcode位于哪里?

![image-20210329105314683](https://bbs.kanxue.com/upload/attach/202103/867232_662AVJKV6KYBA84.jpg)

|  |  |
| --- | --- |
| 1 | `位于VA为`407030`处` |
3. 这段shellcode是如何被编码的?

![image-20210329105452242](https://bbs.kanxue.com/upload/attach/202103/867232_AVFU4CZJV29JH2M.jpg)

|  |  |
| --- | --- |
| 1 | `亦或`0xE7` |
4. 这段shellcode手动导入了哪个函数?

![image-20210329110248864](https://bbs.kanxue.com/upload/attach/202103/867232_KDCUSFT58ZJB34Q.jpg)

|  |  |
| --- | --- |
| 1 |  |
5. 这段shellcode和什么网络主机进行通信?

![image-20210329112027179](https://bbs.kanxue.com/upload/attach/202103/867232_PVH7QC62NV32C34.jpg)

|  |  |
| --- | --- |
| 1 | `IP为`192.168`.`200.2` `端口为`13330` |
6. 这段shellcode做了什么?

![image-20210329112425557](https://bbs.kanxue.com/upload/attach/202103/867232_37SPEJ9SWUBZDDD.jpg)

|  |  |
| --- | --- |
| 1 | `启动cmd执行远程命令` |

## Lab19-03

1. 这个PDF中使用了什么漏洞?

![image-20210329112723865](https://bbs.kanxue.com/upload/attach/202103/867232_N9WRJAQ42WFDWHK.jpg)

|  |  |
| --- | --- |
| 1 | `以文本方式打开pdf文件，搜索script，可以定位到js代码。漏洞为CVE`-`2010`-`0188` |
2. 这段shellcode是如何编码的？

|  |  |
| --- | --- |
| 1 | `未进行编码，硬编码显示` |
3. 这段shellcode手动导入了哪个函数?

![image-20210329113543028](https://bbs.kanxue.com/upload/attach/202103/867232_ZRF3ZZE4Y5J3PQF.jpg)

|  |  |
| --- | --- |
| 1 |  |
4. 这段shellcode在文件系统上留下了什么迹象?

![image-20210329114446765](https://bbs.kanxue.com/upload/attach/202103/867232_DK83JBH3HAEPJUU.jpg)

![image-20210329114556794](https://bbs.kanxue.com/upload/attach/202103/867232_G6YYQUU86H6TNR7.jpg)

|  |  |
| --- | --- |
| 1 |  |
5. 这段shellcode做了什么?

|  |  |
| --- | --- |
| 1 | `将exe与pdf释放至临时目录，运行exe后打开pdf伪装正常pdf文档。` |

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
