
```
public static byte[] Encrypt(byte[] input)
{
    uint[] array = new uint[]
    {
        1423373290u,
        363609949u,
        512121596u,
        1703126449u
    };
    byte[] array2 = new byte[0];
    int num = input.Length;
    byte[] array3 = new byte[8];
    int num2 = 7 - num % 8;
    array3[0] = (byte)num2;
    for (int i = 0; i < num2; i++)
    {
        array3[i + 1] = (byte)(200 + num2 - i);
    }
    for (int j = 0; j < 7 - num2; j++)
    {
        array3[j + num2 + 1] = input[j];
    }
    uint[] array4 = new uint[]
    {
        MyClass.ConvertBytesToUInt(array3, 0),
        MyClass.ConvertBytesToUInt(array3, 4)
    };
    array4[0] = (array4[0] ^ array[0]);
    array4[1] = (array4[1] ^ array[2]);
    array4 = MyClass.Code(array4, array);
    array2 = MyClass.CombineBytes(array2, MyClass.ConvertUIntToBytes(array4[0]));
    array2 = MyClass.CombineBytes(array2, MyClass.ConvertUIntToBytes(array4[1]));
    for (int k = 7 - num2; k < num; k += 8)
    {
        array4[0] ^= MyClass.ConvertBytesToUInt(input, k);
        array4[1] ^= MyClass.ConvertBytesToUInt(input, k + 4);
        array4 = MyClass.Code(array4, array);
        array2 = MyClass.CombineBytes(array2, MyClass.ConvertUIntToBytes(array4[0]));
        array2 = MyClass.CombineBytes(array2, MyClass.ConvertUIntToBytes(array4[1]));
    }
    return array2;
```

算法是XTEA的变种，链接这里：

[030K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6j5g2p5g2m8](https://bbs.kanxue.com/elink@111K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6j5g2p5g2m8)

说实话，我这方法有点麻烦，你们出题有点猥琐。

### 0x2 解密函数解密data

直接上代码，wp写累了：
