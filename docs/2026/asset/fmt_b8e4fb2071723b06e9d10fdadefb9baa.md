
```
static int drbg_ctr_init(PROV_DRBG *drbg)

{

    PROV_DRBG_CTR *ctr = (PROV_DRBG_CTR *)drbg->data;

    size_t keylen;



    if (ctr->cipher_ctr == NULL) {

        ERR_raise(ERR_LIB_PROV, PROV_R_MISSING_CIPHER);

        return 0;

    }

    ctr->keylen = keylen = EVP_CIPHER_key_length(ctr->cipher_ctr);

    if (ctr->ctx_ecb == NULL)

        ctr->ctx_ecb = EVP_CIPHER_CTX_new();            //泄露点1

    if (ctr->ctx_ctr == NULL)

        ctr->ctx_ctr = EVP_CIPHER_CTX_new();            //泄露点2

    if (ctr->ctx_ecb == NULL || ctr->ctx_ctr == NULL) {

        ERR_raise(ERR_LIB_PROV, ERR_R_MALLOC_FAILURE);

        goto err;

    }



    if (!EVP_CipherInit_ex(ctr->ctx_ecb,

                           ctr->cipher_ecb, NULL, NULL, NULL, 1)

        || !EVP_CipherInit_ex(ctr->ctx_ctr,

                              ctr->cipher_ctr, NULL, NULL, NULL, 1)) {

        ERR_raise(ERR_LIB_PROV, PROV_R_UNABLE_TO_INITIALISE_CIPHERS);

        goto err;

    }



    drbg->strength = keylen * 8;

    drbg->seedlen = keylen + 16;



    if (ctr->use_df) {

        /* df initialisation */

        static const unsigned char df_key[32] = {

            0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,

            0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f,

            0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17,

            0x18, 0x19, 0x1a, 0x1b, 0x1c, 0x1d, 0x1e, 0x1f

        };



        if (ctr->ctx_df == NULL)

            ctr->ctx_df = EVP_CIPHER_CTX_new();       //泄露点3

        if (ctr->ctx_df == NULL) {

            ERR_raise(ERR_LIB_PROV, ERR_R_MALLOC_FAILURE);

            goto err;

        }

        /* Set key schedule for df_key */

        if (!EVP_CipherInit_ex(ctr->ctx_df,

                               ctr->cipher_ecb, NULL, df_key, NULL, 1)) {

            ERR_raise(ERR_LIB_PROV, PROV_R_DERIVATION_FUNCTION_INIT_FAILED);

            goto err;

        }

    }

    return drbg_ctr_init_lengths(drbg);



err:

    EVP_CIPHER_CTX_free(ctr->ctx_ecb);

    EVP_CIPHER_CTX_free(ctr->ctx_ctr);

    ctr->ctx_ecb = ctr->ctx_ctr = NULL;

    return 0;    

}
```

进一步回溯堆栈，发现了一处可疑操作：

```
int RAND_bytes_ex(OSSL_LIB_CTX *ctx, unsigned char *buf, int num)

{

    EVP_RAND_CTX *rand;

#ifndef OPENSSL_NO_DEPRECATED_3_0

    const RAND_METHOD *meth = RAND_get_rand_method();



    if (meth != NULL && meth != RAND_OpenSSL()) {

        if (meth->bytes != NULL)

            return meth->bytes(buf, num);

        ERR_raise(ERR_LIB_RAND, RAND_R_FUNC_NOT_IMPLEMENTED);

        return -1;

    }

#endif



        //当rand不为NULL时，使用rand作为上下文生成随机数，之后未作rand的释放动作，直接返回了。

        //而rand做为一个局部变量，未作资源释放，是否会产生资源泄露？还是做了其它特殊处理？

    rand = RAND_get0_public(ctx);

    if (rand != NULL)

        return EVP_RAND_generate(rand, buf, num, 0, 0, NULL, 0);  



    return 0;

}
```

