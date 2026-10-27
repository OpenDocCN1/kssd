
# LLVM IR 指令转译为字节流

## `packValue` 函数

```cpp
#define GET_PACK_VALUE(value) (packValue(value, &value_map))
// pack type to a vector(2)
// {size, TypeID}
std::vector<uint8_t> type_to_hex(Type *type)
{
std::vector<uint8_t> res;
res.push_back(modDataLayout->getTypeAllocSize(type));
res.push_back(type->getTypeID());
return res;
}
// pack a value
std::vector<uint8_t> packValue(Value *value, std::map<Value *, int> *value_map)
{
std::vector<uint8_t> res;
std::vector<uint8_t> packed;
std::vector<uint8_t> packType = type_to_hex(value->getType());
if (ConstantData *CD = dyn_cast<ConstantData>(value))
{
packed = pack_const_value(value);
}
else
{
// if value not in map
if (value_map->find(value) == value_map->end())
{
// check value is not a GlobalVariable
if (GlobalVariable *gv = dyn_cast<GlobalVariable>(value))
{
// is a GlobalVariable and not in value_map
// put it into value_map
insert_to_value_map(value_map, value, curr_data_offset);
// also put it into gv_value_map
gv_value_map.insert(pair<GlobalVariable *, int>(gv, curr_data_offset));
int res_size = modDataLayout->getTypeAllocSize(gv->getType());
curr_data_offset += res_size;
}
else
{
assert(value_map->find(value) != value_map->end());
}
}
packed = pack((*value_map)[value], POINTER_SIZE);
// variable，packtype->TypeID=0
packType[1] = 0;
}
res.insert(res.end(), packType.begin(), packType.end());
res.insert(res.end(), packed.begin(), packed.end());
return res;
}
```

## `alloca` 指令

最終`alloca`指令會被轉譯成如下字節流，保存在`gv_code_seg`中。

```cpp
// | 字節碼 | 打包的返回值 | 申請的空間 |
<ALLOCA_OP> | <var_size, var_type, var_data_off> | <alloca_area>
```

同時看看VM解釋器是怎樣解析上述字節流的。

`get_byte_code()`會返回`gv_code_seg[ip++]`，但`alloca`的返回值固定是8字節的變量，因此`var_size`和`var_type`都沒有用。

`unpack_code()`會以小端形式從`gv_code_seg`讀取指定大小的值。`data_seg_addr`是運行時`gv_data_seg`的實際地址，最後那句代碼會把`data_seg_addr + area_offset`這個值保存到`data_seg_addr + var_offset`這個地址。

```cpp
#ifdef IS_INLINE_FUNC
__inline__ __attribute__((always_inline))
#endif
void alloca_handler() {
// size and type of pointer is useless
uint8_t var_size = get_byte_code();
uint8_t var_type = get_byte_code();
// get pointer var offset
uint64_t var_offset = unpack_code(POINTER_SIZE);
// get alloca area offset
uint64_t area_offset = unpack_code(POINTER_SIZE);
// store area virtual address to var
// set_var(var_offset, POINTER_SIZE, data_seg_addr+area_offset);
pack_store_addr(data_seg_addr+var_offset, data_seg_addr+area_offset, var_size);
}
```

## `LoadInst` 指令

`LoadInst`指令：( 例子 ⇒ `%val = load i32, i32* %ptr` )

- `return`：`LoadInst`的返回值。

- `PointerOperand`：`LoadInst`中第`0`個操作數，即某條`alloca`指令的返回值。

```cpp
else if (LoadInst *inst = dyn_cast<LoadInst>(ins))
{
// return
int res_offset = curr_data_offset;
insert_to_value_map(&value_map, inst, curr_data_offset);
int res_size = modDataLayout->getTypeAllocSize(inst->getType());
curr_data_offset += res_size;
std::vector<uint8_t> packed_res = GET_PACK_VALUE(inst);
// nglog: LoadInst指令的getPointerOperand()會返回「指針操作數」(即某個AllocaInst的返回值)
// PointerOperand
std::vector<uint8_t> packed_pointer_operand = GET_PACK_VALUE(inst->getPointerOperand());
std::vector<uint8_t> hex_code;
ins_to_hex(hex_code, pack_op(LOAD_OP), packed_res, packed_pointer_operand);
vm_code.insert(vm_code.end(), hex_code.begin(), hex_code.end());
}
```

## `StoreInst` 指令

`StoreInst`指令：( 例子 ⇒ `store i32 3, ptr %ptr` )

- `StoreInst`沒有返回值。

- `ValueOperand`和`PointerOperand`可從`StoreInst`的例子明顯看出。

```cpp
else if (StoreInst *inst = dyn_cast<StoreInst>(ins))
{
// ValueOperand
std::vector<uint8_t> packed_value_operand = GET_PACK_VALUE(inst->getValueOperand());
// PointerOperand
std::vector<uint8_t> packed_pointer_operand = GET_PACK_VALUE(inst->getPointerOperand());
std::vector<uint8_t> hex_code;
ins_to_hex(hex_code, pack_op(STORE_OP), packed_value_operand, packed_pointer_operand);
vm_code.insert(vm_code.end(), hex_code.begin(), hex_code.end());
}
```

