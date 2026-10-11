
# Acprotect1.0x-1.2x 脱壳脚本

> 原文：[`www.pediy.com/kssd/pediy06/pediy6917.htm`](https://www.pediy.com/kssd/pediy06/pediy6917.htm)

历时 7 小时写成，要兼容各版本 `Ac`、语言、处理方法，哎，累。  

环境，`Win2k/xp`，`UnkillOD`，`Flyod1.10`，`脚本器 0.92`，忽略除内存异常外的全部异常。  

```
// Acprotect OEP or FOEP and Patch IAT  v0.3  

// 作者: Mr.David  

// 主页: [www.chinadfcg.com](http://www.chinadfcg.com)  

msg "请设置 OD 异常设置不忽略内存异常，其它全部忽略，然后从菜单处继续运行脚本"  

pause  

dbh  //隐藏调试器  

var cbase  

gmi eip, CODEBASE  

mov cbase, $RESULT     
    
log cbase            //记录代码段基地址,一般是 401000  

var k  

var imgbase  

gmi eip,MODULEBASE     //模块基地址 400000  

mov imgbase,$RESULT  

mov k,imgbase  

add k,3C               //40003C  

mov k,[k]  

add k,imgbase         

add k,f8               //第一区段名  

log k  

add k,8               //地址加 8 字节的偏移就是区段大小       
    
mov k,[k]            //401000 处第一区段大小  

log k         

var addr1           //通用临时变量  

var addr2  

mov addr2,ebp      //保存 ebp=12fff0 判断后面是否到达 OEP 或 FOEP  

var addr3  

sub ebp,30        //实际取 ebp=12ffc0 判断后面是否到达 OEP 或 FOEP  

mov addr3,ebp  

add ebp,30      //修改了寄存器值得还原  

var addr6      //Replace Code 判断变量  

Msg "请耐心等候 IAT 处理"  

gpa "GlobalAlloc","kernel32.dll"  

mov addr1,$RESULT                    //捷径 API 断点 GlobalAlloc  

bp addr1  

run  

bc addr1    //Clear break point  //取消断点  

gpa "GetProcAddress","kernel32.dll"  

mov addr1,$RESULT                   //AC 敏感捷径 API 断点 GetProcAddress  

add addr1,5                        //GetProcAddress+5  

bp addr1     

eoe end2     //Acprotect1.10 Build123 之 C 语言篇的 IAT 处理在 INT1 中断前面，设置 INT1 陷阱             

esto  

loop:                      //循环标记  

Bc addr1  

rtu                      //Alt+F9  

cmp eip,70000000       //判断是否在系统领空  

ja loop               //地址大于 70000000 则循环  

find eip,#0BC0#    //查找特征指令 or eax,eax 以定位处理 MessageBoxA 的位置  

                    //有些版本这里返回后还要 bp GetProcAddress+5 中断一次后返回才处理 IAT,如记事本                               

cmp eip,$RESULT  //如果当前返回指令是 or eax,eax 则需要继续中断处理  

je BGPA  //跳到标签 BGPA 即是 Bp GetProcAddress 缩写  

Back1:  //返回或正常流程  

find eip,#750A#    //特征指令   

mov addr1,$RESULT 

repl addr1, #750A#, #EB??#,10      //有病治病，无病强身，处理 MessageBoxA  

find eip,#7F??#    //特征指令     //jg xxxxx 不确定调转路径用 7F??，处理 IAT  

mov addr1,$RESULT 

repl addr1, #7F??#, #EB??#,10     //有病治病，无病强身  //这里结束处理 IAT 是否加密  

//下面开始分界，有些 Acprotect 有 Replace Code，有些没有，给脚本编写加大难度，这里处理方法不同。  

cmp addr6,1  //如果确认没有 Replace Code,则直接跳到取 OEP 位置代码处  

je label69  

gpa "GlobalAlloc","kernel32.dll"  

mov addr1,$RESULT                    //如果要寻找 Replace Code 处理位置就得继续下捷径 API 断点 GlobalAlloc  

bp addr1  

eoe end  

run  //INT1 陷阱,异常则去 end 处理  

bpmc //内存不中断执行这里则清除内存断点。  

bc addr1  //清除 GlobalAlloc 断点。  

rtu  

label32: //这里我们寻找 Replace Code 位置。  

find eip,#8BF8#    //如果当前返回点代码就是 mov edi,eax，则可确认就是 Replace Code 位置，否则循环。  

cmp eip,$RESULT 

je label77       //如果发现 mov edi,eax 则跳出循环。  

label66:  

gpa "GlobalAlloc","kernel32.dll"  

mov addr1,$RESULT                    //捷径 API 断点 GlobalAlloc  

bp addr1  

run  

rtu  

jmp label32  

label77:  

bc addr1  

Msg "IAT 已经修复，想狸猫换太子吗?请修改信息框处 EAX 值，处理完从菜单处继续运行!"  

pause  

label69:  

bprm cbase, k //然后就是跳 OEP 或 FOEP 了,内存镜像断点  

esto //Shift+F9  

label444:  //这里我用一个条件判断 OEP OR FOEP，有些 AC 壳到 OEP 或 FOEP 时候 EBP 一般是 12fff0,或是 12ffc0,401000 处中断一般都是 OEP。  

cmp eip,401000           //汇编比较常用  

je label333  

cmp ebp,addr2         //12fff0 一般没有抽代码 

je label333  

cmp ebp,addr3       //12ffc0,抽代码常见  

je label333  

cmp ebp,12fff2    //12fff2,VB 类  

je label333  

var addr4  

add addr4,1  

cmp addr4,70   //定义循环上限，如果 Code 段执行 70 行代码仍然无法发现 OEP 则认为死循环。  

ja Sorry  

esto  

jmp label444  //循环直到条件成立，当然有些 AC 版本会跑飞。  

label333:  

Msg "IAT 已经修复，FOEP 请补上被盗字节"  

cmt eip,"OEP Or Next Shell To Get,Please dumped it,Enjoy!"  

bpmc  

ret  

BGPA:  //继续 GetProcAddress+5 中断  

gpa "GetProcAddress","kernel32.dll"  

mov addr1,$RESULT                    //捷径 API 断点 GetProcAddress+5  

add addr1,5  

bp addr1  

run  

bc addr1  

rtu  

jmp Back1  

end:     //INT1  

coe  

bprm 401000, k //内存镜像断点     //意外情况，没有 Replace Code 的程序，这时要跳 FOEP 了，得拦住  

bc addr1 //清除 GlobalAlloc 断点  

jmp label444  

Sorry:  

Msg "估计脚本陷入死循环，提前结束。"  

ret  

end2:   //INT1  

coe  

mov addr6,1 //如果 INT1 在 IAT 处理以前,这些版本的 AC 一般没有 Replace Code,赋标记 1 让他直接去内存镜像断点处。  

esto  

jmp loop
```

`4dd466d0e1e0cc123afbe504da2462ca`
