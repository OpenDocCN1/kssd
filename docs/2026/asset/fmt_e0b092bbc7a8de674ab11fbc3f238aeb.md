
```
	int arrInt[2][3] = { {1, 2, 3}, {4, 5, 6} };

001A10E8  mov         dword ptr [ebp-1Ch],1  

001A10EF  mov         dword ptr [ebp-18h],2  

001A10F6  mov         dword ptr [ebp-14h],3  

001A10FD  mov         dword ptr [ebp-10h],4  

001A1104  mov         dword ptr [ebp-0Ch],5  

001A110B  mov         dword ptr [ebp-8],6      //从地址ebp-0x1C开始给每个4字节的元素赋值，这里可以看出二维数组的初始化和一维数组的初始化过程并没有区别

	int(*pInt)[3] = arrInt;

001A1112  lea         eax,[ebp-1Ch]  

001A1115  mov         dword ptr [ebp-28h],eax  //取出数组的首地址赋值给pInt

	printf("arrInt[1][1]=%d,arrInt[1]=%d,arrInt=%d\n", arrInt[1][1], arrInt[1], arrInt);

001A1118  lea         eax,[ebp-1Ch]  //取出数组首地址并压栈

001A111B  push        eax  

001A111C  mov         ecx,0Ch          //将ecx赋值为0xC也就是数组的第二维的字节大小乘以3

001A1121  shl         ecx,0            //左移操作，这里是为了对移动大小进行乘法，公式是2^(n-1)次方，这里n就是我们指定的arrInt[1]中的1，通过这个方法来算出偏移

001A1124  lea         edx,[ebp+ecx-1Ch]      //根据偏移得出数据的地址并压栈

001A1128  push        edx  

001A1129  mov         eax,0Ch  

001A112E  shl         eax,0  

001A1131  lea         ecx,[ebp+eax-1Ch]  //跟上面的分析知道这里是根据偏移得到arrInt[1]的位置

001A1135  mov         edx,4              //赋值为4，这是因为每个整型是4字节

001A113A  shl         edx,0              //左移操作，功能与上相同

001A113D  mov         eax,dword ptr [ecx+edx]   //将偏移与arrInt[1]的地址相加得出arrInt[1][1]的位置，取出数据后压栈

001A1140  push        eax  

001A1141  push        2131B0h            //压入字符串的地址

001A1146  call        001A11B0           //调用函数

001A114B  add         esp,10h  

	printf("*(*(pInt + 1) + 1)=%d,pInt+1=%d,pInt=%d", *(*(pInt + 1) + 1), pInt + 1, pInt);

00FC114E  mov         eax,dword ptr [ebp-28h]  //取出pInt的值，里面保存的是arrInt的首地址

00FC1151  push        eax  

00FC1152  mov         ecx,dword ptr [ebp-28h]  //取出pInt的值

00FC1155  add         ecx,0Ch           //加0xC也就是3*4由于此时指针是指向3维数组的指针所以地址加一都要跨过三个整型大小的字节才能得到对应的值

00FC1158  push        ecx          

00FC1159  mov         edx,dword ptr [ebp-28h]  //取出pInt的值

00FC115C  mov         eax,dword ptr [edx+10h]  //首先pInt是指向3维数组的指针所以他加1需要跨过12个字节                                             

00FC115F  push        eax              //对pInt+1解引用之后得到了指向整型的指针这个时候在+1需要跨过4个字节，所以这里需要+0x10   

00FC1160  push        10331D8h                 // 压入字符串地址

00FC1165  call        00FC11B0                 //调用printf

00FC116A  add         esp,10h
```

由上我们可以看出，其实二维数组和一维数组没有什么太大的差别，只是他寻找数据的时候要多跨一个维度，而且这个维度的大小和第二维的大小有关。指向数组的指针在进行寻址的时候也是如此，他需要考虑所指数组的大小。

# 四.字符串