跟进`RAND_get0_public`函数一探究竟：

```
EVP_RAND_CTX *RAND_get0_public(OSSL_LIB_CTX *ctx)

{

    RAND_GLOBAL *dgbl = rand_get_global(ctx);

    EVP_RAND_CTX *rand, *primary;



    if (dgbl == NULL)

        return NULL;



    //使用线程局部存储，减少了频繁的初始化及反初始化，以优化速度。

    rand = CRYPTO_THREAD_get_local(&dgbl->public);

    if (rand == NULL) {

        primary = RAND_get0_primary(ctx);

        if (primary == NULL)

            return NULL;



        ctx = ossl_lib_ctx_get_concrete(ctx);

        /*

         * If the private is also NULL then this is the first time we've

         * used this thread.

         */

        if (CRYPTO_THREAD_get_local(&dgbl->private) == NULL

                && !ossl_init_thread_start(NULL, ctx, rand_delete_thread_state))

            return NULL;

        rand = rand_new_drbg(ctx, primary, SECONDARY_RESEED_INTERVAL,

                             SECONDARY_RESEED_TIME_INTERVAL);

        CRYPTO_THREAD_set_local(&dgbl->public, rand);

    }

    return rand;

}
```

同时，可以发现`RAND_get0_private`同`RAND_get0_public`，使用了同样的优化机制：

```
EVP_RAND_CTX *RAND_get0_private(OSSL_LIB_CTX *ctx)

{

    RAND_GLOBAL *dgbl = rand_get_global(ctx);

    EVP_RAND_CTX *rand, *primary;



    if (dgbl == NULL)

        return NULL;



    //使用线程局部存储，减少了频繁的初始化及反初始化，以优化速度。

    rand = CRYPTO_THREAD_get_local(&dgbl->private);

    if (rand == NULL) {

        primary = RAND_get0_primary(ctx);

        if (primary == NULL)

            return NULL;



        ctx = ossl_lib_ctx_get_concrete(ctx);

        /*

         * If the public is also NULL then this is the first time we've

         * used this thread.

         */

        if (CRYPTO_THREAD_get_local(&dgbl->public) == NULL

                && !ossl_init_thread_start(NULL, ctx, rand_delete_thread_state))

            return NULL;

        rand = rand_new_drbg(ctx, primary, SECONDARY_RESEED_INTERVAL,

                             SECONDARY_RESEED_TIME_INTERVAL);

        CRYPTO_THREAD_set_local(&dgbl->private, rand);

    }

    return rand;

}
```

# 总结

看来，"内存泄露"的原因已经明确了，源于OpenSSL为了优化速度，对`EVP_RAND_CTX`的生成使用了线程局部存储机制，当RSA在新建线程中使用完毕时，如果未作线程资源清理动作，每调用一次`RSA_public_encrypt`或`RSA_private_decrypt`便会造成（3*0x1a4）字节的内存资源未释放。测试代码的新线程使用`RSA_public_encrypt`和`RSA_private_decrypt`各一次，因此共计（2*3*0x1a4）字节，与预期一致。

其它Rsa函数如`RSA_public_decrypt`等，未做深究，想来应该会有同样的问题存在。

由于在IOCP模型下不会频繁的创建新线程，这个问题便被很好地'掩盖'了, 而在加入了WinHttp异步操作后，由于频繁的在WinHttp回调函数(新线程)中执行Rsa函数，这个问题便不可避免的暴露出来了，考虑到具体应用场景，将Rsa相关操作迁移到IOCP回调中执行即可。

PS：

补充一点，在分析32位应用时，最好使用32位`gflag.exe`进行配置，同时使用32位任务管理器生成dmp文件且用32位`WinDBG`进行分析，否则可能会出现一些影响分析的兼容性问题。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2021-5-10 15:47

被Anakin Stone编辑

，原因：

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
