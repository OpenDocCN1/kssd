
# 某游戏登录流程

> 原文：[`www.pediy.com/kssd/pediy11/107112.html`](https://www.pediy.com/kssd/pediy11/107112.html)

// 这里是游戏界面的加解密

```c
__declspec(naked) VOID EnDePackEx(PBYTE pbyBuffer, int nLen, PBYTE pbyKEY, int nKeyLen)
{
    __asm
    {
        mov     eax, dword ptr [esp+8]
        xor     ecx, ecx
        xor     edx, edx
        test    eax, eax
        jbe     RETURN
        push    ebx
        push    ebp
        mov     ebp, dword ptr [esp+0x14]
        push    esi
        mov     esi, dword ptr [esp+0x10]
        push    edi
        mov     edi, dword ptr [esp+0x20]
        lea     esp, dword ptr [esp]
LABEL:
        mov     bl, byte ptr [ecx+ebp]
        xor     byte ptr [edx+esi], bl
        add     ecx, 1
        cmp     ecx, edi
        jb      LABEL1
        xor     ecx, ecx
LABEL1:
        add     edx, 1
        cmp     edx, eax
        jb      LABEL
        pop     edi
        pop     esi
        pop     ebp
        pop     ebx
RETURN:
        retn
    }
}
```

// 这里是登录界面的加解密

```c
__declspec(naked) VOID EnDePack(PBYTE pbyBuffer, int nLen, PBYTE pbyKEY, int nKeyLen)
{
    __asm
    {
        push    esi
        mov     esi, dword ptr [esp+0xC]
        shr     esi, 1
        mov     eax, 0
        mov     ecx, eax
        je      RETURN
        mov     edx, dword ptr [esp+8]
        push    ebx
        mov     ebx, dword ptr [esp+0x14]
        push    ebp
        push    edi
        mov     edi, dword ptr [esp+0x20]
        shr     edi, 1
LABLE:
        mov     bp, word ptr [ebx+eax*2]
        xor     word ptr [edx+ecx*2], bp
        add     eax, 1
        cmp     eax, edi
        jb      LABLE1
        xor     eax, eax
LABLE1:
        add     ecx, 1
        cmp     ecx, esi
        jb      LABLE
        pop     edi
        pop     ebp
        pop     ebx
RETURN:
        lea     eax, dword ptr [esi+esi]
        pop     esi
        retn
    }
}
```

// 效验封包头

```c
DWORD WINAPI GetCheckPack(PBYTE pbyBuffer)
{
    DWORD   dwResult = 0;
    WORD    nLen = *(PWORD)ULongToPtr(&pbyBuffer[0]);
    int i = 0;
    if ( nLen > 0 )
    {
        do
        {
            dwResult += *(PBYTE)ULongToPtr(&pbyBuffer[i]);
            --nLen;
            i ++;
        } while (nLen);
    }
    return dwResult;
}
```

// 修改 KEY

```c
VOID WINAPI ChangeKEY(PBYTE pbyKEY)
{
    for ( int i = 0; i < 0x10; i ++ )
        pbyKEY[i] = pbyKEY[i] + i;
}
```

// 加密发送封包

```c
VOID CGameSend::EnSend(PSOCKET_ENGINE pSE, PBYTE pbyBuffer, int nLen)
{
    DWORD dwResult = GetCheckPack(pbyBuffer);
    *(PWORD)ULongToPtr(&pSE->bySendBuffer[0])   = nLen + 8;
    *(PWORD)ULongToPtr(&pSE->bySendBuffer[2])   = 0x0007;
    *(PDWORD)ULongToPtr(&pSE->bySendBuffer[4])  = dwResult;
    memcpy(&pSE->bySendBuffer[8], pbyBuffer, nLen);
    ((VOID (*)(PBYTE, int, PBYTE, int))ULongToPtr(pSE->pEnDeCode))(&pSE->bySendBuffer[4], nLen + 4, pSE->bySendKEY, 0x10);
    pSE->nSendLen = nLen + 8;
    send(pSE->Socket, (const char *)pSE->bySendBuffer, pSE->nSendLen, 0);
    ChangeKEY(pSE->bySendKEY);
}
```

// 发送加密解密 KEY

```c
VOID CGameSend::OnSendKEY(PSOCKET_ENGINE pSE)
{
#pragma pack(1)
    typedef struct
    {
        WORD  wSize;
        WORD  wHead;
        BYTE  byKEY[0x10];
        DWORD dwUnknow;
    } SEND_ENDEKEY, *PSEND_ENDEKEY;
#pragma pack()

    SEND_ENDEKEY    SENK;
    SENK.wSize      = 0x0018;
    SENK.wHead      = 0x0008;
    // 自己创建一个 KEY 发送给服务器
    for ( int i = 0; i < 0x10; i ++ )
    {
        SENK.byKEY[i] = 1;
    }
    SENK.dwUnknow = 0x01010101;
    // 记录这个 KEY
    memcpy(pSE->bySendKEY, (BYTE *)&SENK, 0x10);
    send(pSE->Socket, (const char *)&SENK, 0x18, 0);
}
```

// 发送心跳包

```c
VOID CGameSend::OnKeepLive(PSOCKET_ENGINE pSE, PPACK_INFOEX pPI)
{
#pragma pack(1)
    typedef struct
    {
        WORD wSize;
        WORD wHead;
        WORD wNum;
        WORD wUnknow;
    } SEND_DATA, *PSEND_DATA;
#pragma pack()

    SEND_DATA SD;
    SD.wSize    =   0x0008;
    SD.wHead    =   0x000B;
    SD.wNum     =   (WORD)pPI->dwPackCheck;
    SD.wUnknow  =   0;
    send(pSE->Socket, (const char *)&SD, sizeof(SD), 0);
}
```

// 发送登录帐号包

```c
VOID CGameSend::OnSendAccount(PSOCKET_ENGINE pSE)
{
#pragma pack(1)
    typedef struct
    {
        WORD   wSize;
        WORD   wHead;
        DWORD  dwVersion;
        char   szAccount[51];
        char   szPassword[21];
    } SEND_ACCOUNT_PACK, *PSEND_ACCOUNT_PACK;
#pragma pack()

    SEND_ACCOUNT_PACK SAP;
    memset(&SAP, 0, sizeof(SAP));
    SAP.wSize       = 0x0050;
    SAP.wHead       = 0x0201;
    SAP.dwVersion   = m_dwVersion;
    memcpy(SAP.szAccount, m_LC.szName, lstrlen(m_LC.szName));
    memcpy(SAP.szPassword, m_LC.szPass, lstrlen(m_LC.szPass));
    EnSend(pSE, (PBYTE)&SAP, sizeof(SAP));
}
```

// 选择线路

```c
VOID CGameSend::OnSelectLine(PSOCKET_ENGINE pSE)
{
// 0x05, 0x00, 0x07, 0x02, 0x09
#pragma pack(1)
    typedef struct
    {
        WORD wSize;
        WORD wHead;
        BYTE byLine;
    } SEND_DATA, *PSEND_DATA;
#pragma pack()

    SEND_DATA SD;
    SD.wSize    = 0x0005;
    SD.wHead    = 0x0207;
    SD.byLine   = m_LC.bySelectLine;
    EnSend(pSE, (PBYTE)&SD, sizeof(SD));
}
```

// 开始游戏

```c
VOID CGameSend::OnStartGame(PSOCKET_ENGINE pSE, PPACK_INFO pPI)
{
#pragma pack(1)
    typedef struct
    {
        WORD    wLen;
        WORD    wHead;
        DWORD   dwUnknow1;
        DWORD   dwUnknow2;
        WORD    wUnknow3;
        WORD    wNum;
        struct
        {
            DWORD   dwAccID;
            DWORD   dwSID;
            char    szName[24];
            DWORD   dwUnknow4;
            DWORD   dwProfessional; //  职业
            BYTE    byUnknow5[59];
        } LIST[8];  //  最多 8 个人物
    } RECV_DATA, *PRECV_DATA;
#pragma pack()

#pragma pack(1)
    typedef struct
    {
        WORD    wLen;
        WORD    wHead;
        DWORD   dwAccID1;
        DWORD   dwAccID2;
    } SEND_DATA, *PSEND_DATA;
#pragma pack()

    PRECV_DATA pRD = (PRECV_DATA)pPI;
    SEND_DATA  SD;
    SD.wLen         = sizeof(SD);
    SD.wHead        = 0x030A;
    SD.dwAccID1     = pRD->LIST[m_LC.bySelectPlayerLocation].dwAccID;
    SD.dwAccID2     = pRD->LIST[m_LC.bySelectPlayerLocation].dwSID;
    EnSend(pSE, (PBYTE)&SD, sizeof(SD));
}
```

// 进入游戏

```c
VOID CGameSend::OnEnterGame(PSOCKET_ENGINE pSE)
{
#pragma pack(1)
    typedef struct
    {
        WORD    wSize;
        WORD    wHead;
        DWORD   dwEnterID1;
        DWORD   dwEnterID2;
        DWORD   dwVersion;
        BYTE    byGMID[16];
    } SEND_DATA, *PSEND_DATA;
#pragma pack()

    SEND_DATA       SD;
    SD.wSize        = 0x0020;
    SD.wHead        = 0x0301;
    SD.dwEnterID1   = m_dwEnterID1;
    SD.dwEnterID2   = m_dwEnterID2;
    SD.dwVersion    = m_dwVersion;
    // 这个是进入游戏包,版本更新也得更新
    BYTE byBuffer[16] = {   0xC4, 0x75 ,0x29, 0xF0, 0xB2, 0x76, 0x24, 0x81, 0xE2, 0x21,
                            0x01, 0x5F, 0x84, 0xB0, 0x70, 0x2A};
    memcpy(SD.byGMID, byBuffer, 16);
    EnSend(pSE, (PBYTE)&SD, sizeof(SD));
}
```

```c
VOID CGameRecv::DeRecvPack(PSOCKET_ENGINE pSE, PPACK_INFOEX pPIEX)
{
    switch ( pPIEX->wTag + -1  )
    {
        //  5 的标志不用解密,用于心跳
        case 0x0005:
        {
```
