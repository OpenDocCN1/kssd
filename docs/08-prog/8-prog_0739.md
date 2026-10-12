
# 网络用户登录密码验证过程

> 原文：`www.pediy.com/kssd/pediy09/pediy09-634.htm`

> 通常网络密码验证过程是这样的：客户端在获取用户输入的用户名和密码信息之后创建 `socket` 套接字与远程服务器建立一个连接，发送用户名和密码并等待服务器返回消息，服务器收到用户名和密码之后查询服务器数据库，如果用户名密码正确返回给客户端一个登录成功消息，否则返回一个密码错误的消息！

> 下面是用 `VC++` 实现验证过程中的部分代码，代码有点乱：

```cpp
//获取用户输入信息 
void CClientDlg::OnOK() 
{ 
//之前 socket 已经经过初始化 
UpdateData(); 
if(m_username.IsEmpty()||m_userID.IsEmpty()) 
  MessageBox("请输入所有信息!","用户登录",MB_OK); 
else 
{ 
  if(Load()) //调用验证函数 
  { 
//   close(destSocket); 
   closesocket(destSocket); 
   CDialog::OnOK(); 
   MessageBox("登陆成功!","用户登录",MB_OK);//验证成功 
   CChat chat; 
   chat.m_username=m_username; 
//   chat.res=res; 
   chat.DoModal(); 
  } 
  else 
  {      //验证失败 
   MessageBox("尚未注册或\n 用户名/密码有误。","用户登录",MB_OK); 
  } 
} 

BOOL CClientDlg::Load() //发送"Load:" 
{ 
//////////////////////发送登陆信息并接收反馈信息///////////////////////////// 
char sendText[100],recvText[5025]; 
int numsnt,numrcv,status; 
sprintf(sendText,"Load:%s,%s",m_username,m_userID); 
numsnt=send(destSocket, sendText, strlen(sendText) + 1, NO_FLAGS_SET); 
if (numsnt != (int)strlen(sendText) + 1) 
   { 
  MessageBox("ERROR: Connection terminated!","用户登录",MB_OK); 
  status=closesocket(destSocket); 
  if (status == SOCKET_ERROR) 
   MessageBox("ERROR: closesocket unsuccessful!","用户登录",MB_OK); 
    status=WSACleanup(); 
  if (status == SOCKET_ERROR) 
   MessageBox("ERROR: WSACleanup unsuccessful!","用户登录",MB_OK); 
  return false; 
   } 
numrcv=recv(destSocket, recvText, 5025, NO_FLAGS_SET); 
if ((numrcv == 0) || (numrcv == SOCKET_ERROR)) 
   { 
     MessageBox("ERROR: Connection terminated!","用户登录",MB_OK); 
     status=closesocket(destSocket); 
     if (status == SOCKET_ERROR) 
   MessageBox("ERROR: closesocket unsuccessful!","用户登录",MB_OK); 
     status=WSACleanup(); 
     if (status == SOCKET_ERROR) 
   MessageBox("ERROR: WSACleanup unsuccessful!","用户登录",MB_OK); 
     return false; 
   } 
recvText[numrcv]='\0'; 
//////////////////////////////////////////////////////////////////////////// 
/////////////////////读取返回标志和用户信息/////////////////////////////////// 
char Flag[10]; 
char *pos=strchr(recvText,'!'); 
pos+=1; 
int len_text=strlen(recvText); 
int len_pos=strlen(pos); 
int len=len_text-len_pos; 
for(int i=0;i<len;i++) 
  Flag[i]=recvText[i]; 
Flag[len]='\0'; 
if(strcmp(Flag,"success!")!=0) 
  return false; 
///////////////////////////////////////////////////////////////////////////// 
/////////////////////将用户信息写入与用户名对应的文件//////////////////////// 
else 
{ 
  CString filename; 
  filename.Format(".\\data\\%s.db",m_username); 
  FILE *file; 
  if((file=fopen(filename,"w"))!=NULL) 
  { 
   fprintf(file,"%s",pos); 
  } 
  fclose(file); 
} 
///////////////////////////////////////////////////////////////////////////// 
return true; 
} 

//附网络初始化代码 
BOOL CClientDlg::Init_net() 
{ 
/////////////////////////网络初始化/////////////////////////////////// 
status=WSAStartup(MAKEWORD(1, 1), &Data); 
if (status != 0) 
  MessageBox("ERROR: WSAStartup unsuccessful!","用户登录",MB_OK); 
destAddr=inet_addr(DEST_IP_ADDR);  //暂时定为本机使用 
memcpy(&destSockAddr.sin_addr, &destAddr,sizeof(destAddr)); 
destSockAddr.sin_port=htons(606); 
destSockAddr.sin_family=AF_INET; 
destSocket=socket(AF_INET, SOCK_STREAM, 0); 
if (destSocket == INVALID_SOCKET) 
{ 
  MessageBox("ERROR: socket unsuccessful!","用户登录",MB_OK); 
  status=WSACleanup(); 
  if (status == SOCKET_ERROR) 
   MessageBox("ERROR: WSACleanup unsuccessful!","用户登录",MB_OK); 
  return false; 
} 
//////////////////////////////////////////////////////////////////// 
///////////////////////连接网络//////////////////////////////////////////// 
status=connect(destSocket, (LPSOCKADDR) &destSockAddr,sizeof(destSockAddr)); 
if (status == SOCKET_ERROR) 
{ 
  MessageBox("ERROR: connect unsuccessful!","用户登录",MB_OK); 
  status=closesocket(destSocket); 
  if (status == SOCKET_ERROR) 
   MessageBox("ERROR: closesocket unsuccessful!","用户登录",MB_OK); 
  status=WSACleanup(); 
  if (status == SOCKET_ERROR) 
   MessageBox("ERROR: WSACleanup unsuccessful!","用户登录",MB_OK); 
  return false; 
} 
///////////////////////////////////////////////////////////////////////////// 
return true; 
} 

///////////////////////////////////服务端代码///////////////////////////////// 
//建立一个基于消息的异步套接字 
BOOL CSkyQQsrvDlg::InitSocket() 
{ 
m_socket=WSASocket(AF_INET,SOCK_DGRAM,0,NULL,0,0); //WSASocket 创建套接字 
if(INVALID_SOCKET==m_socket) 
{ 
  MessageBox("创建套接字失败！"); 
  return FALSE; 
} 
SOCKADDR_IN addrSock; 
addrSock.sin_addr.S_un.S_addr=htonl(INADDR_ANY); 
addrSock.sin_family=AF_INET; 
addrSock.sin_port=htons(606);//设置 606 为监听端口 
if(SOCKET_ERROR==bind(m_socket,(SOCKADDR*)&addrSock,sizeof(SOCKADDR))) 
{ 
  MessageBox("绑定失败！"); 
  return FALSE; 
} 
if(SOCKET_ERROR==WSAAsyncSelect(m_socket,m_hWnd,UM_SOCK,FD_READ))//请求一个基于消息的网络事件通知异步套接字 
{ 
  MessageBox("注册网络读取事件失败！"); 
  return FALSE; 
} 
return TRUE; 
} 

//建立消息映射，在 SkyQQsrvDlg.h 中添加 
#define UM_SOCK  WM_USER+1 //定义消息 

//消息映射 
BEGIN_MESSAGE_MAP(CSkyQQsrvDlg, CDialog) 
//{{AFX_MSG_MAP(CSkyQQsrvDlg) 
// ON_WM_SYSCOMMAND() 
// ON_WM_PAINT() 
// ON_WM_QUERYDRAGICON() 
ON_MESSAGE(UM_SOCK,OnSock) 
// ON_BN_CLICKED(IDC_BTN_SEND, OnBtnSend) 
//}}AFX_MSG_MAP 
END_MESSAGE_MAP() 

BOOL bchar=FALSE;//密码检测结果 
SOCKET m_socket; 

void CSkyQQsrvDlg::OnSock(WPARAM wParam,LPARAM lParam) 
{ 
switch(LOWORD(lParam)) 
{ 
case FD_READ:  //读取事件 
  WSABUF wsabuf; 
  wsabuf.buf=new char[200]; 
  wsabuf.len=200; 
  DWORD dwRead; 
  DWORD dwFlag=0; 
  SOCKADDR_IN addrFrom; 
  int len=sizeof(SOCKADDR); 
  CString str; 
  CString strTemp; 
  HOSTENT *pHost; 
  if(SOCKET_ERROR==WSARecvFrom   (m_socket,&wsabuf,1,&dwRead,&dwFlag,//WSARecvFrom 接受数据函数 
     (SOCKADDR*)&addrFrom,&len,NULL,NULL)) 
  { 
   MessageBox("接收数据失败！"); 
   return; 
  } 
  pHost=gethostbyaddr((char*)&addrFrom.sin_addr.S_un.S_addr,4,AF_INET); 
  addrFromIP.sin_addr.S_un.S_addr=addrFrom.sin_addr.S_un.S_addr; 
  char(wsabuf.buf);//调用密码检测函数 
} 
CString strSend; 
if(bchar) 
{//发送验证成功的消息 
  strsend="success"; 
} 
else    strsend="FALSE"; 
SOCKADDR_IN addrTo; 
addrTo.sin_family=AF_INET; 
addrTo.sin_addr.S_un.S_addr=addrFromIP.sin_addr.S_un.S_addr; 
addrTo.sin_port=Port;//将获取的数据端口设置为发送数据端口 
len=strSend.GetLength();    //长度 
wsabuf.buf=strSend.GetBuffer(len); 
wsabuf.len=len+1; 
if(SOCKET_ERROR==WSASendTo(m_socket,&wsabuf,1,&dwSend,0,  //WSASend 发送数据 
  (SOCKADDR*)&addrTo,sizeof(SOCKADDR),NULL,NULL)) 
{ 
  MessageBox("发送数据失败！"); 
  return; 
} 

void char(char * wsabuf) 
{ 
_ConnectionPtr m_pConnection; 
_RecordsetPtr m_pRecordset 
CString admin,passwd; 
huanyuan(wsabuf,admin,passwd); 
//假设服务器上已经有一个 ACCESS 数据库文件，文件名为 skywd,并建有 user(admin,password)表 
//初始话 OLE/COM 库环境 
::CoInitialize(NULL); 
try 
{  
```



