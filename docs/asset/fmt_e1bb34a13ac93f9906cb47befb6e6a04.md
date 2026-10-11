
# 汇编代码分析

```asm
00424B12  |.  6A 04             PUSH 4                            ; /Arg1 = 00000004  
00424B14  |.  8B4D E0           MOV ECX,DWORD PTR SS:[EBP-20]        ; |  
00424B17  |.  83C1 74          ADD ECX,74                          ; |  
00424B1A  |.  E8 71F4FFFF      CALL VideoCon.00423F90              ; \VideoCon.00423F90  
00424B1F  |.  8845 F9          MOV BYTE PTR SS:[EBP-7],AL          ;  第五位[ebp-7]  

00424B22  |.  6A 05            PUSH 5                            ; /Arg1 = 00000005  
00424B24  |.  8B4D E0          MOV ECX,DWORD PTR SS:[EBP-20]        ; |  
00424B27  |.  83C1 74          ADD ECX,74                          ; |  
00424B2A  |.  E8 61F4FFFF      CALL VideoCon.00423F90              ; \VideoCon.00423F90  
00424B2F  |.  8845 F7          MOV BYTE PTR SS:[EBP-9],AL          ;  第六位[ebp-9]  

00424B32  |.  6A 06            PUSH 6                            ; /Arg1 = 00000006  
00424B34  |.  8B4D E0          MOV ECX,DWORD PTR SS:[EBP-20]        ; |  
00424B37  |.  83C1 74          ADD ECX,74                          ; |  
00424B3A  |.  E8 51F4FFFF      CALL VideoCon.00423F90              ; \VideoCon.00423F90  
00424B3F  |.  8845 FE          MOV BYTE PTR SS:[EBP-2],AL          ;  第七位[ebp-2]  

00424B42  |.  6A 07            PUSH 7                            ; /Arg1 = 00000007  
00424B44  |.  8B4D E0          MOV ECX,DWORD PTR SS:[EBP-20]        ; |  
00424B47  |.  83C1 74          ADD ECX,74                          ; |  
00424B4A  |.  E8 41F4FFFF      CALL VideoCon.00423F90              ; \VideoCon.00423F90  
00424B4F  |.  8845 FB          MOV BYTE PTR SS:[EBP-5],AL          ;  第八位[ebp-5]  

00424B52  |.  0FB655 EF        MOVZX EDX,BYTE PTR SS:[EBP-11]      ;  码 1  
00424B56  |.  0FB645 FC        MOVZX EAX,BYTE PTR SS:[EBP-4]       ;  假注册码第 1 位的 ASCII  
00424B5A  |.  83E8 30          SUB EAX,30                          ;  转成数字  
00424B5D  |.  3BD0            CMP EDX,EAX                         ;  要相等  
00424B5F  |.  75 3C           JNZ SHORT VideoCon.00424B9D  
00424B61  |.  0FB64D F8        MOVZX ECX,BYTE PTR SS:[EBP-8]  
00424B65  |.  0FB655 FD        MOVZX EDX,BYTE PTR SS:[EBP-3]  
00424B69  |.  83EA 30          SUB EDX,30  
00424B6C  |.  3BCA            CMP ECX,EDX  
00424B6E  |.  75 2D           JNZ SHORT VideoCon.00424B9D         ;  码 2 与第二位注册码  
00424B70  |.  0FB645 FF        MOVZX EAX,BYTE PTR SS:[EBP-1]  
00424B74  |.  0FB64D F6        MOVZX ECX,BYTE PTR SS:[EBP-A]  
00424B78  |.  83E9 30          SUB ECX,30  
00424B7B  |.  3BC1            CMP EAX,ECX  
00424B7D  |.  75 1E           JNZ SHORT VideoCon.00424B9D         ;  码 3  
00424B7F  |.  0FB655 FA        MOVZX EDX,BYTE PTR SS:[EBP-6]  
00424B83  |.  0FB645 F5        MOVZX EAX,BYTE PTR SS:[EBP-B]  
00424B87  |.  83E8 30          SUB EAX,30  
00424B8A  |.  3BD0            CMP EDX,EAX  
00424B8C  |.  75 0F           JNZ SHORT VideoCon.00424B9D         ;  码 4  
00424B8E  |.  0FB64D F4        MOVZX ECX,BYTE PTR SS:[EBP-C]  
00424B92  |.  0FB655 F9        MOVZX EDX,BYTE PTR SS:[EBP-7]  
00424B96  |.  83EA 30          SUB EDX,30  
00424B99  |.  3BCA            CMP ECX,EDX  
00424B9B  |.  74 58           JE SHORT VideoCon.00424BF5         ;  码 5  
00424B9D  |>  0FB645 FC        MOVZX EAX,BYTE PTR SS:[EBP-4]       ;  上面码 1 至码 5 如果有一个不符合，则运算出的注册码不对  
00424BA1  |.  83F8 35          CMP EAX,35                          ;  但是还有一次机会，通用注册码，51897629  
00424BA4  |.  0F85 A7000000   JNZ VideoCon.00424C51  
00424BAA  |.  0FB64D FD        MOVZX ECX,BYTE PTR SS:[EBP-3]  
00424BAE  |.  83F9 31          CMP ECX,31  
00424BB1  |.  0F85 9A000000   JNZ VideoCon.00424C51  
00424BB7  |.  0FB655 F6        MOVZX EDX,BYTE PTR SS:[EBP-A]  
00424BBB  |.  83FA 38          CMP EDX,38  
00424BBE  |.  0F85 8D000000   JNZ VideoCon.00424C51  
00424BC4  |.  0FB645 F5        MOVZX EAX,BYTE PTR SS:[EBP-B]  
00424BC8  |.  83F8 39          CMP EAX,39  
00424BCB  |.  0F85 80000000   JNZ VideoCon.00424C51  
00424BD1  |.  0FB64D F9        MOVZX ECX,BYTE PTR SS:[EBP-7]  
00424BD5  |.  83F9 37          CMP ECX,37  
00424BD8  |.  75 77           JNZ SHORT VideoCon.00424C51  
00424BDA  |.  0FB655 F7        MOVZX EDX,BYTE PTR SS:[EBP-9]  
00424BDE  |.  83FA 36          CMP EDX,36  
00424BE1  |.  75 6E           JNZ SHORT VideoCon.00424C51  
00424BE3  |.  0FB645 FE        MOVZX EAX,BYTE PTR SS:[EBP-2]  
00424BE7  |.  83F8 32          CMP EAX,32  
00424BEA  |.  75 65           JNZ SHORT VideoCon.00424C51  
00424BEC  |.  0FB64D FB        MOVZX ECX,BYTE PTR SS:[EBP-5]  
00424BF0  |.  83F9 39          CMP ECX,39  
00424BF3  |.  75 5C           JNZ SHORT VideoCon.00424C51         ;  也不是通用注册码，就注册失败，否则成功  
```

