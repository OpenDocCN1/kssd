
```
using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using System.IO;

namespace TENCENT2017

{

    class Program

    {

        static void Main(string[] args)

        {

            EncryptDataFile();

            DecryptDataFile();

            Console.WriteLine("Hello GSLab!");

        }

        public static void EncryptDataFile()

        {

            byte[] array;

            using (FileStream fileStream = new FileStream("areyouok.png", FileMode.Open))

            {

                using (BinaryReader binaryReader = new BinaryReader(fileStream))

                {

                    array = binaryReader.ReadBytes(Convert.ToInt32(fileStream.Length));

                    array = Encrypt(array);

                }

            }

            using (FileStream fileStream2 = new FileStream("areyouok_encrypted", FileMode.Create))

            {

                using (BinaryWriter binaryWriter = new BinaryWriter(fileStream2))

                {

                    binaryWriter.Write(array);

                }

            }

        }

        public static void DecryptDataFile()

        {

            byte[] array;

            using (FileStream fileStream = new FileStream("data.encrypted", FileMode.Open))

            {

                using (BinaryReader binaryReader = new BinaryReader(fileStream))

                {

                    array = binaryReader.ReadBytes(Convert.ToInt32(fileStream.Length));

                    array = Decrypt(array);

                }

            }

            using (FileStream fileStream2 = new FileStream("de-data.encrypted.png", FileMode.Create))

            {

                using (BinaryWriter binaryWriter = new BinaryWriter(fileStream2))

                {

                    binaryWriter.Write(array);

                }

            }

        }

        public static byte[] Decrypt(byte[] input)

        {

            int num = input.Length;

            uint[] array = new uint[]

	        {

		        1423373290u,

		        363609949u,

		        512121596u,

		        1703126449u

	        };

            byte[] array2 = new byte[0];

            uint[] array4 = new uint[]

	        {

		        0,

		        0

	        };

            uint[] array5 = new uint[]

	        {

		        0,

		        0

	        };

            for (int k = num-4; k >=12; k -= 8)

            {

               array4[1] = ConvertBytesToUInt(input, k );

               array4[0] = ConvertBytesToUInt(input, k-4);

               array4 = DeCode(array4, array);

               array4[1] ^= ConvertBytesToUInt(input, k -8);

               array4[0] ^= ConvertBytesToUInt(input, k - 12);

               array2 = CombineBytes2(array2, ConvertUIntToBytes(array4[1]));

               array2 = CombineBytes2(array2, ConvertUIntToBytes(array4[0]));

               

            }

            return array2;

        }

        private static uint[] DeCode(uint[] v, uint[] k)

        {

            uint num = v[0];

            uint num2 = v[1];

            uint num3 = 0u;

            uint num4 = Convert.ToUInt32(Math.Floor((Math.Pow(5.0, 0.5) - 1.0) * Math.Pow(2.0, 31.0)));

            uint num5 = 32u;

            while (num5-- > 0u)

            {

                num3 += num4;

            }

            num5 = 32u;

            while (num5-- > 0u)

            {

                num2-= (num << 4 ^ (num >> 5) + num ^ num3 + k[(int)((ushort)(num3 >> 11 & 3u))]);

                num3 -= num4;

                num -= (num2 << 4 ^ (num2 >> 5) + num2 ^ num3 + k[(int)((ushort)(num3 & 3u))]);

               Console.WriteLine(" {0} {1} {2}.", num, num2, num3);

                

            }

            return new uint[]

	{

		num,

		num2

	};

        }

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

		ConvertBytesToUInt(array3, 0),

		ConvertBytesToUInt(array3, 4)

	};

            array4[0] = (array4[0] ^ array[0]);

            array4[1] = (array4[1] ^ array[2]);

            array4 = Code(array4, array);

            array2 = CombineBytes(array2, ConvertUIntToBytes(array4[0]));

            array2 = CombineBytes(array2, ConvertUIntToBytes(array4[1]));

            for (int k = 7 - num2; k < num; k += 8)

            {

                array4[0] ^= ConvertBytesToUInt(input, k);

                array4[1] ^= ConvertBytesToUInt(input, k + 4);

                array4 = Code(array4, array);

                array2 = CombineBytes(array2, ConvertUIntToBytes(array4[0]));

                array2 = CombineBytes(array2, ConvertUIntToBytes(array4[1]));

            }

            return array2;

        }

        private static byte[] ConvertUIntToBytes(uint input)

        {

            return new byte[]

	            {

		            (byte)(input & 255u),

		            (byte)(input >> 8 & 255u),

		            (byte)(input >> 16 & 255u),

		            (byte)(input >> 24 & 255u)

	            };

        }

        private static uint ConvertBytesToUInt(byte[] input, int pos)

        {

            uint num = (uint)input[pos];

            num += (uint)((uint)input[pos + 1] << 8);

            num += (uint)((uint)input[pos + 2] << 16);

            return num + (uint)((uint)input[pos + 3] << 24);

        }

        private static byte[] CombineBytes(byte[] bytes1, byte[] bytes2)

        {

            byte[] array = new byte[bytes1.Length + bytes2.Length];

            Buffer.BlockCopy(bytes1, 0, array, 0, bytes1.Length);

            Buffer.BlockCopy(bytes2, 0, array, bytes1.Length, bytes2.Length);

            return array;

        }

        private static byte[] CombineBytes2(byte[] bytes1, byte[] bytes2)

        {

            byte[] array = new byte[bytes1.Length + bytes2.Length];

            Buffer.BlockCopy(bytes2, 0, array, 0, bytes2.Length);

            Buffer.BlockCopy(bytes1, 0, array, bytes2.Length, bytes1.Length);

            return array;

        }

        private static uint[] Code(uint[] v, uint[] k)

        {

            uint num = v[0];

            uint num2 = v[1];

            uint num3 = 0u;

            uint num4 = Convert.ToUInt32(Math.Floor((Math.Pow(5.0, 0.5) - 1.0) * Math.Pow(2.0, 31.0)));

            uint num5 = 32u;

            while (num5-- > 0u)

            {

                num += (num2 << 4 ^ (num2 >> 5) + num2 ^ num3 + k[(int)((ushort)(num3 & 3u))]);

                num3 += num4;

                num2 += (num << 4 ^ (num >> 5) + num ^ num3 + k[(int)((ushort)(num3 >> 11 & 3u))]);

               // Console.WriteLine(" {0} {1} {2}.", num, num2, num3);

            }

            return new uint[]

	{

		num,

		num2

	};

        }

    }

}
```

### 0x3 最后的战役

这一问好开放，只能靠脑洞了。解密出来的是一个藏着数据的png。修复头之后提示格式问题，去熟悉格式。

又不会，直接链接如下，一起学习之：

[715K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3&6C8N6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6#2)9J5c8X3p5&6z5e0p5J5y4K6m8X3y4K6k6U0y4U0j5I4x3K6N6W2k6e0l9$3x3e0V1$3y4#2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@ce2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3&6C8N6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6#2)9J5c8X3p5&6z5e0p5J5y4K6m8X3y4K6k6U0y4U0j5I4x3K6N6W2k6e0l9$3x3e0V1$3y4#2)9J5k6h3S2@1L8h3H3`.)
