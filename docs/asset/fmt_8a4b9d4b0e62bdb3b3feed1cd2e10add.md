
```assembly
.text:26047054             mov     eax, [ebp+var_C]
.text:26047057             xor     ecx, ecx
.text:26047059             mov     cl, [eax+0F4h]  //这里是有重复字符的那组的索引
.text:2604705F             and     ecx, 2           //屏蔽低 2 位
.text:26047062             test    ecx, ecx
.text:26047064             jz      short loc_26047075 //不能为 0。这说明只能第三组注册码里有重复
```

看到这里就明白了。有重复字符那组的累加和要等于`64h`，并且只能是第三组注册码。:)

上面还有个`CALL`没有分析，在这个位置啦：

```assembly
.text:2604EC0E             push    ecx
.text:2604EC0F             call    sub_2604E130     //这里先略过
```

这个我没有去分析，猜测应该是不同协议类型的验证。等有时间再分析了。

反正是老外的软件，最后给一组能用的注册码：`CB5W-MK5S-JUUG`

注册名用你自己的就可以了。

【全文完】

`4dd466d0e1e0cc123afbe504da2462ca`
