
```
for(i=0;i<2;i++)
        buf2[len+i]=sha1_hash_str1[seq2[i]];
buf2[len+i] = '\0';

Tiger_init(&tiger);
Tiger_Hash(&tiger, tiger_hash, buf2, strlen(buf2));
for(i = 0; i < 24 ; ++i)
{
        sprintf(tiger_hash_str+i*2, "%02X ", tiger_hash[i]);
}
tiger_hash_str[i*2] = '\0';
printf("Tiger input: %s\n", buf2);
printf("Tiger Hash:  %s\n", tiger_hash_str);
for(i=0;i<11;i++)
        buf3[i] = (seq3[i] & 0x8000) ? tiger_hash_str[seq3[i]&0xff] : sha1_hash_str1[seq3[i]&0xff] ;
buf3[i]='\0';
SHA1Reset(&sha);
SHA1Input(&sha, (const unsigned char *) buf3, strlen(buf3));
SHA1Result(&sha, sha1_hash2);
for(i = 0; i < 20 ; ++i)
{
        sprintf(sha1_hash_str2+i*2, "%02X ", sha1_hash2[i]);
}
sha1_hash_str2[i+20] = '\0';
printf("SHA1 input: %s\n", buf3);
printf("SHA1 hash:  %s\n", sha1_hash_str2);
Blowfish_Init(&blowfish, sha1_hash2, 0x14);
L = 0;
R = 0;
Blowfish_Encrypt(&blowfish, &L, &R);

/*        验证代码      */

/*
        strcpy(Key_base64, Key+6);
        Key_base64[0x40] = '\0';
        len = DecodeBase64(Key_base64, Key_bin, 100);
        for(i=0;i<0x30;i++)
        {
                L0 = L;
                R0 = R;
                Blowfish_Encrypt(&blowfish, &L, &R);
                buf4[i] = Key_bin[i] ^ (unsigned char)(L>>24);
                L = L0;
                R = R0;
                L = (L << 8) | (R>>24);
                R = (R << 8) | ((unsigned long)Key_bin[i]);
        }
        buf4[i] = '\0';
        if (strcmp(buf4, tiger_hash_str)==0){
                printf("OK\n");
        }
*/

/* 反算求KeyLicense */
        for(i=0;i<0x30;i++)
        {
                L0 = L;
                R0 = R;
                Blowfish_Encrypt(&blowfish, &L, &R);
                Key_bin[i] = tiger_hash_str[i] ^ (unsigned char)(L>>24);
                L = L0;
                R = R0;
                L = (L << 8) | (R>>24);
                R = (R << 8) | ((unsigned long)Key_bin[i]);
        }
        Key_bin[i] = '\0';
        len = EncodeBase64(Key_bin, 0x30, Key_base64, 100);
        strcpy(Key, "RMDBK");
        Key[5] = tiger_hash_str[0x23];
        Key[6] = '\0';
        strcat(Key, Key_base64);
        len = strlen(Key);
        Key[len] = sha1_hash_str1[0x11];
        Key[len+1] = '\0';
        printf("User:%s\n", Username);
        printf("Key: %s\n", Key);
        return 0;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
