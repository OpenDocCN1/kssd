
# 比较早的一个keygen练习， 参考`hambo`教程。 (1千字)

/* keygen for MIrc v5.82 by arbiter */

/* long long ago... do it as a standard keygen practice */

/* level : easy! */

```
#include <stdio.h>
#include <conio.h>
#include <stdlib.h>
typedef unsigned long DWORD;
typedef unsigned char BYTE;
const BYTE magicMul[] = { 0xb, 0x6, 0x11, 0xc, 0xc, 0xe, 0x5, 0xc,
              0x10, 0xa, 0xb,
0x6, 0xe, 0xe, 0x4, 0xb,
              0x6, 0xe, 0xe,
0x4, 0xb, 0x9, 0xc, 0xb,
              0xa, 0x8, 0xa,
0xa, 0x10, 0x8, 0x4, 0x6,
              0xa, 0xc, 0x10,
0x8, 0xa, 0x4, 0x10 };
void GenRegCode( char* buf, DWORD part_a, DWORD part_b );
void main( void )
{
    int i, j = 0, len;
    DWORD tmp = 0;
    DWORD nZ1 = 0;
    DWORD nZ2 = 0;
    char userName[30] = { 0 };
    char userCode[50] = { 0 };
    clrscr();
    fprintf( stderr, "Mirc v5.82 key generator by arbiter.\n\n");
    fprintf( stderr, "Your userName : " );
    gets( userName );
    len = strlen( userName );
    if( len < 5 )
    {
        fprintf( stderr, "\nName length must great\nthan 5!" );
        exit( 0 );
    }
    /* Gen. Part one */
    for( i = 3; i < len; i++, j++ )
    {
        nZ1 += userName[i] * magicMul[j];
        j = ( j > 38 )? 0 : j;
    }
    j &= 0;
    /* Gen. Part two */
    for( i = 3; i < len; i++, j++ )
    {
        tmp = userName[i] * userName[i - 1];
        nZ2 += tmp * magicMul[j];
        j = ( j > 38 )? 0 : j;
    }
    GenRegCode( userCode, nZ1, nZ2 );
    fprintf( stderr, "You userCode : %s", userCode );
    getch();
}
void GenRegCode( char* buf, DWORD part_a, DWORD part_b )
{
    char tmpBuf[50] = { 0 };
    DWORD temp = 0;
    int i = 0;
    while( part_a > 0 )
    {
        temp = part_a % 10;
        part_a /= 10;
        tmpBuf[i++] = temp + 0x30;
    }
    sprintf( buf, "%s", strrev( tmpBuf ) );
    buf[i++] = 0x2d; /* '-' */
    i &= 0;
    while( part_b > 0 )
    {
        temp = part_b % 10;
        part_b /= 10;
        tmpBuf[i++] = temp + 0x30;
    }
    strcat( buf, strrev( tmpBuf ) );
}
```
