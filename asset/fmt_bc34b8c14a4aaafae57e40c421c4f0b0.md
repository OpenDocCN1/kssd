
注册码为：`code1-cdoe2-code3-code4` 

注册成功后保存在注册表：

`[HKEY_CURRENT_USER\Software\FolderView\Registration]`

`"Name"="qhj"`

`"Code"="473-12920-413-3553"`

===================================================

附：VB 注册码源代码：(初学)

```vb
Private Sub 计算()  

If Text1.Text = "" Then  

MsgBox "注意：用户名不能为空!", 48, "绝密档案"  

Text1.SetFocus  

Exit Sub  

End If 

Dim I, name， name_h, name_h2, code1, code2, code3, code4 

'''先把名称串成一串  

name = Trim(Text1.Text)  '清除左右空格  

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))    ''' 求出相应的 ASC 码值  

ebp = name_h2 + 50  

ecx = ecx + ebp  

code1 = ecx  

Next I 

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))  

ebp = name_h2 * 40  

ecx = ecx + ebp  

code2 = ecx  

Next I 

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))  

ebp = name_h2 + 30  

ecx = ecx + ebp  

code3 = ecx  

Next I 

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))  

ebp = name_h2 * 11  

ecx = ecx + ebp  

code4 = ecx  

Next I 

Text2.Text = code1 & "-" & code2 & "-" & code3 & "-" & code4 

Private Sub Text1_KeyPress(KeyAscii As Integer)  

If KeyAscii > 255 Or KeyAscii < 0 Then  

KeyAscii = 0  

MsgBox "暂不支持中文", 48, "绝密档案"  

End If  

End Sub 

''暂不支持中文，请高手指点! 

绝密档案 2003-05-19  

[hongjian.126.com](http://hongjian.126.com)
```

算法分析本身就不完整，当然支持不了中文……贴个 TC 2.0 注册机，写得不好，请多多见谅，支持中文注册！

```c
main()  

{  

int i;  

char name[80];  

long sn1=0,sn2=0,sn3=0,sn4=0;  

clrscr();  

printf("\n\nFolderView V1.8 KeyGen *Make by PaulYoung*\n\nPlease input your name:");  

gets(name);  

for (i=0;name[i];i++)  

{  

sn1+=name[i]+0x32;  

sn2+=name[i]*0x28;  

sn3+=name[i]+0x1E;  

sn4+=name[i]*0xB;  

}  

printf(" Your RegisterCode is:%lu-%lu-%lu-%lu\n\n",sn1,sn2,sn3,sn4);  

getch();  

}   

4dd466d0e1e0cc123afbe504da2462ca
```
