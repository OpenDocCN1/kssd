
```
                        invoke        lstrcat,addr szRegNum,addr szTmp3
                        mov        BYTE ptr [szRegNum+5],2dh
                        invoke        StrMidBuffer,addr szTmp1,6,5,addr szTmp3
                        invoke        lstrcat,addr szRegNum,addr szTmp3
                        mov        BYTE ptr [szRegNum+11],2dh
                        invoke        StrMidBuffer,addr szTmp1,0Bh,1,addr szTmp3
                        invoke        lstrcat,addr szRegNum,addr szTmp3
                        invoke        lstrcat,addr szRegNum,addr szStrCon
                        invoke        SetDlgItemText,hDlg,IDC_REG,addr szRegNum
                .endif
        .endif
        popad
        ret
GetRegKey        endp
```

【版权声明】   本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

---

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
