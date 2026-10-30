
```
Breakpoint 1 hit
eax=00000003 ebx=0013ef98 ecx=052e9018 edx=109f5228 esi=051c5718 edi=00000000
eip=10499052 esp=0013eca8 ebp=0013ecc8 iopl=0         nv up eiplzrnapenc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00200246
xul!nsObjectLoadingContent::OnChannelRedirect:
10499052 8b4c2408        movecx,dwordptr [esp+8] ss:0023:0013ecb0=00000000
0:000>ddesp+8
0013ecb0  00000000 057369f0 00000000 00897640//第一个参数null 第二个参数new Object，第三个参数0记住这个057369f0,后面用到
0013ecc0  00000000 0013ef68 0013ef68 10129438
0013ecd0  052e9018 00000003 00000003 0013ed80
0013ece0  005092f0 054f9c80 00000000 00379b06
0013ecf0  06060b0c 00000000 030c0ecc 008af0e0
0013ed00  06799403 051c5718 00379def 07122024
0013ed10  052e9018 00000003 0013ed80 00000001
0013ed20  087d1934 80570009 07dcd340 05736460
```

这个函数汇编代码
```
xul!nsObjectLoadingContent::OnChannelRedirect:
10499052 8b4c2408        movecx,dwordptr [esp+8]//aOldChannel
10499056 56              push    esi
10499057 8b742408        movesi,dwordptr [esp+8]//aNewChannel
1049905b 3b4e1c          cmpecx,dwordptr [esi+1Ch]//mChannel=0
1049905e 7407          je      xul!nsObjectLoadingContent::OnChannelRedirect+0x15 (10499067)//跳
10499060 b802004b80      mov     eax,804B0002h
10499065 eb1a            jmpxul!nsObjectLoadingContent::OnChannelRedirect+0x2f (10499081)
10499067 8b4624          moveax,dwordptr [esi+24h] ds:0023:052e903c=00000000//mClassifier
1049906a 85c0            test    eax,eax
1049906c 57              push    edi
1049906d 8b7c2414        movedi,dwordptr [esp+14h]
10499071 7408            je      xul!nsObjectLoadingContent::OnChannelRedirect+0x29 (1049907b)//跳
10499073 8b10            movedx,dwordptr [eax]
10499075 57              push    edi
10499076 51              push    ecx
10499077 50              push    eax
10499078 ff5210          call    dwordptr [edx+10h]
1049907b 897e1c          movdwordptr [esi+1Ch],edi //0写入mChannel
```

对应源码：
```
NS_IMETHODIMP
nsObjectLoadingContent::OnChannelRedirect(nsIChannel *aOldChannel,
nsIChannel *aNewChannel,
                                          PRUint32    aFlags)
{
  // If we're already busy with a new load, cancel the redirect
if (aOldChannel != mChannel) {
return NS_BINDING_ABORTED;
  }
if (mClassifier) {
mClassifier->OnRedirect(aOldChannel, aNewChannel);
  }
mChannel = aNewChannel;
return NS_OK;
}
```

从上述源码可知,利用new object来替换`mChannel`,而new Object这个对象会在js函数onChannelRedirect执行完就会被释放(可以再这个js函数执行后紧跟一个申请内存的操作,大小和new object一样,即可站位mChannel :).

```
F5继续执行,firefox里面有2个LoadObject函数
先执行原型为nsresult
nsObjectLoadingContent::LoadObject(nsIURI* aURI,
PRBoolaNotify,
constnsCString&aTypeHint,
PRBoolaForceLoad)
的LoadObject,执行后,又跳到原型为nsresult
nsObjectLoadingContent::LoadObject(constnsAString&aURI,
PRBoolaNotify,
constnsCString&aTypeHint,
PRBoolaForceLoad)
函数,再次跳到nsObjectLoadingContent::LoadObject(nsIURI* aURI,
PRBoolaNotify,
constnsCString&aTypeHint,
PRBoolaForceLoad)
函数执行:
nsresult
nsObjectLoadingContent::LoadObject(nsIURI* aURI,
PRBoolaNotify,
constnsCString&aTypeHint,
PRBoolaForceLoad)

{	…..
  // From here on, we will always change the content. This means that a
  // possibly-loading channel should be aborted.
if (mChannel) {
LOG(("OBJLC [%p]: Cancelling existing load\n", this));

if (mClassifier) {
mClassifier->Cancel();
mClassifier = nsnull;
    }

    // These three statements are carefully ordered:
    // - onStopRequest should get a channel whose status is the same as the
    //   status argument
    // - onStopRequest must get a non-null channel
mChannel->Cancel(NS_BINDING_ABORTED);//exploit
if (mFinalListener) {
      // NOTE: Since mFinalListener is only set in onStartRequest, which takes
      // care of calling mFinalListener->OnStartRequest, mFinalListener is only
      // non-null here if onStartRequest was already called.
mFinalListener->OnStopRequest(mChannel, nsnull, NS_BINDING_ABORTED);
mFinalListener = nsnull;
    }
mChannel = nsnull;
  })
	…
}
```

汇编代码:
```
1080df61 837b5000        cmpdwordptr [ebx+50h],0
1080df65 7456            je      xul!nsObjectLoadingContent::LoadObject+0x137 (1080dfbd)
1080df67 8d7358          lea     esi,[ebx+58h]
1080df6a 8b06            moveax,dwordptr [esi]  ds:0023:052e903c=00000000
1080df6c 85c0            test    eax,eax
1080df6e 740f            je      xul!nsObjectLoadingContent::LoadObject+0xf9 (1080df7f)
1080df70 8b08            movecx,dwordptr [eax]
1080df72 50              push    eax
1080df73 ff5114          call    dwordptr [ecx+14h]
1080df76 33d2            xoredx,edx
1080df78 8bce            movecx,esi
1080df7a e841da91ff      call    xul!nsCOMPtr_base::assign_with_AddRef (1012b9c0)
1080df7f 8b4350          moveax,dwordptr [ebx+50h]
1080df82 8b08            movecx,dwordptr [eax]
1080df84 be02004b80      mov     esi,804B0002h
1080df89 56              push    esi
1080df8a 50              push    eax
1080df8b ff5118          call    dwordptr [ecx+18h]
1080df8e 8d4b38          lea     ecx,[ebx+38h]
1080df91 e85210b0ff      call    xul!nsCOMPtr<nsIPrefBranch>::operator nsIPrefBranch * (1030efe8)
….
```

单步日志
