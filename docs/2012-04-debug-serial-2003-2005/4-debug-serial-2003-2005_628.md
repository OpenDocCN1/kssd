
# GreenBrowser V2.3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6626.htm`](https://www.pediy.com/kssd/pediy06/pediy6626.htm)

`GreenBrowser` 是一个多窗口浏览器，它基于 `IE`，并且提供了更多的特色功能。收集器、鼠标手势、鼠标拖动页面、搜索引擎、页面背景色、工具栏皮肤、代理、标签栏、快捷键、自动滚屏、自动保存、自动填表、启动模式、自动隐藏工具栏、鼠标拖放、页内链接分析、群组、状态栏显示设定、悬浮监视窗、设置收藏夹路径、特定的下载控制方案、搜索栏、扩展工具栏、运行脚本、下载管理器、页面缩放...

我用了这个软件，感觉特别好用，但是需要注册 `![](img/620448a77d884bd6b2c4a7cfef437501.jpg)`

主程序用 `Aspack2.12` 加壳，`Pescan` 直接脱掉了，`VC++`

下断 `bpx GetWindowTextA`，断在：

```
0049B868  |.  FF15 C8154C00      call dword ptr ds:[<&USER32.GetWindowTextA>]   ; \GetWindowTextA
0049B86E  |.  8B4C24 08         mov ecx,dword ptr ss:[esp+8]
0049B872  |.  6A FF            push -1
0049B874  |.  E8 54340000       call <test.CString::ReleaseBuffer(int)>
```

单步跟踪，下面就是对用户名和注册码进行复制和转移，跟踪到：

