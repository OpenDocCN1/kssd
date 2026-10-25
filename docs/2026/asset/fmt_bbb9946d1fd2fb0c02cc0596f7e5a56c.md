
那么我们现在获取到了2个消除的下标，我们就要找到消除的`call`和它的参数（再模拟它）

![](https://bbs.kanxue.com/upload/attach/201704/585205_03k8p47hxm36din.jpg)

老规矩栈回溯，出去外面找到个比较容易模仿的`call`（参数越少越好）

![](https://bbs.kanxue.com/upload/attach/201704/585205_mz2q978kdyzz9wd.png)

我们发现这一层不是最外面的，继续回溯，并且我们发现函数结尾`retn 0x18`（24/4=6）很明显`call`是6个参数

![](https://bbs.kanxue.com/upload/attach/201704/585205_q9fhjco28gbvpgw.png)

现在数组下标都有了只要调用`call`就可以了

只要模拟这6个参数就可以了

![](https://bbs.kanxue.com/upload/attach/201704/585205_666i4v8l817fbge.png)

```
//模拟代码
//6个参数
//00129BB8   00000000  无视
//00129BBC   0012BB50  地图首地址
//00129BC0   00129BEC  棋子地址A
//00129BC4   00129BF4  棋子地址B
//00129BC8   019B8400  这个地址是13683D0+30得来的
//公式:
//mov eax, 0x45DCF8
//mov eax, dword ptr[eax]                    
//mov ecx, dword ptr[eax + 0x1E84]

//00129BCC   00000004  固定值

push 4                                  //固定值                                                                                                                
mov ebx,ReEcx
add ebx,0x30
push ebx                                //13683D0+30
lea eax,[FirstDot]                      //坐标A
lea ebx, [SecondDot]                    //坐标B
push ebx
push eax

/******************获取到12BB50地图首地址***************************/
mov eax, 0x45DCF8
mov ecx, dword ptr[eax]
lea eax, dword ptr[ecx + 0x1e84]
mov eax, dword ptr[eax]
mov eax, dword ptr[eax + 4]
push eax                                                        
/*******************获取到12BB50地图首地址**************************/
push 0                              //死值0
mov eax,0x41C68E                     
call eax                           //不支持call 立即数
```

属性选择静态库中使用 MFC

测试代码：

![](https://bbs.kanxue.com/upload/attach/201704/585205_teh6q2h3lwpd1sm.png)

代码跟OD调试信息跟注入器我都打包起来了（9M好像直接传不上去，我分开来传）

**致谢**

**感谢15PB老师们的辛勤栽培！**

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`注入器.part1.rar`](javascript:void(0);)（2.00MB，143次下载）
- [`注入器.part2.rar`](javascript:void(0);)（246.05kb，128次下载）
