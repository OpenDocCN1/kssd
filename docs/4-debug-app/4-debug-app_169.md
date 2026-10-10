

# QQ 辅助工具源代码

> 原文：[`www.pediy.com/kssd/pediy12/142072.html`](https://www.pediy.com/kssd/pediy12/142072.html)

心灰意冷了……大家共同学习。  
部分代码有 bug，慎用。另外只提供了和 QQ 有关的代码，不全的代码找个 dll 自己逆向吧。  
  
节选部分代码：  
  
在框内写消息  

代码:

```
//这个还是在看雪上面看见的
typedef struct CTXBSTR
{
  int zero;//貌似要一直是 0
  int count;//引用计数
  int len1;
  int len2;
  wchar\_t str\[4100\];
} TXStr, \* pTXStr;
void QQMsgInFrame(unsigned long QQUIN,LPCTSTR lpText,int save)
{
  HWND hWnd = Util::ChatSession::GetContactChatSessionMainHWnd(QQUIN);
  if(!IsWindow(hWnd) || !g\_hwnd\_inner) return;    //避免无效窗口

  TXStr MyMsg;
  MyMsg.zero = 0;
  MyMsg.count = 3;
  wcsncpy(MyMsg.str, lpText,4096);
  MyMsg.str\[4096\]=0;
  MyMsg.len1 = MyMsg.len2 = wcslen(MyMsg.str) \* sizeof(wchar\_t);

  if(MyMsg.len1<=0) return;//不显示空文本

  if(WriteMsgTip&&CreateChatFrame)
  {
    DWORD pointer = 0;//0x11111111;
    CreateChatFrame(QQUIN, 0, &pointer, 0);
    WriteMsgTip(pointer, 0, MyMsg.str, 0);
  }
  else
  {
    if(WriteMsgTip2)
    {
      WriteMsgTip2(QQUIN, 0, 0, MyMsg.str,0);
    }
  }
}

```

去掉所有右下角弹窗  
代码:

```
long \_\_cdecl MyRawCreateGFElementByXtml(wchar\_t  \*p1,DWORD p2,DWORD p3,DWORD p4)
{
  if(wndf && oneminits && wcsstr(p1,L"PopupTip.xml")!=0) return 0;

  if(RawCreateGFElementByXtml) return RawCreateGFElementByXtml(p1,p2,p3,p4);
  return 0;
}

```

  
一些用到的函数原型  
代码:

```
#ifndef \_\_QQDEF\_H\_\_
#define \_\_QQDEF\_H\_\_


class CTXStringW;
struct ITXMsgPack;

#pragma comment(lib, "lib\\\\KernelUtil.lib")
#pragma comment(lib, "lib\\\\AppUtil.lib")
#pragma comment(lib, "lib\\\\Common.lib")

#define DLLIMPORT \_\_declspec (dllimport)

namespace Registry
{
    namespace API
    {
        DLLIMPORT int \_\_cdecl GetRegSubKeyBoolField(wchar\_t \*,wchar\_t \*,wchar\_t \*,int \*,enum \_\_MIDL\_\_\_MIDL\_itf\_IRegistry\_0000\_0003,enum \_\_MIDL\_\_\_MIDL\_itf\_IRegistry\_0000\_0004);
    }
}
namespace Util
{
    namespace ChatSession
    {
        //DLLIMPORT int \_\_cdecl CreateChatFrameType(unsigned long,int,struct IAFChatFrameType \* \*,struct ITXData \*);
        //DLLIMPORT void \_\_cdecl WriteMsgTipInChatSession(unsigned long,int,enum IconIndex,class CTXStringW,int);
        DLLIMPORT long \_\_cdecl SendAutoReplyMsgToBuddy(unsigned long);
        DLLIMPORT void \_\_cdecl OpenContactChatSession(unsigned long,struct ITXData \*);
        DLLIMPORT int \_\_cdecl IsChatSessionOpen(unsigned long,int);
    }
    namespace Misc
    {
        DLLIMPORT void \_\_cdecl SwitchUser(void);
        DLLIMPORT int \_\_cdecl IsServerControlBitOn(unsigned long);
        DLLIMPORT int \_\_cdecl IsTencentTrusted(wchar\_t const \*);
        //DLLIMPORT int \_\_cdecl MessageBoxExByHwnd(struct HWND\_\_ \*,wchar\_t \*,wchar\_t \*,unsigned int,struct ITXData \*,struct ITXData \* \*);
        //DLLIMPORT int \_\_cdecl MessageBoxExByHwnd(struct HWND\_\_ \*,wchar\_t \*,wchar\_t \*,unsigned int,struct ITXData \*,struct ITXData \* \*);
    }
    namespace AFChatSession
    {
        DLLIMPORT void \_\_cdecl WriteMsgTipInChatSession(struct IAFChatFrameType \*,enum IconIndex,class CTXStringW,int);
    }
    namespace Contact
    {
        DLLIMPORT int \_\_cdecl IsFlagValid(unsigned long,unsigned long);
        DLLIMPORT unsigned long \_\_cdecl GetSelfUin(void);
        DLLIMPORT unsigned short \_\_cdecl GetIMVersion(unsigned long);
        DLLIMPORT unsigned short \_\_cdecl GetStatus(unsigned long);
        DLLIMPORT class CTXStringW \_\_cdecl GetNickname(unsigned long);
        DLLIMPORT class CTXStringW \_\_cdecl GetPublicName(unsigned long);
        
        namespace Self
        {
            DLLIMPORT int \_\_cdecl GetAutoRelyContent(class CTXStringW &);
        }
    }
    namespace ChatSession
    {
        DLLIMPORT struct HWND\_\_ \* \_\_cdecl GetContactChatSessionMainHWnd(unsigned long);
    }
    namespace Group
    {
        DLLIMPORT int \_\_cdecl CheckMsgImage(struct ITXMsgPack \*,class CTXStringW &);
    }
    namespace Msg
    {
        DLLIMPORT class CTXStringW \_\_cdecl GetMsgAbstract(struct ITXMsgPack \*);
    }
    namespace Convert
    {
        DLLIMPORT int \_\_cdecl ConvertXMLStrToTXData(wchar\_t \*,wchar\_t \* \*,struct ITXData \*,wchar\_t \*);
    }
    namespace Boot
    {
        DLLIMPORT int \_\_cdecl InitPluginCoreConfig(wchar\_t \*);
    }
    namespace Com
    {
        DLLIMPORT long \_\_stdcall CreateObjectFromDllFile(wchar\_t const \*,struct \_GUID const &,struct \_GUID const &,void \* \*,struct IUnknown \*);
    }
    namespace CoreCenter
    {
        DLLIMPORT int \_\_cdecl EnablePlugin(wchar\_t \*);
        DLLIMPORT int \_\_cdecl DisablePlugin(wchar\_t \*);
    }
}

namespace Version
{
    DLLIMPORT unsigned long \_\_cdecl GetBuildVer(void);
    DLLIMPORT int \_\_cdecl Init(void);
}

namespace FS
{
    DLLIMPORT int \_\_cdecl IsFileExist(wchar\_t const \*);
}

//DLLIMPORT void \_\_cdecl TXLog\_DoTXLogVW(struct tagLogObj \*,wchar\_t const \*,wchar\_t const \*,char \*);

//typedef unsigned long (WINAPI \*GetAccountName)(struct ITXMsgPack \*,unsigned long);
//GetAccountName GetNickname = NULL;
//GetAccountName GetPublicName = NULL;

//typedef unsigned long (WINAPI \*MsgAbstract)(struct ITXMsgPack \*,struct ITXMsgPack \*);
//MsgAbstract GetMsgAbstract = NULL;


typedef void (WINAPI \*MyGetAccountName)(const wchar\_t \*\*,unsigned long);
#define GetNickname ((MyGetAccountName)Util::Contact::GetNickname)
#define GetPublicName ((MyGetAccountName)Util::Contact::GetPublicName)
#define GetMsgAbstract ((MyGetAccountName)Util::Msg::GetMsgAbstract)

#endif // \_\_QQDEF\_H\_\_

```

下载源代码：

上传的附件
| ![](img/31a2c422ae120fd0b6c241a4e6e83632.jpg) || CWUB2\_SRC.7z  
 |