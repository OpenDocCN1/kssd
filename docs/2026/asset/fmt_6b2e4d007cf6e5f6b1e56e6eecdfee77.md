
```
   len,p :integer;

begin

    st:='87ae2401my69';

    repeat

          write('Please input the name:');

          readln(name);

          len:=length(name);

    until len>=6;

    ch:=st[(len mod 12)+1];

    code:='6582-'+ch;

    for p:=1 to len do

    if (p mod 2)=1 then

    begin

       ch:=name[p];

       ch:=chr(ord(ch) div 2+32);

       if (ch>'Z') and (ch<'a') then ch:=chr(ord(ch)+6);

       if (ch>'9') and (ch<'A') then ch:=chr(ord(ch)+8);

       code:=code+ch;

    end;

    writeln('Your Register Code is:',code);

    writeln;

    writeln('Crack by RoBa         Thank you');

end.
```

我实在不明白代码中出现很多次的`repnz` `scasb`是什么意思，请大侠指教。
