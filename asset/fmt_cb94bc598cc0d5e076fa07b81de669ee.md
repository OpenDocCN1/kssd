
```vb
FF (a, b, c, d, x[ 8], S11, 0x698098d8); /* 9 */   
FF (d, a, b, c, x[ 9], S12, 0x8b44f7af); /* 10 */   
FF (c, d, a, b, x[10], S13, 0xffff5bb1); /* 11 */   
FF (b, c, d, a, x[11], S14, 0x895cd7be); /* 12 */   
FF (a, b, c, d, x[12], S11, 0x6b901122); /* 13 */   
FF (d, a, b, c, x[13], S12, 0xfd987193); /* 14 */   
FF (c, d, a, b, x[14], S13, 0xa679438e); /* 15 */   
FF (b, c, d, a, x[15], S14, 0x49b40821); /* 16 */   
/* 第二轮循环 */   
GG (a, b, c, d, x[ 1], S21, 0xf61e2562); /* 17 */   
GG (d, a, b, c, x[ 6], S22, 0xc040b340); /* 18 */   
GG (c, d, a, b, x[11], S23, 0x265e5a51); /* 19 */   
GG (b, c, d, a, x[ 0], S24, 0xe9b6c7aa); /* 20 */   
GG (a, b, c, d, x[ 5], S21, 0xd62f105d); /* 21 */   
GG (d, a, b, c, x[10], S22,  0x2441453); /* 22 */   
GG (c, d, a, b, x[15], S23, 0xd8a1e681); /* 23 */   
GG (b, c, d, a, x[ 4], S24, 0xe7d3fbc8); /* 24 */   
GG (a, b, c, d, x[ 9], S21, 0x21e1cde6); /* 25 */   
GG (d, a, b, c, x[14], S22, 0xc33707d6); /* 26 */   
GG (c, d, a, b, x[ 3], S23, 0xf4d50d87); /* 27 */   
GG (b, c, d, a, x[ 8], S24, 0x455a14ed); /* 28 */   
GG (a, b, c, d, x[13], S21, 0xa9e3e905); /* 29 */   
GG (d, a, b, c, x[ 2], S22, 0xfcefa3f8); /* 30 */   
GG (c, d, a, b, x[ 7], S23, 0x676f02d9); /* 31 */   
GG (b, c, d, a, x[12], S24, 0x8d2a4c8a); /* 32 */   
/* 第三轮循环 */   
HH (a, b, c, d, x[ 5], S31, 0xfffa3942); /* 33 */   
HH (d, a, b, c, x[ 8], S32, 0x8771f681); /* 34 */   
HH (c, d, a, b, x[11], S33, 0x6d9d6122); /* 35 */   
HH (b, c, d, a, x[14], S34, 0xfde5380c); /* 36 */   
HH (a, b, c, d, x[ 1], S31, 0xa4beea44); /* 37 */   
HH (d, a, b, c, x[ 4], S32, 0x4bdecfa9); /* 38 */   
HH (c, d, a, b, x[ 7], S33, 0xf6bb4b60); /* 39 */   
HH (b, c, d, a, x[10], S34, 0xbebfbc70); /* 40 */   
HH (a, b, c, d, x[13], S31, 0x289b7ec6); /* 41 */   
HH (d, a, b, c, x[ 0], S32, 0xeaa127fa); /* 42 */   
HH (c, d, a, b, x[ 3], S33, 0xd4ef3085); /* 43 */   
HH (b, c, d, a, x[ 6], S34,  0x4881d05); /* 44 */   
HH (a, b, c, d, x[ 9], S31, 0xd9d4d039); /* 45 */   
HH (d, a, b, c, x[12], S32, 0xe6db99e5); /* 46 */   
HH (c, d, a, b, x[15], S33, 0x1fa27cf8); /* 47 */   
HH (b, c, d, a, x[ 2], S34, 0xc4ac5665); /* 48 */   
/* 第四轮循环 */   
II (a, b, c, d, x[ 0], S41, 0xf4292244); /* 49 */   
II (d, a, b, c, x[ 7], S42, 0x432aff97); /* 50 */   
II (c, d, a, b, x[14], S43, 0xab9423a7); /* 51 */   
II (b, c, d, a, x[ 5], S44, 0xfc93a039); /* 52 */   
II (a, b, c, d, x[12], S41, 0x655b59c3); /* 53 */   
II (d, a, b, c, x[ 3], S42, 0x8f0ccc92); /* 54 */   
II (c, d, a, b, x[10], S43, 0xffeff47d); /* 55 */   
II (b, c, d, a, x[ 1], S44, 0x85845dd1); /* 56 */   
II (a, b, c, d, x[ 8], S41, 0x6fa87e4f); /* 57 */   
II (d, a, b, c, x[15], S42, 0xfe2ce6e0); /* 58 */   
II (c, d, a, b, x[ 6], S43, 0xa3014314); /* 59 */   
II (b, c, d, a, x[13], S44, 0x4e0811a1); /* 60 */   
II (a, b, c, d, x[ 4], S41, 0xf7537e82); /* 61 */   
II (d, a, b, c, x[11], S42, 0xbd3af235); /* 62 */   
II (c, d, a, b, x[ 2], S43, 0x2ad7d2bb); /* 63 */   
II (b, c, d, a, x[ 9], S44, 0xeb86d391); /* 64 */   
state[0] += a;   
state[1] += b;   
state[2] += c;   
state[3] += d;   
MD5_memset ((POINTER)x, 0, sizeof (x));   
}   

static void Encode (output, input, len)   
unsigned char *output;   
UINT4 *input;   
unsigned int len;   
{   
  unsigned int i, j;   
  for (i = 0, j = 0; j < len; i++, j += 4) {   
output[j] = (unsigned char)(input[i] & 0xff);   
output[j+1] = (unsigned char)((input[i] >> 8) & 0xff);   
output[j+2] = (unsigned char)((input[i] >> 16) & 0xff);   
output[j+3] = (unsigned char)((input[i] >> 24) & 0xff);   
  }   
}   

static void Decode (output, input, len)   
UINT4 *output;   
unsigned char *input;   
unsigned int len;   
{   
  unsigned int i, j;   
  for (i = 0, j = 0; j < len; i++, j += 4)   
output[i] = ((UINT4)input[j]) | (((UINT4)input[j+1]) << 8) |   
  (((UINT4)input[j+2]) << 16) | (((UINT4)input[j+3]) << 24);   
}   

static void MD5_memcpy (output, input, len)   
POINTER output;   
POINTER input;   
unsigned int len;   
{   
  unsigned int i;   
  for (i = 0; i < len; i++) output[i] = input[i];   
}   

static void MD5_memset (output, value, len)   
POINTER output;   
int value;   
unsigned int len;   
{   
  unsigned int i;   
  for (i = 0; i < len; i++)   
((char *)output)[i] = (char)value;   
}
```

