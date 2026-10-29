
# 脱壳记录

## 代码片段

```c
v60 = v3;

v59 = v95;

v57 = "mZips";

}

goto LABEL_91;

}

v48 = (unk_6226A980)(v3, "dalvik/system/DexPathList$Element");

v49 = (unk_6226A9AE)(v3, v48, "<init>", "(Ljava/io/File;Ljava/util/zip/ZipFile;Ldalvik/system/DexFile;)V");

if ( v49
  || ((unk_6226C8AC)(v3),
      (v49 = (unk_6226A9AE)(v3, v48, "<init>", "(Ljava/io/File;Ljava/io/File;Ldalvik/system/DexFile;)V")) != 0) )
{
  v50 = (unk_6226A994)(v3, v48, v49);
}
else
{
  (unk_6226C8AC)(v3);
  if ( !(unk_6226A9AE)(v3, v48, "<init>", "(Ljava/io/File;ZLjava/io/File;Ldalvik/system/DexFile;)V") )
  {
    v51 = 0;
LABEL_87:
    v52 = (unk_6226A980)(v3, "dalvik/system/DexPathList$Element");
    v53 = (unk_6226C928)(v3, v96 + 1, v52, 0);
    (unk_6226A9E2)(v3, v53, 0, v51);
    for ( m = 0; m < v96; (unk_6226A9E2)(v3, v53, m, v55) )
      v55 = (unk_6226A9D4)(v3, v101, m++);
    v56 = *(v9 + 212);
    v57 = *(v9 + 216);
    v58 = *(v9 + 220);
    v59 = v106;
    v60 = v3;
LABEL_91:
    (unk_6226B008)(v60, v59, v56, v57);
    dword_62283728 = (*(*v3 + 84))(v3, v94);
    Log(3, "SecShell", &unk_6227F13A);
    (unk_6226E6A4)(&v114);
    (unk_6226E6A4)(&v112);
    goto LABEL_108;
  }
  v50 = (unk_6226A994)(v3, v48);
}
v51 = v50;
goto LABEL_87;
}
*(v38 + 4) = 1;
*(v38 + 8) = v45;
if ( v47 == 10 )
  *(v85 + 16) = v105;
}
*(v38 + 12) = 0;
goto LABEL_78;
}
Log(3, "SecShell", "wrong code");
v35 = (unk_6227B32C)("/dev/zero", 2);
v36 = (unk_6227B44C)(0, v88);
(unk_6227B38C)(v35);
if ( v36 )
{
  (unk_6227B28C)(v36, v108, v88);
  v105 = v36;
  goto LABEL_53;
}
v21 = "SecShell";
v22 = "mmap fail";
LABEL_51:
Log(3, v21, v22);
LABEL_108:
result = v110;
if ( v119 != *v110 )
  sub_6227B24C(v110);
return result;
}
```

## 说明

文档写的有些流水账，只是调试过程中的一些记录，各位看官勿喷。如有误欢迎交流与指点。

同时脱壳过程中参考了论坛上各位大佬们的帖子，学习了许多。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 附件

上传的附件：

- [记一次某锁机APP脱壳过程.pdf](javascript:void(0);)

  （963.54kb，89次下载）
