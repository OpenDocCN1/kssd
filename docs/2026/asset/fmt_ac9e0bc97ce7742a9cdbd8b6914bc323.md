
![image-20210416211552618](https://bbs.kanxue.com/upload/attach/202104/856790_VWDA94SB753H6WK.jpg)

使用`IDA`打开`DLL1.dll`，找到`Exports`窗口，找到`DLL1Print`，双击找到函数；

![image-20210416212033278](https://bbs.kanxue.com/upload/attach/202104/856790_UFSU4BQ6XAYG92V.jpg)

至于`sub_10001038`，经过验证是一个打印字符串函数；

综上所述，是一个打印函数；

> 5、当`Lab09-03.exe`调用`WriteFile`的函数时，它写入的文件名是什么？

![image-20210416213125399](https://bbs.kanxue.com/upload/attach/202104/856790_QFRMYDFA6Q8GKRC.jpg)

所以我们还是得先去看看`DLL2ReturnJ`返回了什么；

![image-20210416213240603](https://bbs.kanxue.com/upload/attach/202104/856790_BP8HCJ4PJUSTK8D.jpg)

双击进去；

![image-20210416213326284](https://bbs.kanxue.com/upload/attach/202104/856790_HVCY9DWBU239NAD.jpg)

我们会发现这是一个未定义数据空间，我们摁`x`查看交叉引用；

![image-20210416213628915](https://bbs.kanxue.com/upload/attach/202104/856790_MFWRAPNUFXES6FZ.jpg)

只有这个是赋值操作，双击进去；

![image-20210416213756686](https://bbs.kanxue.com/upload/attach/202104/856790_ZKNDH4EY4EGA854.jpg)

找到了，所以返回的文件名是`temp.text`；

> 6、当`Lab09-03.exe`使用`NetScheduleJobAdd`创建一个job时，从哪里获取第二个参数的数据？

首先在`Lab09-03.exe`找到`NetScheduleJobAdd`；

![image-20210416214208954](https://bbs.kanxue.com/upload/attach/202104/856790_WSW37UX5PWE2KQA.jpg)

往上找，谁交叉引用了`[ebp+Buffer]`了；

![image-20210416215637406](https://bbs.kanxue.com/upload/attach/202104/856790_4GA9WFCCSTJXAWD.jpg)

去`DLL3.dll`找到`DLLGetStructure`瞅一瞅；

![image-20210416215856263](https://bbs.kanxue.com/upload/attach/202104/856790_DH2RDB6M3TR9QAV.jpg)

查看交叉引用；

![image-20210416215953774](https://bbs.kanxue.com/upload/attach/202104/856790_F8JXAV642B3JXWE.jpg)

点进去查看；

![image-20210416220103554](https://bbs.kanxue.com/upload/attach/202104/856790_UTS8UGPUBCTUGDN.jpg)

所以可以看见，`ptr [eax], 36EE80h`*（实际汇编是不允许内存直接存取立即数的，这里只是为了清晰直观）*，意思是在以`buff`位置的值为地址存放`0x36EE80`的**偏移**，并不是就是存放`0x36EE80`；

这里要说一下的是，因为`DLL3.dll`加载时如果发生了重定位，这个偏移是会动的；

> 7、在运行或调试`Lab09-03.exe`时，你会看到`Lab09-03.exe`打印处三块神秘数据。`DLL1.dll`的神秘数据、`DLL2.dll`的神秘数据、`DLL3.dll`的神秘数据分别是什么？

都是以下的步骤：

1. 先把`dll`拖进`IDA`；
2. 导出函数（`Exports`）找到打印函数；
3. 双击进去，找到字符串；
4. 摁`x`找到赋值语句，向上找赋值源；

直接给出答案：

- `DLL1.dll`：

  ![image-20210416234333801](https://bbs.kanxue.com/upload/attach/202104/856790_K32S8W7HT7ENS2N.jpg)

  所以是当前进程标识；

- `DLL2.dll`：

  ![image-20210416234515724](https://bbs.kanxue.com/upload/attach/202104/856790_W98ANH87S4YSDEK.jpg)

  是文件句柄；

- `DLL3.dll`：

  找这个的时候遇到点以外，没有找到本来想找的`mov`，但是转念一想，函数可以，所以找了`push`，找到了；

  ![image-20210416235309090](https://bbs.kanxue.com/upload/attach/202104/856790_KXD6NHXZDNYSP77.jpg)

  `MultiByteToWideChar`的功能是将一个字符串映射为宽字符（Unicode）字符串；

  根据MSDN，可以看见最后是把`ping a54K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2S2L8s2N6S2M7X3g2S2L8X3q4D9P5i4y4A6M7$3u0G2L8$3E0Q4x3X3g2U0L8$3@1`的位置赋给了`WideCharStr`；

> 8、如何将`DLL2.dll`加载到`IDA Pro`中，使得它与`OllyDbg`使用的加载地址匹配？

不匹配，但是可以手动调整加载到`IDA Pro`中的地址；

### Lab9-3分析总结

这个题目相对简单，需要注意的就是关于PE中的重定位问题；

至于找东西，有两种思路：

1. 分析就是自上向下看过程；
2. 做题就是自下往上找调用，找变动；

---

## 总结

这一篇写了四五天了，期间是各种突发情况，眼看月底近了，真的是心急如焚，希望能抓紧写完，抓住银四的尾巴找到工作吧；

这三题，第一题写到很细，分析的有点因小失大，拘泥于字眼之间了；

最后用两句诗句与君共勉吧；

**行路难，行路难，多歧路，今安在！**

**长风破浪会有时，直挂云帆济沧海。**

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于2021-4-17 00:18被平头猿小哥编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

上传的附件：

- [Chapter\_9L.zip](javascript:void(0);)（99.24kb，27次下载）