————————————————C 代码结束——————————

### 在 VB 中实现 MD5 算法

——————————————————————————————————————————

```vb
Option Explicit   
Dim w1 As String, w2 As String, w3 As String, w4 As String   

Function MD5F(ByVal tempstr As String, ByVal w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
MD5F = BigMod32Add(RotLeft(BigMod32Add(BigMod32Add(w, tempstr), BigMod32Add(Xin, qdata)), rots), X)   
End Function   

Sub MD5F1(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(z, BigAND(X, BigXOR(y, z)))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Sub MD5F2(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(y, BigAND(z, BigXOR(X, y)))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Sub MD5F3(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(X, BigXOR(y, z))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Sub MD5F4(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(y, BigOR(X, BigNOT(z)))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Function MD5_Calc(ByVal hashthis As String) As String   
ReDim buf(0 To 3) As String   
ReDim Xin(0 To 15) As String   
Dim tempnum As Integer, tempnum2 As Integer, loopit As Integer, loopouter As Integer, loopinner As Integer   
Dim a As String, b As String, c As String, d As String   
    tempnum = 8 * Len(hashthis)   
    hashthis = hashthis + Chr$(128) 'Add binary 10000000   
    tempnum2 = 56 - Len(hashthis) Mod 64   
    If tempnum2 < 0 Then   
        tempnum2 = 64 + tempnum2   
    End If   
    hashthis = hashthis + String$(tempnum2, Chr$(0))   
    For loopit = 1 To 8   
        hashthis = hashthis + Chr$(tempnum Mod 256)   
        tempnum = tempnum - tempnum Mod 256   
        tempnum = tempnum / 256   
    Next loopit   
    buf(0) = "67452301"   
    buf(1) = "efcdab89"   
    buf(2) = "98badcfe"   
    buf(3) = "10325476"   
For loopouter = 0 To Len(hashthis) / 64 - 1   
        a = buf(0)   
        b = buf(1)   
        c = buf(2)   
        d = buf(3)   
        ' Get the 512 bits   
        For loopit = 0 To 15   
            Xin(loopit) = ""   
            For loopinner = 1 To 4   
                Xin(loopit) = Hex$(Asc(Mid$(hashthis, 64 * loopouter + 4 * loopit + loopinner, 1))) + Xin(loopit)   
                If Len(Xin(loopit)) Mod 2 Then Xin(loopit) = "0" + Xin(loopit)   
            Next loopinner   
        Next loopit   
' 第一轮循环   
MD5F1 a, b, c, d, Xin(0), "d76aa478", 7   
MD5F1 d, a, b, c, Xin(1), "e8c7b756", 12   
```
