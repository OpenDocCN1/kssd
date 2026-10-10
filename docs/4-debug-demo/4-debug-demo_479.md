

# GTA2

> 原文：[`www.pediy.com/kssd/pediy03/forum487.htm`](https://www.pediy.com/kssd/pediy03/forum487.htm)

安装完游戏,光碟先不要拿出.   
  bpx GetDriveTypeA   
  执行程序,弹出 sice. 返回到 gta2 区域,向上的一个 cmp XXX   
  就是第一个检测点. 第一步完成.   
  
  到游戏主界面,如果没有光碟,此时执行 start game,就会   
  来到第二个陷阱. security fail.   
  
  所以执行前 bpx ReadFile,按 F10 执行程序,会找到第二个检   
  测点,很简单啦.   
  
  无壳,无反跟踪,西西，我只会做最简单的啦.   
  
  关键是开始我被迷惑了，因为我下第一个断点时,没有放光碟,   
  这个程序上来直接读一下光碟上的东西,此时检测,如果读错误   
  就跳走了,所以不会断到 GetDriveTypeA 出.   
  
  完了.  算个开始吧.   
  
  向各位兄弟学习!  4dd466d0e1e0cc123afbe504da2462ca