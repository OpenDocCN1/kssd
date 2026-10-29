
```java
public final boolean mo40148a(C28418vg vgVar, boolean z) {

     AppMethodBeat.m3378i(58774);

  if (!C2700g.aaU()) {

         C8953ab.m13552e("MicroMsg.SyncDoCmdDelegate", "account storage disabled, discard all commands");

         AppMethodBeat.m3379o(58774);

  return false;

     }

  long azp = C9015bo.azp();

  byte[] a = C2976aa.m5552a(vgVar.zbu);//1

  C8953ab.m13557i("MicroMsg.SyncDoCmdDelegate", "doCmd %d cmdid:%d 
buf:%d thr:[%d]", Long.valueOf(azp), Integer.valueOf(vgVar.zbt), 
Integer.valueOf(C9015bo.m13690co(a)), 
Long.valueOf(Thread.currentThread().getId()));

  if (C9015bo.m13689cn(a)) {

         C8953ab.m13552e("MicroMsg.SyncDoCmdDelegate", "docmd: no protobuf found.");

         AppMethodBeat.m3379o(58774);

  return false;

     }

  try {

  if (this.xUP != null) {

  this.xUP.mo36123a(vgVar, a, z);//这句调用mo36123a，byte[] a = C2976aa.m5552a(vgVar.zbu);

  }

         C8953ab.m13557i("MicroMsg.SyncDoCmdDelegate", "doCmd FIN %d 
cmdid:%d Time:%d", Long.valueOf(azp), Integer.valueOf(vgVar.zbt), 
Long.valueOf(C9015bo.m13715hn(azp)));

         AppMethodBeat.m3379o(58774);

  return true;

     } catch (Exception e) {

         C8953ab.printErrStackTrace("MicroMsg.SyncDoCmdDelegate", e, "", new Object[0]);

         AppMethodBeat.m3379o(58774);

  return false;

     }

 }
```

在往上`com.tencent.mm.modelmulti.o$a$1.onTimerExpired`，明显是判断超时了怎么办，在上面就是一些系统调用了，到此为止了

```java
public final boolean onTimerExpired() {

     AppMethodBeat.m3378i(58399);

  if (C2700g.aaU() && !C2586a.aaa()) {

         C2700g.aba();

  if (C2700g.aaZ() != null) {

             C2700g.aba();

  if (C2700g.aaZ().aaH() != null) {

                 LinkedList<C28418vg> linkedList = C21737a.this.gDu.yUs.kUI;

                 C27641c cVar = new C27641c();

                 cVar.mo40149cN(C21737a.this.gDw);

  long azp = C9015bo.azp();

  while (C21737a.this.gCe < linkedList.size()) {

                     linkedList.size();

  if (!cVar.mo40148a((C28418vg) linkedList.get(C21737a.this.gCe), false)) {

                         C6339e.sxI.idkeyStat(99, 46, 1, false);

                     }

                     C21737a.this.gCe++;

  long hn = C9015bo.m13715hn(azp);

                     C8953ab.m13557i("MicroMsg.SyncService", 
"processResp %s time:%s size:%s index:%s", C21737a.this.gDw, 
Long.valueOf(hn), Integer.valueOf(linkedList.size()), 
Integer.valueOf(C21737a.this.gCe - 1));

  if (hn >= 500) {

  break;

                     }

                 }

                 cVar.mo40150cO(C21737a.this.gDw);

  if (C21737a.this.gCe < linkedList.size()) {

                     C8953ab.m13557i("MicroMsg.SyncService", 
"processResp %s time:%s size:%s index:%s Shold Continue.", 
C21737a.this.gDw, Long.valueOf(C9015bo.m13715hn(azp)), 
Integer.valueOf(linkedList.size()), Integer.valueOf(C21737a.this.gCe - 
1));

                     AppMethodBeat.m3379o(58399);

  return true;

                 }

                 C21733o.m34801a(C21737a.this.gDt, C21737a.this.gDu, C21737a.this.gDw);

                 C21737a.this.gDv.mo34011no(linkedList.size());

                 AppMethodBeat.m3379o(58399);

  return false;

             }

         }

     }
```

最后，简单总结一下：

### 1
从`com.tencent.mm.sdk.g.c.c$1$1.run`开始，进入系统调用，开启`loop`消息循环，通过厂商自己重写的`dispatchMessage`和`handleMessage`处理`message`。

### 2
进入`com.tencent.mm.modelmulti.o$a$1.onTimerExpired`，判断是否超时，记得超时了好像撤回的选项就没有了，这个大家自己跟吧

### 3
从`com.tencent.mm.r.b.b(SourceFile:40)`  
`com.tencent.mm.plugin.messenger.foundation.c.a(SourceFile:165)`  
`com.tencent.mm.plugin.messenger.foundation.c.a(SourceFile:1059)`  
`com.tencent.mm.plugin.messenger.foundation.f.a(SourceFile:118)`  
都是处理上层的`protobuf`，有的直接继承参数，有的剥离外层在继承

### 4
函数`com.tencent.mm.model.f.a(SourceFile:145)`和`com.tencent.mm.model.f.a(SourceFile:352)`  
通过上层函数参数`aVar`的数据结构生成下层函数的`map`参数，这个参数很关键，保存了  
`(".sysmsg.revokemsg.newmsgid")`和`(".sysmsg.revokemsg.replacemsg")`两个键值。这里也很适合`hook`获得这个`map`修改  
`newmsgid`和  
`replacemsg`完成防撤回

### 5
进入本文的关键函数`com.tencent.mm.model.f.a(SourceFile:145)` `m34535a`和`com.tencent.mm.model.f.a(SourceFile:352)`  
`mo7343a`；函数`mo7343a`调用了`m34535a`，直接`hook`这个函数，当`p0=revokemsg`时直接返回就可以屏蔽掉撤回了。

### 6
进入数据库封装的上层`com.tencent.mm.cf.f.update(SourceFile:774)`和`com.tencent.mm.cf.h.update(SourceFile:601)`  
，这里主要注意`sqlite`实例的获得，分为加密和不加密，中间还有一些优化的功能。

### 7
进入最终更新数据库的`com.tencent.wcdb.database.SQLiteDatabase.updateWithOnConflict(Native Method)`  
`com.tencent.wcdb.database.SQLiteDatabase.update(SourceFile:1726)`，更新数据库，完成功能

ps：中间有一些类和函数我没有仔细去看，可能会有些错误，但大致流程应该没啥问题，今天就开始上班了，时间比较急，中间整理的有些乱，大家见谅吧，干活去喽！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-3-7 12:35

被挤蹭菌衣编辑，原因：

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
