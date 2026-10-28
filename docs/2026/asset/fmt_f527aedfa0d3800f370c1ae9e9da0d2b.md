
```asm
                mov                hGoodsMap2,eax

                invoke        MapViewOfFile,hGoodsMap2,FILE\_MAP\_READ OR FILE\_MAP\_WRITE,0,0,0

                mov                lpGoodsView2,eax

                ;因为文件才创建 所以我们先写入自己预设一点数据

                invoke        lstrcpy,lpGoodsView2,addr szSaveS

                invoke        FlushViewOfFile,lpGoodsView2,sizeof szSaveS

        .endif

        .else;文件存在则打开map

                mov                hGoodsFile2,eax

                invoke        CreateFileMapping,hGoodsFile2,NULL,PAGE\_READWRITE,0,4096\*50,NULL

                mov                hGoodsMap2,eax

                invoke        MapViewOfFile,hGoodsMap2,FILE\_MAP\_READ OR FILE\_MAP\_WRITE,0,0,0

                mov                lpGoodsView2,eax

        .endif

        popad

        ret

\_CreateGoods        endp
```
;释放已打开的资源

```asm
\_goodsClose        proc

        pushad

        invoke        UnmapViewOfFile,lpGoodsView

        invoke        CloseHandle,hGoodsMap

        invoke        CloseHandle,hGoodsFile

        invoke        UnmapViewOfFile,lpGoodsView2

        invoke        CloseHandle,hGoodsMap2

        invoke        CloseHandle,hGoodsFile2

        popad

        ret

\_goodsClose        endp
```
;向List控件写入不需要丢弃的物品 (不扔掉的物品 每个字符之间用,分隔,最后一个以0结尾)

```asm
\_readGoods        proc

        local        @buf[30]:BYTE

        pushad

        invoke        SendDlgItemMessage,hWin1,IDC\_LIST1,LB\_RESETCONTENT,0,0;首先删除列表里面所有的项目

        mov        esi,lpGoodsView

        lea        edi,@buf

        mov        edx,0;用这个来标识是否有数据

gs1:

        mov        al,','

        cmp        BYTE ptr [esi],0

        je        gs2        ;字符如果为0了表示到了结尾 然后看edx中是否有数据有就取

        cmp        [esi],al

        jz        gs2;如果数据为','则表示这句完了可以插入控件了

        mov        al,[esi]

        mov        [edi],al

        inc        esi

        inc        edi

        inc        edx

        jmp        gs1

gs2:

        inc        esi

        mov        BYTE ptr [edi],0;把在整个字符最后面插入个0

        .if        edx

                invoke        SendDlgItemMessage,hWin1,IDC\_LIST1,LB\_INSERTSTRING,-1,addr @buf

        .else

                jmp        \_Ret

        .endif

        lea        edi,@buf;把edi又设置为开头重新取新数据

        mov        edx,0

        jmp        gs1

\_Ret:

        popad

        ret

\_readGoods        endp
```
;保存新物品名称进文件里面,同时更新了list控件

```asm
\_addGoods        proc

        local                @buf[256]:BYTE

        invoke        GetDlgItemText,hWin1,IDC\_ADDGOODS,addr @buf,sizeof @buf

        .if        eax;eax是读取到的名字长度。不为0就表示是字符喽

                ;查找指定字符串 没有找到返回0 找到返回查找到的字符后面的地址

                invoke        \_SearchString,lpGoodsView,addr @buf;

                .if        eax==0;如果没有相等的就添加

                        invoke        lstrlen,lpGoodsView

                        mov                esi,eax

                        add                esi,lpGoodsView;指向最后的字符0

                        .if                eax ;这里的eax是指的总字符的长度，如果长度不为0才需要添加',' 如果为0表示是第一个字符，就不需要添加','了

                                mov                al,','

                                mov                [esi],al;设置最后一个为','

                                inc                esi

                        .endif

                        invoke        lstrcpy,esi,addr @buf;这里esi是指向了最终的0

                        invoke        \_readGoods;添加了则更新一次控件

                        invoke        SendDlgItemMessage,hWin1,IDC\_LIST1,LB\_GETCOUNT,0,0;选中最后添加的一项

                        DEC        eax;选中是基于0

                        invoke        SendDlgItemMessage,hWin1,IDC\_LIST1,LB\_SETCURSEL,eax,0

                .else

                        invoke        MessageBox,hWin1,addr szErrGed,addr szTitle,MB\_OK OR MB\_ICONINFORMATION

                .endif

        .else

                invoke        MessageBox,hWin1,addr szErrAdd,addr szTitle,MB\_OK OR MB\_ICONINFORMATION

        .endif

        ret

\_addGoods        endp
```
;在文件里删除指定的物品名称,同时更新了list控件

```asm
\_delGoods        proc

        local                @buf[256]:BYTE

        invoke        SendDlgItemMessage,hWin1,IDC\_LIST1,LB\_GETCURSEL,0,0

        .if        eax!= LB\_ERR;eax是当前选中的索引 如果没有选中则是返回 LB\_ERR

                mov                ebx,eax

                invoke        SendDlgItemMessage,hWin1,IDC\_LIST1,LB\_GETTEXT,ebx,addr @buf

                ;查找指定字符串 没有找到返回0 找到返回查找到的字符后面的地址

                invoke        \_SearchString,lpGoodsView,addr @buf;

                .if        eax!=0;查找到了字符，开始移除

                        mov        esi,eax;指向该字符后面的从','开始的

                        mov        edi,eax;指向该字符后面的从','开始的

                        invoke        lstrlen,addr @buf;取得当前文字的长度

                        inc                eax;再+1 因为还有一个','需要去掉

                        .while        eax;把这段字符都设置为0

                                mov        BYTE ptr [edi],0

                                dec        eax

                                dec        edi

                        .endw

                        inc                esi;因为esi指向的当前','被设为0 所以要增加一下指向后面

                        inc                edi;edi已经被上面的设置指向了前面的',' 需要保留一个','所以也增加一次

                        invoke        lstrlen,esi;取后面的长度

                        .while        eax;有长度就复制过去

                                mov         cl,[esi]

                                mov        [edi],cl

                                mov        BYTE ptr [esi],0;然后本当前的就不需要保留 删除掉

                                inc        esi

                                inc        edi

                                dec        eax

                        .endw

                        invoke        lstrcat,lpGoodsView,esi;合并成新的

                        invoke        lstrlen,lpGoodsView;如果最后一个字符是',' 则去掉 在这防止 出错

                        .if                eax

                                add                eax,lpGoodsView

                                dec                eax

                                mov                bl,','

                                cmp                BYTE ptr [eax],bl

                                jnz                @F

                                mov                BYTE ptr [eax],0

                                @@:

                        .endif

                        invoke        \_readGoods;添加了则更新一次控件

                .endif

        .else
```
