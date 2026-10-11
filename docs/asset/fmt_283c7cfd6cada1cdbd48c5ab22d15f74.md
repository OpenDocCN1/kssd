
### Visual Basic 代码示例

```
j = Asc(Mid(strTmp, i, 1))

strSN = strSN & CStr((107 * lngLen + j) \ i)

Next i

MsgBox "注册码为：" & strSN
```

### Delphi 代码示例

```
procedure TForm1.Button1Click(Sender: TObject);

var  //定义变量关键词

  name:string;    //定义存放注册名的字符串变量

  namelen:integer;    //定义注册名长度

  code:array of integer;  //定义存放注册码的动态数组，在用动态数组时，在程序中一定要分配空间

  i:integer;      //定义循环变量

  reg:string;    //注册码

begin

  name:=edit1.Text ; //取注册名

  namelen:=strlen(pchar(name));  //计算注册名长度

  setlength(code,namelen);    //给动态数组 `code` 分配数量为 `namelen` 长度的空间

  reg:='';               //初始化注册码

  for i:=1 to namelen do

  begin

    code[i-1]:= (107*namelen+ord(name[i])) div i;  //计算注册码

    reg:=reg+inttostr(code[i-1]);  //将注册码转换成字符串

  end;

  edit2.Text :=reg;    //显示注册码

end;
```

### BASIC 风格代码示例

```
acce "请输入用户名:" to N

L = Len(N)

If L <= 0 

    MessageBox("用户名不能为空！") 

    retu

EndIf

C = ""

For i = 1 To L 

    j = Asc(subs(N, i, 1)) 

    C = C+allt(Str(int((107*L + j)/i))) 

endf

MessageBox("注册码为：" +C)
```

### 汇编代码示例 (masm16kg.asm)

```
;==============masm16kg.asm===========

;编译方法(当然得用 `masm 5.0` 或其他 16 位编译器啦)：

;`masm masm16kg;`

;`link masm16kg;`

;`debug masm16kg.exe` --> `rcx` 将 `16F` 改为 `6F` --> `n masm16kg.com` --> `w` --> `q`

;运行你的作品吧。

code segment 

    assume cs:code,ds:code,ss:code 

    org 100h 

start: 

    mov ah,9 

    mov dx,offset Msg 

    int 21h 

    mov ah,0ah 

    mov dx,offset szName 

    int 21h 

    xor cx,cx 

    lea si,Buf 

    mov ax,107 

    mov bl,ReadLen 

    mul bl 

    mov bp,ax 

    mov ah,9 

    mov dx,offset Msg2 

    int 21h 

Next: 

    xor ax,ax 

    lodsb 

    add ax,bp 

    inc cx 

    xor dx,dx 

    div cx 

    push cx 

    xor cx,cx 

Next1: 

    test ax,ax 

    jz Disp 

    xor dx,dx 

    mov di,10 

    div di 

    add dl,'0' 

    push dx 

    inc cx 

    jmp Next1 

Disp: 

    pop dx 

    mov ah,2 

    int 21h 

    loop Disp 

    pop cx 

    cmp cl,bl 

    jl Next 

    int 20h 

Msg db "Name:$" 

Msg2 db 0ah,"Code:",24h 

szName Label byte 

NameLen  db 82 

ReadLen  db 80 

Buf      Label byte 

code ends 

    end start
```

`4dd466d0e1e0cc123afbe504da2462ca`
