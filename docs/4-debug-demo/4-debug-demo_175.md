

# \[最弱智\]考题

> 原文：[`www.pediy.com/kssd/pediy03/forum509.htm`](https://www.pediy.com/kssd/pediy03/forum509.htm)

DWORD algo( DWORD m )   
{   
  int i;   
  for( i=0; i<32; i++ )   
    if( m & 1 )   
      m = ( m>>1 ) ^ 0x12345678;   
    else   
      m >>= 1;   
  return m;   
}   
  
问题：   
(1) 该函数是否存在逆算法？   
(2) 如果存在，给出逆算法，并说明逆算法有什么限制。   
(3) 如果不存在，请说明理由，并对原算法稍做修改使之可逆。

该算法部分可逆。用于异或的 0x12345678 是“部分可逆”的原因。当   
algo 的参数在一个范围之内的时候，算法可逆。这个范围留给各位去找找吧。   
  
逆算法：   
  
unsigned long ogla( unsigned long m )   
{   
  int i;   
  for( i=0; i<32; i++ )   
    if( m & 0x10000000 )   
      m = ( ( m ^ 0x12345678 ) << 1 ) | 1;   
    else   
      m <<= 1;   
  return m;   
}  4dd466d0e1e0cc123afbe504da2462ca