在C/C++中，用""包围的字符被称为字符串，每个字符串的最后都是以0作为字符串的结束。那么字符串在内存中是如何保存的和我们的字符有什么不同。看下面这个实例：

```
	char arrTest1[] = { 'H', 'e', 'l', 'l', 'o', ' ', '1', '9', '0', '0' };

00081038  mov         byte ptr [ebp-10h],48h  

0008103C  mov         byte ptr [ebp-0Fh],65h  

00081040  mov         byte ptr [ebp-0Eh],6Ch  

00081044  mov         byte ptr [ebp-0Dh],6Ch  

00081048  mov         byte ptr [ebp-0Ch],6Fh  

0008104C  mov         byte ptr [ebp-0Bh],20h  

00081050  mov         byte ptr [ebp-0Ah],31h  

00081054  mov         byte ptr [ebp-9],39h  

00081058  mov         byte ptr [ebp-8],30h  

0008105C  mov         byte ptr [ebp-7],30h          //将单引号所表示的字符的数分别赋值给相应的arrTest1数组的位置

	char arrTest2[] = { "Hello 1900" };

00081060  mov         eax,dword ptr ds:[000F31B0h]  

00081065  mov         dword ptr [ebp-24h],eax  

00081068  mov         ecx,dword ptr ds:[000F31B4h]  

0008106E  mov         dword ptr [ebp-20h],ecx  

00081071  mov         dx,word ptr ds:[000F31B8h]  

00081078  mov         word ptr [ebp-1Ch],dx  

0008107C  mov         al,byte ptr ds:[000F31BAh]  

00081081  mov         byte ptr [ebp-1Ah],al        //从000F31B0地址处取出4个字节的数据赋值给arrTest2数组相应的位置

	char *arrTest3 = "Hello 1900";

00081084  mov         dword ptr [ebp-30h],0F31B0h    //将000F31B0地址赋值给arrTest3
```

那么这里的的`0x000F31B0`存储了什么内容，我们可以在内存窗口查看

![](https://bbs.kanxue.com/upload/attach/202109/835440_WC5DDVW6GYQUMSW.jpg)

可以看到这个地址保存的就是我们需要用到的字符数据，而这个地址所在的位置也被称为全局数据区。

由此我们可以得出结论，相对于单引号的一个一个赋值，双引号包含的字符串放在了全局数据区，当我们对数组进行赋值的时候，程序会从这个地址中把数据赋值到相应的位置，而当我们是对指针进行赋值的时候，程序会把这个地址直接赋值给相应的指针变量。

最后我们可以在变量窗口查看相应的赋值结果![](https://bbs.kanxue.com/upload/attach/202109/835440_69X672Z6SQFDN92.jpg)

# 五.引用

由于指针的错误使用，往往会造成比较严重的后果，C++推出了引用。

```
11:       int x = 0;

00401068   mov         dword ptr [ebp-4],0        //为x赋值为0

12:       int &y = x;

0040106F   lea         eax,[ebp-4]               //将x地址取出

00401072   mov         dword ptr [ebp-8],eax     //将地址给y

13:

14:       printf("%d\n", x);

00401075   mov         ecx,dword ptr [ebp-4]    

00401078   push        ecx

00401079   push        offset string "%d\n" (00427008)    //输出x

0040107E   call        printf (00401190)

00401083   add         esp,8

15:       y = 1;

00401086   mov         edx,dword ptr [ebp-8]        //取出y，此时是x的地址

00401089   mov         dword ptr [edx],1            //对这个地址赋值为1，其实就是为x赋值

16:       printf("%d\n", x);

0040108F   mov         eax,dword ptr [ebp-4]

00401092   push        eax

00401093   push        offset string "%d\n" (00427008)

00401098   call        printf (00401190)

0040109D   add         esp,8
```

可以看出引用和指针在内存中存储形式是一样的，引用只是编译器对指针的封装罢了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2021-10-20 11:18

被1900编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
