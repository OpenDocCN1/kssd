
## 代码实现

```cpp
OnKeepLive(pSE, pPIEX);  

}  

break;  

//  接收验证 KEY  

case 0x0007:  

{  

    memcpy(pSE->byRecvKEY, (PBYTE)pPIEX, 0x10);  

    //  发送加解密 KEY 给游戏  

    OnSendKEY(pSE);  

}  

break;  

//  只需要解密  

case 0x0006:  

{  

    ((VOID (\*)(PBYTE, int, PBYTE, int))ULongToPtr(pSE->pEnDeCode))((BYTE \*)&pPIEX->dwPackCheck, pPIEX->wSize - 4,  pSE->byRecvKEY, 0x10);  

    ChangeKEY(pSE->byRecvKEY);  

    OnRecv(pSE, (PPACK_INFO)&pPIEX->wLen);  

}  

break;  

//  需要解密并解压缩  

case 0x000D:  

{  

    ((VOID (\*)(PBYTE, int, PBYTE, int))ULongToPtr(pSE->pEnDeCode))((BYTE \*)&pPIEX->dwPackCheck, pPIEX->wSize - 4,  pSE->byRecvKEY, 0x10);  

    PBYTE pbyBuffer = new BYTE\[pPIEX->wLen\];  

    uncompress(pbyBuffer, (ULONG \*)&pPIEX->wLen, (LPVOID)&pPIEX->wHead, pPIEX->wSize - 0xA);  

    ChangeKEY(pSE->byRecvKEY);  

    OnRecv(pSE, (PPACK_INFO)pbyBuffer);  

    delete pbyBuffer;  

}  

break;  

}  

}  

//  获得进入游戏 ID  

VOID CGameRecv::OnGetEnterID(PSOCKET_ENGINE pSE, PPACK_INFO pPI)  

{  

    m_dwEnterID1 = \*(PDWORD)(&pPI->byBuffer\[8\]);  

    m_dwEnterID2 = \*(PDWORD)(&pPI->byBuffer\[0\]);  

}  

VOID CGameRecv::OnRecv(PSOCKET_ENGINE pSE, PPACK_INFO pPI)  

{  

    //ClientMain->PackPrint("接收", (PBYTE)pPI, pPI->wLen);  

    switch (pPI->wHead)  

    {  

        //  登录状态处理  

        case 0x0202:  

            //  数据开始 作为标识  

            //  C 代表线路堵塞？  

            //  D 代表版本好错误  

            //  2 代表密码错误  

            //  6 代表人物还在游戏中  

        break;  

        //  发送帐号进入消息  

        case 0x0206:  

            OnSendAccount(pSE);  

        break;  

        //  选择线路  

        case 0x0205:  

            OnSelectLine(pSE);  

        break;  

        //  开始游戏  

        case 0x0208:  

            OnGetEnterID(pSE, pPI);  

        break;  

        //  进入游戏  

        case 0x0302:  

            OnEnterGame(pSE);  

        break;  

        //  开始游戏  

        case 0x0307:  

            OnStartGame(pSE, pPI);  

        break;  

        //  开始游戏成功  

        case 0x0303:  

        break;  

        //  初始化人物数据  

        case 0x0310:  

        break;  

        //  自身或周围玩家信息  

        case 0x1501:  

        break;  

    }  

}
```

## 具体登录流程

连接服务器 -> 接收服务器发来的解密 `KEY` -> 本地生成一个解密 `KEY`(用于服务器解密客户发送过去的封包) -> 发送给服务器 -> 发送登录包(上面代码里有) -> 服务器返回人物信息(部分封包都采用压缩函数去压缩的,所以还需要解压缩封包才能使用,使用`zlib1.dll`的`uncompress`函数) -> 然后就各显神威吧.

搞这游戏真是觉得有点耐不住性子了..

希望这些代码对有些人有用..

准备过年了...祝大家虎年行大运,财源滚滚,虎气冲天,天天开心!
