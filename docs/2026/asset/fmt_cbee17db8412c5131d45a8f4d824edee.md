
```java
public static C1268b m38177a(C1267a aVar, C24153v vVar) {
    AppMethodBeat.m3378i(TXLiteAVCode.EVT_RTMP_PUSH_PUBLISH_START);

    C27985cr crVar = aVar.foR;

    if (10008 == C2977ae.hiZ && C2977ae.hja != 0) {
        C8953ab.m13557i(
            "MicroMsg.MessageSyncExtension",
            "dkmsgid  set svrmsgid %d -> %d",
            Long.valueOf(crVar.rCB),
            Integer.valueOf(C2977ae.hja)
        );
        crVar.rCB = (long) C2977ae.hja;
        C2977ae.hja = 0;
    }

    if (((C5985k) C2700g.m5022ac(C5985k.class)).ciy().mo10747kF(crVar.rCB)) {
        C8953ab.m13556i(
            "MicroMsg.MessageSyncExtension",
            "ignore, because reSync the deleted msg perhaps the IDC has change has swtiched"
        );
        AppMethodBeat.m3379o(TXLiteAVCode.EVT_RTMP_PUSH_PUBLISH_START);
        return null;
    }

    String a = C2976aa.m5550a(crVar.yCi);
    String a2 = C2976aa.m5550a(crVar.yCj);

    if (!a.equals(C2837u.akn()) || !a2.equals("newsapp") || crVar.oXG == 51) {
        C8953ab.m13557i(
            "MicroMsg.MessageSyncExtension",
            "dkAddMsg from:%s to:%s id:[%d,%d,%d] status:%d type:%d time:[%d %s] diff:%d imgstatus:%d imgbuf:%d src:%d push:%d content:%s",
            a,
            a2,
            Long.valueOf(crVar.rCB),
            Integer.valueOf(crVar.rCz),
            Integer.valueOf(crVar.yCp),
            Integer.valueOf(crVar.kVf),
            Integer.valueOf(crVar.oXG),
            Integer.valueOf(crVar.CreateTime),
            C9015bo.m13727nW((long) crVar.CreateTime),
            Long.valueOf(C9015bo.azo() - ((long) crVar.CreateTime)),
            Integer.valueOf(crVar.yCl),
            Integer.valueOf(C2976aa.m5553a(crVar.yCm, new byte[0]).length),
            Integer.valueOf(C9015bo.nullAsNil(crVar.yCn).length()),
            Integer.valueOf(C9015bo.nullAsNil(crVar.yCo).length()),
            C9015bo.atw(C2976aa.m5551a(crVar.yCk, ""))
        );

        C8953ab.m13557i(
            "MicroMsg.MessageSyncExtension",
            "parseMsgSource has been Deprecated by dk. at 20151218 [%s] %s ",
            crVar.yCn,
            ""
        );

        C24154w.m38170h(crVar);

        if (a.equals("readerapp")) {
            crVar.yCi = C2976aa.m5556wH("newsapp");
            crVar.oXG = 12399999;
        }

        if ((a.equals("blogapp") || a.equals("newsapp")) && crVar.oXG != 10002) {
            crVar.oXG = 12399999;
        }

        if (crVar.oXG == 52) {
            crVar.oXG = 1000052;
        }

        if (crVar.oXG == 53) {
            crVar.oXG = 1000053;
        }

        C21628bi.m34464c(aVar);

        boolean z = false;
        C1268b bVar = null;
        C1264e bG = C1266d.m3510bG(Integer.valueOf(crVar.oXG));

        if (bG == null) {
            bG = C1266d.m3510bG(a);
        }

        if (bG != null) {
            bVar = bG.mo5953b(aVar); // 这一句调用上面函数 mo5953b，传入 aVar
            C9124bi biVar = bVar == null ? null : bVar.cRE;

            if (biVar == null) {
                C8953ab.m13561w(
                    "MicroMsg.MessageSyncExtension",
                    "summerbadcr extension declared but skipped msg, type=%d, svrId=%d, MsgSeq=%d, createTime=%d, addMsgInfo=%s",
                    Integer.valueOf(crVar.oXG),
                    Long.valueOf(crVar.rCB),
                    Integer.valueOf(crVar.yCp),
                    Integer.valueOf(crVar.CreateTime),
                    aVar
                );
            } else if (!m38176UL(a)) {
                C8953ab.m13550d(
                    "MicroMsg.MessageSyncExtension",
                    " msg , id =" + biVar.field_msgId + "  " + vVar
                );

                if (biVar.field_msgId > 0 && vVar != null && bVar.gnh) {
                    vVar.mo10881a(biVar, crVar);
                }
            }

            z = true;
        }

        C24154w.m38168b(5, crVar);

        if (!z) {
            C8953ab.m13555f(
                "MicroMsg.MessageSyncExtension",
                "unknown add msg request, type=%d. drop now !!!",
                Integer.valueOf(crVar.oXG)
            );
        }

        AppMethodBeat.m3379o(TXLiteAVCode.EVT_RTMP_PUSH_PUBLISH_START);
        return bVar;
    }

    C8953ab.m13561w(
        "MicroMsg.MessageSyncExtension",
        "msgid:%d type:%d this fucking msg from mac weixin, someone send msg to newsapp at mac weixin, givp up.",
        Long.valueOf(crVar.rCB),
        Integer.valueOf(crVar.oXG)
    );

    AppMethodBeat.m3379o(TXLiteAVCode.EVT_RTMP_PUSH_PUBLISH_START);
    return null;
}
```

