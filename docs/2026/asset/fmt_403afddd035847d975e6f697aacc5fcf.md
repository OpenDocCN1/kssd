
`LoadReadlyAddress();`
`String modulePath = "unidbg-android/src/test/resources/example_binaries/ollvm_fla/libnative-lib.so";`
`byte[] sodata = readFile(modulePath);`
`// 遍历真实块。然后直接修改成跳转真实块`
`for (int i = 0; i < readlyBlock.size(); i++) {`
`if (i < readlyBlock.size() - 1) {`
`// 获取当前真实块`
`Pair<Long, Capstone.CsInsn[]> block = readlyBlock.get(i);`
`System.out.println(String.format("curBlock address:0x%x", block.getKey()));`
`// 获取下一个真实块`
`Pair<Long, Capstone.CsInsn[]> nextBlock = readlyBlock.get(i + 1);`
`// 取出当前真实块最后一个指令的地址`
`int end_address = GetEndAddress(block.getValue()).intValue();`
`if (end_address <= 0) {`
`continue;`
`}`
`// 获取下一个真实块第一个指令的地址`
`int start_address = GetStartAddress(nextBlock.getValue()).intValue();`
`// 这里是最后跳转出结束的地方。由于已经被我们nop掉了循环。所以如果下一个块是跳转出 while 的。可以直接跳过了`
`if (nextBlock.getKey().intValue() == 0x4000f76c) {`
`continue;`
`}`
`// 准备转换汇编代码进行替换`
`try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm64, KeystoneMode.LittleEndian)) {`
`int subAddress = start_address - end_address;`
`// 用来patch修改的asm指令。这里是要计算出当前地址的相对地址跳转。所以上面要减一下。`
`String asmStr = String.format("b #0x%x", subAddress);`
`// 这个是我们显示日志看结果的。看看和我们之前手动分析的是不是差不多`
`String showStr = String.format("b #0x%x", start_address);`
`System.out.println(String.format("address:0x%x chang asm:%s", end_address, showStr));`
`// 转换汇编`
`KeystoneEncoded encoded = keystone.assemble(asmStr);`
`byte[] patch = encoded.getMachineCode();`
`if (patch.length <= 0) {`
`System.out.println("转换汇编失败");`
`return;`
`}`
`System.out.println(bytesToHexString(patch));`
`// 替换原来的字节数据`
`for (int y = 0; y < patch.length; y++) {`
`sodata[end_address + y] = patch[y];`
`}`
`}`
`}`
`}`
`// 循环的地方给nop掉`
`byte[] nop_byte = new byte[]{0x1F, 0x20, 0x03, (byte)0xD5};`
`int nop_address = 0xF778;`
`for (int y = 0; y < nop_byte.length; y++) {`
`sodata[nop_address + y] = nop_byte[y];`
`}`
`String savepath = "unidbg-android/src/test/resources/example_binaries/ollvm_fla/libnative-lib.patch.so";`
`writeFile(sodata, savepath);`
`System.out.println("处理完成");`

然后看看日志。因为我们之前已经静态分析解混淆过一次了。所以结果我们是清楚的。那么下面的结果对比我们手动的答案要不同

### 日志输出示例

```
address:0xf484 chang asm:b #0xf5f8
address:0xf620 chang asm:b #0xf624
address:0xf644 chang asm:b #0xf648
address:0xf654 chang asm:b #0xf750
address:0xf758 chang asm:b #0xf76c
```

问题其实就是因为我们没有处理分支。只是单纯根据执行结果在跳转。那么这里我们需要判断一下`csel`来特殊处理一下。

需要处理的地方有两处。

1、分支的当前关联。也就是仿照我们手动关联时的`cmp + b.eq + b`来实现跳转两个地方

2、未执行分支的后续关联。手动关联是我们虽然不知道执行哪个分支。但是看`v6`的变化可以找到下一个真实块。但是自动化时。我们前面获取到的真实块。都是基于执行流程的。未执行部分的`block`块我们并未保存下来。所以我们要不就是一步步解析未执行分支的后续关联。要不就是直接修改分支判断的寄存器。把另一条分支的执行块也给记录下来。

我们先调整下`hook`部分的代码。将分支不同的执行块结果保存出来。

