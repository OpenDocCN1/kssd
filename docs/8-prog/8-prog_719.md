
# MX 发信 核心源码

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-728.htm`](https://www.pediy.com/kssd/pediy08/pediy8-728.htm)

所谓 MX 发送邮件就是不通过 `SMTP` 中转服务器，直接将邮件投递至目标邮件服务器

这种发信方式可以伪造发信者的邮件地址（但你的 `IP` 一样会被邮件服务器记录），同时也不需要用户名和密码

我看到好多木马在用这种发信方式。

因为不需要用户名和密码，所以不存在用户名和密码被嗅探的问题。

目前有些邮件服务器会把这种没有通过中转的信件认为是垃圾邮件，会拒收。

昨天抓包研究了一下如何获得 `MX` 地址信息，研究出来之后，才发现微软有专门的 `API` 来获取 `MX` 地址。这样节省了好多工作。

我把核心代码贴出来，细节可能需要自己去完善。

目前只是实现了文本方式的邮件发送，不支持附件，想要其它功能，自己研究邮件格式吧。

```
代码:

* * *

/\*-------------
Code by:允恒
Date:2006.08.14
转载请包含此信息
\--------------\*/
#include <winsock2.h>
#include <stdio.h>
#include <Windns.h>
#pragma comment(lib,"Dnsapi")
#pragma comment(lib,"ws2_32")

struct SMTPINFO 
{
  LPSTR SendToMail;
  LPSTR RecvFromMail;

  LPSTR From;
  LPSTR To;

  int Priority;
  LPSTR Subject;

  LPSTR Body;
};

unsigned char Chr2Hex( char c ) 
{ 
  if ( c >= 'a' && c <= 'z' ) // it's strtoupper 
   c = c - 'a' + 'A'; 
  if ( c >= '0' && c <= '9' ) 
   return ( int )( c - '0' ); 
  else if ( c >= 'A' && c <= 'F' ) 
   return ( int )( c - 'A' + 10 ); 
  else 
   return -1; 
} 

char Hex2Chr( unsigned char n )
{
  n &= 0xF;
  if ( n < 10 )
   return ( char )( n + '0' );
  else
   return ( char )( n - 10 + 'A' );
}

int QPEncode( char * const aDest,char * aSrc, int aLen )
{
  char * p = aDest;
  int    i = 0;

  while ( i++ < aLen )
  {
   *p++ = '=';
   *p++ = Hex2Chr( *aSrc >> 4 );
   *p++ = Hex2Chr( *aSrc++ );
  }
  *p = 0;  //  aDest is an ASCIIZ string
  return ( p - aDest );  //  exclude the end of zero
}

int QPDecode(unsigned char * const aDest, const char * aSrc ) 
{ 
  unsigned char * p = aDest; 
  int n = strlen( aSrc ); 
  unsigned char ch, cl; 

  while ( *aSrc ) // aSrc is an ASCIIZ string 
  { 
   if ( ( *aSrc == '=' ) && ( n - 2 > 0 ) ) 
   { 
    ch = Chr2Hex( aSrc[1] ); 
    cl = Chr2Hex( aSrc[2] ); 
    if ( ( ch == ( unsigned char )-1 ) || ( cl == ( unsigned char )-1 ) ) 
     *p++ = *aSrc++; 
    else 
    { 
     *p++ = ( ch << 4 ) | cl; 
     aSrc += 3; 
    } 
   } 
   else 
    *p++ = *aSrc++; 
  } 
  return ( p - aDest ); 
} 

BOOL SendData(SOCKET Socket,LPSTR sendbody,int sendlen,LPSTR recvflags)
{
  do
  {
   int len = send(Socket,sendbody,sendlen,0);

   if (len == SOCKET_ERROR)
    return FALSE;

   sendbody += len;
   sendlen -= len;
  }while(sendlen != 0);

  char recvbody[1024];
#ifdef  _DEBUG
  ZeroMemory(recvbody,sizeof(recvbody));
#endif

  if (SOCKET_ERROR != recv(Socket,recvbody,sizeof(recvbody),0))
  {
#ifdef  _DEBUG
   printf("%s\r\n",recvbody);
#endif
   if (0 == strncmp(recvbody,recvflags,lstrlenA(recvflags)))
   {
    return TRUE;
   }
   else
   {
    return FALSE;
   }
  }

  return TRUE;
}

