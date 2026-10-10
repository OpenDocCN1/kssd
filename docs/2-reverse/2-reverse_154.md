

# 自己修改 OD 让主窗口不再出现 xx.exe

> 原文：[`www.pediy.com/kssd/pediy12/130721.html`](https://www.pediy.com/kssd/pediy12/130721.html)

说明下我用的是 52 的 OD，其它版本的可能不一样。  
   
大家使用 OD 调试的时候，OD 的主窗口会出现被调试程序的 文件名.exe  
如下图:  
  
Anti.exe 就会出现在 OD 的主窗口中。有的程序就会枚举所有窗口，检测标题是不是含有  
程序文件名.exe 字串，比如我上传这个附件。  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILm6MSMRk6GGOMZ+iJPnMAyp.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILm6MSMRk6GGOMZ+iJPnMAyp.jpg)  
StrongOD 好像没有这个功能。  
  
那么现在我们就开始自己修改吧！  
  
1：把 OD 加到 OD 中。（直接复制一个 OD 扔进去就行了）。  
2：F9 让他运行起来。效果如下：  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILmjkPV3xri/63EbbdOiBB8U.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILmjkPV3xri/63EbbdOiBB8U.jpg)  
  
以下:调试 OD 的 OD 称为 OD1, 被调试的 OD 称为 OD2  
  
3：就把一个不会让 OD 挂悼的程序拖到 OD2 中，  
OD1 中 bp SetWindowTextA 下这个断点。  
在 OD2 中点<< 重新运行  
  
(SetWindowText 这个是设置窗口文本的。OD2 的标题就会改变。)  
  
那么 OD1 断下后堆栈如下:  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILk5Y5sxzRyfD0mNZhEniojs.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILk5Y5sxzRyfD0mNZhEniojs.jpg)  
  
具体分析过会在进行先整体看下。 按 又 OD1 F9 运行如下  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILnPd0+0j0B20ox5sD20dmdF.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILnPd0+0j0B20ox5sD20dmdF.jpg)  
  
  
现在分析一下：  
TEXT 里的内容和我们 OD2 的标题内容是一样的。说明我们找对地址了。  
调用 SetWindowTextA 的是 StrongOD 插件里的一个函数。  
  
现在我们在 OD2 里，注意是 OD2 里，点那个<<标致，就是重新运行。  
  
这次我们要回溯到调用的地方。断下后如下：  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILmTAcqdQmtfvVwSKfv6O89F.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILmTAcqdQmtfvVwSKfv6O89F.jpg)  
  
Ctrl+f9  3 下就回到了程序领空 向上拉一点就看到了：如图  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILmsaT/d9ZEPRVgrIDhtYeY3.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILmsaT/d9ZEPRVgrIDhtYeY3.jpg)  
  
好 在 call SetWindowTextA 这句上下断点 下断点。  
  
继续 F9 重复上面的动作 把其它调用 SetWindowTextA 的地方找出来， 找到三个: 全设断  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILmETTpmsYldfFZN6yFdzPEK.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILmETTpmsYldfFZN6yFdzPEK.jpg)  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILk+SOIlv5j+svNFvmL0Y5Jj.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILk+SOIlv5j+svNFvmL0Y5Jj.jpg)  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILlpIy61hAbm+z2tRs68juMJ.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILlpIy61hAbm+z2tRs68juMJ.jpg)  
  
  
  
找到 3call SetWindowTextA 个。  
全部设断 取消 bp SetWindowText 断点  
(经测试 475bc7 这个不要改，改了以后按<< 或 ctrl+f2 被调试程序就会结束) 只改 2 个。  
  
思路:  
让主窗口不显示内容的话那么只要把   
BOOL SetWindowText(  
  HWND hWnd,         // handle to window or control  
  LPCTSTR lpString   // title or text  
);  
参数 lpString 变成空就行了。   
我们在 OD1 里写个子程序, 就是一个函数。 然后修改上面的  
3 个的 call SetWindowTextA 这句， 让它调用我们的函数，在我们函数里面把  
参数 lpString 就是标题的内容变为空, 然后和 OD 原来一样调用 StrongOD 里的函数。  
最后 retn 返回就可以了。  
  
  
现在自己找个空白处写函数：如下  
  
0x4af810:  
push ebp  
mov ebp,esp  
push eax  
mov eax,dword ptr ss:\[ebp+0xC\]  
mov byte ptr ds:\[eax\],0x0  
pop eax  
push dword ptr ss:\[ebp+0xC\]  
push dword ptr ss:\[ebp+0x8\]  
call 004AF5BE  
mov esp,ebp  
pop ebp  
retn 0x8  
  
简要说明下：  
0x4af810: // 我是用的 NonaWrite 写的哈 手动写就不要这句了  
push ebp  
mov ebp,esp  
// 这就是把 lpString 变为空  
push eax  
mov eax,dword ptr ss:\[ebp+0xC\]  
mov byte ptr ds:\[eax\],0x0  
pop eax  
// 下面三句就是模仿的原来 Call SetWindowTextA 的调用  
push dword ptr ss:\[ebp+0xC\]  
push dword ptr ss:\[ebp+0x8\]  
call 004AF5BE    
mov esp,ebp  
pop ebp  
// 这句很重要，要使堆栈平衡, 为什么是 0x8 因为你原来的 call  
// push 了 2 个参数嘛  
retn 0x8  
  
因为我们设了断的嘛，所以按 alt+b 打开断点  
把 2 个 call 都改成 call 4af810 (4af810 你自己函数的地址)  
  
然后 右键->复制可执行文件->所有修改->全部复制->保存文件……  
  
  
如果出现下面的这个：  
可以在调试 OD2 之前改一下 OD2 的 rawsize.  
  
那么可能是因为 rawsize 太小了 用 PE Explorer 改大一下就行. 或者换个位置。。  
 ![`link2.youzhaopian.com/pplink/qqEcQ7PKILnYx562JvHKqEz41uRbEDBY.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILnYx562JvHKqEz41uRbEDBY.jpg)  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILk5W4zgpn0dwkVx8wiGQdpC.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILk5W4zgpn0dwkVx8wiGQdpC.jpg)  
  
视图->区段头->区段编辑器  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILlBujDwbeNzG0ZzCNyzKXA+.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILlBujDwbeNzG0ZzCNyzKXA+.jpg)  
点左边那个锁  
改 RAW 数据大小 ae800 为 000AFA00 文件另存为 xx 然后就调试这个改过的.  
  
 这个就可以写入代码了…….  
最后如果对你有用的话,请回下帖![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")！光是图片就搞得头都大了！  
上效果图  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILkh0TZc/QitiQi6AT5qHhdu.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILkh0TZc/QitiQi6AT5qHhdu.jpg)  
HELLO 表示过了检测 oh 表示没过检测  
![`link2.youzhaopian.com/pplink/qqEcQ7PKILkVwIyiyeYTDRIyfMNwK0wQ.jpg`](http://link2.youzhaopian.com/pplink/qqEcQ7PKILkVwIyiyeYTDRIyfMNwK0wQ.jpg)

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || anti.rar  
 |