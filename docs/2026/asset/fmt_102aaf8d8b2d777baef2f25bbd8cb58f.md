
```cpp
size\_t \_\_cdecl sub\_42940(\_DWORD \*fun3, BYTE \*user\_name, int user\_name\_len, char \*password)
{
v73 = \_\_readgsdword(0x14u);
v4 = -64;
v5 = 0;
do
v5 = \*((\_WORD \*)fun3 + (\*((unsigned \_\_int8 \*)fun3 + v4++ + 68) ^ HIBYTE(v5)) + 42) ^ (v5 << 8);
while ( v4 ); // 又见到了这个算法，这个算法已经出现了不止一次
// 不过我突然觉着这个算法可能没有我之前分析的那么简单
// 因为我发现对于整个算法来讲，最重要的是v5的高位
// 参与算法的并不是当前v5的低位，而是上一次 v5 的低位
// 这种情况可以将v5拆成两个uint8，以便更好的分析算法
// 这里使用 res 记录当前v5高位，old\_data 记录上次 v5 低位
//
// 这里对这个算法进行重新分析：
// 将 v5 转为 uint8 类型后整个算法逐渐变的更加清晰
// 首先通过公式 id=(data[i] ^ v5) \* 2 + 84 得到 id 作为下一次从 data 中取数据的下标
// 随后通过 id + 1 将高位取出，再和上一次的 v5 低位异或得到该轮循环的值
// 注意：这种情况最终的结果将不再是 uint16 类型，因为这里是为了更清晰的分析算法
// 如果需要，可以添加一个一直指向 res 的指针，最终取出 uint16 的数据
// 通过之前分析 buff3 的内存结构可以知道，data 的数据是应用启动时生成的一段随机 hash
// 这里返回的 v5 如果应用不重启，那么这个值是是固定的，如果应用重启这个值就会变化
//
// int id = 0;
// unsigned char res = 0;
// unsigned char old\_data = 0;
//
// for (int i = 4; i < 68; i++)
// {
// id = (data[i] ^ res) \* 2 + 84;
// res = data[id + 1] ^ old\_data;
// old\_data = data[id];
// }
v62 = v5; // v61 = v5
\*((\_WORD \*)password + 1) = v5; // password[2-3] = v5
if ( user\_name )
{
if ( user\_name\_len > 0 ) // 判断用户名长度大于0
{
v6 = 0;
if ( (unsigned int)user\_name\_len < 0x10 ) // 判断用户名长度小于 16
goto LABEL\_33;
v6 = user\_name\_len & 0xFFFFFFF0; // v16 = 16
v7 = 0; // v7 = 0
si128 = \_mm\_load\_si128((const \_\_m128i \*)(&(&off\_1A25DC)[-11539] + 1));// si128 = 0F0F 0F0F 0F0F 0F0F 0F0F 0F0F 0F0F 0F0F
v59 = \_mm\_load\_si128((const \_\_m128i \*)(&off\_1A25DC - 92307));// v59 = 9F9F 9F9F 9F9F 9F9F 9F9F 9F9F 9F9F 9F9F
v9 = \_mm\_load\_si128((const \_\_m128i \*)(&(&off\_1A25DC)[-11538] + 1));// v9 = 3030 3030 3030 3030 3030 3030 3030 3030
v10 = \_mm\_load\_si128((const \_\_m128i \*)(&off\_1A25DC - 92299));// v10 = 5757 5757 5757 5757 5757 5757 5757 5757
v11 = \_mm\_load\_si128((const \_\_m128i \*)(&(&off\_1A25DC)[-11537] + 1));// v11 = 0909 0909 0909 0909 0909 0909 0909 0909
do
{
v12 = \_mm\_loadu\_si128((const \_\_m128i \*)&user\_name[v7]);// 将 username 加载到寄存器 v12 中
//
v13 = \_mm\_and\_si128(\_mm\_srli\_epi16(v12, 4u), si128);// username分为 8 组 16bit 的数据，对它们分别逻辑右移4位，高位补零
// 再与上 0F0F 0F0F 0F0F 0F0F 0F0F 0F0F 0F0F 0F0F
// v13 = username>>4^si128
//
v14 = \_mm\_cmpeq\_epi8(\_mm\_min\_epu8(v12, v59), v12);// username 和 9F9F 9F9F 9F9F 9F9F 9F9F 9F9F 9F9F 9F9F 的最小值
// 而字符串必然小于9F，所以v14 = FFFFFFFFFFFFFFFF
//
v15 = \_mm\_or\_si128(\_mm\_andnot\_si128(v14, \_mm\_add\_epi8(v13, v10)), \_mm\_and\_si128(\_mm\_or\_si128(v13, v9), v14));//
// v15 = (!v14 & (v13+v10))|((v13|v9)&v14)
// 注意 andnot 会将 v14 反转，后边的 v14 不再是ffff而是0
// 稍微化简即可得到 v15 = v13|v9
v16 = \_mm\_and\_si128(v12, si128); // v16 = username & si128
v17 = \_mm\_cmpeq\_epi8(\_mm\_subs\_epu8(v16, v11), (\_\_m128i)0LL);// (v16分组减v11) 和 0比较，v16 每个字节不会超过 0f， v11 每个字节是 09
// 所以 v17 为 ff 和 00 组成
v18 = \_mm\_or\_si128(\_mm\_andnot\_si128(v17, \_mm\_add\_epi8(v16, v10)), \_mm\_and\_si128(\_mm\_or\_si128(v16, v9), v17));// 通过动态调试得知，v18 = username
\*(\_\_m128i \*)&key[2 \* v7 + 16] = \_mm\_unpackhi\_epi8(v15, v18);// 将 v15 和 v18 的低 64 位数以 8 位为单位进行交错
// key[16-31] = username>>4^si128 低64位分组交错 username
\*(\_\_m128i \*)&key[2 \* v7] = \_mm\_unpacklo\_epi8(v15, v18);// key[0-15] = username>>4^si128 高64位分组交错 username
v7 += 16;
}
while ( v6 != v7 ); // 看似循环，实际只执行一次
// 小结：
// key[0-15] = (username 8bit分组右移四位) ^ si128 再高64位分组交错 username
// key[16-31] = (username 8bit分组右移四位) ^ si128 再低64位分组交错 username
//
if ( v6 != user\_name\_len ) // 判断不成立
{
......
}
}
}
HMAC\_CTX\_init(ctx);
md = EVP\_sha256();
HMAC\_Init\_ex(ctx, key, 32, md, 0);
HMAC\_Update((int)ctx, (int)&v62, 2); // 这里的 v61 就是每次重启应用才会改的那个值，计算这个值的hash
len = 0;
HMAC\_Final((int)ctx, (int)hash, (int)&len); // 得到 hash
v26 = -32;
key\_hash\_ = 0;
do
key\_hash\_ = \*((\_WORD \*)fun3 + ((unsigned \_\_int8)v71[v26++] ^ HIBYTE(key\_hash\_)) + 42) ^ (key\_hash\_ << 8);
while ( v26 ); // 越界访问 key，通过(key[i] ^ v27) \* 2 + 84 得到 id
// 作为下一次从 data 中取数据的下标，最终得到 v27
key\_hash = key\_hash\_; // v76 = v27
uesrname\_reverse = 0LL;
v28 = user\_name;
if ( user\_name\_len > 0 )
{
v29 = 0;
if ( (unsigned int)user\_name\_len <= 0x1F )
goto LABEL\_21; // 跳转到 LABEL\_21
......
v28 = user\_name;
if ( v29 != user\_name\_len )
{
LABEL\_21:
v35 = user\_name\_len - v29; // v35 = 16
uesrname\_reverse\_ = &ctx[v29 - 16]; // v36 = \_\_int128 v69 = 0
do
\*uesrname\_reverse\_++ = v28[--v35]; // 将 username 倒序存入 username\_reverse 中
while ( v35 );
}
}
username\_ = (int)v28;
memset(dest, 0, sizeof(dest));
sub\_425F0(v57, (int)hash, username\_, 12, (int)&uesrname\_reverse, 16, (int)&key\_hash, 2, dest);// 总结：
// dest[0-1] = encrydata[0-1]
// dest[2-9] = tag[0-7]
// dest[10-17] = tag[8-15]
\*(\_WORD \*)password = \*(\_WORD \*)dest; // password[0-1] = dest[0-1]
v38 = \*(\_QWORD \*)&dest[2];
\*(\_QWORD \*)(password + 12) = \*(\_QWORD \*)&dest[10];// password[12-19] = dest[10-17]
\*(\_QWORD \*)(password + 4) = v38; // password[4-11] = dest[2-9]
memset(password\_, 0, 0x400u); // v77 被清零，v65也被越界清空
v65 = \*((\_DWORD \*)password + 4); // v64 = password[16-19]
v39 = \*(\_QWORD \*)password; // v39 = password[0-7]
\*(\_QWORD \*)&password\_[8] = \*((\_QWORD \*)password + 1);// v77[8-15] = password[8-15]
\*(\_QWORD \*)password\_ = v39; // v77[0-7] = password[0-7]
v40 = sub\_43070(fun3, password\_, 0x14u, src); //
// 首先修改fun3的数据并计算出一个hash1，放到src[0-1]
// 再次修改fun3的数据并计算出一个hash2，通过hash2和一些常数的计算
// 得到贯穿整个函数的v18（这个值最终决定了password长度），
// 在43-60行的代码中，反复修改fun3的数据并计算hash
// 来设置数组 s, 随后将数组 s拷贝到 src 中，此时函数先将src的[40 + v18]到[40 + v18]保存了一份
// 调用函数 sub\_43990 中通过一系列的位移变换，再次修改src中的数据
// 然后将保存的数据进行恢复，返回值为 v18 + 56
memset(v72, 0, 256);
memset(v71, 0, sizeof(v71));
for ( i = 0; i != 256; ++i )
{
v71[i] = i; // 初始化v71 = 0 1 2 3 4 5 6 ...
v72[i] = \*((\_BYTE \*)fun3 + (i & 0x3F) + 4); // 使用 fun3 的数据初始化 v72
}
v42 = 0;
v43 = -256;
do
{
v44 = (unsigned \_\_int8)v72[v43];
v45 = v44 + v42 + (unsigned \_\_int8)v72[v43 + 256];
v45 %= 256;
v46 = v71[v45];
v71[v45] = v44;
v72[v43++] = v46; // 不断变换交换 v71 和 v72 的数据
v42 = v45;
}
while ( v43 );
v58 = v40;
if ( v40 > 0 ) // v40 = 上个函数中的v18 + 56 判断成立
{
v47 = 0;
v48 = 0;
src\_ = src;
v50 = v58;
do
{
v51 = v47 + 1 - ((v47 + ((unsigned int)((v47 + 1) >> 31) >> 24) + 1) & 0xFFFFFF00);
v52 = (unsigned \_\_int8)v71[v51];
v48 = (v52 + v48) % 256;
v53 = v50;
v54 = v71[v48];
v71[v48] = v52; // 变换 v71 中的数据
v71[v51] = v54;
\*src\_++ ^= v71[(unsigned \_\_int8)(v71[v48] + v54)];// 循环将 v71 的数据异或再赋值给src
v50 = v53 - 1;
v55 = v53 == 1;
v47 = v51;
}
while ( !v55 );
}
memcpy(password, src, v58); // 将 src 的数据拷贝到 password
return v58; // 返回拷贝数据的长度
}
```

## 按钮 check3

可以轻易发现，主要是调用了 `MainActivity.this.getck` 方法，通过其返回值判断成功失败

```java
class c implements View.OnClickListener {
c() {
}
@Override // android.view.View.OnClickListener
public void onClick(View view) {
Context applicationContext;
String str;
String obj = MainActivity.this.y.getText().toString();
String obj2 = MainActivity.this.z.getText().toString();
if (obj.length() != 16) {
Toast.makeText(MainActivity.this.getApplicationContext(), "用户名长度必须为16字节", 0).show();
return;
}
if (MainActivity.this.getck(obj.getBytes(), com.test.pac.demo.a.a.b(obj2)) > 0) {
applicationContext = MainActivity.this.getApplicationContext();
str = "verify success!";
} else {
applicationContext = MainActivity.this.getApplicationContext();
str = "verify failed!";
}
Toast.makeText(applicationContext, str, 0).show();
}
}
```

### 分析函数 MainActivity\_getck

可以发现主要就是在调用函数 `dword_1A50BC[0] + 4` 而这个地址其实就是在 `_init` 初始化的 `[19ca8c + image_base]+4`（函数 `sub_43280`）
