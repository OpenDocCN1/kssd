
### 汇编代码分析

```
004635EE    .  8D95 48FDFE>lea edx,dword ptr ss:[ebp+FFFEFD48]  

004635F4    .  8B45 FC     mov eax,dword ptr ss:[ebp-4]  

004635F7    .  8B80 000300>mov eax,dword ptr ds:[eax+300]  

004635FD    .  E8 AA66FDFF call NVIDIA_G.00439CAC  //得到我们的假注册码  

00463602    .  8B85 48FDFE>mov eax,dword ptr ss:[ebp+FFFEFD48]  

00463608    .  50         push eax  

00463609    .  8D95 44FDFE>lea edx,dword ptr ss:[ebp+FFFEFD44]  

0046360F    .  B8 86170000 mov eax,1786  //这个是一个常数  

00463614    .  E8 CB55FAFF call NVIDIA_G.00408BE4 //转换成字符串“6022”  

00463619    .  8B85 44FDFE>mov eax,dword ptr ss:[ebp+FFFEFD44]  

0046361F    .  5A         pop edx  

00463620    .  E8 A316FAFF call NVIDIA_G.00404CC8 //验证我们的注册码是否含“6022”  

00463625    .  85C0       test eax,eax  
```

OK，这下知道了吧，你输入 `WDW6022` 就可以安装了，不过为了和我们刚才的注册码结合起来，我们可以用这个 `WDW222226-346E696C6022`。

### 总结

- 用户名：`4nil`
- 注册码：`WDW222226-346E696C6022`

另外在注册前要在 `WINDOWS` 目录下新建一个文件叫“`system.sys`”。

完美破解，谢谢观赏。

\------------------------------------------------------------------------

### 破解总结

注册码是通用的，所以注册机也不写，不过算法很简单的。可以自己看看。

\------------------------------------------------------------------------

### 版权声明

本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

注册机的过程是这样的:

```
len=strlen(name);  
printf("S/N:WDW2222%d-",26+len);  
for(int i=0;i<len;i++)  
        printf("%X",name[i]);  
printf("6022\n"); 
```

`4dd466d0e1e0cc123afbe504da2462ca`
