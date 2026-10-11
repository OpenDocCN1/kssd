
# 智伦报关系统

> 原文：[`www.pediy.com/kssd/pediy05/pediy50417.htm`](https://www.pediy.com/kssd/pediy05/pediy50417.htm)

一个 VF 程序的破解---智伦报关系统

目标软件：智伦报关系统

软件大小：7.54 MB

软件性质：商业软件

应用平台：Win9x/NT/2000/XP

软件介绍：海关报关软件，能用 10,10 天以后得注册。

用工具：`fi`, `regsnap`, `unfoxall`, `uedit`。

## 破解过程

1. 先用 `fi` 查壳，没壳，奇了，明明是 VF，怎么 `fi` 显示 `VC5.0` 呢？
    用 `regsnap` 查，发现在 `system` 下多了几个 VF 的控件。
2. 用 `unfoxall` 反编辑程序，会有很多文件产生，用 Windows 的查找包含文字“注册号错误”，打开对应的程序文件，注册算法就在那里！如果安装了 VF，打开 form 表，查找对应注册窗口也可找到程序文件。
3. 分析其算法

```
＊＊＊＊＊＊＊＊

DIMENSION A_PASS( 13 )//注册码对应表

A_PASS = ''

M.ACT_PASS = ''//注册码对应表 A_PASS＝ABC0123456789

FOR I = 0 TO 2

  A_PASS( I + 1 ) = CHR(65 + I)//65 为 A 的 ascii 码,48 为 0 的 ascii 码

ENDFOR   

FOR I = 0 TO 9

  A_PASS( 4 + I ) = CHR(48 + I)

ENDFOR   

M.SYS_PASS = ALLTRIM(STR(100 * RAND(-1) + 10))//产生系统注册号

M.SYS_PASS = M.SYS_PASS + ALLTRIM(STR(100 * RAND()))

M.SYS_PASS = M.SYS_PASS + ALLTRIM(STR(100 * RAND()))

M.SYS_PASS = M.SYS_PASS + ALLTRIM(STR(100 * RAND()))

M.SYS_PASS = M.SYS_PASS + ALLTRIM(STR(100 * RAND()))

M.SYS_PASS = 'Z' + PADR(ALLTRIM(M.SYS_PASS),10,'8')//在系统注册号前加字符'Z'

＊＊＊＊＊＊

 M.ACT_PASS = 'Z'　　　　　　　　　　　　　　　　　　　　　　　　　　　//真确注册码算法

FOR I = 1 TO 10

  M.TMP_VAR = INT(I + VAL(SUBSTR(M.SYS_PASS,I + 1,1)) * 61.8 + 13.8)

  J = MOD(M.TMP_VAR,13) + 1

  M.ACT_PASS = M.ACT_PASS + A_PASS(J)

ENDFOR 

＊＊＊＊＊＊＊＊＊＊＊＊＊＊

PROCEDURE VcontrolOk

PARAMETER P_OK

SELECT HMST50

IF P_OK = 1

  IF EMPTY(PO_DATE) .OR. EMPTY(PO_CMP) .OR. EMPTY(CUST_CNAME) .OR. EMPTY(CUST_CADDR) .OR.  ;

  EMPTY(CUST_TEL) .OR. EMPTY(CUST_FAX) .OR. EMPTY(CUST_EMAIL) .OR.  ;

  EMPTY(ZIP_CODE)

    DO LMSG WITH 'MSG<贵公司资料不完整,请检查!>bmp<atten.bmp>time<08>'

    RETURN   

  ENDIF   

  IF M.REG_PASS <> M.ACT_PASS                  //注册码比较

    DO LMSG WITH 'MSG<注册号错误!>bmp<atten.bmp>time<08>'

    RETURN .F.

  ENDIF   

  M.LIMIT_DATE = EN(DTOC(DATE() + M.LLUSEDDAYS))

  M.W_TODAY = EN(DTOC(DATE()))

  DIMENSION A_DATA( 1 , 6 )

  A_DATA = ''

  M.DIR = SET('DEFAULT') + SYS(2003) + '\\bmp'

  = ADIR(A_DATA,M.DIR,'D')

  IF EMPTY(A_DATA)

    DO LMSG WITH 'MSG<EMPTY a SYSTEM directory!>BMP<atten.bmp>'

    CANCEL   

  ENDIF   

  M.DIR_DATE = EN(DTOC(A_DATA(3)))

  M.DIR_TIME =  ;

      EN(IIF(EMPTY(AT(SPACE(1),A_DATA(4))),A_DATA(4),STUFF(A_DATA(4),AT(SPACE(1),A_DATA(4)),1,'0')))

  M.NOTE =  ;

      'lsatdltatdfdhftasfotrtelsidf989t-[o' + M.DIR_DATE + 'iropjidofdoidfj90,[z' +  ;

  M.LIMIT_DATE + 'z]93490583478kif(x' + M.DIR_TIME + 'x)jdukg{d' + M.W_TODAY +  ;

  'dfgdfo'

  SELECT HMST50

  M.W_DATE = DATE() + M.LLUSEDDAYS

  REPLACE DATE1 WITH M.W_DATE , DATE2 WITH DATE() , DATE3 WITH M.W_DATE , DATE4 WITH  ;

      DATE() , DATE5 WITH M.W_DATE , DATE6 WITH DATE() + M.LLUSEDCNT ,  ;

      UPD_DATE WITH DATE()

  REPLACE NOTE WITH M.NOTE

  DO LMSG WITH 'MSG<注册成功!>bmp<atten.bmp>time<08>'

ELSE   

  CLEAR READ   

ENDIF   

RETURN   

ENDPROC

*------*
```

4. 写注册机

由于不懂 `vp`，大概编程的思路还是看得懂的，把它翻译成 `VC` 注册机就出来了

```
void CAbcDlg::OnButton1()   

{

//               CString m_syspass,m_actpass;    //m_syspass 为系统注册号，m_actpass 为实际注册号

UpdateData(TRUE);

CString regpass("ABC0123456789");//注册码对应表

m_actpass="Z";

int i,j,temp;

for(i=0;i<(m_syspass.GetLength()-1);i++)

{

temp=(int)((i+1+(m_syspass.GetAt(i+1)-48)*61.8+13.8));

j=temp%13;

m_actpass=m_actpass+apass.GetAt(j);

}

UpdateData(FALSE);

}   4dd466d0e1e0cc123afbe504da2462ca
```
