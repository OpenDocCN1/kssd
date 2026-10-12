
```
const  
  OperateAryLen = 5;  

const  
  OperateAry : array[1..OperateAryLen] of Integer =($0D, $01, $14, $05,$02);  

function DoubleStr(Str: string): string;    //取字符串偶位数字符  

var  
  i: Integer;  

begin  
   Result := '';  
   for i := 2 to Length(Str) do  
   if i mod 2 = 0 then  
   Result := Result + Str[i];  
end;  

function  OperateStr(const aStr: string): string; //xor 运算  

var  
  i,n: Integer;  

begin  
  SetLength(Result, Length(aStr));  
  for i := 1 to Length(aStr) do begin  
    n := i mod OperateAryLen;  
    if n = 0 then  
    n := 5;  
    Result[i] := Char(Ord(astr[i]) xor OperateAry[n]);  
  end;  
end;  

function  OpeateStr(const aStr: string): string;　//or 运算  

var  
  i,n: Integer;  

begin  
  SetLength(Result, Length(aStr));  
  for i := 1 to Length(aStr) do begin  
    n := i mod OperateAryLen;  
    if n = 0 then  
    n := 5;  
    Result[i] := Char(Ord(aStr[i]) or OperateAry[n]);  
  end;  
end;  

function StrToBack(bStr: string): string;   //将字符串倒转过来  

  var i:integer;  

begin  
    for i:=1 to length(bStr) do  
    begin  
    Result :=bStr[i]+Result;  
    end;  
end;  

function SiftStr(cStr: string): string;  //过滤字符串  

  var i,j:integer;  

begin  
    Result:='';  
    j:=Length(cStr);  
    for i:=0 to j do  
    begin  
    if cStr[i] in ['0'..'9','a'..'z','A'..'Z','~','!',  
    '@','#','$','%','^','&','\*','(',')','\[','\]','{','}','-','\+'] then  
    Result:=Result+cStr[i];  
    end;  
end;  

procedure TForm1.Button1Click(Sender: TObject);  

var s,s1,s2,s3,s4:string;  

begin  
     If Length(edit1.text) =0 Then exit  
     else  
     s:=edit1.text+'dflk5632129T.cOm'+DoubleStr(edit1.Text+'dflk5632129T.cOm');  
     s1:=  OperateStr(s);  
     s2:=  StrToBack(OpeateStr(s));  
     s3:=  concat(s2,s1);  
     s4:=  SiftStr(s3);  
     edit2.text:=copy(s4,1,10);  

end; 

 4dd466d0e1e0cc123afbe504da2462ca
```
