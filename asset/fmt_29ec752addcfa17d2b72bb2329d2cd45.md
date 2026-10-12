
```
:0040B011 8B7C2418                mov edi, dword ptr [esp+18]         <-EDI=假注册码   

:0040B015 33C0                    xor eax, eax   

:0040B017 85ED                    test ebp, ebp   

:0040B019 C70700000000            mov dword ptr [edi], 00000000       <-将[EDI]所指向的地址按 DW 格式清零   

:0040B01F 7617                    jbe 0040B038   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040B036(C)   

|                                                                   第五次循环   

:0040B021 8BC8                    mov ecx, eax---------------   

:0040B023 83E103                  and ecx, 00000003                 <-ECX=4 时，EAX=0   

:0040B026 8A1C39                  mov bl, byte ptr [ecx+edi]    

:0040B029 8D3439                  lea esi, dword ptr [ecx+edi]   
    
:0040B02C 8A0C10                  mov cl, byte ptr [eax+edx]      |此循环将中间数 4 前 4 位依次放入 EDI 所指向的   

:0040B02F 02D9                    add bl, cl                       |地址中，第 5 位数继续放入地址的第一位，但   

:0040B031 40                      inc eax     <-EAX 加 1，计数器  /  要加上此地址中原来存在的数；第 6 位放入地址   

:0040B032 3BC5                    cmp eax, ebp                    /  的第二位并加上原来存在的数，依此类推，直   

:0040B034 881E                    mov byte ptr [esi], bl          /   至中间数取完。得到的 4 位数的十进制就是注册码。   

:0040B036 72E9                    jb 0040B021_________________/ 结果：E4D9EC67=67ECD9E4=174375524 <-注册码:)   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040B01F(C)   

|   

:0040B038 5F                      pop edi   

:0040B039 5E                      pop esi   

:0040B03A 5D                      pop ebp   

:0040B03B 5B                      pop ebx   

:0040B03C C3                      ret                      <-返回 ESI=67ECD9E4 到主程序   
```

---

### 【算法总结】

通过上面对算法 `CALL` 的逐句分析我们可以看出，软件是通过我们输入的用户名与给定的字符表中的字符进行四次循环计算后得出一个中间数，然后把这个中间数进行按 4 位拆分进行相加计算，即如果中间数是七位，则把第一位与第五位相加，结果放入第一位地址；第二位与第六位相加，结果放入第二位地址，依此类推，直到中间数取完为止。最后得到一个 4 位数（16 进制的），这个数由算法 `CALL` 返回到主程序，与我们输入的假注册码的 16 进制进行比对。正确就写注册表，错误就提示注册码错误。

另，用户名超过 10 位注册码会是个负数，不过一样注册码，没事的。:)

---

### 【注册信息】

用 `Regmon` 可以监测到软件在注册成功后在注册里写入注册信息：

`HKCUSoftwareXnViewLicenseName`  <-用户名"xbb-NCG"

`HKCUSoftwareXnViewLicenseNumber`  <-注册码"1743575524"

取注册注册只需把 `LicenseName` 和 `LicenseNumber` 两个键值删除即可。

---

### 【注册机】

我的编程非常差，虽然能找追算法，可还不够能力用代码还原它，正在学习 C，争取能在以后能写出注册机。

xbb-NCG
2003.12.15 上午

`4dd466d0e1e0cc123afbe504da2462ca`
