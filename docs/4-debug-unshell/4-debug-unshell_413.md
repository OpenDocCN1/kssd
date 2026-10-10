

# Customizer XP1.7.9 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50049.htm`](https://www.pediy.com/kssd/pediy05/pediy50049.htm)

ASPROtect 1.22 加壳的 Customizer XP1.7.9 beta 2 脱壳(超简单)   
crack by fwnl   
软件主页：http://www.tweaknow.com/   
软件简介：   
　　Customizer XP 是为 Windows 2000 及 XP 设计的系统性能优化工具。软件的用户界面十分清爽，它可以让你查看并修改 Windows 中隐藏   
  
的设置选项，你只需点击相应复选框即可。Customizer XP 可以让你的 Internet 访问速度大大提高，仅仅需要点击相应按钮，软件会自动显   
  
示推荐使用的数值。Customizer XP 同样提供了许多附加的 Windows 修改选项，例如：选择不同登录时显示不同的背景，发生错误时自动重启   
  
Windows 资源管理器以免造成你的工作成果丢失等。   
  
(要在 winn2000/xp 下安装程序)   
一.找入口点   
  
    由于是 delphi 程序，所以利用快速寻找 aspr 入口的方法:   
  
    执行 Customizer，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 UltraEdit32 打开 dump.exe，ALT+F3，填 runtime,选中查找 ASCⅡ,执   
  
行搜索，搜到后,向上查找离 runtime 最近的机器码为 55 8B   
  
EC 的地方就是程序的 oep，而在 AhaView.exe 里这个位置在 offset:0016b240 处，用 peditor 的 flc 功能将它转换为 virtual   
  
address 就是 0056bc40----oep。   
  
二.用 trw2000 初步脱壳   
  
  下面要在入口处脱壳，打开 SuperBPM，点 erase，用 trw 载入 Customizer XP，下 g 56bc40，接着下 pedump 脱出程序.   
  
  
三.修复 import table   
  
  打开原加壳程序,在 Import REConstructor v1.4.2+ 的 Attach to an Active Process 窗口中选取 Customizer 的进程,然后在下方的 oep   
  
处填入 0016bc40,点 IAI AutoSearch,再点 Get Imports 然后点 Show Invalid,在 Imported Functions Found 窗口里的无效地址上点鼠标右键，选   
  
Trace Leve11(disasm),再点 show invaids，发现还有几个没有修复，再次在那几个没有修复的地址上点鼠标右键.   
  选中 Plugin Tracer(Asprotect 1.2X Emul),再点 show invaids 应发现所有的 dll 显示 valid:Yes 了。然后点 Fix Dump,选中你用 trw2000   
  
pedump 出的文件修复.   
  
呵呵,这样程序就脱壳了,是不是超简单.主要是作者不会用 ASPR,今天的最新版没加壳了,不知是不是作者在研究 ASPR 的加壳呀   
这个公司还有个软件 RAM Idle Pro  的脱法也是一样,呵呵   
                                                                                      fwnl   
                                                                              Beginner's Cracking Group                                                                                                  The Free Cracking Group Of China   
                                                                                    2002.4.2   
                                                                                      长沙  4dd466d0e1e0cc123afbe504da2462ca