## `GetElementPtrInst` 指令

`GetElementPtrInst`指令：( 例子 ⇒ `%elem_ptr = getelementptr i32, ptr %array_ptr, i32 2` )

- 收集了所有索引，但只會用只後一個索引。

- 分成`struct`和`array`兩種情況，前者要計算字段偏移，後者直接記錄索引值即可。

```cpp
else if (GetElementPtrInst *inst = dyn_cast<GetElementPtrInst>(ins))
{
int res_offset = curr_data_offset;
insert_to_value_map(&value_map, inst, curr_data_offset);
int res_size = modDataLayout->getTypeAllocSize(inst->getType());
curr_data_offset += res_size;
std::vector<uint8_t> packed_res = GET_PACK_VALUE(inst);
std::vector<uint8_t> packed_ptr = GET_PACK_VALUE(inst->getPointerOperand());
// get indices
// but only consider last indice
std::vector<Value *> indices;
for (auto curr_idx = inst->idx_begin(); curr_idx != inst->idx_end(); curr_idx++)
{
indices.push_back(*curr_idx);
}
// GEP type
// {0, 0}: structure value is offset
// {x, x}: array, value is offset
Type *srcType = inst->getSourceElementType();
std::vector<uint8_t> gep_type;
std::vector<uint8_t> packed_value;
if (dyn_cast<StructType>(srcType))
{
// is struct type
StructType *st = dyn_cast<StructType>(srcType);
gep_type = {0, 0};
ConstantInt *CI = dyn_cast<ConstantInt>(indices[indices.size() - 1]); // last indice
int element_idx = CI->getSExtValue(); // const value to int
int curr_element_offset = 0;
for (int i = 0; i < element_idx; i++)
{ // calc the offset between curr_element and struct_begin
curr_element_offset += modDataLayout->getTypeAllocSize(st->getElementType(i));
}
// Construct const-offset manually
packed_value = {0, 0};
std::vector<uint8_t> tmp = pack(curr_element_offset, POINTER_SIZE);
packed_value.insert(packed_value.end(), tmp.begin(), tmp.end());
}
else
{
// is array type
gep_type = type_to_hex(inst->getResultElementType());
packed_value = GET_PACK_VALUE(indices[indices.size() - 1]);
}
std::vector<uint8_t> hex_code;
ins_to_hex(hex_code, pack_op(GEP_OP), gep_type, packed_res, packed_ptr, packed_value);
vm_code.insert(vm_code.end(), hex_code.begin(), hex_code.end());
}
```

為什麼只指最後一個索引就可以？看一個實際例子。

```cpp
int test16(int arr[3][2]) {
return arr[2][0];
}
```

在`-O0`優化下，上述代碼對應的llvm ir如下。

可以看到在取`arr[2][0]`時分成了2條`gep`指令，第1條`gep`指令返回`&arr[2]`。

而第2條`gep`指令有兩個索引，第1個0代表`&arr[2]`( 因為`<ty>`是`[2 x i32]` )，第2個0代表`&arr[2][0]`。

由此可見在`-O0`優化下會把多層的指針操作拆分到多條`gep`指令來執行，單條`gep`指令最多只有2個索引( 不確定是否有例外？)，只有最後那個索引才是「有用」的。

```llvm
; Function Attrs: mustprogress noinline nounwind optnone uwtable
define dso_local noundef i32 @_Z6test16PA2_i(ptr noundef %arr) #4 {
entry:
%arr.addr = alloca ptr, align 8
store ptr %arr, ptr %arr.addr, align 8
%0 = load ptr, ptr %arr.addr, align 8
%arrayidx = getelementptr inbounds [2 x i32], ptr %0, i64 2
%arrayidx1 = getelementptr inbounds [2 x i32], ptr %arrayidx, i64 0, i64 0
%1 = load i32, ptr %arrayidx1, align 4
ret i32 %1
}
```

然後看看`-O1`優化，它把2條`gep`指令合併成1條，類型被修改為`i8`，索引只有1個`16`，相當於返回`((uint8_t*)arr + 16)`，它等價於`&arr[2][0]`。

```llvm
; Function Attrs: mustprogress nofree norecurse nosync nounwind willreturn memory(argmem: read) uwtable
define dso_local noundef i32 @_Z6test16PA2_i(ptr nocapture noundef readonly %arr) #7 {
entry:
%arrayidx = getelementptr inbounds i8, ptr %arr, i64 16
%0 = load i32, ptr %arrayidx, align 4, !tbaa !5
ret i32 %0
}
```

## `BranchInst` 指令

`BranchInst`指令：( 例子 ⇒ `br i1 %cmp, label %if.then, label %if.end` )

- 分成有條件和無條件跳轉，`0`代表無條件，`1`代表有條件。

- 跳轉地址暫時置空( `vm_code`傳入`padding`來占位 )，跳轉信息先保存在`br_map`，在遍歷完整個函數後統一處理。
