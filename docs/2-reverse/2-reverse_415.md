

# IDA 搜索 UNICODE 中文字符串 python 脚本

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-334.htm`](https://www.pediy.com/kssd/pediy09/pediy09-334.htm)

> # -\*- coding: utf-8 -\*-  
> #使用方法：SearchUnicodeString() 或 SearchUnicodeString(1) ＝》1 为地址加上注释  
> #脚本有些乱，请多包含  
> #虽然搜索出一些其实不是字符串的字符，但它符合 Unicode 的范围，实则没办法  
> #脚本作者为:Steve QQ:344843150  
>   
>   
> szPattern=ur'''\[  
>         #\\u3040-\\u318f| #日文平假名，及繁体注音符号  
>         \\u3300-\\u337f| #带圆括号象形字  
>         #\\u3400-\\u3d2d| #CJK 扩展 A  
>         \\u4e00-\\u9fff| #CJK 统一表意符号  
>         #\\uf900-\\ufaff| #CJK 兼容象形文字  
>         #\\uac00-\\ud7af\] #未知字符  
>         {1,}\[\\u0000\]  
>         '''  
> ##        szPattern=szPattern.encode("utf-16")  
> ##        szPattern="\[\\u4e00-\\u9fa5\]"  
> Pattern=re.compile(szPattern,re.UNICODE | re.VERBOSE)  
>   
>   
> def GetBytes(ea,size):  
>     bytes=""  
>     for byte\_ea in range(ea,ea+size):  
>         bytes+=chr(get\_byte(byte\_ea))  
>     return bytes  
>   
> def showUStr(ea,size,bCmt):  
>     if (ea!=BADADDR and size!=0):  
>         gbkstr=""  
>   
>         sFind=GetBytes(ea,size)  
>         if isData(GetFlags(ea)):  
>             if Pattern.match(unicode(sFind,'utf-16','ignore')):  
>                 #print "Find:0x%x" %ea  
>               
>             #    if get\_word(ea)>=0x4e00 and get\_word(ea)<=0x9fa5 : # and get\_word(ea+2)>=0x4e00 and get\_word(ea+2)<=0x9fa5 :  
>                     #if get\_word(get\_item\_end(ea)-2)==0:  
>                 try:  
>                     gbkstr=unicode(GetBytes(ea,size),'utf-16').encode('gbk')  
>                 except BaseException:  
>                     gbkstr=""  
>                 else:  
>                     if bCmt: set\_cmt(ea, gbkstr, 1)  
>     return gbkstr  
>   
> def SearchUStr(ea\_start,ea\_end,bCmt=0):  
>     """Search Unicode String"""  
>     ea=ea\_start  
>     while ea<ea\_end:  
> ##        print 'ea=0x%x'%(ea)  
>    
> ##        if isASCII(get\_flags\_novalue(ea)):  
> ##            #stype=typeinfo\_t()  
> ##            #stype.strtype=0  
> ##            #stype=get\_typeinfo(ea,0,FF\_DWRD,stype)  
> ##            stype=GetStringType(ea)  
> ##            code=stype  
> ##            print 'stype=%s'%code  
>   
>         size=get\_item\_size(ea)  
>         #print 'size:%d'%size  
>         if size>2:  
>             szUStr=showUStr(ea,size,bCmt)  
>             szUStr=szUStr.strip(' \\n\\0')  
>       #szUStr=szUStr.strip().strip('\\n').strip(chr(0xA)).strip('\\0')  
>             if szUStr<>'':  
>                 print 'ea=0x%x:%s'%(ea,szUStr)  
>               
> ##            if code==ASCSTR\_UNICODE or stype==ASCSTR\_ULEN2 or stype==ASCSTR\_ULEN4:  
> ##                delta=0  
> ##                if code==ASCSTR\_ULEN2:  
> ##                    delta=2  
> ##                if code==ASCSTR\_ULEN4:  
> ##                    delta=4  
> ##  
> ##                size=get\_item\_size(ea)  
> ##                print 'size:%d'%size  
> ##                if size>delta:  
> ##                    print showUStr(ea,size,0)  
> ##              
>         ea=NextHead(ea,ea\_end)  
>   
> #bCmt 为地址加上注释  
> def SearchUnicodeString(bCmt=0):  
>     """Seach Unicode String"""  
>     SearchUStr(MinEA(),MaxEA(),bCmt)  
>       
>   
>   
> 附件（请把 txt 后辍名改为 py 附件（编码 utf-8)）：unicode\_gen.txt  

 4dd466d0e1e0cc123afbe504da2462ca