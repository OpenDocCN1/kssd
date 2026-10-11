
比较 `CM_INDEX` 和 `HCELL_INDEX` 大小，它们正好是 2 倍的关系。

在 `Index->Signature == CM_KEY_FAST_LEAF || Index->Signature == CM_KEY_HASH_LEAF` 时，其对应的结构是：

```
typedef struct _CM_KEY_FAST_INDEX {

    USHORT      Signature;              // also type selector
    USHORT      Count;

    CM_INDEX    List[1];                // Variable sized array

} CM_KEY_FAST_INDEX, *PCM_KEY_FAST_INDEX;
```

而其他时候对应的结构是下面的。

```
typedef struct _CM_KEY_INDEX {

    USHORT      Signature;              // also type selector
    USHORT      Count;

    HCELL_INDEX List[1];                // Variable sized array

} CM_KEY_INDEX, *PCM_KEY_INDEX;
```

通过上面比较可以看出，在 `CM_KEY_FAST_LEAF` 和 `CM_KEY_HASH_LEAF` 这两种情况下，这里使用的是 `CM_INDEX` 索引。因此需要 `*2`。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | drv.rar |
| --- | --- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | HIVE 文件读写.rar |
