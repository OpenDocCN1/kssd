
## LIF-Content

`LIF`文件中的`content`虽然是`ANY`类型，但可以通过`contentType`来判断`content`的类型。而`oid1.3.6.1.4.1.44485.2.1`正是指`content`需要使用`LIF-Content`进行编解码。

```asn1
LIF-Content.asn1
LIF-CONTENT DEFINITIONS ::= BEGIN
    LIF-Content ::= [PRIVATE 228] IMPLICIT SEQUENCE
    {
        sw-specs [PRIVATE 161] IMPLICIT Software-Specs,
        clocks [PRIVATE 100] IMPLICIT Clocks,
        license-description [PRIVATE 125] IMPLICIT License-Description,
        binding [PRIVATE 136] IMPLICIT Binding-Scheme,
        cmact-options [PRIVATE 133] IMPLICIT CmAct-Options,
        mask [PRIVATE 142] IMPLICIT Mask OPTIONAL
    }
    Software-Specs ::= [PRIVATE 161] IMPLICIT SEQUENCE
    {
        creator-name [PRIVATE 181] IMPLICIT UTF8String,
        creator-version [PRIVATE 165] IMPLICIT Format-Version,
        required-version [PRIVATE 122] IMPLICIT Format-Version
    }
    Format-Version ::= [PRIVATE 122] IMPLICIT SEQUENCE
    {
        sfl [PRIVATE 78] IMPLICIT UInt8,
        sfh [PRIVATE 84] IMPLICIT UInt8,
        feature-flags [PRIVATE 32] IMPLICIT UInt32
    }
    UInt8 ::= [PRIVATE 31] IMPLICIT INTEGER
    UInt32 ::= [PRIVATE 34] IMPLICIT INTEGER
    Clocks ::= [PRIVATE 100] IMPLICIT SEQUENCE
    {
        box-time [PRIVATE 178] IMPLICIT Seconds-Since-2000,
        certified-time [PRIVATE 179] IMPLICIT Seconds-Since-2000
    }
    Seconds-Since-2000 ::= [PRIVATE 180] IMPLICIT INTEGER
    License-Description ::= [PRIVATE 125] IMPLICIT SEQUENCE
    {
        license-description [PRIVATE 30] IMPLICIT OCTET STRING, -- BMPString, it does't work well in asn1tools, beacase asn1tools only support utf-8, but here is utf-16
        firm-code [PRIVATE 114] IMPLICIT Firm-Code
    }
    Firm-Code ::= [PRIVATE 114] IMPLICIT INTEGER
    Binding-Scheme ::= [PRIVATE 136] IMPLICIT SEQUENCE
    {
        cmact-id [PRIVATE 148] IMPLICIT CmAct-ID,
        telephone-id [PRIVATE 127] IMPLICIT Telephone-ID,
        binding-method [PRIVATE 128] EXPLICIT Binding-Method
    }
    CmAct-ID ::= [PRIVATE 148] IMPLICIT IA5String
    Telephone-ID ::= [PRIVATE 127] IMPLICIT INTEGER
    Binding-Method ::= CHOICE
    {
        smartbind [PRIVATE 129] IMPLICIT SmartBind-Parameters,
        custom-binding [PRIVATE 131] IMPLICIT CustomBinding-Parameters,
        dcbn-classic [PRIVATE 132] IMPLICIT DCBN-Parameters,
        nonebind [PRIVATE 231] IMPLICIT NULL,
        cm-server-ip [PRIVATE 232] IMPLICIT NULL,
        serial-number [PRIVATE 233] IMPLICIT NULL,
        random [PRIVATE 234] IMPLICIT NULL
    }
    SmartBind-Parameters ::= [PRIVATE 129] IMPLICIT SEQUENCE
    {
        heuristic [PRIVATE 31] IMPLICIT UInt8,
        redundancy-level [PRIVATE 130] IMPLICIT Redundancy-Level
    }
    Redundancy-Level ::= [PRIVATE 130] IMPLICIT INTEGER
    CustomBinding-Parameters ::= [PRIVATE 131] IMPLICIT IA5String
    DCBN-Parameters ::= [PRIVATE 132] IMPLICIT SEQUENCE
    {
        disk [PRIVATE 1] IMPLICIT BOOLEAN,
        cpu-type [PRIVATE 1] IMPLICIT BOOLEAN,
        board [PRIVATE 1] IMPLICIT BOOLEAN,
        network [PRIVATE 1] IMPLICIT BOOLEAN,
        tolerance [PRIVATE 2] IMPLICIT UInt8 OPTIONAL
    }
    CmAct-Options ::= [PRIVATE 133] IMPLICIT SEQUENCE
    {
        allow-vm [PRIVATE 219] IMPLICIT BOOLEAN,
        allow-reimport [PRIVATE 220] IMPLICIT BOOLEAN
    }
    Mask ::= [PRIVATE 142] IMPLICIT INTEGER
END
```

### 测试例子

```python
>>> lif = bytes.fromhex("FF816481C7FF812137DF813508636D626F7870676DFF812512DF4E020001DF54020001DF20050000000000FF7A12DF4E020001DF54020001DF20050000000000FF6412DF81320500272ADC71DF8133050027048F5FFF7D3ADE3054006500720072006100200043006F006E007400610069006E00650072002000760032002E0033002E0031002E003400DF720500005B9126FF810828DF81140432303031DF7F050000000000FF810014FF810410C101FFC101FFC101FFC10100C2020001FF81050ADF815B0100DF815C0100")
>>> asn_lif_res = asn1_def.decode("LIF-Content", lif)
>>> print_asn1_data(asn_lif_res)
<sw-specs>
  <creator-name> cmboxpgm </creator-name>
  <creator-version>
    <sfl> 1 </sfl>
    <sfh> 1 </sfh>
    <feature-flags> 0 </feature-flags>
  </creator-version>
  <required-version>
    <sfl> 1 </sfl>
    <sfh> 1 </sfh>
    <feature-flags> 0 </feature-flags>
  </required-version>
</sw-specs>
<clocks>
  <box-time> 657120369 </box-time>
  <certified-time> 654610271 </certified-time>
</clocks>
<license-description>
  <license-description> 54 00 65 00 72 00 72 00 61 00 20 00 43 00 6F 00 6E 00 74 00 61 00 69 00 6E 00 65 00 72 00 20 00 76 00 32 00 2E 00 33 00 2E 00 31 00 2E 00 34 00 </license-description>
  <firm-code> 6000934 </firm-code>
</license-description>
<binding>
  <cmact-id> 2001 </cmact-id>
  <telephone-id> 0 </telephone-id>
  <binding-method>
    <dcbn-classic>
      <disk> True </disk>
      <cpu-type> True </cpu-type>
      <board> True </board>
      <network> False </network>
      <tolerance> 1 </tolerance>
    </dcbn-classic>
  </binding-method>
</binding>
<cmact-options>
  <allow-vm> False </allow-vm>
  <allow-reimport> False </allow-reimport>
</cmact-options>
```

## Certificate

`LIF`文件中的数字证书用于签名。多个证书还构成了证书链，证书链中的每个父节点都可以校验子节点下的任何一份证书。
