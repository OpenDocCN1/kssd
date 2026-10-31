
# [原创]SENSE4加密锁浅析之基础篇

【文章标题】: SENSE4加密锁浅析之基础篇  
【文章作者】: yaoyuan[CCG]  
【下载地址】: 自己搜索下载  
【保护方式】: SENSE4加密锁  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

关于SENSE4加密锁，论坛上资料还比较少，简单写一下这个锁的破解思路，欢迎一起交流。

SENSEIV锁通常的调用方式如下：

### 第一步：枚举设备，分配空间

```
DWORD WINAPI S4Enum
(
    OUT     SENSE4_CONTEXT   *pS4CtxList,
    IN OUT  DWORD            *pdwCtxListSize
);
```

枚举系统连接了多少个设备，这一步不用判断返回值，第一个参数输入`NULL`，如果得到的设备上下文内存数量为0或者不为`sizeof(SENSE4_CONTEXT)`的倍数，说明没有找到设备或者列举设备出错。根据返回的设备个数，分配空间给程序使用。

```
s4CtxList = (SENSE4_CONTEXT *)malloc(dwSize);
```

`s4CtxList` 结构如下：

```
typedef struct
{
    DWORD                   dwIndex;                                        /** device index; index begins at zero*/
    DWORD                   dwVersion;                                      /** device version*/
    S4HANDLE                hLock;                                          /** device handle*/
    BYTE                    reserve[12];                                    /** reserved*/
    BYTE                    bAtr[MAX_ATR_LEN];                              /** ATR*/
    BYTE                    bID[MAX_ID_LEN];                                /** device ID*/
    DWORD                   dwAtrLen;                                       /** ATR length*/
}SENSE4_CONTEXT,*PSENSE4_CONTEXT;
```

### 第二步：再次枚举设备，连接设备

```
//根据分配的空间，再次枚举设备，这次要判断函数返回值。
S4Enum(s4CtxList,&dwSize)
S4OpenEx(&s4CtxList[0], &S4_OpenInfo)
```

`S4_OpenInfo` 结构如下：

```
typedef struct _S4OPENINFO
{
    DWORD dwS4OpenInfoSize;                                                 /* size of the structure*/
    DWORD dwShareMode;                                                      /* share mode*/
} S4OPENINFO;
```

### 第三步：获取设备信息

```
DWORD WINAPI S4Control
(
    IN      SENSE4_CONTEXT    *pS4Ctx,
    IN      DWORD             dwCtlCode,
    IN      VOID              *pInBuffer,
    IN      DWORD             dwInBufferLen,
    OUT     VOID              *pOutBuffer,
    IN      DWORD             dwOutBufferLen,
    OUT     DWORD             *pdwBytesReturned
);
```

```
#define S4_LED_UP                               0x00000004      /** LED up*/
#define S4_LED_DOWN                             0x00000008      /** LED down*/
#define S4_LED_WINK                             0x00000028      /** LED wink*/
#define S4_GET_DEVICE_TYPE                      0x00000025      /** get the device type*/
#define S4_GET_SERIAL_NUMBER                    0X00000026      /** get the device serial number*/
#define S4_GET_VM_TYPE                          0X00000027      /** get the virtual machine type*/
#define S4_GET_DEVICE_USABLE_SPACE              0x00000029      /** get the total space of the device*/
#define S4_SET_DEVICE_ID                        0x0000002a      /** set the device ID*/
#define S4_RESET_DEVICE                         0x00000002      /** reset the device*/
#define S4_DF_AVAILABLE_SPACE                   0x00000031      /** get the free space of current directory*/
#define S4_EF_INFO                              0x00000032      /** get specified file information in current directory*/
#define S4_SET_USB_MODE                         0x00000041      /** set the device as a normal usb device*/
#define S4_SET_HID_MODE                         0x00000042      /** set the device as a HID device*/
#define S4_GET_CUSTOMER_NAME                    0x0000002b      /** get the customer number*/
#define S4_GET_MANUFACTURE_DATE                 0x0000002c      /** get the manufacture date of the device*/
#define S4_GET_CURRENT_TIME                     0x0000002d      /** get the current time of the clock device*/
```

### 第四步：更改目录，校验用户pin码，执行锁内程序得到返回值

```
DWORD WINAPI S4ChangeDir
(
    IN      SENSE4_CONTEXT     *pS4Ctx,
    IN      LPCSTR             lpszPath
);

DWORD WINAPI S4VerifyPin
(
    IN      SENSE4_CONTEXT     *pS4Ctx,
    IN      BYTE               *pbPin,
    IN      DWORD              dwPinLen,
    IN      DWORD              dwPinType
);

DWORD WINAPI S4ExecuteEx
(
    IN      SENSE4_CONTEXT     *pS4Ctx,
    IN      LPCSTR             lpszFileID,
    IN      DWORD              dwFlag,
    IN      VOID               *pInBuffer,
    IN      DWORD              dwInbufferSize,
    OUT     VOID               *pOutBuffer,
    IN      DWORD              dwOutBufferSize,
    OUT     DWORD              *pdwBytesReturned
);
```

### 最后处理返回值，关闭设备

```
DWORD WINAPI S4Close
(
    IN      SENSE4_CONTEXT    *pS4Ctx
);
```

实战篇中我们将以某国产软件的模块验证程序`Products.exe`为例来分析具体加密过程及处理方法。

---

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007年01月29日 11:32:39

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
