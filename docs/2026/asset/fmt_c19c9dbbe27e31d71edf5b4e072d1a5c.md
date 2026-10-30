
```asm
                LDR        R3, =unk_1007BAE0 //这里其实是返回值

                MOV        R2, #1

                STMIA        R3, {R2,R4}//第一个数值保存为1，第二个数值保存为

                LDR        R0, [SP,#0x14C+var_20]

                BL        sub_1006DAA8

                ADD        SP, SP,        #0x130

                LDMFD        SP!, {R4-R9,LR}

                BX        LR

; End of function sub_1003D864

; ---------------------------------------------------------------------------

off_1003D9C8        DCD unk_1007BAE0        ; DATA XREF: sub_1003D864:loc_1003D9A8r

off_1003D9CC        DCD dword_10003300        ; DATA XREF: sub_1003D864+B8r

off_1003D9D0        DCD aWindows                ; DATA XREF: sub_1003D864+90r

                                        ; "\\Windows"

off_1003D9D4        DCD aPkg_close                ; DATA XREF: sub_1003D864+5Cr

                                        ; "Pkg_Close"

off_1003D9D8        DCD aPkg_getpkginfo        ; DATA XREF: sub_1003D864+4Cr

                                        ; "Pkg_GetPkgInfo"

off_1003D9DC        DCD aPkg_openpackag        ; DATA XREF: sub_1003D864+3Cr

                                        ; "Pkg_OpenPackageByGUID"

off_1003D9E0        DCD aPkg_setbaselin        ; DATA XREF: sub_1003D864+28r

                                        ; "Pkg_SetBaselineDirectories"

off_1003D9E4        DCD aPackageinfoapi        ; DATA XREF: sub_1003D864+14r

                                        ; "PackageInfoAPI.DLL"

off_1003D9E8        DCD unk_1007B9D4        ; DATA XREF: sub_1003D864+8r

; Exported entry  17. GetVersionEx

; Exported entry 717. GetVersionExW

; =============== S U B        R O U T        I N E =======================================

/*

typedef struct _OSVERSIONINFO{

  DWORD dwOSVersionInfoSize;

  DWORD dwMajorVersion;

  DWORD dwMinorVersion;

  DWORD dwBuildNumber;

  DWORD dwPlatformId;

  TCHAR szCSDVersion[128];

} OSVERSIONINFO;

*/

                EXPORT GetVersionExW

GetVersionExW                                ; DATA XREF: .pdata:1007F438o

                STMFD        SP!, {R4,R5,LR}        ; GetVersionEx

                MOV        R4, R0

                LDR        R5, =unk_1007BAE0 //R5中存放unk_1007BAE0的地址

                LDR        R3, [R5]          //R3为获取R5所指向的int的值

                CMP        R3, #0            //看R3是否为0

                BLEQ        sub_1003D864  //如果R3中的值为0，则执行sub_1003D864

                LDR        R3, [R5,#4]       //获取Build number

                MOV        R0, #2      //表示minor version

                MOV        LR, #3       //表示platformid

                MOV        R2, #0x114  //dwOSVersionInfoSize =R2=0x114

                MOV        R1, #5   //Major version

                MOV        R5, #0

                ADD        R12, R4, #8     //指向minorversion

                STMIA        R12, {R0,R3,LR}//保存minor version, buildnumber, 以及platformid

                MOV        R0, #1   //返回1，表示成功

                STR        R2, [R4] //dwOSVersionInfoSize =R2=0x114

                STR        R1, [R4,#4] //dwMajorVersion = R1= 5

                STRH        R5, [R4,#0x14]

                LDMFD        SP!, {R4,R5,LR}

                BX        LR

; End of function GetVersionExW

sub_1006DAA8                                ; CODE XREF: sub_1000AA8C+F4p //这个函数是干什么的？大概是堆栈的检查动作具体没看

                                        ; sub_1000ABEC+11Cp ...

var_C                = -0xC

var_8                = -8

var_4                = -4

arg_4                =  4

                LDR        R12, =unk_1007B9D4

                LDR        R12, [R12]

                CMP        R0, R12

                MOVEQS        R12, R0,LSR#16

                BXEQ        LR

loc_1006DABC                                ; DATA XREF: .pdata:100813A8o

                STR        LR, [SP,#var_4]!

                SUB        SP, SP,        #0xC

                LDR        R12, =unk_1007B9D0

                LDR        R12, [R12]

                STR        R12, [SP,#0xC+var_4]

                LDR        R12, =unk_1007B9D4

                LDR        R12, [R12]

                STR        R12, [SP,#0xC+var_8]

                LDR        R12, =0xB064

                STR        R12, [SP,#0xC+var_C]

                BL        __report_gsfailure

                ADD        SP, SP,        #0xC

                LDR        LR, [SP],#arg_4

                BX        LR

; End of function sub_1006DAA8

; ---------------------------------------------------------------------------

off_1006DAF4        DCD unk_1007B9D4        ; DATA XREF: sub_1006DAA8r

                                        ; sub_1006DAA8+28r

off_1006DAF8        DCD unk_1007B9D0        ; DATA XREF: sub_1006DAA8+1Cr

dword_1006DAFC        DCD 0xB064                ; DATA XREF: sub_1006DAA8+34r
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