```java
emulator.getBackend().hook_add_new(new BlockHook() {
    @Override
    public void hook(Backend backend, long address, int size, Object user) {
        // 这里的insns是整个block。
        Capstone.CsInsn[] insns = emulator.disassemble(address, size, 0);
        System.out.println(String.format("address:0x%x size:0x%x", address, insns.length));
        if (brance_data == 0) {
            blocks.add(new Pair<Long, Capstone.CsInsn[]>(address, insns));
        } else {
            branchBlocks.add(new Pair<Long, Capstone.CsInsn[]>(address, insns));
        }
    }
}, 0x4000F44C, 0x4000F778, null);

// 碰到csel的时候把w8的值修改下。控制走其他分支
emulator.getBackend().hook_add_new(new CodeHook() {
    @Override
    public void hook(Backend backend, long address, int size, Object user) {
        // 这里的insns是整个block。
        Capstone.CsInsn[] insns = emulator.disassemble(address, size, 0);
        for (Capstone.CsInsn ins : insns) {
            // 这里就是控制如果brance_data为 0 就固定走第一个分支。为 1 就固定走第二个分支
            if (ins.mnemonic.equals("csel")) {
                int w9 = emulator.getBackend().reg_read(Arm64Const.UC_ARM64_REG_W9).intValue();
                int w10 = emulator.getBackend().reg_read(Arm64Const.UC_ARM64_REG_W10).intValue();
                if (brance_data == 0) {
                    emulator.getBackend().reg_write(Arm64Const.UC_ARM64_REG_W10, w9);
                } else {
                    emulator.getBackend().reg_write(Arm64Const.UC_ARM64_REG_W9, w10);
                }
            }
        }
    }
}, 0x4000F44C, 0x4000F778, null);

// 调用一个返回值为 object 的静态的jni函数
StringObject res = mainActivityDvm.callStaticJniMethodObject(emulator, "stringFromJNI()Ljava/lang/String;");
System.out.println(res.toString());
brance_data = 1;
StringObject res2 = mainActivityDvm.callStaticJniMethodObject(emulator, "stringFromJNI()Ljava/lang/String;");
System.out.println(res2.toString());
LoadReadlyAddress();
```

然后就可以在跳转的地方获取到两个跳转的真实块地址了。然后打上`patch`替换掉原来的三行汇编

```java
// 获取当前块中的csel地址。如果没有csel则为 0
Long cselAddress = GetCselddress(block.getValue());
// 存在csel指令说明是有分支。然后要特殊处理
if (cselAddress > 0) {
    // 获取另一个分支的下一个真实块
    int start_address2 = GetBranchReadlyAddress(block.getKey());
    try (Keystone keystone = new Keystone(KeystoneArchitecture.Arm64, KeystoneMode.LittleEndian)) {
        /*
         * 要改以下三行的汇编。前面获取出来的cselAddress是CSEL的地址。
         * TST             W8, #1
         * CSEL            W8, W9, W10, NE
         * STR             W8, [SP,#0x50+var_2C]
         */
        int subAddress1 = start_address - (cselAddress.intValue()) + 4;
        int subAddress2 = start_address2 - (cselAddress.intValue()) + 4;
        String asm1 = "cmp w8,#0x1";
        String showAsm1 = String.format("b.eq 0x%x", start_address);
        String showAsm2 = String.format("b #0x%x", start_address2);
        System.out.println(String.format("address:0x%x chang asm1:%s", cselAddress, showAsm1));
        System.out.println(String.format("address:0x%x chang asm2:%s", cselAddress, showAsm2));
        String asm2 = String.format("b.eq 0x%x", subAddress1);
        String asm3 = String.format("b 0x%x", subAddress2);
        // 转换汇编
        KeystoneEncoded encoded = keystone.assemble(Arrays.asList(asm1, asm2, asm3));
        byte[] patch = encoded.getMachineCode();
        if (patch.length <= 0) {
            System.out.println("转换汇编失败");
            return;
        }
        System.out.println(bytesToHexString(patch));
        // 这里去掉基址。再往上一个指令的位置开始写入
        int replace_address = cselAddress.intValue() - 4;
        // 替换原来的字节数据
        for (int y = 0; y < patch.length; y++) {
            sodata[replace_address + y] = patch[y];
        }
    }
    continue;
}
```

贴上执行完成后解析的结果

```c
__int64 __usercall calcKey@<X0>(__int64 a1@<X0>, __int64 a2@<X8>)
{
    __int64 v2;
    // x0
    __int64 v4;
    // [xsp+10h] [xbp-40h]
    __int64 v5;
    // [xsp+18h] [xbp-38h]

    v5 = a2;
    v4 = a1;
    if ( (unsigned __int64)sub_F77C(a1) > 0xA == 1 )
    {
        sub_F830(v4, "ceshi");
        v2 = sub_F8E4(v5, v4);
    }
    else
    {
        v2 = sub_F77C(v4);
    }
    return sub_F77C(v2);
}
```

对比之前我们手动还原的结果看了下。还是差了点。但是当前的执行流程的反混淆结果可以看了。未执行部分的分支如何去关联我想了好久越想越复杂了。就没有继续倒腾了。有知道的大佬麻烦提点下。

中间还有一些细节处理的地方没有贴代码，我就直接丢上地址了。代码我都尽量写上注释了。
