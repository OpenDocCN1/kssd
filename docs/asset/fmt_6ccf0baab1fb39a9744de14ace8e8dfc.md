
`site_code.keys = {VENDOR_KEY1, VENDOR_KEY2, VENDOR_KEY3, VENDOR_KEY4} */`

`LM_CODE(site_code, ENCRYPTION_SEED1, ENCRYPTION_SEED2, VENDOR_KEY1,`
`VENDOR_KEY2, VENDOR_KEY3, VENDOR_KEY4, VENDOR_KEY5);`

... ...

`int main(int argc, char **argv)`

{
... ...
`/*设置 site_code.data = {ENCRYPTION_SEED1, ENCRYPTION_SEED2} */`

`LM_CODE_GEN_INIT(&site_code);`

`if (lc_init((LM_HANDLE *)0, VENDOR_NAME, &site_code, &lm_job))`

{
`lc_perror(lm_job, "lc_init failed");`

`exit(-1);`

}

... ...

`/*调用链接 dofilecrypt() -> dofpcrypt() -> lc_cryptstr() */`

`estat |= dofilecrypt(infilename, outfilename, &site_code);`

`return 0;`
}

这是 `lmcrypt.c` 的简明源代码及其自身说明。仔细注意到在 `lmclient.h` 中定义的两个宏 `LM_CODE` 和 `LM_CODE_GEN_INIT`。前者初始化 `site_code.data` 为与 `VENDOR_KEY5` 进行与或的加密种子（与 `omcode.c` 一致），但后者很快逆向它为原始的加密种子。我是否曾告诉你 `FLEXLM` 有很糟的代码类型？：）

不管怎样，原始的加密种子和零 `job->mem_ptr2_bytes` 值被用于密钥产生（`keygen`），它不同于检查（`checkout`）。很自然的事情是从检查处理拷贝 `code->data` 到 `lmseeds.h` 作为加密种子 1 和 2，并重新编译 `lmcrypt.exe`。但它并不工作，因为在表格 `52xxxxB8` 和 `75yyyy0F` 中，`code->data` 是随机的，而 `xxxx` 和 `yyyy` 在每次运行中都将改变。相同的是，对于 `job->mem_ptr2_bytes` 是真实的。我们推断，加密种子必定被模糊并存储在 `vendor` 软件中的两个地方：`code->data[]` 和 `job->mem_ptr2_bytes[]`（它们应该是关系很近的一对），因为它提供给最终用户而原始种子需要保护。相反，`lmcrypt.exe` 仅对 `vendor` 可获得，因此加密种子可以以一般形式出现。

`4dd466d0e1e0cc123afbe504da2462ca`
