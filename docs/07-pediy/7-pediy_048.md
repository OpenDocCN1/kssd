
# 再谈给应用程序 DIY 启动画面和 Java 源代码补丁修改

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-263.htm`](https://www.pediy.com/kssd/pediy08/pediy8-263.htm)

搞 DIY 的朋友经常把自己设计的 logo 或者是打点广告窗口加进别人的应用程序作为启动画面。

偶也搞了小例子给大家，并且编译了一个小 Java 补丁小工具一下就搞定（只限本程序，程序异常概无负责任）。

现在偶把看雪论坛的一个 logo 加进 Windows 自带的进程管理器中。

附件下载。

系统：Microsoft Windows XP Professional 版本 2002 Service Pack 2

修改程序：Windows 自带的进程管理器，快捷键 `Ctrl+Alt+Del` 调出。

目录地址：`C:\WINDOWS\system32\taskmgr.exe`

第一步：先把看雪论坛的一个 logo 编译进 `logo.dll` 库。这一步骤不说了，用 C，汇编很容易实现它。

第二步：再把 `logo.dll`，加进 `taskmgr.exe` 里当做启动画面 logo...

需要给五个地方修改补丁一下...

第一处：

VA：`1005944`
RVA：`5944`
Offset：`4D44`
字节大小：`5 byte size`
修改内容：`E9 A6 F5 00 00`

第二处：

VA：`01014EEF`
RVA：`00014EEF`
Offset：`000142EF`
字节大小：`46 byte size`
修改内容：
```
68 20 55 01 01 FF 15 E4 10 00 
01 85 C0 74 10 FF 15 DC 10 00 
01 E8 5E D1 FF FF E9 4A 09 FF 
FF 90 90 90 6A 00 68 30 55 01 
01 68 40 55 01 01 6A 00 FF 15 
28 21 00 01 EB DD    
```

第三处：

VA：`01015520`
RVA：`00015520`
Offset：`00014920`
字节大小：`8 byte size`
修改内容：`6C 6F 67 6F 2E 64 6C 6C`

第四处：

VA：`01015530`
`7 byte size`
```
FF FE 02 5F 38 5E 00   
```

第五处：

VA: `01015540`
`34 byte size`
```
FF FE 6E 00 6F 00 20 00 6C 00 6F 00 61 00 64 00 
20 00 6C 00 6F 00 67 00 6F 00 2E 00 64 00 6C 00    
6C 00     
```

源代码如下：

```java
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import javax.swing.*;

public class Pediy implements ActionListener
{
   Frame f;
   boolean flag=false;
   byte[] buff;
   String strfile;
   Label t1,t2,t3;
   FileDialog fd;
   TextField tf1,tf2,tf3;
   Button b1,b2,b3,b4;
   byte[] one;
   byte[] onebuf={-23,-90,-11,0,0};//offset=4d44(19780)
   byte[] twobuf={
       104, 32, 85 ,1 ,1, -1 ,21,  -28 ,16 ,0,
       1 ,-123 ,-64 ,116, 16 ,-1 ,21 ,-36 ,16 ,0,
       1 ,  -24, 94, -47 ,-1 ,-1 ,-23, 74, 9,
       -1 ,-1 ,-112 ,-112 ,-112 ,106 ,0 , 104 ,48 ,85,
       1 ,1 ,104 ,64 ,85,  1, 1, 106 ,0,
       -1 , 21, 40,  18 ,0 ,1, -21, -25 }; //offset=000142EF (82671)
   byte[] threebuf={108, 111 ,103, 111 ,46 ,100 ,108 ,108};//offset=14920(84256)
   byte[] fourbuf={-1 , -2 , 2 , 95 ,56 ,94, 0};//offset=14930;(84272)
   byte[] fivebuf={ -1, -2, 110 ,0, 111, 0 ,32, 0 ,108, 0, 111, 0, 103, 
   0 ,111, 0 ,46 ,0 ,100,0,108, 0 ,108 ,0};//offset=14940(84288)
 
public void display()
 {
    f=new Frame("http//www.pediy.com-qiweixue ");
               f.setSize(350,230);
               f.setBackground(Color.lightGray);
               f.setLocation(300,240);
               f.setLayout(new FlowLayout());
               t1=new Label(" taskmgr.exe(Window xp sp2):");
               t2=new Label("程序入口指针（十进制）:");
               tf1=new TextField("",40) ;
               tf2=new TextField("Offset1=4d44,Offset2=000142EF,Offset3=14920....",40);
               tf3=new TextField("",40) ;     
               t3=new Label("http:www.pediy.com");
               b1=new Button("打开文件");
               b2=new Button("修改");
               b3=new Button("撤消"); 
               b4=new Button("about");
               f.add(t1);
               f.add(tf1); 
               f.add(t2);
               f.add(tf2);
               f.add(t3);
               f.add(tf3);
               f.add(b1);
               f.add(b2);
               f.add(b3);
               f.add(b4);       
               b1.addActionListener(this);
               b2.addActionListener(this);
               b3.addActionListener(this);
               b4.addActionListener(this);
               f.addWindowListener(new WinClose());
               f.setVisible(true);
               tf2.setBackground(Color.gray);
               tf3.setBackground(Color.GRAY);
              //tf2.setForeground(Color.red);
              tf2.enable(false);
              tf3.setText("Don't close me first!程序异常请撤消退到原来");
             tf3.enable(false);
             b2.enable(false);
             b3.enable(false);
 }
 
public static void main(String[] args)
   {
              (new Pediy()).display(); 
    }
    
