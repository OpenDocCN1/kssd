
```c
    memset(ctx, 0, sizeof(ctx));

}

/* The four core functions - F1 is optimized somewhat */

/* #define F1(x, y, z) (x & y | ~x & z) */

#define F1(x, y, z) (z ^ (x & (y ^ z)))

#define F2(x, y, z) F1(z, x, y)

#define F3(x, y, z) (x ^ y ^ z)

#define F4(x, y, z) (y ^ (x | ~z))

/* This is the central step in the MD5 algorithm. */

#define MD5STEP(f, w, x, y, z, data, s) \
    ( w += f(x, y, z) + data,  w = w<<s | w>>(32-s),  w += x )

void MD5Transform(buf, in)
    uint32 buf[4]; uint32 in[16];
{
    register uint32 a, b, c, d;

    a = buf[0];

    b = buf[1];

    c = buf[2];

    d = buf[3];

    MD5STEP(F1, a, b, c, d, in[0] + 0xd76aa478, 7);

    MD5STEP(F1, d, a, b, c, in[1] + 0xe8c7b756, 12);

    MD5STEP(F1, c, d, a, b, in[2] + 0x242070db, 17);

    MD5STEP(F1, b, c, d, a, in[3] + 0xc1bdceee, 22);

    MD5STEP(F1, a, b, c, d, in[4] + 0xf57c0faf, 7);

    MD5STEP(F1, d, a, b, c, in[5] + 0x4787c62a, 12);

    MD5STEP(F1, c, d, a, b, in[6] + 0xa8304613, 17);

    MD5STEP(F1, b, c, d, a, in[7] + 0xfd469501, 22);

    MD5STEP(F1, a, b, c, d, in[8] + 0x698098d8, 7);

    MD5STEP(F1, d, a, b, c, in[9] + 0x8b44f7af, 12);

    MD5STEP(F1, c, d, a, b, in[10] + 0xffff5bb1, 17);

    MD5STEP(F1, b, c, d, a, in[11] + 0x895cd7be, 22);

    MD5STEP(F1, a, b, c, d, in[12] + 0x6b901122, 7);

    MD5STEP(F1, d, a, b, c, in[13] + 0xfd987193, 12);

    MD5STEP(F1, c, d, a, b, in[14] + 0xa679438e, 17);

    MD5STEP(F1, b, c, d, a, in[15] + 0x49b40821, 22);

    MD5STEP(F2, a, b, c, d, in[1] + 0xf61e2562, 5);

    MD5STEP(F2, d, a, b, c, in[6] + 0xc040b340, 9);

    MD5STEP(F2, c, d, a, b, in[11] + 0x265e5a51, 14);

    MD5STEP(F2, b, c, d, a, in[0] + 0xe9b6c7aa, 20);

    MD5STEP(F2, a, b, c, d, in[5] + 0xd62f105d, 5);

    MD5STEP(F2, d, a, b, c, in[10] + 0x02441453, 9);

    MD5STEP(F2, c, d, a, b, in[15] + 0xd8a1e681, 14);

    MD5STEP(F2, b, c, d, a, in[4] + 0xe7d3fbc8, 20);

    MD5STEP(F2, a, b, c, d, in[9] + 0x21e1cde6, 5);

    MD5STEP(F2, d, a, b, c, in[14] + 0xc33707d6, 9);

    MD5STEP(F2, c, d, a, b, in[3] + 0xf4d50d87, 14);

    MD5STEP(F2, b, c, d, a, in[8] + 0x455a14ed, 20);

    MD5STEP(F2, a, b, c, d, in[13] + 0xa9e3e905, 5);

    MD5STEP(F2, d, a, b, c, in[2] + 0xfcefa3f8, 9);

    MD5STEP(F2, c, d, a, b, in[7] + 0x676f02d9, 14);

    MD5STEP(F2, b, c, d, a, in[12] + 0x8d2a4c8a, 20);

    MD5STEP(F3, a, b, c, d, in[5] + 0xfffa3942, 4);

    MD5STEP(F3, d, a, b, c, in[8] + 0x8771f681, 11);

    MD5STEP(F3, c, d, a, b, in[11] + 0x6d9d6122, 16);

    MD5STEP(F3, b, c, d, a, in[14] + 0xfde5380c, 23);

    MD5STEP(F3, a, b, c, d, in[1] + 0xa4beea44, 4);

    MD5STEP(F3, d, a, b, c, in[4] + 0x4bdecfa9, 11);

    MD5STEP(F3, c, d, a, b, in[7] + 0xf6bb4b60, 16);

    MD5STEP(F3, b, c, d, a, in[10] + 0xbebfbc70, 23);

    MD5STEP(F3, a, b, c, d, in[13] + 0x289b7ec6, 4);

    MD5STEP(F3, d, a, b, c, in[0] + 0xeaa127fa, 11);

    MD5STEP(F3, c, d, a, b, in[3] + 0xd4ef3085, 16);

    MD5STEP(F3, b, c, d, a, in[6] + 0x04881d05, 23);

    MD5STEP(F3, a, b, c, d, in[9] + 0xd9d4d039, 4);

    MD5STEP(F3, d, a, b, c, in[12] + 0xe6db99e5, 11);

    MD5STEP(F3, c, d, a, b, in[15] + 0x1fa27cf8, 16);

    MD5STEP(F3, b, c, d, a, in[2] + 0xc4ac5665, 23);

    MD5STEP(F4, a, b, c, d, in[0] + 0xf4292244, 6);

    MD5STEP(F4, d, a, b, c, in[7] + 0x432aff97, 10);

    MD5STEP(F4, c, d, a, b, in[14] + 0xab9423a7, 15);

    MD5STEP(F4, b, c, d, a, in[5] + 0xfc93a039, 21);

    MD5STEP(F4, a, b, c, d, in[12] + 0x655b59c3, 6);

    MD5STEP(F4, d, a, b, c, in[3] + 0x8f0ccc92, 10);

    MD5STEP(F4, c, d, a, b, in[10] + 0xffeff47d, 15);

    MD5STEP(F4, b, c, d, a, in[1] + 0x85845dd1, 21);

    MD5STEP(F4, a, b, c, d, in[8] + 0x6fa87e4f, 6);

    MD5STEP(F4, d, a, b, c, in[15] + 0xfe2ce6e0, 10);

    MD5STEP(F4, c, d, a, b, in[6] + 0xa3014314, 15);

    MD5STEP(F4, b, c, d, a, in[13] + 0x4e0811a1, 21);

    MD5STEP(F4, a, b, c, d, in[4] + 0xf7537e82, 6);

    MD5STEP(F4, d, a, b, c, in[11] + 0xbd3af235, 10);

    MD5STEP(F4, c, d, a, b, in[2] + 0x2ad7d2bb, 15);

    MD5STEP(F4, b, c, d, a, in[9] + 0xeb86d391, 21);

    buf[0] += a;

    buf[1] += b;

    buf[2] += c;

    buf[3] += d;

}

char eulav(long i);

void zhconvert(verylong a, char *dbuff, int CodeLen){

    static    char *b;

    register long i,j,cnt = 0;

    b = (char *)malloc( (size_t)(a[0] << 3) );

    if (!b) return;

    do    {
        b[cnt] = eulav(a[1] & 15);

        cnt++;

        zrshift(a, (long) 4, &a);

    }

    while ( (a[1] != 0) || (a[0] != 1) || (cnt < CodeLen) );

    b[cnt]&=0x00;

    j=cnt-1;

    for    (i=0; i<cnt; i++)    {
        dbuff[i]=b[j];

        j--;

    }

    dbuff[i]=0x00;

    free(b);

}

void GenerateCode(HWND hDlg){

    static char modulus_n[]="24DFDA27FA14D3F27DDF62CEA5D2381F9";

    static char private_d[]="1E2D9B52ADCBC20DCCDE3C721AA740E83";

    static char hextab[]="0123456789ABCDEF";

    char       szName[31], szOrg[31], szH[62], signature[16], md[33];

    struct MD5Context md5c;

        char       Regcode[42];

    int       i,j,nLen,oLen;

    BYTE      t, u;

    verylong  d = 0;

    verylong  m = 0;

    verylong  n = 0;

    nLen=GetDlgItemTextA(hDlg, EDIT_NAME, szName, 31);

        oLen=GetDlgItemTextA(hDlg, EDIT_ORG, szOrg, 31);

    if    (nLen < 1) {
        SetDlgItemTextA(hDlg, EDIT_CODE, "Name must contain at least 1 char.");

        return;

    }

    if    (oLen < 1) {
        SetDlgItemTextA(hDlg, EDIT_CODE, "Organization must contain at least 1 char.");

        return;

    }

    //concatenate the strings

    for (i=0; i<nLen; i++) szH[i] = szName[i];

    for (i=0; i<oLen; i++) szH[nLen+i] = szOrg[i];

    szH[nLen+oLen] = '\0';

    MD5Init(&md5c);

    MD5Update(&md5c, szH, nLen+oLen);

    MD5Final(signature, &md5c);

    j=0;

    for    (i=0; i<16; i++){
        u=t=signature[i];

        md[j]=hextab[((t & 0xF0) >> 4)];

        md[j+1]=hextab[(u & 0x0F)];

        j+=2;

    }

    md[j]&=0x00;

    /* Read in Bignums */

    zstrtozbas(modulus_n, (long) 16, &n);

    zstrtozbas(private_d, (long) 16, &d);

    zstrtozbas(md, (long) 16, &m);

    /* m = m^d % n */

    zexpmod(m,d,n,&m);

    /* Convert m to Hex String, 32 Chars long */

    zhconvert(m, md, 32);

        /* Clean up */

    zfree(&d);

    zfree(&m);

    zfree(&n);

    /* Show Regcode */

    SetDlgItemTextA(hDlg, EDIT_CODE, md);

}

BOOL CALLBACK DlgP (HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)

{

    switch (message)

    {

        case WM_INITDIALOG :
            SendDlgItemMessageA(hDlg, EDIT_NAME, EM_SETLIMITTEXT, (WPARAM) 30, 0);

            SetWindowTextA(hDlg, "KeyGen For Bishop Lockless CrackMe4 By 娃娃/[CCG]");

            SetDlgItemTextA(hDlg, EDIT_CODE, "请输入用户名和组织名.");

            return TRUE;

        case WM_COMMAND :
            switch (LOWORD (wParam))

            {

                case BT_GENERATE :
                    GenerateCode(hDlg);

                    SetFocus(GetDlgItem(hDlg, EDIT_NAME));

                    break;

                case IDCANCEL :
                    EndDialog (hDlg, 0);

                    break;

            }

            break;

    }

    return FALSE;

}
```
