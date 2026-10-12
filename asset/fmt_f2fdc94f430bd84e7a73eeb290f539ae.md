
`00CA536A    8B03            MOV EAX,DWORD PTR DS:[EBX]`  

修改完后 `ALT+M` 打开内存页面，在 `text` 段下断:

![](img/370e69efd132006ffcdc6d0d7a6dbaf0.jpg)  

下断完毕 `F9` 就到 `OEP` 处：  

`004014AC    6D              INS DWORD PTR ES:[EDI],DX                ; 中断在这里，虽然到这里代码不能用，但可以获取到正确的 IAT`  
`004014AD    66:16           PUSH SS`  

……  

断下后用 `ImportREC` 对 `PROCESS` 进行 `IAT` 修复操作，`IAT` 的开始地址用我们上面记下的 `533000-400000=133000`,大小可以写大一点(当然你也可以计算的).cut 无效的指针再 `FIXDUMP` 就行了。  

修复 `IAT` 完毕(注：如果是 `3.7X` 或以上版本还有乱序，操作多一点)。  

### 第四部分：修复程序/优化  

程序现在已经基本搞完了，现在做一点收尾的工作。关掉两个 `OD`，用 `lordpe` 改回 `OEP` 的代码。把没有用的段清除掉然后用 `LordPE` `FIXDUMP` 一下：  

Finished! .  

Greetz:  

`Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!`  

`By loveboom[DFCG][FCG][US]`  

`Email:loveboom#163.com`  

`Date:3/27/2006 4:57:00 PM` 

`4dd466d0e1e0cc123afbe504da2462ca`