    public void actionPerformed(ActionEvent e)
    {
      if(e.getSource()==b1)
      {
               fd = new FileDialog(f,"qiweixue open",FileDialog.LOAD);
               fd.show(true); 
         strfile=fd.getDirectory()+fd.getFile();
         if (strfile!=null)
         { 
            tf1.setText(strfile);
             }
         b2.enable(true);
          b3.enable(false);
       }
        
        else if(e.getSource()==b2)
        {
               try
                 {
                   //String str=new String(tf2.getText());   
                 // long offset=Long.parseLong(str); 
                 // long fixset=offset; 
                   RandomAccessFile Taskraf=new RandomAccessFile(strfile,"rw"); 
                   long offsetone=19780;
                   one=new byte[5]; 
                   Taskraf.read(one);                 
                   offsetone =19780;    
                   Taskraf.seek(offsetone);
                   long offsettwo=82671;
                   Taskraf.write(onebuf);
                   Taskraf.seek(offsettwo);
                   Taskraf.write(twobuf);
                   long offsethree=84256;
                   Taskraf.seek(offsethree);
                   Taskraf.write(threebuf);
                   long offsetfour=84272;
                   Taskraf.seek(offsetfour);
                   Taskraf.write(fourbuf);
                   long offsetfive=84288;
                   Taskraf.seek(offsetfive);
                   Taskraf.write(fivebuf);
                   JOptionPane.showMessageDialog(f,"修改成功");
                   Taskraf.close();
                   this.tf1.enable(false);
                     }
                       catch(Exception e1)
                       {
                         e1.printStackTrace();
                       }
                       tf2.enable(false);
                       flag=true;
                       b2.enable(false);
                       b3.enable(true);
        }
        
        else if(e.getSource()==b3)
        {  
           try{           
                       RandomAccessFile raf=new RandomAccessFile(strfile,"rw");                    
             long offseton=19780;
             raf.seek(offseton);
             raf.write(one);          
             raf.close();   
               JOptionPane.showMessageDialog(f,"撤消成功");
               b3.enable(false);
            }
           catch(Exception e1)
            {
          e1.printStackTrace();   
            }
       }
       
       else 
       {
       final Frame f2=new Frame("www.pediy.com");
          Label b1=new Label("Author:qiweixue ,CopyRight:http//www.pediy.com");
           f2.add(b1);
           f2.setBackground(Color.gray);
          f2.addWindowListener(new WindowAdapter(){
          public void windowClosing(WindowEvent e)
  {
    f2.hide();
  }
           });
      f2.setLocation(300,240);
       f2.setSize(370,100);
       f2.setVisible(true);
    }
 }
}

class WinClose extends WindowAdapter
{
  public void windowClosing(WindowEvent e)
  {
    System.exit(0);
  }
}
```

\========

程序有 bug，`FreeLibrary` 有的异常，但是不影响运行的效果。有点奇怪，刚发现的不愿意改了。

大家注意就 ok...

有错误之处，多指教。

运行环境 `jdk1.5` 或 `jre1.5`，运行文件 `PediyRun.bat`。

Copyright http//www.pediy.com

`4dd466d0e1e0cc123afbe504da2462ca`
