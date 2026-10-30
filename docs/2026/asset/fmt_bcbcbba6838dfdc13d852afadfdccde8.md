
```python
import re, struct, json, sys, io, os
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8', errors='replace')
SETUP\_PE = r'D:\Z370\_DICOPEN\z370.bin\_output\regions\region-bios\volume-0\file-899407d7-99fe-43d8-9a21-79ec328cac21\section1\section0.pe'
STRINGS\_JSON = r'D:\Z370\_DICOPEN\setup\_strings.json'
OUTPUT\_JSON = r'D:\Z370\_DICOPEN\all\_questions.json'
if not os.path.isfile(SETUP\_PE):
print(f"[ERR] need {SETUP\_PE}"); sys.exit(1)
if not os.path.isfile(STRINGS\_JSON):
print(f"[ERR] run phase3 first to produce {STRINGS\_JSON}"); sys.exit(1)
blob = open(SETUP\_PE, 'rb').read()
strings = {int(k):v for k,v in json.load(open(STRINGS\_JSON, encoding='utf-8')).items()}
print(f"PE32 size: {len(blob)} bytes, {len(strings)} strings loaded")
# --- Step 1: Locate Forms Package (type 0x02) ---
# Header: 4 bytes -- 3B length + 1B type=0x02
# Right after header should be a FormSet opcode (0x0E).
print("\n[Step 1] Locate Forms Package (type 0x02)")
forms\_pkgs = []
for i in range(0, len(blob) - 32):
plen = int.from\_bytes(blob[i:i+3], 'little')
ptype = blob[i+3]
if ptype != 0x02: continue
if plen < 0x100 or plen > 0x800000: continue
if blob[i+4] == 0x0E and blob[i+5] >= 0x18 and i+plen <= len(blob):
forms\_pkgs.append((i, plen))
for fp in forms\_pkgs:
print(f" candidate @ 0x{fp[0]:x}, length 0x{fp[1]:x}")
if not forms\_pkgs:
print("[ERR] no forms package found"); sys.exit(1)
fp\_pos, fp\_len = max(forms\_pkgs, key=lambda x: x[1])
print(f"using Forms package @ 0x{fp\_pos:x} (length 0x{fp\_len:x})")
# --- Step 2: Walk IFR opcodes ---
print("\n[Step 2] Walk IFR opcode stream (with 7-bit length masking)")
OP\_NAMES = {0x01:'Form',0x05:'OneOf',0x06:'CheckBox',0x07:'Numeric',0x09:'OneOfOpt',
0x0A:'SuppressIf',0x0E:'FormSet',0x19:'GrayOutIf',0x1E:'DisableIf',
0x23:'OrderedList',0x24:'VarStore',0x29:'End',0x5A:'Default'}
varstores = {}
questions = []
form\_stack = []
p = fp\_pos + 4 # skip 4-byte package header
end = fp\_pos + fp\_len
while p < end:
if p + 2 > len(blob): break
op = blob[p]
raw\_len = blob[p+1]
ln = raw\_len & 0x7F # <-- critical
scope = (raw\_len >> 7) & 1
if ln < 2: break
body = blob[p+2:p+ln]
if op == 0x24: # VarStore
guid = body[:16].hex()
vsid = struct.unpack('<H', body[16:18])[0]
vsz = struct.unpack('<H', body[18:20])[0]
ne = body.find(b'\x00', 20)
nm = body[20:ne].decode('ascii','replace') if ne>=0 else '?'
varstores[vsid] = {'name':nm, 'size':vsz, 'guid':guid}
elif op == 0x01: # Form
fid = struct.unpack('<H', body[:2])[0]
ttl = struct.unpack('<H', body[2:4])[0]
form\_stack.append({'fid':fid, 'title':strings.get(ttl,'?')})
elif op in (0x05, 0x06, 0x07, 0x23):
# Question: prompt(2) help(2) qid(2) vsid(2) voff(2) flags(1)
if len(body) >= 11:
prompt = struct.unpack('<H', body[0:2])[0]
help\_ = struct.unpack('<H', body[2:4])[0]
qid = struct.unpack('<H', body[4:6])[0]
vsid = struct.unpack('<H', body[6:8])[0]
voff = struct.unpack('<H', body[8:10])[0]
flags = body[10]
questions.append({
'op': op, 'opname': OP\_NAMES.get(op,'?'),
'prompt': prompt, 'help': help\_, 'qid': qid,
'vsid': vsid, 'voff': voff, 'flags': flags,
'len': ln, 'pos': p,
'form': form\_stack[-1]['title'] if form\_stack else None,
})
elif op == 0x29 and form\_stack: # End closes form scope
form\_stack.pop()
p += ln
print(f" walked OK: {len(varstores)} VarStores, {len(questions)} Questions")
# --- Step 3: Show key VarStores (especially Setup) ---
print("\n[Step 3] VarStores discovered (showing first 10)")
for vsid, vs in sorted(varstores.items())[:10]:
print(f" VsId=0x{vsid:04X} name={vs['name']!r:30s} size={vs['size']:>5} guid={vs['guid'][:16]}...")
# --- Step 4: Find Questions referencing our target StringIds ---
print("\n[Step 4] Match Questions to DCI/Debug menu StringIds")
TARGETS = {
0x07BE: 'DCI enable (HDCIEN)',
0x033D: 'Debug Interface',
0x033F: 'Debug Interface Lock',
0x019C: 'JTAG C10 Power',
}
print(f"{'Target StringId':<16} {'Menu Text':<28} {'VsId':<8} {'VarOffset':<15} {'Form'}")
print('-'\*100)
for sid, name in TARGETS.items():
matches = [q for q in questions if q['prompt'] == sid]
for q in matches:
vs = varstores.get(q['vsid'], {'name':'?'})
print(f" 0x{sid:04X} {name:<28} VsId=0x{q['vsid']:x} 0x{q['voff']:04X} ({q['voff']:>4}) {q['form']}")
# Save all questions for Phase 5
with open(OUTPUT\_JSON, 'w', encoding='utf-8') as f:
json.dump(questions, f, ensure\_ascii=False)
print(f"\nSaved {len(questions)} questions → {OUTPUT\_JSON}")
```

![图片描述](upload/attach/202605/992284\_V3TG74G9D5D5VQW.webp)

具体如何解析的可以看代码。

接下来就需要找到`0x904`在哪，这个解析出来的不是文件office，是NVRAM容器格式的偏移。

NVRAM容器的Guid是：`cef5b9a3-476d-497f-9fdc-e98143e0422c`。而我们需要修改的是他出厂的默认设置：

```
NVRAM 容器
└── NVAR 记录 "StdDefaults" (14715 字节) ← 包所有 Setup 变量的出厂默认值
└── 嵌套 NVAR "Setup" (4263 字节) ← 我们的 Setup 变量默认值就这里
└── 这 4263 字节里第 0x904 个就是 DciEn 字节
```

首先需要解析 AMI NVAR 的私有格式。好在通过Ai搜寻 UEFITool 源码中是有参考的。这里写了一个脚本来寻找并且Patch一下：

![图片描述](upload/attach/202605/992284\_N7G2BGQJ59EGB4A.webp)

解析代码如下：
