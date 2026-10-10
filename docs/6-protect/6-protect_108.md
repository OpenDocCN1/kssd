

# 一个无驱型 USB 加密锁的 HID 通信分析 4 楼部分更新应用程序枚举 HID 设备进行找狗

> 原文：[`www.pediy.com/kssd/pediy12/130803.html`](https://www.pediy.com/kssd/pediy12/130803.html)

【文章标题】: 一个无驱型加密锁的 HID 通信分析  
【文章作者】: zailaiyp  
【作者邮箱】: 70306682@qq.com  
【操作系统】: Windows 2003  
【使用工具】: OD+Bus Hound+UsbView  
【作者声明】: 只是感兴趣，没有其他目的。错漏之处还请各位  
              高手指点  
  
  
  
  
楼层说明：  
一楼 HID 设备 描述符  加密狗的枚举  
二楼 USB 设备标准 11 条命令 HID 描述符 分析狗的思路  
三楼 重要的 HID 及 SETUPDIxxxxx 函数 共 19 个。原型及简单说明  
四楼 应用程序枚举 HID 设备进行找狗  
六楼 确定本加密狗的内置控制命令码 及详细分析过程  
  
待添加  
  
  
  
  
  
【讨论说明】  
前段时间从朋友处得来一套他做生意淘汰下来的软件，带有 2 个加  
密狗，上周末就仔细学习研究了一下，深感资料匮乏。。。整整  
两天时间都贡献给这个狗了。本文主要从 HID 通信的角度来书写，  
没有任何技术含量，目的是做为学习总结记录之用，毕竟这花了  
我一个周末整整两天的时间，总要写点什么出来。内容浅陋，不  
值打狗高手们一提。。。。。。  
我之前对加密狗了解很少，这篇帖子仅以我手中的这种狗为例。  
  
实际上，HID 设备的分析基本上都差不多，而 HID 加密狗算是很  
简单的了 起码从报告描述符这方面来讲，比 HID 鼠标什么的还简  
单。  
  
\=========================================  
另外，本文虽然标题为加密锁云云，但实际上大部分内容为 HID 相关，  
加密锁只是作为一个具体例子来讲。对其他的 HID 设备，都是适用的。  
以学习交流为主。  
我是一只菜菜鸟，基本上什么都不会，文章写的更是差劲，还请各位  
朋友多多关照 多多指点。  
\=========================================  
  
  
USB 协议中有个 HID（人机接口设备）类，符合 HID 规范的就叫 HID  
设备，无驱型加密锁就是 HID 设备，HID 库中提供了很多函数，用  
做应用程序和 HID 设备进行通信。  
当把 HID 设备 比如这个加密狗插进主机，系统就会进行枚举等操  
作，下面先讲一下枚举。  
  
  
Bus Hound 是分析 USB 设备的不可少工具，使用很简单，就不多说  
了。先插入狗，打开 Bus Hound，打开设备项，挂勾设备，返回  
主显示界面，运行右下角的 RUN，再把狗拔下， 再插进去，信息  
就出来了。  
  
  
  
Device  Phase  Data                      Description         
\------  -----  ------------------------  ----------------  ------------------  
  
//80 表示要求数据输入 06 是获取描述符 00 表示语言 ID  03 表示获取字符串描述符  
  CTL    80 06 00 03  00 00 ff 00  GET DESCRIPTOR   
  
//设备回答：     
  IN     04 03 09 04               ....                 
  CTL    80 06 01 03  09 04 ff 00  GET DESCRIPTOR     
  IN     0e 03 55 00  53 00 42 00  ..U.S.B.            
         4b 00 65 00  79 00        K.e.y.           
  CTL    80 06 00 01  00 00 12 00  GET DESCRIPTOR         
  IN     12 01 00 02  00 00 00 08  ........                
         89 36 62 87  00 02 01 01  .6b.....          
         00 01                     ..              
  CTL    80 06 00 02  00 00 09 00  GET DESCRIPTOR        
  IN     09 02 22 00  01 01 01 80  ..".....               
         19                        .                 
  CTL    80 06 00 02  00 00 22 00  GET DESCRIPTOR          
  IN     09 02 22 00  01 01 01 80  ..".....               
         19 09 04 00  00 01 03 01  ........         
         00 00 09 21  11 01 00 01  ...!....         
         22 20 00 07  05 81 03 08  " ......         
  CTL    00 09 01 00  00 00 00 00  SET CONFIG             
  CTL    21 0a 00 00  00 00 00 00  SET IDLE                
  CTL    81 06 00 22  00 00 60 00  GET DESCRIPTOR          
  IN     06 00 ff 09  01 a1 01 85  ........                
         01 09 01 15  00 26 ff 00  .....&..           
         75 08 95 08  b1 06 85 02  u.......          
         09 03 96 14  00 b1 06 c0  ........         
  
  
  
以上就是主机和加密狗的部分通信内容，简单说明一下：  
信息从获取描述符开始，之前还有些步骤，没显示出来，有兴趣  
的朋友可以自行了解。  
主机通过默认的控制通道使用标准的 USB 设备请求命令发送到 HID  
设备，这样的命令基本上所有的 USB 设备都支持， 每条命令 8 字节，共有 11 条命令。  
上面的大部分做了注释，很容易理解。如有需要 可以去查看 USB  
的官方文档。后面还会具体讲到。  
  
  
HID 设备要被主机识别，就要提供足够多的信息给主机。而 HID 设  
备本身不会主动发消息给主机。HID 设备插入后，依靠的是主机  
使用控制传输来从设备获取信息。  
  
Descriptor 就是描述符，是一个存储在 USB 设备的固件中的数据  
结构，用于描述该设备的属性，主机就是通过使用控制传输来  
要求设备提供这些信息的。让主机知道设备具有哪些功能、属  
于具体哪类设备、使用哪些传输方式 等等。  
  
  
刚开始我参照官方文档，一个字节一个字节的把上面的几个描  
述符给扣了出来，再然后 悲剧的发现，使用 UsbView 直接就能  
显示出来了。  
以下是直接显示的结果  
  
Device Descriptor:  
bcdUSB:             0x0200  
bDeviceClass:         0x00  
bDeviceSubClass:      0x00  
bDeviceProtocol:      0x00  
bMaxPacketSize0:      0x08 (8)  
idVendor:           0x3689  
idProduct:          0x8762  
bcdDevice:          0x0200  
iManufacturer:        0x01  
0x0409: "USBKey"  
0x0409: "USBKey"  
0x0409: "USBKey"  
iProduct:             0x01  
0x0409: "USBKey"  
0x0409: "USBKey"  
0x0409: "USBKey"  
iSerialNumber:        0x00  
bNumConfigurations:   0x01  
  
ConnectionStatus: DeviceConnected  
Current Config Value: 0x01  
Device Bus Speed:      Low  
Device Address:       0x01  
Open Pipes:              1  
  
Endpoint Descriptor:  
bEndpointAddress:     0x81  
Transfer Type:   Interrupt  
wMaxPacketSize:     0x0008 (8)  
bInterval:            0x0A  
  
Configuration Descriptor:  
wTotalLength:       0x0022  
bNumInterfaces:       0x01  
bConfigurationValue:  0x01  
iConfiguration:       0x01  
0x0409: "USBKey"  
0x0409: "USBKey"  
0x0409: "USBKey"  
bmAttributes:         0x80 (Bus Powered )  
MaxPower:             0x19 (50 Ma)  
  
Interface Descriptor:  
bInterfaceNumber:     0x00  
bAlternateSetting:    0x00  
bNumEndpoints:        0x01  
bInterfaceClass:      0x03 (HID)  
bInterfaceSubClass:   0x01  
bInterfaceProtocol:   0x00  
iInterface:           0x00  
  
HID Descriptor:  
bcdHID:             0x0111  
bCountryCode:         0x00  
bNumDescriptors:      0x01  
bDescriptorType:      0x22  
wDescriptorLength:  0x0020  
  
Endpoint Descriptor:  
bEndpointAddress:     0x81  
Transfer Type:   Interrupt  
wMaxPacketSize:     0x0008 (8)  
bInterval:            0x0A  
  
单词基本上可以望文生义，具体的值请参考 USB 官方文档-------  
尽管不愿意总说 但是没办法 资料少的可怜，两天时间，我一直  
开着金山词霸。  
  
  
  
特别的对于 HID 设备，还有一个最重要的描述符----报告描述符。  
这是个复杂的东东，没有什么工具可以直接显示，只能使用手工。  
81 06 00 22  00 00 60 00 这个命令就是获取设备的报告描述  
符了。  
  
主机和 HID 设备使用“报告”来传输信息，比如无驱的 USB 键盘   
鼠标 等等，他们都是 HID 设备 都是依靠 报告 来和主机交换信  
息，对于报告，主机就需要使用该设备的报告描述符来解析具  
体的内容，否则，对于设备传来的数据，主机是根本无法知道  
表示什么内容的。   
对于一些符合 HID 规范的设备，比如键盘鼠标游戏杆等等，他们  
的报告描述符也符合规范，插上就能用，对于自定义的 HID 设备，  
则不一定需要具体的描述“报告”中的数据所表示的意义，因为  
应用程序提取到数据后，可以按照需要自己进行解析使用。  
  
这个加密狗的报告描述符就很简单，下面是我反出来的。  
  
  
USAGE\_PAGE (Vendor Defined Page 1)  06 00 FF  
USAGE (Vendor Usage 1)      09 01  
COLLECTION (Application)    A1 01   
  REPORT\_ID (1)        85 01   
  USAGE (Vendor Usage 1)    09 01  
  LOGICAL\_MINIMUM (0)      15 00   
  LOGICAL\_MAXIMUM (255)      26 FF 00   
  REPORT\_SIZE (8)      75 08   
  REPORT\_COUNT (8)      95 08   
  FEATURE (Data,Var,Rel)    B1 06   
  REPORT\_ID (2)        85 02   
  USAGE (Undefined)      09 03  
  REPORT\_COUNT (276)      96 14 01   
  FEATURE (Data,Var,Rel)    B1 06   
END\_COLLECTION  C0  
  
简单说下 这个加密狗是自定义 HID 设备，报告数据的用法由设备  
商自定义（也就是在在程序里自己处理吧）  
这个狗有 2 种报告，第一种是特征报告，ID 为 1，数据表示范围  
在 0-255 之间，报告大小为 8 位，有 8 个，也就是共 8 个字节的数据。第二种也是特征报告，ID 为 2，数据大小也为 8 位，有 276 个，也就是共有 276 个字节。  
这里我有点没搞明白，报告 2 里的计数为 96 14 00 ，但是我怎么  
也想不出 96 怎么可以得出 14 00 的？只能得出 96 14 01 即 276 个  
字节。  
  
实际上，对于这种简单的自定义 HID 设备来说，取得它的报告描  
述符的意义似乎并不大，如果是一些复杂的自定义 HID 设备，通  
过分析报告描述符，是可以取得很多有用信息的。  
具体到这只狗。。。只知道会有 2 个报告，第一个 8 字节，第二个  
276 字节（这里有疑问 因为实际报告大小似乎都是 21 字节）  
  
\===========================================  
  
以上是狗的插入枚举部分。这时候狗已经可以正常使用了。下  
面就开始运行软件。  
  
\-------原来整理记录也是很累人的   
  
 稍后待续 

刚才发现枚举部分的注释没写上去。  
USB 设备的标准请求命令共有 11 个， 大小为 8 字节。  
  
构成如下：  
bmRequestType 请求类型位图 1 字节  
bRequest      请求代码 1 字节  
wValue         2 字节  
wIndex         2 字节  
wLength        2 字节 如果有数据传送 就是传送的字节数  
  
比如上面的 81 06 00 22  00 00 60 00    
  
81 传输方向为设备到主机的接口   
06 为请求代码表示是 GET\_DESCRIPTOR 请求  
00 索引 22 种类        
00 00   
60 00 这里是 wLength 字段，应该表示传送的字节数，但实际  
传送的只是 32 个字节，即使是再大的值，也是按照实际有的传送  
  
其他命令可以参考 USB 规范，这 11 个命令是所有 USB 设备都支持的。  
你也可以使用 BUS HOUND 的发送命令功能自己向 USB 设备发送命令  
，这个功能很有爱的...你懂的。  
  
  
再补充一下 对于 USB 设备 是没有 HID Descriptor 的。  
USB 的 HID 类设备才有。  
在这个狗中  
HID Descriptor:  
bcdHID:             0x0111  
bCountryCode:         0x00  
bNumDescriptors:      0x01  ；表示作为 HID 设备有 1 个报告描述符  
bDescriptorType:      0x22  ；下个描述符类型为报告描述符  
wDescriptorLength:  0x0020  ；下个描述符的大小。这里也就  
解释了为什么 81 06 00 22  00 00 60 00 中的字节虽然为 0X60 但  
是实际发送却为 32 个字节了。因为这里 0X0020 指定了。  
  
  
\=========================================  
观看的朋友可能很郁闷 讲了这么多，有毛用啊？ 咳。。。  
这个。。。确实没毛用。 本来我是只想 OD 看看 F7 F8 搞搞  
就算了，没想到跟到 HID 函数，再到 HID 设备 再到 USB 通信  
再到各种描述符 再去啃协议，。。。。。。差点深陷其  
中，两天时间都浪费了 连非诚勿扰都没看，好在我悬崖勒  
马了。这个有瘾，玩玩就算了。不可深入。  
不过了解一下 感觉还是收获蛮多的。  
这种通信数据没有加密的狗，无论是复制还是模拟，都是  
秒秒钟的事，下面大家就会看到了。但是本文的目的不在  
于此，所以具体的狗型号及软件名称都隐身，关键点蒙薄纱  
，会 HELLOWORLD 的都能揭掉。另外，我手中的这 2 个狗应该  
很古老了。相信早就淘汰 N 久了。  
\=========================================  
  
学习的过程是快乐。希望与您分享。  
\=========================================  
  
  
  
继续：  
  
一般的 HID 的键盘鼠标等等设备，它们的报告描述符中都指定了  
各种用法啊范围啊什么什么的，比如说带有音量旋钮的 USBHID  
键盘，比如鼠标的 XY 轴，比如游戏杆的力反馈 。。。主机都  
会在枚举时通过读取设备的报告描述符来作为解析“报告”的  
依据。 一个 HID 设备再复杂，它只要依据 HID 规范已有的报告描述  
符项目去设计，然后固化进 HID 设备，那么此 HID 设备就可以在任  
何支持 USBHID 设备的主机上即插即用，不再需要安装任何驱动。  
  
  
  
但是作为自定义的 HID 设备，报告描述符中并没有指定数据的用法，  
报告 中的数据用法就需要 HID 设备自己去处理 ，在主机端，则是  
需要应用程序来处理。这同样不需要任何驱动，而是依靠 HID 库中  
的各种函数来实现。  
  
加密锁需要提供文档和 SDK 或者是加狗程序，如是加狗程序，则会  
在被加狗的程序的外壳中动态导入 HID 库，如果采用源代码的 SDK  
方式加密，在 SDK 函数中也会导入 HID 库。我想作为自定义的 HID 狗  
，应该不会有狗厂家会直接让人使用 HID 函数吧，牵扯到报告内数  
据的用法，极不安全，还是再封装一层成 SDK 提供。  
  
\-------------------------------------------  
上面部分是我自己的理解，如有错误，还望高手指出。  
  
  
   
  
顺着这个思路：  
对于 HID 设备，要去分析它接收的数据的格式及  
控制命令，它在自己的设备内部根据控制命令去运算数据，然后  
会有返回值。  
  
对于应用程序，以跟踪 HID 函数为主，要注意函数对数据的用法。  
  
  
  
有关 HID 函数的介绍 非常少，在 WDK 中都可以查到，但都是 E 文。  
金山词霸也对绝大部分单词表示有压力。百度谷歌的中文资料也  
很少，即使有 也都是抄来抄去 实际内容也就是那几篇。。。。。。  
  
在这种情况下，我做出了一个艰难的决定----------查看应用  
程序的导入表，发现共有 15 个 HID 库函数 4 个 SetupDixxxx 函数，  
于是我只查这 19 个函数的使用。然而  我发现 要想了解这些函  
数，就要看 USBHID 协议 因为它们的参数都和 USB/HID 协议有关。  
所以 啃协议 啃各种描述符 是不可少的功课。  
  
  
为节约您的时间，后面我把相关函数及原型列出，其中的大部分  
参数涉及到前面我讲的描述符，主要是报告描述符。  
  
  
休息  待续 

本楼列出重要的 HID 及 SETUPDIxxxxx 函数 共 19 个。原型及简单说明。  
也利于其他朋友搜索。  
  
  
也许您看到这些参数会觉得很挠头。。。 刚开始我也这样，但是了解描述符 尤其是报告描述符  
以后，就很容易理解了。所以 如果想理解并使用 HID 函数，就必须要看 USB HID。  
看得懂报告描述符，对这些函数也就能搞明白了。  
  
  
\=================================================  
  
  
SetupDiGetClassDevsA   
查询所有已安装的 HID 设备，得到一个指向该 HID 设备集合的句柄  
HDEVINFO  
  SetupDiGetClassDevs(  
    IN LPGUID  ClassGuid,  OPTIONAL  
    IN PCTSTR  Enumerator,  OPTIONAL  
    IN HWND  hwndParent,  OPTIONAL  
    IN DWORD  Flags  
    );  
  
\===========================================  
  
  
SetupDiDestroyDeviceInfoList   
释放设备信息集合,和上面  
的函数是一对  
WINSETUPAPI BOOL WINAPI  
  SetupDiDestroyDeviceInfoList(  
    IN HDEVINFO  DeviceInfoSet  
    );  
  
  
  
\============================================  
  
  
SetupDiEnumDeviceInterface   
查询 HID 设备集中每一个设备的接口信息  
WINSETUPAPI BOOL WINAPI  
  SetupDiEnumDeviceInterfaces(  
    IN HDEVINFO  DeviceInfoSet, //HID 设备集合句柄  
    IN PSP\_DEVINFO\_DATA  DeviceInfoData,  OPTIONAL  
    IN LPGUID  InterfaceClassGuid,  
    IN DWORD  MemberIndex,//索引  
    OUT PSP\_DEVICE\_INTERFACE\_DATA  DeviceInterfaceData  
    );  
  
参数说明  
SP\_DEVICE\_INTERFACE\_DATA 结构定义了一个设备接口  
typedef struct \_SP\_DEVICE\_INTERFACE\_DATA {  
  DWORD  cbSize; //结构体大小  
  GUID  InterfaceClassGuid; //接口的 GUID  
  DWORD  Flags;  
  ULONG\_PTR  Reserved;  
} SP\_DEVICE\_INTERFACE\_DATA, \*PSP\_DEVICE\_INTERFACE\_DATA;  
  
\===============================================  
  
  
  
SetupDiGetDeviceInterfaceDetail   
获取其详细的信息  
WINSETUPAPI BOOL WINAPI  
  SetupDiGetDeviceInterfaceDetail(  
    IN HDEVINFO  DeviceInfoSet, //HID 设备集合句柄  
    IN PSP\_DEVICE\_INTERFACE\_DATA  DeviceInterfaceData,//设备接口信息  
    OUT PSP\_DEVICE\_INTERFACE\_DETAIL\_DATA             DeviceInterfaceDetailData,  OPTIONAL  //接口详细信息  
    IN DWORD  DeviceInterfaceDetailDataSize,  
    OUT PDWORD  RequiredSize,  OPTIONAL  
    OUT PSP\_DEVINFO\_DATA  DeviceInfoData  OPTIONAL  
    );  
  
参数说明  
SP\_DEVICE\_INTERFACE\_DETAIL\_DATA 结构体提供了一个特定设备接口的详细信息。  
typedef struct \_SP\_DEVICE\_INTERFACE\_DETAIL\_DATA {  
  DWORD  cbSize; //结构体大小  
  TCHAR  DevicePath\[ANYSIZE\_ARRAY\];  //设备路径  
} SP\_DEVICE\_INTERFACE\_DETAIL\_DATA, \*PSP\_DEVICE\_INTERFACE\_DETAIL\_DATA;  
  
  
\====================================================  
  
  
HidD\_GetHidGuid     
获取 HID 设备的类标识  
VOID  
HidD\_GetHidGuid(  
    OUT LPGUID  HidGuid  
    );  
  
好象 HID 类的 GUID 标识为:   
4d1e55b2-f16f-11cf-88-cb-00-11-11-00-00-30  
  
\===================================================  
  
HidD\_SetFeature     
  
设置特征报告  
  
BOOLEAN  
  HidD\_SetFeature(  
    IN HANDLE  HidDeviceObject,  
    IN PVOID  ReportBuffer,  
    IN ULONG  ReportBufferLength  
    );  
\===============================================  
  
  
HidD\_GetFeature     
  
获取特征报告  
  
BOOLEAN  
  HidD\_GetFeature(  
    IN HANDLE  HidDeviceObject,  
    OUT PVOID  ReportBuffer,  
    IN ULONG  ReportBufferLength  
    );  
  
  
\====================================================  
  
  
HidD\_GetPreparsedData   
返回: 一个指向缓存区指针，该缓存区包含有设备的功能信息  
  
BOOLEAN  
  HidD\_GetPreparsedData(  
    IN HANDLE  HidDeviceObject,  
    OUT PHIDP\_PREPARSED\_DATA  \*PreparsedData  
    );  
  
  
\===============================================  
  
  
HidD\_FreePreparsedData  
和上面一对的。  
BOOLEAN  
  HidD\_FreePreparsedData(  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData  
    );  
  
\==============================================  
  
  
HidP\_SetUsages   
  
设置用法  
  
NTSTATUS \_\_stdcall  
  HidP\_SetUsages(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage,    
    IN USHORT  LinkCollection,   
    IN OUT PUSAGE  UsageList,     
    IN OUT PULONG  UsageLength,   
    IN PHIDP\_PREPARSED\_DATA  PreparsedData,  
    IN OUT PCHAR  Report,  
    IN ULONG  ReportLength  
    );  
  
\==================================================  
  
HidP\_GetUsages  
  
获取用法  
  
NTSTATUS \_\_stdcall  
  HidP\_GetUsages(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage,  
    IN USHORT  LinkCollection,  
    IN OUT USAGE  \*UsageList,  
    IN OUT ULONG  \*UsageLength,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData,  
    IN PCHAR  Report,  
    IN ULONG  ReportLength  
    );  
  
\===========================================  
  
  
  
  
HidP\_GetCaps    
  
获得描述设备能力的结构  
  
NTSTATUS  
  HidP\_GetCaps(  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData,  
    OUT PHIDP\_CAPS  Capabilities  
    );  
  
结构  
typedef struct \_HIDP\_CAPS {  
  USAGE  Usage;  
  USAGE  UsagePage;  
  USHORT  InputReportByteLength;  
  USHORT  OutputReportByteLength;  
  USHORT  FeatureReportByteLength;  
  USHORT  Reserved\[17\];  
  USHORT  NumberLinkCollectionNodes;  
  USHORT  NumberInputButtonCaps;  
  USHORT  NumberInputValueCaps;  
  USHORT  NumberInputDataIndices;  
  USHORT  NumberOutputButtonCaps;  
  USHORT  NumberOutputValueCaps;  
  USHORT  NumberOutputDataIndices;  
  USHORT  NumberFeatureButtonCaps;  
  USHORT  NumberFeatureValueCaps;  
  USHORT  NumberFeatureDataIndices;  
} HIDP\_CAPS, \*PHIDP\_CAPS;  
  
\===============================================  
  
  
HidP\_SetUsageValue   
  
设置一数值  
  
NTSTATUS  
  HidP\_SetUsageValue(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage,  
    IN USHORT  LinkCollection,  
    IN USAGE  Usage,  
    IN ULONG  UsageValue,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData,  
    IN OUT PCHAR  Report,  
    IN ULONG  ReportLength  
    );  
  
\====================================================  
  
HidP\_GetUsageValue   
  
获取一个用法值  
  
NTSTATUS  
  HidP\_GetUsageValue(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage,  
    IN USHORT  LinkCollection,  
    IN USAGE  Usage,  
    OUT PULONG  UsageValue,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData,  
    IN PCHAR  Report,  
    IN ULONG  ReportLength  
    );  
  
\=====================================================  
  
  
HidP\_GetSpecificValueCaps   
  
获取在报告中数值的能力   这个要求能够设置用法页 用法 或连接集合  
  
NTSTATUS  
  HidP\_GetSpecificValueCaps(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage,  
    IN USHORT  LinkCollection,  
    IN USAGE  Usage,  
    OUT PHIDP\_VALUE\_CAPS  ValueCaps,  
    IN OUT PULONG  ValueCapsLength,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData  
    );  
  
\==================================================  
  
HidP\_GetSpecificButtonCaps   
  
获取在报告中按键的能力 这个要求能够设置用法页 用法 或连接集合  
  
NTSTATUS  
  HidP\_GetSpecificButtonCaps(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage,  
    IN USHORT  LinkCollection,  
    IN USAGE  Usage,  
    OUT PHIDP\_BUTTON\_CAPS  ButtonCaps,  
    IN OUT PULONG  ButtonCapsLength,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData  
    );  
  
\===================================================  
  
HidP\_MaxUsageListLength    
  
获得报告中能够获取的用法最大数值  
  
ULONG  
  HidP\_MaxUsageListLength(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage  OPTIONAL,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData  
    );  
  
\=====================================================  
  
HidD\_GetAttributes   
  
获取 USB 设备的有关属性。它包含了设备的厂商 ID、产品 ID 及产品的版本号等。可以根据这些信息判断该设备是否为目标设备  
  
BOOLEAN  
  HidD\_GetAttributes(  
    IN HANDLE  HidDeviceObject,  
    OUT PHIDD\_ATTRIBUTES  Attributes  
    );  
  
  
结构  
typedef struct \_HIDD\_ATTRIBUTES {  
  ULONG  Size;  
  USHORT  VendorID;  
  USHORT  ProductID;  
  USHORT  VersionNumber;  
} HIDD\_ATTRIBUTES, \*PHIDD\_ATTRIBUTES;  
  
\===================================================  
  
HidP\_SetScaledUsageValue    
  
采用一个有符号且实际比例的数值  将其转换成设备所使用的逻辑表示式 并将其插入报告中  
  
NTSTATUS  
  HidP\_SetScaledUsageValue(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage,   
    IN USHORT  LinkCollection  OPTIONAL,  
    IN USAGE  Usage,  
    IN LONG  UsageValue,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData,  
    IN OUT PCHAR  Report,  
    IN ULONG  ReportLength  
    );  
  
\===================================================  
  
HidP\_GetScaledUsageValue   
  
获取一个以其比例因子所调整后的有符号数值  
  
NTSTATUS  
  HidP\_GetScaledUsageValue(  
    IN HIDP\_REPORT\_TYPE  ReportType,  
    IN USAGE  UsagePage, 用法页  
    IN USHORT  LinkCollection  OPTIONAL,  
    IN USAGE  Usage,用法  
    OUT PLONG  UsageValue,  
    IN PHIDP\_PREPARSED\_DATA  PreparsedData,  
    IN PCHAR  Report,  
    IN ULONG  ReportLength  
    ); 

本楼内容为应用程序枚举 HID 设备进行找狗。  
  
  
  
在一楼讲了主机对 HID 设备的枚举，成功完成后 HID 设备已经能正常使用了。   
但是对于应用程序来说，它还需要通过 枚举 来查找在所有的 HID 设备中是否有它需要的 HID 设备。  
  
  
既然是规范性的东东，那么就应该尽量先从理论上大概的了解一下，这样才能有的放矢，避免浪费精神体力。对于应用程序对加密狗的寻找，当然是要了解程序枚举 HID 设备的流程了。  
  
HidD\_GetHidGuid 获取 HID 设备的 GUID  
setupdigetclassdevs 获取 HID 设备类信息  
setupdienumdeviceinterfaces 枚举设备接口  
setupdigetinterfacedevicedetail 获取设备路径信息  
createfile 打开设备  
hidd\_attributes 获取设备详细信息  
判断是否为需要的设备  
是则成功，否则关闭设备重新去枚举接口。  
  
SetupDiDestroyDeviceInfoList 是释放 setupdigetclassdevs 获取的 HID 类设备集合句柄，所以可以把这个函数当成是应用程序枚举 HID 设备结束的标志。为了验证，可以在 SetupDiDestroyDeviceInfoList 上设断点，F9，  
假如是有狗，那么只会在这儿中断一次，  
假如是无狗，那么就会中断 FF 次，我猜想中断一次就可以知道有无狗了，又重复那么多次，大概是保险起见又多枚举整个集合 FE 次吧。  
因为即使有多个 HID 设备，重新枚举接口并不需要释放集合句柄从头来过。释放集合句柄表示已经把该集合内的所有 HID 设备都枚举过了。  
  
  
  
  
SetupDiDestroyDeviceInfoList 以后，只是表示有无枚举到锁，还不代表找到正确的锁。  
  
紧接着，会调用 10 号控制命令，把软件的读密码发送给狗，返回狗的读密码，由此判断该狗是否是和软件配套的正确的狗。  
  
枚举狗---10 号命令判断读密码---4 号功能解密运算  
  
本楼内容只是想讲程序对狗的枚举。  
  
  
  
  
  
  
如果可以仔细看看加密锁的 SDK 和文档，并且根据文档提供的各种例子去分析一下，那就事半功倍了。  
不过我基本上没看，所以在分析时特别吃力。  
  
  
  
打开 bus hound 便于随时查看。  
然后 OD 载入程序，HidD\_GetHidGuid 处下断，F9，中断在 HID 中，  
ALT+F9 返回主程序，然后在 HidD\_GetHidGuid 调用的上一句 F2，重新运行  
  
00408E4B  |> \\8D4424 24     LEA EAX,DWORD PTR SS:\[ESP+24\]  
00408E4F  |.  50            PUSH EAX  
00408E50  |.  E8 AB0D0000   CALL <JMP.&HID.HidD\_GetHidGuid>  
00408E55  |.  8B7C24 5C     MOV EDI,DWORD PTR SS:\[ESP+5C\]  
  
HidD\_GetHidGuid 函数请参考 3 楼。  
  
这个函数只有一个参数，就是提供个缓冲区指针给函数 用来返回  
GUID。  我这里是 CBF940，位于堆栈中。F8 过 CALL 以后查看 CBF940，内容为 4D1E55B211CFF16F1100CB8830000011，这个高低字节要注意下，  这串就是 HID 设备的 GUID 了。这就是应用程序枚举狗的开始。  
  
00408E55  |.  8B7C24 5C     MOV EDI,DWORD PTR SS:\[ESP+5C\]  
00408E59  |.  6A 12         PUSH 12  
00408E5B  |.  56            PUSH ESI  
00408E5C  |.  8D4C24 2C     LEA ECX,DWORD PTR SS:\[ESP+2C\]  
00408E60  |.  56            PUSH ESI  
00408E61  |.  51            PUSH ECX  
00408E62  |.  8937          MOV DWORD PTR DS:\[EDI\],ESI  
00408E64  |.  FF15 C0114000 CALL DWORD PTR DS:\<&SETUPAPI.SetupDiGet>;  SETUPAPI.SetupDiGetClassDevsA  
00408E6A  |.  8BE8          MOV EBP,EAX  
  
SetupDiGetClassDevsA 查询所有已安装的 HID 设备，得到一个指向该 HID 设备集合的句柄，用到上面得到的 GUID。  
  
  
  
\================================  
  
这几天没时间写东东。。。。。。部分更新  待编辑。。。。。。 

本楼内容为确定 HID 设备的控制命令 即本加密狗的内置命令码 详细分析过程整理后编辑  
  
  
  
这个获得控制命令。。。真是让人好好害羞。。。因为。。。  
这完全就是赤果果的纯休闲活动。  
  
我根据读出的设备描述符信息，下载了狗的配套开发工具，挨个命令运行一遍，并且找了开发工具的不同版本，都大概试了一下，全部 OK。。。。。。  
  
虽然这种方法很取巧，但是。。。也很海皮。  
如果不愿意，还可以用分析的方式来取得控制命令，不过应该会很累的。  
  
另外 我不知道这样的控制命令商家是自己留存 还是会以某种方式提供给开发用户？  
  
\========================================================  
  
我尽量想以有依据的格式化的形式来解读数据，而不是以一般的统计规律的形式来分析。 也许这样会有些困难。。。但以学习的目的来说 还是要知其所以然。  
  
  
  
回头看一下我在一楼分析出的报告描述符  
  
USAGE\_PAGE (Vendor Defined Page 1)  06 00 FF  
USAGE (Vendor Usage 1)      09 01  
COLLECTION (Application)    A1 01   
  REPORT\_ID (1)        85 01   
  USAGE (Vendor Usage 1)    09 01  
  LOGICAL\_MINIMUM (0)      15 00   
  LOGICAL\_MAXIMUM (255)      26 FF 00   
  REPORT\_SIZE (8)      75 08   
  REPORT\_COUNT (8)      95 08   
  FEATURE (Data,Var,Rel)    B1 06   
  REPORT\_ID (2)        85 02   
  USAGE (Undefined)      09 03  
  REPORT\_COUNT (276)      96 14 01   
  FEATURE (Data,Var,Rel)    B1 06   
END\_COLLECTION  C0  
  
  
  
可知这个自定义的 HID 狗有 2 种报告，报告 ID1 只有 8 个字节，显然不足以容纳什么，而报告 ID2  
则要大的多，根据 HID 规范，凡是有报告 ID 的，都必须在报告的  
最前面用一个字节来表示报告 ID。所以在 BUSHOUND 中，我们看到  
主机发送给设备的报告最前一个字节都是 02，这表示是 ID2 的报告。  
\---------------------------------------------------  
ID2 的 REPORT\_COUNT 字段我一直没搞明白，95 后可跟到 FF，96 以后就会增加一个字节，但增加的不可能是 00，至少要为 01，用来表示计数的最高位。 而从这个狗的描述符中读到的是 96 14 00，无论如何都反不出来到底是什么。。。。。。 所以我就把它当成 96 14 01 了。如有高手知道，还请指点。这个地方我十分纠结。  
\------------------------------------------------------  
  
最前一个字节为报告 ID，紧跟着的就是厂家自定义用法了，在这里就可以当成是控制命令，根据这种粗略的报告描述符无法确定厂家定义的控制命令----在 HID 规范中 应该称为 USAGE 用法---到底位于报告的哪个位置，但是猜想因为报告中也一定会有其他数据，数据不定长，控制命令除非放在最前面或者是有特殊前缀或后缀来区分，否则不易和其他数据分开，使用前后缀会浪费字节，而放在最前面是最好的选择了。。。。。。即紧跟着报告 ID 的就是控制命令。  
  
  
后面将贴出分析过程，因为那 2 天光记录就搞了 N 多，发出来的话就要仔细核对，需要点时间。  
  
这里先给出我所能分析出的所有厂家自定义用法，不一定全，也许还有其他用法。。。 仅供参考。   
  
01 锁版本   
02 锁 ID  
03 普通加密  
04 普通解密  
05 锁扩展版本  
06 设定算法密钥  
07 ？我向设备发送这个控制，有功能，但是分析不出具体用途。  
08 增强算法加密  
09 设定增强算法密钥  
10 这个控制很有爱，它会将调用者的读密码发送给设备，而设备会将设备自己的读密码隐藏一字节后回传。于是。。。完整的应用程序锁壳读密码 隐藏一字节的锁内读密码都亮相了，这个用法的设计很威武。  
11 设定密码  
12 读字符串  
13 写字符串  
  
\==================================================  
分析过程 待编辑。 

刚开始的主机枚举设备描述符中就有用来识别设备的厂家信息，11 条 USB 标准命令是所有 USB 设备都必须支持的，作为设备型号的识别，一个最简单的实现方法就是写代码发送获取设备描述符的命令，然后和自己搜集的厂家型号代码对比。我个人猜想，如果可以这样实现，那么插上就可识别，唯一的问题就是你能搜集到的加密锁的具体型号数量了。  
  
前几天调试时只记录了文字，OD 调试没有做记录，只好重新弄了，所以带有 OD 具体调试内容的部分弄好才能发了。  
现在我是真正感受到------------- 发帖子是多么辛苦的一件事了，而发这样的帖子，更是辛苦![