
```
invoke SetDlgItemText,hWnd,IDC_EDT2,addr NameBuffer

ret

GetRegistKey endp

end start

;---------------------------------------------------------------------------------

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;>>>       keygen.rc       >>>>

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#define MANIFEST 24

#define IDD_MAINDLG 1000

#define IDOK 1

#define IDC_EDT1 1003

#define IDC_EDT2 1004

#define IDC_STC1 1005

#define IDC_STC2 1006

#define IDC_GRP1 1007

#define IDM_MUSIC 50

IDD_MAINDLG DIALOGEX 4,4,145,112

CAPTION "Kegen for program"

FONT 10,"宋体"

STYLE 0x10CA0880

EXSTYLE 0x00000001

BEGIN
  CONTROL "计算",IDOK,"Button",0x50010000,52,93,41,15,0x00000000
  CONTROL "",IDC_EDT1,"Edit",0x50010000,46,49,87,12,0x00000200
  CONTROL "",IDC_EDT2,"Edit",0x50010800,46,67,87,12,0x00000200
  CONTROL "用户名：",IDC_STC1,"Static",0x50000201,13,49,30,12,0x00000000
  CONTROL "注册码：",IDC_STC2,"Static",0x50000201,13,69,30,12,0x00000000
  CONTROL "",IDC_GRP1,"Button",0x50000007,7,40,132,47,0x00000000
END

1008 ICON DISCARDABLE "ccg.ico"

IDM_MUSIC RCDATA DISCARDABLE "keygen.xm"                 ;自己找一个 xm 文件换上就可以了

;---------------------------------------------------------------------------------
```

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

附件:`mfmplayer.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
