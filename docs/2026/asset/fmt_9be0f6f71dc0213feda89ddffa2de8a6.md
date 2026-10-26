
```
                compute_backup[3] >>= 8;

                g_email_table[8] = (unsigned char)(compute_backup[2] & 0xff);

                compute_backup[2] >>= 8;

                g_email_table[9] = (unsigned char)(compute_backup[2] & 0xff);

                compute_backup[2] >>= 8;

                g_email_table[10] = (unsigned char)(compute_backup[2] & 0xff);

                compute_backup[2] >>= 8;

                g_email_table[11] = (unsigned char)(compute_backup[2] & 0xff);

                compute_backup[2] >>= 8;

                g_email_table[12] = (unsigned char)(compute_backup[1] & 0xff);

                compute_backup[1] >>= 8;

                g_email_table[13] = (unsigned char)(compute_backup[1] & 0xff);

                compute_backup[1] >>= 8;

                g_email_table[14] = (unsigned char)(compute_backup[1] & 0xff);

                compute_backup[1] >>= 8;

                g_email_table[15] = (unsigned char)(compute_backup[1] & 0xff);

                compute_backup[1] >>= 8;

        }

        /*打印出计算结果，注意这个不是最后的注册码，还需要根据结果查g_index表*/

        for (seq=0; seq<16; seq++)

        {

                printf("%x ", (unsigned char)g_email_table[seq]);

        }

        printf("\n");

        return 0;

}
```

代码写的不好，希望大家给我提出宝贵意见。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
