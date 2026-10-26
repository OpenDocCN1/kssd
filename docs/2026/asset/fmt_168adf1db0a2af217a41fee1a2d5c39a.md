
![](https://bbs.kanxue.com/upload/attach/201101/304196_apuu92lgjibnvyh.jpg)

双击“（未注册）”这一行，来到主程序窗口，

![](https://bbs.kanxue.com/upload/attach/201101/304196_rb4q4yje05az4qd.jpg)

从`004086C6`这一行往上观察，发现`004086C4`这一行跳转正好跳过“未注册”字样这一行，在`004086C4`这一行单击鼠标右键，选择汇编，将`jnz`改为`jmp`,

![](https://bbs.kanxue.com/upload/attach/201101/304196_72uc8rbu2dkvnqc.jpg)

在汇编窗口单击鼠标右键，选择复制到可执行文件—所有修改，然后选择全部复制，然后单击鼠标右键，选择保存文件。

重启中国象棋V2.0，直接进入软件，且没有“未注册”字样。OK！现在就是正式用户了！

## 方法三：追踪注册码，制作内存注册机

根据前文分析，`00430B67`这一行是关键`CALL`，用`OD`载入中国象棋V2.0，`Ctrl+G`输入`00430B67`

![](https://bbs.kanxue.com/upload/attach/201101/304196_4790nmujzxmxza3.jpg)

点击确定，来到`00430B67`这一行，`F2`下断点，

![](https://bbs.kanxue.com/upload/attach/201101/304196_hv8nbxe4qpsrofj.jpg)

`F9`运行软件，输入注册名和注册码，

![](https://bbs.kanxue.com/upload/attach/201101/304196_r3pcgapxvq546a6.jpg)

点击“注册”，软件被`OD`断下来， `F8`单步，观察堆栈窗口，

![](https://bbs.kanxue.com/upload/attach/201101/304196_oxl5lkaexf5mzob.jpg)

正在进行真假码对照，复制"`9eb9a33259ad034db8b0a8462dac9aee`"字符串进行注册，注册成功。

下面制作内存注册机，见上面分析，软件被`OD`断在`00430B67`这一行，`F7`单步进入，`F8`单步走，来到`0042F4A0`这一行，观察寄存器窗口，

![](https://bbs.kanxue.com/upload/attach/201101/304196_nt05oxuveey37a2.jpg)

`EAX`寄存器中就是真的注册码，打开`keymake`注册机，选择其它—内存注册机，浏览选择中国象棋`ChineseChess`应用程序，点击添加，在是中断位置填写`0042F4A0`,中断次数位置填写`1`,第一字节位置填写`FF`，指令长度位置填写`6`,

![](https://bbs.kanxue.com/upload/attach/201101/304196_0wd6pkl5t1dn3rz.jpg)

选中“保存下列信息为注册码”，选中内存方式，寄存器方式选择`EAX`，然后点击添加，关闭添加窗口，点击生成按钮，

![](https://bbs.kanxue.com/upload/attach/201101/304196_bf61wzqsuumtwz8.jpg)

选择界面三，点击确定，保存注册机。运行内存注册机，注册码会自动出现，OK！

![](https://bbs.kanxue.com/upload/attach/201101/304196_a6teiu6r4lt4s19.jpg)

## 方法四：算法分析，写注册机

目前正在学习中，有点难度。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.JPG](javascript:void(0);)
  （72.97kb，305次下载）
- [2.jpg](javascript:void(0);)
  （73.57kb，304次下载）
- [3.jpg](javascript:void(0);)
  （63.51kb，305次下载）
- [4.jpg](javascript:void(0);)
  （34.33kb，306次下载）
- [5.jpg](javascript:void(0);)
  （46.09kb，301次下载）
- [6.jpg](javascript:void(0);)
  （32.40kb，305次下载）
- [7.jpg](javascript:void(0);)
  （73.61kb，304次下载）
- [8.jpg](javascript:void(0);)
  （70.04kb，302次下载）
- [9.jpg](javascript:void(0);)
  （69.81kb，299次下载）
- [10.jpg](javascript:void(0);)
  （108.85kb，302次下载）
- [11.jpg](javascript:void(0);)
  （450.03kb，307次下载）
- [12.jpg](javascript:void(0);)
  （85.21kb，300次下载）
- [13.jpg](javascript:void(0);)
  （186.09kb，298次下载）
- [14.jpg](javascript:void(0);)
  （146.87kb，300次下载）
- [15.jpg](javascript:void(0);)
  （81.94kb，299次下载）
- [16.jpg](javascript:void(0);)
  （172.37kb，288次下载）
- [17.jpg](javascript:void(0);)
  （92.84kb，287次下载）
- [18.jpg](javascript:void(0);)
  （190.70kb，289次下载）
- [19.jpg](javascript:void(0);)
  （59.21kb，283次下载）
- [20.jpg](javascript:void(0);)
  （182.86kb，283次下载）
- [21.jpg](javascript:void(0);)
  （184.00kb，284次下载）
- [22.jpg](javascript:void(0);)
  （88.82kb，278次下载）
- [23.jpg](javascript:void(0);)
  （183.49kb，282次下载）
- [24.jpg](javascript:void(0);)
  （88.35kb，276次下载）
- [25.jpg](javascript:void(0);)
  （96.50kb，278次下载）
- [26.jpg](javascript:void(0);)
  （128.58kb，281次下载）
- [27.jpg](javascript:void(0);)
  （155.54kb，280次下载）
- [28.jpg](javascript:void(0);)
  （153.34kb，280次下载）
- [29.jpg](javascript:void(0);)
  （62.51kb，269次下载）
