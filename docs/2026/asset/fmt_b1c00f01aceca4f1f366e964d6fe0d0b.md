
图 24 看门狗kill了调试进程

## 4.总结

本文通过记录智能家居设备的部分研究过程，向读者分享了嵌入式产品简单的固件分析、固件篡改、调试环境的建立流程。希望对各位读者的安全研究有帮助。

由于现有设备的特殊性，这些方法只是达到阶段目标的可行方式之一。比如烧录固件，也可以通过`uboot`的`tftp`功能烧录，还可以通过`uboot`加载DIY的固件等等，这些功能比较简单，但归根结底还是向`flash`中写入数据，并将`flash`区域中的数据读取到内存空间后进行启动的过程，单独成文的必要性不大，有兴趣的同学可以单独沟通。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-4-8 20:48

被fengzhidian编辑，原因：改了图

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)

[#技术分享](https://bbs.kanxue.com/forum-128-1-168.htm)

[#家用设备](https://bbs.kanxue.com/forum-128-1-173.htm)

[#漏洞挖掘](https://bbs.kanxue.com/forum-128-1-178.htm)

[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)

[#PCB分析](https://bbs.kanxue.com/forum-128-1-169.htm)

[#工控设备](https://bbs.kanxue.com/forum-128-1-174.htm)

[#汽车安全](https://bbs.kanxue.com/forum-128-1-175.htm)