## 算法总结

- 用户名至少 2 位，注册码至少 8 位
- 第一位的 `ASCII` 码 `OR` `41`（十六进制），除以 `A`（十六进制），余数作为码 1
- 第二位的 `ASCII` 码 `OR` `56`（十六进制），除以 `A`（十六进制），余数作为码 2
- 第一位的 `ASCII` 码 `OR` `49`（十六进制），除以 `A`（十六进制），余数作为码 3
- 第二位的 `ASCII` 码 `OR` `43`（十六进制），除以 `A`（十六进制），余数作为码 4
- 用户名所有位数 `ASCII` 码累加，除以 `A`，余数作为码 5
- 码 1 至码 5 分别与注册码前五位比较，全同则注册成功
- 否则判断注册码前八位是否为通用注册码 `51897629`，是则注册成功，否则注册失败

### 示例

- 用户名：`alifriend`
- 注册码：`76512XXXXXXXXXX`（`XX` 随便，注册码至少八位就行）

## 注册机

### C++ 代码

```cpp
#include<iostream.h>  

#include <stdio.h>  

void main()  
{  
int i=0,j=0;  

char name[100];  

gets(name);  

cout<<endl;  

cout<<(name[0]|65)%10;  

cout<<(name[1]|86)%10;  

cout<<(name[0]|73)%10;  

cout<<(name[1]|67)%10;  

while(name[i]!='\0')  
{  
j=j+name[i];  

i++;  

}  

cout<<j%10<<"000"<<endl;  

}  
```

### Delphi 代码

```delphi
procedure Tkeygen.genButton1Click(Sender: TObject);  

var  

s,s1,s2,s3,s4,s5,s6:String;  

begin  

s:=nameEdit.Text;  

if length(s)<1 then  

begin  

ShowMessage('请正确输入用户名！');  

exit;  

end;  

s1:=inttostr((ord(s[1]) or $41)mod $a);  

s2:=inttostr((ord(s[2]) or $56)mod $a);  

s3:=inttostr((ord(s[1]) or $49)mod $a);  

s4:=inttostr((ord(s[2]) or $43)mod $a);  

s5:=inttostr((ord(s[1])+ord(s[2])+ord(s[3])+ord(s[4])+ord(s[5])))mod $a);  

s6:=inttostr(random(9))+inttostr(random(3))+inttostr(random(6));//随便设置随机数  

codeEdit.Text:=s1+s2+s3+s4+s5+s6;  

end;  
```

`4dd466d0e1e0cc123afbe504da2462ca`