这里主要还是关注一下参数的传递，关键是 `C27985cr` 这个类，其他没啥好说的。

```java
public final void mo33997a(C28418vg vgVar, byte[] bArr, boolean z, C24153v vVar) {
    // (Lcom/tencent/mm/protocal/protobuf/vg;[BZLcom/tencent/mm/plugin/messenger/foundation/a/v;)V

    int i = 0;
    AppMethodBeat.m3378i(TXLiteAVCode.EVT_CAMERA_CLOSE);

    switch (vgVar.zbt) {
        case 5:
            C27985cr crVar = (C27985cr) new C27985cr().parseFrom(bArr); // 1

            if (crVar != null) {
                C1267a aVar = new C1267a(crVar, false, false, false); // 2

                m38177a(aVar, vVar); // 这句调用了上面 m38177a，aVar 继承自 bArr，C27985cr crVar = (C27985cr) new C27985cr().parseFrom(bArr);C1267a aVar = new C1267a(crVar, false, false, false);

                // 而这个 C27985cr 就是 com.tencent.p130mm.protocal.protobuf，我们已经触摸到协议层了，有兴趣可以去看 C27985cr 的数据结构，反复 hook 就能搞明白

                if (!aVar.gnc) {
                    C21637bj.m34487a(
                        C2976aa.m5550a(crVar.yCi),
                        crVar.rCB,
                        ((long) crVar.CreateTime) * 1000,
                        crVar.oXG
                    );
                }
            }

            AppMethodBeat.m3379o(TXLiteAVCode.EVT_CAMERA_CLOSE);
            return;

        case 8:
            C28483ya yaVar = (C28483ya) new C28483ya().parseFrom(bArr);
            String a = C2976aa.m5550a(yaVar.zdL);
            int i2 = yaVar.zdO;
            Cursor di = ((C5985k) C2700g.m5022ac(C5985k.class)).ciy().mo10730di(a, i2);

            if (di.moveToFirst()) {
                while (!di.isAfterLast()) {
                    C9124bi biVar = new C9124bi();
                    biVar.convertFrom(di);
                    C21628bi.m34468m(biVar);
                    di.moveToNext();
                }
            }

            di.close();
            ((C5985k) C2700g.m5022ac(C5985k.class)).ciy().mo10729dh(a, i2);
            AppMethodBeat.m3379o(TXLiteAVCode.EVT_CAMERA_CLOSE);
            return;

        case 9:
            C28490yh yhVar = (C28490yh) new C28490yh().parseFrom(bArr);
            LinkedList<Integer> linkedList = yhVar.zdR;

            while (true) {
                int i3 = i;

                if (i3 >= linkedList.size()) {
                    break;
                } else {
                    C21628bi.m34484y(
                        C2976aa.m5550a(yhVar.zdL),
                        (long) ((Integer) linkedList.get(i3)).intValue()
                    );
                    i = i3 + 1;
                }
            }
    }

    AppMethodBeat.m3379o(TXLiteAVCode.EVT_CAMERA_CLOSE);
}
```

在往上是 `com.tencent.mm.plugin.messenger.foundation.f.a(Lcom/tencent/mm/protocal/protobuf/vg;[BZ)V` = `mo36123a`，很简单，自己看看吧。

```java
public final void mo36123a(C28418vg vgVar, byte[] bArr, boolean z) {
    AppMethodBeat.m3378i(1062);

    C24149s By = C24150a.m38158By(vgVar.zbt);

    if (By != null) {
        try {
            By.mo33997a(vgVar, bArr, z, this.qvv); // 这一句调用前面 mo33997a，无脑传参，没啥意思
            AppMethodBeat.m3379o(1062);
        } catch (IOException e) {
            C8953ab.m13552e(
                "MicroMsg.SyncDoCmdExtensions",
                "docmd: parse protobuf error, " + e.getMessage()
            );

            RuntimeException runtimeException = new RuntimeException("docmd: parse protobuf error");
            AppMethodBeat.m3379o(1062);
            throw runtimeException;
        }
    } else {
        C8953ab.m13561w(
            "MicroMsg.SyncDoCmdExtensions",
            "SyncDoCmdExtension for cmd id [%s] is null.",
            Integer.valueOf(vgVar.zbt)
        );
        AppMethodBeat.m3379o(1062);
    }
}
```

在上 `com.tencent.mm.plugin.zero.c.a(Lcom/tencent/mm/protocal/protobuf/vg;Z)Z` = `mo40148a`，仍然是注意一下参数传递。其实每次参数传递和变换，就是把外层协议层层剥离的过程，每一次消耗一些参数进行判断或其他动作，大家应该很容易看明白。
