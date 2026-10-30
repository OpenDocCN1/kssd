
然后退出该容器，再根据该镜像重新启动一个容器，可以发现在重新启动的容器中的`passwd`文件也已经被更改，证明该镜像已经被持续的更改。但该更改并不是永久性的，短时间内的新建容器会出现这种情况。

![1655176451937.png](https://bbs.kanxue.com/upload/attach/202206/921173_Z3PUMAC3CPSMS69.jpg)

在多个同一基础镜像容器同时启动的情况下，由于共享基础镜像，在某一容器中该漏洞的触发会波及到其他容器。下图演示了在其中一个只读容器中更改文件内容后，其他容器的文件系统也跟着发生了变化。

![dirtypipe.gif](https://bbs.kanxue.com/upload/attach/202206/921173_5RD75BD9V8GY8EJ.jpg)

这就是容器的隔离性远不如虚拟机的地方，容器被称作为一种轻量级的操作系统虚拟化，多个容器与宿主据共享一个内核，这就导致了首先容器无法隔离宿主机出现的内核漏洞，一旦内核出了问题，很容器出现容器逃逸的情况；其次同一宿主机的容器之间的隔离性也不强，容器与容器之间很容器出现隐蔽通道、信息泄露的问题。

# 参考链接

漏洞披露：

`5fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1K9i4u0@1P5i4m8A6M7r3g2Q4x3X3g2U0L8e0c8S2L8r3I4Q4x3X3g2U0L8$3#2Q4x3V1j5`.

breeze漏洞分析：

`f53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3g2F1j5h3!0@1K9h3q4F1i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3W2)9J5k6o6l9^5y4o6M7`.

脏管道漏洞与容器：

`43cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8Y4W2C8i4K6u0W2K9h3!0Q4x3V1k6T1L8r3!0Y4i4K6u0r3k6r3W2J5N6s2W2Q4x3X3c8H3K9i4m8W2i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5k6r3y4$3k6g2)9J5k6o6t1H3x3U0u0Q4x3X3b7H3z5o6b7%4i4K6u0V1j5$3!0F1N6r3q4A6L8X3g2J5K9i4A6W2k6q4)9J5k6r3q4H3M7r3I4A6j5$3q4@1K9h3!0F1M7#2)9J5c8R3`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
