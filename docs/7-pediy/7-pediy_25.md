

# 一个另类的 loader.

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-7.htm`](https://www.pediy.com/kssd/pediy09/pediy09-7.htm)

> 以前一直在潜水，从论坛上学到了很多东西，也没有为论坛做个贡献。看到大侠们都是用 asm、c++、delphi 写的 loader。  
> 正巧在学习 python，看到有一个 debug 框架，于是用 python 写了个 loader.py.软件破解过程比较简单，就不献丑了。  
> 下面是 loader.py 源码。  
> #!/usr/bin/env python  
> #--\*-- coding =utf-8 --\*--  
>   
>   
> """  
> Author: <nmweizi@gmail.com>, 2007.1.6  
> crack xxxxx 11.0.2  
> xxxxx must install at C:\\\\Program Files\\\\SlickEdit\\win\\\\vs.exe  
> (bp\_address,mem\_address,s\_value,d\_value,times,delay,begintime)  
>   
> Usage:  
>     python loader.py  
> """  
>   
> import pydbg, time  
> import os,sys  
>   
> def busca\_pid(dbg, nombre):  
>     for pid,proc in dbg.enumerate\_processes():  
>             if proc.lower() == nombre.lower(): return pid  
>     return -1  
>   
> def modificaDados(dbg):  
>     global bp\_address,mem\_address,s\_value,d\_value,times,delay,begintime  
>     if dbg.read\_process\_memory(mem\_address,length=1) == s\_value and \\  
>         (time.clock()-begintime) >= delay:  
>         times = times -1   
>         if times == 0:  
>             dbg.write\_process\_memory(mem\_address,d\_value,length=1)  
>             print 'Patched!'  
>               
>             #TODO if use bp\_set(bp\_address) int 3 then del # at this line  
>             #dbg.bp\_del(bp\_address)   
>             dbg.bp\_del\_hw(bp\_address)   
>             print "Remove breakpoint %s ok!" % hex(bp\_address).upper()  
>     return pydbg.DBG\_CONTINUE  
>   
> def handler\_breakpoint (dbg):  
>     if dbg.first\_breakpoint:  
>         dbg.hide\_debugger()  
>         
>         #print "first\_breakpoint"  
>     if dbg.exception\_handler\_single\_step:  
>         modificaDados(dbg)  
>     return pydbg.DBG\_CONTINUE  
>   
> def handler\_event(dbg):  
>     if dbg.event\_handler\_create\_process:  
>         dbg.bp\_set\_hw(bp\_address,2,pydbg.HW\_EXECUTE) #not sucessed ,why ?  
>         #print 'CREATE\_PROCESS\_DEBUG\_EVENT'  
>     return pydbg.DBG\_CONTINUE  
>       
>   
>   
> if \_\_name\_\_ == '\_\_main\_\_':  
>     global dbg,bp\_address,mem\_address,s\_value,d\_value,times,delay,begintime  
>       
>     bp\_address,mem\_address,s\_value,d\_value,times,delay,begintime = \\  
>                     (0x406dba,0x406dbc,"\\x66","\\x67",1,0,time.clock())  
>     dbg=pydbg.pydbg()  
>     path = "C:\\\\Program Files\\\\SlickEdit\\win\\\\vs.exe"  
>     cmdline = ""  
>     if not os.path.exists(path):  
>         print "File '%s' not exist!" % path  
>         sys.exit(1)  
>       
>     dbg.set\_callback(pydbg.EXCEPTION\_BREAKPOINT, handler\_breakpoint)  
>       
>     #TODO if use bp\_set(bp\_address) int 3 then # this line  
>     dbg.set\_callback(pydbg.CREATE\_PROCESS\_DEBUG\_EVENT, handler\_event)    
>       
>     dbg.load(path,cmdline)  
>       
>     #TODO if use bp\_set(bp\_address) int 3 then del # at this line  
>     #dbg.bp\_set(bp\_address)     #int 3  
>   
>     #winsock\_recv     = dbg.func\_resolve("ws2\_32",  "recv")  
>     #winsock\_recvfrom = dbg.func\_resolve("ws2\_32",  "recvfrom")  
>     #dbg.bp\_set(winsock\_recvfrom)  
>     #dbg.set\_callback(pydbg.USER\_CALLBACK\_DEBUG\_EVENT,modificaDados)  
>     #dbg.set\_callback(pydbg.EXCEPTION\_SINGLE\_STEP,modificaDados)  
>       
>     dbg.run()  
>     #dbg.debug\_event\_loop() 

 4dd466d0e1e0cc123afbe504da2462ca