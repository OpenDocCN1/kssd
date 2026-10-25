
# Microsoft Access FieldList ActiveX 控件实例化内存破坏漏洞（MS10-044）

> 原文：[`www.pediy.com/kssd/pediy11/116780.html`](https://www.pediy.com/kssd/pediy11/116780.html)

## 受影响系统
Microsoft Access 2003 SP3

## 描述
--------------------------------------------------------------------------------  
BUGTRAQ ID: 41444  
CVE ID: CVE-2010-1881  
Microsoft Access 是微软 Office 套件中的关系数据库管理系统。  
Microsoft Access 实例化 FieldList ActiveX 控件的未初始化变量时存在内存破坏漏洞，成功利用此漏洞的攻击者可能以登录用户的身份运行任意代码。

\*来源：Robert Freeman  
链接：http://secunia.com/advisories/40554/  
http://www.microsoft.com/technet/security/Bulletin/MS10-044.mspx?pf=true  
http://www.us-cert.gov/cas/techalerts/TA10-194A.html  
\*

## 建议
--------------------------------------------------------------------------------  

### 临时解决方法
\* 禁止在 Internet Explorer 中运行 COM 对象。要为值为`{53230327-172B-11D0-AD40-00A0C90DC8D9}`的 CLSID 设置 kill bit，请在文本编辑器（如 Notepad）中粘贴以下文本，然后使用`.reg` 文件扩展名保存文件。

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\ActiveX Compatibility\{53230327-172B-11D0-AD40-00A0C90DC8D9}]

"Compatibility Flags"=dword:00000400

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Internet Explorer\ActiveX Compatibility\{53230327-172B-11D0-AD40-00A0C90DC8D9}]

"Compatibility Flags"=dword:00000400
```

\* 将 Internet Explorer 配置为在 Internet 和本地 Intranet 安全区域中运行 ActiveX 控件和活动脚本之前进行提示。
\* 将 Internet 和本地 Intranet 安全区域设置设为“高”，以便在这些区域中运行 ActiveX 控件和活动脚本之前进行提示。

### 厂商补丁
Microsoft  
---------  
Microsoft 已经为此发布了一个安全公告（MS10-044）以及相应补丁:  
MS10-044：Vulnerabilities in Microsoft Office Access ActiveX Controls Could Allow Remote Code Execution (982335)  
链接：http://www.microsoft.com/technet/security/Bulletin/MS10-044.mspx?pf=true
