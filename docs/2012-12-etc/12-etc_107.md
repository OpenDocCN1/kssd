
# 发个 WM 手机开关 GSM 模块的代码（即飞行模式开关）

> 原文：[`www.pediy.com/kssd/pediy10/87504.html`](https://www.pediy.com/kssd/pediy10/87504.html)

哎，真不知道怎么一个两个都把它藏着掖着，查了半天资料以及 IDA 逆了来电精灵和一个一键飞行工具才明白了其中的内幕。

成品界面看附件图。

不多说了，用法是：

`SetPhoneState(LINEEQUIPSTATE_MINIMUM);` //开飞机

`SetPhoneState(LINEEQUIPSTATE_FULL);` //安全着陆

------------------------------------------------------------------------------------

```c
DWORD GetTSPLineDeviceID(const HLINEAPP hLineApp, const DWORD dwNumberDevices, const DWORD dwAPIVersionLow, const DWORD dwAPIVersionHigh, const TCHAR* const psTSPLineName)
{
DWORD dwReturn = 0xffffffff;
for(DWORD dwCurrentDevID = 0 ; dwCurrentDevID < dwNumberDevices ; dwCurrentDevID++)
{
DWORD dwAPIVersion;
LINEEXTENSIONID LineExtensionID;
if(0 == lineNegotiateAPIVersion(hLineApp, dwCurrentDevID,
dwAPIVersionLow, dwAPIVersionHigh,
&dwAPIVersion, &LineExtensionID))
{
LINEDEVCAPS LineDevCaps;
LineDevCaps.dwTotalSize = sizeof(LineDevCaps);
if(0 == lineGetDevCaps(hLineApp, dwCurrentDevID,
dwAPIVersion, 0, &LineDevCaps))
{
BYTE* pLineDevCapsBytes = new BYTE[LineDevCaps.dwNeededSize];
if(0 != pLineDevCapsBytes)
{
LINEDEVCAPS* pLineDevCaps = (LINEDEVCAPS*)pLineDevCapsBytes;
pLineDevCaps->dwTotalSize = LineDevCaps.dwNeededSize;
if(0 == lineGetDevCaps(hLineApp, dwCurrentDevID,
dwAPIVersion, 0, pLineDevCaps))
{
if(0 == _tcscmp((TCHAR*)((BYTE*)pLineDevCaps+pLineDevCaps->dwLineNameOffset),
psTSPLineName))
{
dwReturn = dwCurrentDevID;
}
}
delete[] pLineDevCapsBytes;
}
}
}
}
return dwReturn;
}
```

```c
DWORD SetPhoneState(DWORD state)
{
HLINEAPP m_hLineApp = 0;
DWORD m_dwAPIVersion = TAPI_API_HIGH_VERSION;
DWORD dwNumDevs;
LINEINITIALIZEEXPARAMS liep;
liep.dwTotalSize = sizeof(liep);
liep.dwOptions = LINEINITIALIZEEXOPTION_USEEVENT;
HLINE hLine = 0;
LONG lRC = 0;
DWORD dwState, dwRadioSupport;
TCHAR gszFriendlyAppName[160];
if(lineInitializeEx(&m_hLineApp, 0, 0, gszFriendlyAppName, &dwNumDevs, &m_dwAPIVersion, &liep)) {
return PHONEDEVICE_ERROR;
}
if(NULL == m_hLineApp)
{
return -1;
}
DWORD dwDeviceID = GetTSPLineDeviceID(m_hLineApp, dwNumDevs, TAPI_API_LOW_VERSION, TAPI_API_HIGH_VERSION, CELLTSP_LINENAME_STRING);
lRC = lineOpen(m_hLineApp, dwDeviceID, &hLine, m_dwAPIVersion, 0,
NULL, LINECALLPRIVILEGE_OWNER, LINEMEDIAMODE_DATAMODEM,
NULL);
if(lRC < 0)
{
if(hLine != NULL)
{
lineClose(hLine);
hLine = NULL;
}
return lRC;
}
//lRC = lineGetEquipmentState(hLine, &dwState, &dwRadioSupport);
//if(lRC < 0)
//{
// lineClose(hLine);
// return lRC;
//}
//if((LINEEQUIPSTATE_MINIMUM == state) ||
// (LINEEQUIPSTATE_NOTXRX == state))
//{
// lRC = lineUnregister(hLine);
// if(lRC < 0)
// {
// lineClose(hLine);
// return lRC;
// }
//}
//if(dwState != state)
//{
lRC = lineSetEquipmentState(hLine, state);
//}
if(lRC < 0)
{
lineClose(hLine);
return lRC;
}
if((LINEEQUIPSTATE_FULL == state) ||
(LINEEQUIPSTATE_RXONLY == state) ||
(LINEEQUIPSTATE_TXONLY == state))
{
lRC = lineRegister(hLine, LINEREGMODE_AUTOMATIC, NULL, LINEOPFORMAT_NONE);
if(lRC < 0)
{
lineClose(hLine);
return lRC;
}
}
lRC = lineClose(hLine);
return lRC;
}
![](img/575020d8e7d91bbe07c66179edf4fb39.jpg)