```
00409875   .  E8 C6BF0000       call <test.sub_415840>   //关键代码处

{
0041584E  |.  64:8925 00000000  mov dword ptr fs:[0],esp
00415855  |.  83EC 38           sub esp,38
00415858  |.  56               push esi
00415859  |.  33F6             xor esi,esi
0041585B  |.  8D4C24 4C        lea ecx,dword ptr ss:[esp+4C]
0041585F  |.  897424 44        mov dword ptr ss:[esp+44],esi
00415863  |.  E8 5B140800      call <test.sub_496CC3>
00415868  |.  8D4C24 4C        lea ecx,dword ptr ss:[esp+4C]
0041586C  |.  E8 06140800      call <test.sub_496C77>
00415871  |.  8B4424 4C        mov eax,dword ptr ss:[esp+4C]
00415875  |.  8B40 F8          mov eax,dword ptr ds:[eax-8]
00415878  |.  83F8 0A          cmp eax,0A                        ;注册码长度是否为 10 位
0041587B  |.  74 23            je short <test.loc_4158A0>
0041587D  |.  8D4C24 4C        lea ecx,dword ptr ss:[esp+4C]
00415881  |.  C74424 44 FFFFFFFF mov dword ptr ss:[esp+44],-1
00415889  |.  E8 C08F0800      call <test.sub_49E84E>
0041588E  |.  33C0             xor eax,eax
00415890  |.  5E               pop esi
00415891  |.  8B4C24 38        mov ecx,dword ptr ss:[esp+38]
00415895  |.  64:890D 00000000 mov dword ptr fs:[0],ecx
0041589C  |.  83C4 44          add esp,44
0041589F  |.  C3               retn                              ;不是就返回出错
004158A0 <>|>  A1 74E84E00     mov eax,dword ptr ds:[4EE874]
004158A5  |.  53               push ebx
004158A6  |.  57               push edi
004158A7  |.  894424 0C        mov dword ptr ss:[esp+C],eax
004158AB  |.  894424 10        mov dword ptr ss:[esp+10],eax
004158AF  |.  8D4C24 14        lea ecx,dword ptr ss:[esp+14]
004158B3  |.  6A 05            push 5
004158B5  |.  51               push ecx
.......................................................
00415917  |.  33C0             xor eax,eax                         ;eax 清零
00415919  |.  C74424 1C 02000000 mov dword ptr ss:[esp+1C],2       ;这几句代码是对参数进行初始化
00415921  |.  894424 34        mov dword ptr ss:[esp+34],eax        ;分别从 esp+1C 处到 esp+40 处
00415925  |.  C74424 20 06000000 mov dword ptr ss:[esp+20],6       ;共 10 个 DWORD 值，为十六进制的
0041592D  |.  894424 38        mov dword ptr ss:[esp+38],eax        ;2,6,0xFFFFFFF9(-7),4,FFFFFFFF
00415931  |.  C74424 24 F9FFFFFF mov dword ptr ss:[esp+24],-7      ;8,0,0,0,0
00415939  |.  894424 3C        mov dword ptr ss:[esp+3C],eax
0041593D  |.  C74424 28 04000000 mov dword ptr ss:[esp+28],4
00415945  |.  C74424 2C FFFFFFFF mov dword ptr ss:[esp+2C],-1
0041594D  |.  C74424 30 08000000 mov dword ptr ss:[esp+30],8
00415955  |.  894424 40        mov dword ptr ss:[esp+40],eax
00415959  |.  8D7C24 1C        lea edi,dword ptr ss:[esp+1C]       ;把刚刚初始化的地址放到 edi
0041595D <>|>  8B4C24 0C        /mov ecx,dword ptr ss:[esp+C]      ;注册码的前 5 位，倒序的如输入 1234567890,
00415961  |.  8A17             |mov dl,byte ptr ds:[edi]          ;那么这里就是"54321"
00415963  |.  8A040E           |mov al,byte ptr ds:[esi+ecx]
00415966  |.  02C2             |add al,dl                           ;把注册码从第 5 位到第一位分别和刚刚初始化的数据相加
00415968  |.  3C 30            |cmp al,30                             ;比如第一次是"5"的 ASCII+02,得 37,也就是 7 的 ASCII 码
0041596A  |.  884424 14        |mov byte ptr ss:[esp+14],al        ;上面是比较是不是大于等于 0
0041596E  |.  7D 06            |jge short <test.loc_415976>        ;如果小于 0 就把相加的结果再加上 10
00415970  |.  04 0A            |add al,0A
00415972  |.  884424 14        |mov byte ptr ss:[esp+14],al
00415976 <>|>  3C 39            |cmp al,39                             ;小于等于 9 吗？
00415978  |.  7E 06            |jle short <test.loc_415980>
0041597A  |.  04 F6            |add al,0F6                            ;大于 9 就再加上 0xF6
0041597C  |.  884424 14        |mov byte ptr ss:[esp+14],al
00415980 <>|>  8B5424 14        |mov edx,dword ptr ss:[esp+14]       ;相加后的结果以字节的形式送入 esp+14 处，再以 DWORD 送入 edx
00415984  |.  8D4C24 0C        |lea ecx,dword ptr ss:[esp+C]
00415988  |.  52               |push edx
00415989  |.  56               |push esi
0041598A  |.  E8 17940800      |call <test.CString::SetAt(int,char)>  ;置数，把相加前相应的注册码位设为相加后的值比如第一次置为 7
0041598F  |.  46               |inc esi
00415990  |.  83C7 04          |add edi,4
00415993  |.  83FE 05          |cmp esi,5                             ;前 5 位数处理完了吗？
00415996  |.^ 7C C5           \jl short <test.loc_41595D>
00415998  |.  8D4C24 0C        lea ecx,dword ptr ss:[esp+C]          ;指向转换后的数再倒过来 70660=>06607
0041599C  |.  E8 F3930800      call <test.sub_49ED94>
004159A1  |.  8B7424 10        mov esi,dword ptr ss:[esp+10]         ;这里是注册码的后 5 位
004159A5  |.  8B4424 0C        mov eax,dword ptr ss:[esp+C]          ;这里是前面对前 5 位注册码处理的结果,假如我输入的注册码的前 5 位是 12345,那么处理后会变为 06607
004159A9 <>|>  8A10            /mov dl,byte ptr ds:[eax]            ;下面就开始比较了
004159AB  |.  8A1E            |mov bl,byte ptr ds:[esi]            ;也就说注册码的后 5 位必须和前 5 位的处理结果相同
004159AD  |.  8ACA            |mov cl,dl
004159AF  |.  3AD3            |cmp dl,bl
004159B1  |.  75 1E           |jnz short <test.loc_4159D1>
004159B3  |.  84C9            |test cl,cl
004159B5  |.  74 16           |je short <test.loc_4159CD>
004159B7  |.  8A50 01         |mov dl,byte ptr ds:[eax+1]
004159BA  |.  8A5E 01         |mov bl,byte ptr ds:[esi+1]
004159BD  |.  8ACA            |mov cl,dl
004159BF  |.  3AD3            |cmp dl,bl
004159C1  |.  75 0E           |jnz short <test.loc_4159D1>
004159C3  |.  83C0 02         |add eax,2
004159C6  |.  83C6 02         |add esi,2
004159C9  |.  84C9            |test cl,cl
004159CB  |.^ 75 DC          \jnz short <test.loc_4159A9>
}
```

简单的用 `C++` 写了个注册机 `![](img/e45fac309594d4f9580d902b0ed40a73.jpg)`

```cpp
#include <iostream.h>

#include <string.h>

main()
{
  int arg[10]={2,6,-7,4,-1,8,0,0,0,0};
  char dkey[10];
  int dkey2[5]={0};
  char dkey3[5]={0};
  cout<<"Please input the first 5 Digits of your Donate Key:"<<endl;
  cin>>dkey;
  int i=0;
  i=strlen(dkey);
  if(i!=5)
  {
    cout<<"must 5 digists:"<<endl;
    return -1;
  }
  for(int k=0;k<5;k++)
  {
    dkey2[k]=dkey[4-k];
    dkey2[k]+=arg[k];
    if(dkey2[k]<'0')
    {
      dkey2[k]+=10;
    }
    if(dkey2[k]>'9')
    {
      dkey2[k]+=0xF6;
      dkey3[k]=(char)(dkey2[k]-0xFF);
    }
    dkey3[k]=(char)dkey2[k];
    dkey[9-k]=dkey3[k];
  }
  cout<<"Your Donate Key is ";
  for(int t=0;t<10;t++)
  {
    cout<<dkey[t];
  }
}
```



```cpp
cout << endl;  

return 0;  

}
```

我在密码分析学专题论坛发了个密码分析题目，兄弟们去做做啊，呵，最近公布答案![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
