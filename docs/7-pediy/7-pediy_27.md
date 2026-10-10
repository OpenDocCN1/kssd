

# 屏蔽 RealPlay 10.5 英文版更新补丁并附 java 源代码

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-274.htm`](https://www.pediy.com/kssd/pediy08/pediy8-274.htm)

现在很多软件有自动更新的功能.占这系统资源,占这网络资源...比如说 realplay,我的机器没有连网,可是天天看到它的更新组件 realsched.exe 在我内存里跳来跳去.很郁闷,把它的注册表 key 删除了吧,通常在：  
 "Software\\Microsoft\\Windows\\CurrentVersion\\Run"  
key 里边！  
下一次,启动 realplay 主程序,它有会给你复原回去,没有用.  
所以想屏蔽它一下,我的 realplay　10.5 英文版...  
  
两个方案：  
第一个在主程序找Ｒealplay 加载更新组件的地方,由于主程序太大,加载组件很多,注册表里有大堆 key.估计主程序也用,更新程序也负责查看更新点.很杂乱的,  
我用了第二中偷工减料的方法：  
是直接在目录找到它的更新组件：  
C:\\Program Files\\Common Files\\Real\\Update\_OB\\realsched.exe  
...  
在更新组件中,它负责几个网络组件的更新和加载不鼓励上网的朋友改,对于上不了网的来说,如果它启动有异常,主程序有几处报告网络错误...  
  
思路大概是这样,可以让更新组件把所有网络组件全检查完毕,有更新的就报告你,没有就退出！！  
我是直接让它退出的,没有上网,让它更新干哈.  
我用 java 写的补丁源程序,原理很简单,见笑了,出了错误自己负责,其他版本不兼容,要改自己动手！！！  
１：找入口,  
２：更新检查  
３：退出  
＝＝＝  
由于没有上网直接１与３步骤跳过２,其他人在另 diy  
  
  
import java.awt.\*;  
import java.awt.event.\*;  
import java.beans.\*;  
import java.io.\*;  
import java.util.\*;  
import javax.swing.\*;  
  
public class Pediy implements ActionListener  
{  
   Frame f;  
   byte\[\] buff;  
   String strfile;  
   Label t1,t2,t3;  
   FileDialog fd;  
   TextField tf1,tf2,tf3;  
   Button b1,b2,b3,b4;  
     
 public void display()  
 {  
    f=new Frame("屏蔽 RalyPlay 更新补丁-qiweixue ");  
               f.setSize(350,230);  
               f.setBackground(Color.lightGray);  
               f.setLocation(300,240);  
               f.setLayout(new FlowLayout());  
               t1=new Label("realsched.exe:");  
               t2=new Label("程序入口指针（\*必须是十进制）:");  
               tf1=new TextField("",40) ;  
               tf2=new TextField("90597",40);  
               tf3=new TextField("",40) ;       
               t3=new Label("http:www.pediy.com");  
               b1=new Button("打开文件");  
               b2=new Button("修改");  
               b3=new Button("撤消");   
               b4=new Button("about");  
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
               f.addWindowListener(new WinClose());  
               f.setVisible(true);  
              tf3.setText("Don't destroy me!,如果 RealPlay 运行异常,请撤消操作...");   
             tf3.enable(false);  
            b2.enable(false);  
            b3.enable(false);  
 }  
 public static void main(String\[\] args)  
   {  
            
              (new Pediy()).display();   
    }  
    public void actionPerformed(ActionEvent e)  
    {  
      if(e.getSource()==b1)  
      {  
        
             fd = new FileDialog(f,"qiweixue open",FileDialog.LOAD);  
             fd.show(true);   
         strfile=fd.getDirectory()+fd.getFile();  
         if (strfile!=null)  
         {   
            tf1.setText(strfile);  
              
             }  
         b2.enable(true);  
          b3.enable(true);  
       }  
        else if(e.getSource()==b2)  
        {  
              try  
                {  
                   
                  String str=new String(tf2.getText());    
                  long offset=Long.parseLong(str);  
                  long fixset=offset;  
                  RandomAccessFile raf=new RandomAccessFile(strfile,"rw");                         
                  byte\[\] newbuf={-23,69,01,0,0};  
                  buff=new byte\[5\];  
                  raf.seek(fixset);  
                  raf.read(buff,0,5);  
                  raf.seek(offset);  
                  raf.write(newbuf);  
                   JOptionPane.showMessageDialog(f,"修该成功!");  
                   
                 raf.close();  
                     
                     }  
                       catch(Exception e1)  
                       {  
                         e1.printStackTrace();  
                       }  
                       tf2.enable(false);  
                        
            
        }  
        else if(e.getSource()==b3)  
        {   
          try{            
                       
                     String str=new String(tf2.getText());  
                     long offset=Long.parseLong(str);   
                   RandomAccessFile raf=new RandomAccessFile(strfile,"rw");                         
               //  byte\[\] oldbuf={85,-117,-20,106,-1};  
                 raf.seek(offset);  
                 raf.write(buff);          
                 raf.close();  
                   JOptionPane.showMessageDialog(f,"撤消成功");  
                   tf2.enable(true);  
                   tf3.enable(true);  
            }  
          catch(Exception e1)  
            {  
          e1.printStackTrace();    
            }  
       }  
       else   
       {  
       final Frame f2=new Frame("www.pediy.com");  
         Label b1=new Label("Author:qiweixue ,this is only used to RealPlay version English 10.5 :=>");  
          f2.add(b1);  
         f2.addWindowListener(new WindowAdapter(){  
         public void windowClosing(WindowEvent e)  
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
  
class WinClose extends WindowAdapter  
{  
  public void windowClosing(WindowEvent e)  
  {  
    System.exit(0);  
  }  
}  
  
＝＝＝＝＝＝＝＝＝  
运行要 jdk２,  
一个开执行包程序 Pediy.jar 和源代码 Pediy 

 4dd466d0e1e0cc123afbe504da2462ca