```
_ConnectionPtr pConn(__uuidof(Connection));

pConn->C;

pConn->Open("","","",adConnectUnspecified); //打开对数据库的连接,用户 ID 和密码,同步打开这个连接

m_pC;

}

//捕捉异常

catch(_com_error e)

{   //显示错误信息

//  AfxMessageBox(e.Description());

  AfxMessageBox("数据库连接出错");

}

//设置 INSERT 语句，查询数据

_bstr_t vSQL;

vSQL = "SELECT * FROM user WHERE admin = '" +admin + "'" ; //变量直接加，字符串用""括起来

try

{

m_pRecordset.CreateInstance(__uuidof(Recordset));

m_pRecordset->Open(vSQL,m_pConnection.GetInterfacePtr(), //直接用记录集对象查询

adOpenDynamic,adLockOptimistic,adCmdText);

}

catch(_com_error e)

{

  AfxMessageBox(e.Description());

}

_variant_t  var = m_pRecordset->GetCollect("password");

  if(var.vt!=VT_NULL)

  {

  if(passwd==(LPCTSTR)_bstr_t(var))

  bchar=TRUE;//通过验证

  }

  else

  {//如果答案为空,则重建答案

  bchar=FALSE;

  }

//断开数据库连接

if(m_pRecordset!=NULL)

m_pRecordset->Close();

m_pConnection->Close();

}

zhuanhuan(char * wsabuf,CString admin,CString passwd)

{

    CString str1=wsabuf;

    int n1=str1.Find(":");

    int n2=str1.Find(",");

    int n3=str1.Getlength()-1;

    admin=str1.Left(n2);

    admin=admin.Right(n2-n1-1);

    passwd=str1.Right(n3-n2-1);

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