BOOL SendMail(SMTPINFO *smtpinfo)
{
  LPSTR smtp = strstr(smtpinfo->SendToMail,"@");
  smtp++;

  DNS_RECORDA *p = NULL;

  if (0 == DnsQuery_A(smtp,DNS_TYPE_MX,DNS_QUERY_STANDARD,NULL,(PDNS_RECORD *)&p,NULL))
  {
   for(PDNS_RECORDA i = p; i != NULL; i = i->pNext)
   {
    if (i->wType == DNS_TYPE_MX)
    {
     hostent *hostname = gethostbyname(i->Data.MX.pNameExchange);

     for (int j = 0;hostname != NULL && hostname->h_addr_list[j] != NULL;j++)
     {
      //多 IP
      SOCKET Socket = socket(AF_INET,SOCK_STREAM,0);

      sockaddr_in addr;
      addr.sin_family = AF_INET;
      addr.sin_addr.S_un.S_addr = *(DWORD*)hostname->h_addr_list[j];
      addr.sin_port = htons(25);

      if (SOCKET_ERROR == connect(Socket,(sockaddr*)&addr,sizeof(addr)))
      {
       closesocket(Socket);
       continue;
      }

      char sendbody[1024];

      if (FALSE == SendData(Socket,NULL,0,"220"))
      {
       closesocket(Socket);
       continue;
      }

      char localhostname[MAX_COMPUTERNAME_LENGTH+1];
      gethostname(localhostname,sizeof(localhostname));

      wsprintfA(sendbody,"EHLO %s\r\n",localhostname);
      if (FALSE == SendData(Socket,sendbody,lstrlenA(sendbody),"250"))
      {
       closesocket(Socket);
       continue;
      }

      wsprintfA(sendbody,"MAIL FROM:<%s>\r\n",smtpinfo->RecvFromMail);
      if (FALSE == SendData(Socket,sendbody,lstrlenA(sendbody),"250"))
      {
       closesocket(Socket);
       continue;
      }

      wsprintfA(sendbody,"RCPT TO:<%s>\r\n",smtpinfo->SendToMail);
      if (FALSE == SendData(Socket,sendbody,lstrlenA(sendbody),"250"))
      {
       closesocket(Socket);
       continue;
      }

      if (FALSE == SendData(Socket,"DATA\r\n",lstrlenA("DATA\r\n"),"354"))
      {
       closesocket(Socket);
       continue;
      }

      LPSTR SubjectCode = (LPSTR)GlobalAlloc(GPTR,lstrlenA(smtpinfo->Subject)*4);
      QPEncode(SubjectCode,smtpinfo->Subject,lstrlenA(smtpinfo->Subject));

      LPSTR FromCode = (LPSTR)GlobalAlloc(GPTR,lstrlenA(smtpinfo->From)*4);
      QPEncode(FromCode,smtpinfo->From,lstrlenA(smtpinfo->From));

      LPSTR ToCode = (LPSTR)GlobalAlloc(GPTR,lstrlenA(smtpinfo->To)*4);
      QPEncode(ToCode,smtpinfo->To,lstrlenA(smtpinfo->To));

      LPSTR BodyCode = (LPSTR)GlobalAlloc(GPTR,lstrlenA(smtpinfo->Body)*4);
      QPEncode(BodyCode,smtpinfo->Body,lstrlenA(smtpinfo->Body));

      wsprintfA(sendbody,"Subject: =?GB2312?Q?%s?=\r\n"  \
       "From: \"=?GB2312?Q?%s?=\" <%s>\r\n"    \
       "To: \"=?GB2312?Q?%s?=\" <%s>\r\n"  \
       "X-Priority: %d\r\n"  \
       "Content-Transfer-Encoding: Quoted-Printable\r\n"\
       "Content-Type: text/plain;\r\n.charset=\"GB2312\"\r\n\r\n"\
       "%s\r\n.\r\n",
       SubjectCode,FromCode,smtpinfo->RecvFromMail,ToCode,smtpinfo->SendToMail,smtpinfo->Priority,BodyCode);

      GlobalFree(SubjectCode);
      GlobalFree(FromCode);
      GlobalFree(ToCode);
      GlobalFree(BodyCode);

      if (FALSE == SendData(Socket,sendbody,lstrlenA(sendbody),"250"))
      {
       closesocket(Socket);
       continue;
      }

      if (FALSE == SendData(Socket,"QUIT\r\n",lstrlenA("QUIT\r\n"),"221"))
      {
       closesocket(Socket);
       continue;
      }

      closesocket(Socket);

      DnsRecordListFree((PDNS_RECORD)p,DnsFreeFlat);

      return TRUE;
     }
    }
   }

   DnsRecordListFree((PDNS_RECORD)p,DnsFreeFlat);
  }

  return FALSE;
}

void main()
{
  WSADATA wsaData;
  WSAStartup(0x202,&wsaData);

  SMTPINFO smtpinfo;
  smtpinfo.Priority = 1;               //邮件优先级  1 为最高    5 为最低
  smtpinfo.Subject = "MX 发信测试";        //邮件主题
  smtpinfo.RecvFromMail = "test@test.com";    //发件人信箱
  smtpinfo.SendToMail = "lemony8734@gmail.com";  //收件人信箱
  smtpinfo.From = "张三";            //发件人姓名
  smtpinfo.To = "李四";             //收件人姓名
  smtpinfo.Body = "hoho~~~~~";          //邮件正文 s

  if (SendMail(&smtpinfo))
  {
   printf("邮件发送成功");
  }
  else
  {
   printf("邮件发送失败");
  }

  WSACleanup();
}

* * *

```

`4dd466d0e1e0cc123afbe504da2462ca`
