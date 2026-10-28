
34fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6R3@1y4K6b7#2y4K6m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0V1K6x3K6x3&6z5o6j5I4

![](https://bbs.kanxue.com/upload/attach/201907/833057_95YPEKN7NHR96BH.png)

这里直接给出偏移 2.6.8.52 版本的接收消息的偏移是 `0x315E98`

![](https://bbs.kanxue.com/upload/attach/201907/833057_TZYEPPTU72DXVHA.png)

直接来到接收消息的地方，在这里下断点，然后用小号分享一个名片，让程序断下

![](https://bbs.kanxue.com/upload/attach/201907/833057_UUGZ3KDKQ3JS4VD.png)

程序断下后，我们查看 `[[esp]]` 里面的内容，这里面保存了接收到的消息参数，往下拉

![](https://bbs.kanxue.com/upload/attach/201907/833057_YT75Z2PE57Y6PTU.png)

这里有消息发送者的微信ID和xml格式的消息内容

![](https://bbs.kanxue.com/upload/attach/201907/833057_5VBPVW8CUKPX387.png)

数据窗口跟随进去，里面有我们需要的V1数据，至于其他的数据我们并不关心。OK，分析到这里，自动添加好友分享的名片基本也就完成了。接下来贴代码

# 代码实现自动添加好友分享名片

## 第一步 取出V1数据 我这里用的是字符串查找和匹配的方法，你们可以xml解析库来解析数据

```
void AutoAddCardUser(wstring msg)

{

    //拿到V1

    int v1strat = msg.find(L"v1_");

    int v1end = msg.find(L"@stranger");

    wstring v1;

    v1 = msg.substr(v1strat, v1end - v1strat + 9);



    //调用添加名片好友函数

    AddCardUser((wchar_t*)v1.c_str(), (wchar_t*)L"快通过~快通过~ 吼吼！");

}

```

## 第二步 调用call添加好友

```
void AddCardUser(wchar_t* v1, wchar_t* msg)

{

    DWORD dwWeChatWinAddr = (DWORD)GetModuleHandle(L"WeChatWin.dll");

    DWORD dwParam1 = dwWeChatWinAddr + WxAddWxUserParam1;

    DWORD dwCall1 = dwWeChatWinAddr + WxAddWxUserCall1;

    DWORD dwCall2 = dwWeChatWinAddr + WxAddWxUserCall2;

    DWORD dwCall3 = dwWeChatWinAddr + WxAddWxUserCall3;

    DWORD dwCall4 = dwWeChatWinAddr + WxAddWxUserCall4;

    DWORD dwCall5 = dwWeChatWinAddr + WxAddWxUserCall5;



    struct TextStruct

    {

        wchar_t* pStr;

        int strLen;

        int strMaxLen;



    };



    TextStruct pV1 = { 0 };

    pV1.pStr = v1;

    pV1.strLen = wcslen(v1) + 1;

    pV1.strMaxLen = (wcslen(v1) + 1) * 2;





    char* asmV1 = (char*)&pV1.pStr;

    char buff3[0x100] = { 0 };

    char* buff = buff3;

    __asm

    {

        sub esp, 0x18;

        mov ecx, esp;

        mov dword ptr ss : [ebp - 0xDC], esp;

        push  dwParam1;

        call dwCall1;

        sub esp, 0x18;

        mov eax, buff;

        mov dword ptr ss : [ebp - 0xE4], esp;

        mov ecx, esp;

        push eax;

        call dwCall2;

        push 0x11;

        sub esp, 0x14;

        mov ecx, esp;

        mov dword ptr ss : [ebp - 0xE8], esp;

        push - 0x1;

        mov edi, msg;

        push edi;

        call dwCall3;

        push 0x2;

        sub esp, 0x14;

        mov ecx, esp;

        mov dword ptr ss : [ebp - 0xE0], esp;

        mov ebx, asmV1;

        push ebx;

        call dwCall4;

        mov ecx, eax;

        call dwCall5;

    }

}

```

# 拓展

## 实现自动聊天思路

首先在好友消息的时候保存一下好友的微信ID，然后调用获取到好友的消息内容，并且将消息内容转发给图灵机器人的公众号

![](https://bbs.kanxue.com/upload/attach/201907/833057_JZ7W8BW2WGJRXQG.png)

接着拿到图灵机器人回复的内容，将内容转发给好友

![](https://bbs.kanxue.com/upload/attach/201907/833057_CFS2UYYK4GUVCBQ.png)

这样就实现了自动聊天

## 实现自动同意好友请求思路

我们在接收消息的地方下个断点，让好友发送请求时断下，并且查看 `[[esp]]` 里的消息内容

![](https://bbs.kanxue.com/upload/attach/201907/833057_7XXWVWV66T7HYS5.png)

这里保存有两个重要的数据，一个是V1，一个是V2，我们只要在这个地方取出V1和V2，然后调用同意好友请求的call，就能实现自动同意好友请求

同意好友请求的call，我这里直接给出偏移 微信2.6.8.52版本

```
#define WxAgreeUserRequestCall1 0x1865B0;      //同意好友请求

#define WxAgreeUserRequestCall2 0x4F4F0;      //同意好友请求

#define WxAgreeUserRequestCall3 0xCE4F0;      //同意好友请求

#define WxAgreeUserRequestCall4 0x16BD40;      //同意好友请求

#define WxAgreeUserRequestParam 0x126E050;      //同意好友请求

```

这里需要同时调用4个call，接着附上代码

```
//取出V1和V2

void AutoAgreeUserRequest(wstring msg)

{

    int v1strat = msg.find(L"v1_");

    int v1end = msg.find(L"@stranger");

    wstring v1;

    v1 = msg.substr(v1strat, v1end - v1strat + 9);

    //找到v2

    int v2strat = msg.find(L"v2_");

    int v2end = msg.rfind(L"@stranger");

    wstring v2;

    v2 = msg.substr(v2strat, v2end - v2strat + 9);

    //调用同意好友请求的call

    AgreeUserRequest((wchar_t*)v1.c_str(), (wchar_t*)v2.c_str());

}

```

```
//调用同意好友请求call

void AgreeUserRequest(wchar_t* v1, wchar_t* v2)

{

    struct v1Info

    {

        int fill = 0;

        wchar_t* v1 = 0;

        int v1Len;

        int maxV1Len;

        char fill2[0x41C] = { 0 };

        DWORD v2 = { 0 };

    };



    struct v2Info

    {

        char fill[0x24C] = { 0 };

        DWORD fill3 = 0x25;

        char fill4[0x40] = { 0 };

        wchar_t* v2;

        int v2Len;

        int maxV2Len;

        char fill2[0x8] = { 0 };

    };



    DWORD base = (DWORD)LoadLibrary(L"WeChatWin.dll");

    DWORD callAdd1 = base + WxAgreeUserRequestCall1;

    DWORD callAdd2 = base + WxAgreeUserRequestCall2;

    DWORD callAdd3 = base + WxAgreeUserRequestCall3;

    DWORD callAdd4 = base + WxAgreeUserRequestCall4;

    DWORD params = base + 0x126E050;



    DWORD* asmP = (DWORD*)params;



    v1Info userInfoV1 = { 0 };

    v2Info userInfoV2 = { 0 };

    userInfoV1.v2 = (DWORD)&userInfoV2.fill;

    userInfoV1.v1 = v1;

    userInfoV1.v1Len = wcslen(v1);

    userInfoV1.maxV1Len = wcslen(v1) * 2;

    userInfoV2.v2 = v2;

    userInfoV2.v2Len = wcslen(v2);

    userInfoV2.maxV2Len = wcslen(v2) * 2;



    char* asmUser = (char*)&userInfoV1.fill;

    char buff[0x14] = { 0 };

    char buff2[0x48] = { 0 };

    char* asmBuff = buff2;



    __asm

    {

        mov ecx, asmUser;

        push 0x6;

        sub esp, 0x14;

        push esp;

        call callAdd1;

        mov ecx, asmUser;

        lea eax, buff;

        push eax;

        call callAdd2;

        mov esi, eax;

        sub esp, 0x8;

        mov ecx, asmP;

        call callAdd3;

        mov ecx, asmBuff;

        mov edx, ecx;

        push edx;

        push eax;

        push esi;

        call callAdd4;

    }



}

```

## 实现自动收款思路

同样在接收消息的地方下个断点，接收转账消息，并且查看 `[[esp]]` 里的消息内容

![](https://bbs.kanxue.com/upload/attach/201907/833057_CR4UEXRGEQB24BQ.png)

这里重要的数据只有一个，就是这个 `transferid`，拿到这个转账ID和微信ID，然后调用收款的call，就能实现自动收款了

这里提供收款call的偏移，微信2.6.8.52版本

```
#define WxCllectMoneyCall1 0x676B10        //收款

#define WxCllectMoneyCall2 0x676B90        //收款

```

接着附上代码

```
//取出转账ID

void AutoCllectMoney(wstring msg,wchar_t* wxid)

{

    // 找到<transferid>字符串的位置

    int pos1 = msg.find(L"<transferid>");

    //找到]]></transferid>字符串的位置

    int pos2 = msg.find(L"]]></transferid>");

    //取出多余的字符串长度

    wstring noneed = L"<transferid><![CDATA[";

    int noneedLen = noneed.length();

    //取出转账ID

    wstring transferid;

    transferid = msg.substr(pos1 + noneedLen, (pos2 - pos1) - noneedLen);



    //调用收款call 实现自动收款

    CllectMoney((wchar_t*)transferid.c_str(), wxid);

}

```

```
//调用收款call

void CllectMoney(wchar_t* transferid, wchar_t* wxid)

{

    struct CllectMoneyStruct

    {

        wchar_t* ptransferid;

        int transferidLen;

        int transferidMaxLen;

        char full[0x8] = { 0 };

        wchar_t* pwxid;

        int wxidLen;

        int wxidMaxLen;

        char full2[0x8] = { 0 };

    };



    CllectMoneyStruct cllect;

    cllect.ptransferid = transferid;

    cllect.transferidLen = wcslen(transferid) + 1;

    cllect.transferidMaxLen = (wcslen(transferid) + 1) * 2;

    cllect.pwxid = wxid;

    cllect.wxidLen = wcslen(wxid) + 1;

    cllect.wxidMaxLen = (wcslen(wxid) + 1) * 2;



    char* asmBuff = (char*)&cllect.ptransferid;



    DWORD dwWeChatWinAddr = (DWORD)GetModuleHandle(L"WeChatWin.dll");

    DWORD dwCall1 = dwWeChatWinAddr + WxCllectMoneyCall1;

    DWORD dwCall2 = dwWeChatWinAddr + WxCllectMoneyCall2;





    __asm

    {

        sub esp, 0x30;

        mov ecx, esp;

        mov eax, asmBuff;

        push eax;

        call dwCall1;

        call dwCall2;

        add esp, 0x30;

    }

}

```

最后附上Github地址，还请亲们帮忙点个star

bffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1L8$3&6&6b7$3S2W2L8U0f1$3i4K6u0r3g2$3g2o6K9r3q4@1f1X3!0T1L8%4b7`.
