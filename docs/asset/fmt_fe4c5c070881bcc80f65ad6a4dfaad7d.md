
```
:00424D78 31 2F 35 28 21 73 64 24   1/5(!sd$

:00424D80 4D 71 2E 7B 73 5D 2B 73   Mq.{s\]+s

:00424D88 46 6A 74 4B 70 7A 53 64   FjtKpzSd

:00424D90 74 7A 6F 58 71 6D 62 5E   tzoXqmb^

:00424D98 41 6C 40 64 76 3A 73 3F   Al@dv:s?

:00424DA0 78 2F 00 00 7C 62 21 70   x/..|b!p

:00424DA8 7A 2A 6C 73 3B 72 6E 7C   z*ls;rn|

:00424DB0 6C 66 24 76 69 5E 41 78   lf$vi^Ax

:00424DB8 70 65 29 72 78 35 61 69   pe)rx5ai

:00424DC0 63 26 39 2F 32 6D 35 6C   c&9/2m5l

:00424DC8 73 69 34 40 30 64 6D 5A   si4@0dmZ

:00424DD0 77 39 34 63 6D 71 70 66   w94cmqpf

:00424DD8 68 77 00 00 55 52 4C 3A   hw..URL:

:00424DE0 20 25 73 0D 0D 43 61 6E    %s..Can
```

一个注册机: (`Borland Pascal 7.0`) 正在努力学 `Win32` 汇编, 下一次一定要用汇编写!

```
Program CrackDllShow;

var st1,st2,name,company:string;
    code :real; //好像没有 32 位无符号数,只好用 REAL 代替

Function R(st:string):real;
var p:integer;
    c1,c2,c3,s:real;

begin
    s:=0;
    for p:=1 to length(st) do
    begin
        c1:=ord(st1[p]);
        c2:=ord(st2[p+length(st)]);
        c3:=ord(st[p]);
        s:=s+c1*c2*c3*p;
    end;

    R:=s;
end;

begin
    st1:='|b!pz*ls;rn|lf$vi^Axpe)rx5aic&9/2m5lsi4@0dmZw94cmqpfhw';
    st2:='#serB&nz|mfM1/5(!sd$Mq'; //这里我只截了一段,因为还有一个不可显示的字符很麻烦
                                      //用户名和公司名不要太长就可以了
    write('please input your name:');
    readln(name);
    write('please input your company:');
    readln(company);
    code:=3684031485.0;  //(DB95DB95h or 378h)的结果
    code:=code+R(name)+R(company);
    writeln('code:',code:0:0);
    writeln('Crack by RoBa       Thank you');
end.
```

一个可用的注册码:

```
NAME: RoBa
ORG : China
CODE: 3707936057
```

祝全国的 `CRACKER` 节日快乐!

`4dd466d0e1e0cc123afbe504da2462ca`
