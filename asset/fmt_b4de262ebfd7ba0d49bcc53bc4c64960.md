
```
MessageBox("打开注册表项 ERSvc 出错!");  

return FALSE;  

}  

buf = 0x00000000;  

RegQueryValueEx(hkResult,"Start",0,0,(BYTE\*)&buf,&iResult);  

if (buf == 4)  

{  

buf = 0x00000002;  

RegSetValueEx(hkResult,"Start",0,REG\_DWORD,(BYTE\*)&buf,sizeof(buf));  

}  

RegCloseKey(hkResult);  

//////////////////////////////////////////////////////////////////////////  

lg = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE,  

"system\\\\CurrentControlSet\\\\Services\\\\wscsvc",  

0,  

KEY\_ALL\_ACCESS,  

&hkResult);  

if (lg != ERROR\_SUCCESS)  

{  

MessageBox("打开注册表项 wscsvc 出错!");  

return FALSE;  

}  

buf = 0x00000000;  

RegQueryValueEx(hkResult,"Start",0,0,(BYTE\*)&buf,&iResult);  

if (buf == 4)  

{  

buf = 0x00000002;  

RegSetValueEx(hkResult,"Start",0,REG\_DWORD,(BYTE\*)&buf,sizeof(buf));  

}  

RegCloseKey(hkResult);  

//////////////////////////////////////////////////////////////////////////  

lg = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE,  

"system\\\\CurrentControlSet\\\\Services\\\\wuauserv",  

0,  

KEY\_ALL\_ACCESS,  

&hkResult);  

if (lg != ERROR\_SUCCESS)  

{  

MessageBox("打开注册表项 wuauserv 出错!");  

return FALSE;  

}  

buf = 0x00000000;  

RegQueryValueEx(hkResult,"Start",0,0,(BYTE\*)&buf,&iResult);  

if (buf == 4)  

{  

buf = 0x00000002;  

RegSetValueEx(hkResult,"Start",0,REG\_DWORD,(BYTE\*)&buf,sizeof(buf));  

}  

RegCloseKey(hkResult);  

return TRUE;  

}  

/*
上面功能的实现。
*/  

void CRemoveDlg::OnBtnRemove()  

{  

// TODO: Add your control notification handler code here  

if(!this->PromoteLimit())  

{  

MessageBox("提权失败!");  

return;  

}  

if(!this->CloseProc())  

{  

MessageBox("结束进程失败!");  

return;  

}  

if(!this->RemoveFiles())//册除文件  

return;  

if(!this->RenewReg())  

return;  

if(MessageBox("已完成!\n 是否现在重启计算机?","Finished !",MB\_OKCANCEL) == IDOK)  

system("shutdown -r -f -t 0");  

}  
```
以上是我检测的病毒的分析及处理过程，和官方报出的病特征不太相同，估计是变种，但操作是类似的，建议操作完成后清理一下系统垃圾文件及冗余注册表。
