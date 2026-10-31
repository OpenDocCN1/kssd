
```diff
# Copyright 2023 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
diff --git a/src/common/code-memory-access-inl.h b/src/common/code-memory-access-inl.h
index 4b8ac2e5c7..58542dc4e5 100644
--- a/src/common/code-memory-access-inl.h
+++ b/src/common/code-memory-access-inl.h
@@ -17,6 +17,18 @@
namespace v8 {
namespace internal {
+RwMemoryWriteScope::RwMemoryWriteScope() {
+ SetWritable();
+}
+
+RwMemoryWriteScope::RwMemoryWriteScope(const char \*comment) {
+ SetWritable();
+}
+
+RwMemoryWriteScope::~RwMemoryWriteScope() {
+ SetReadOnly();
+}
+
RwxMemoryWriteScope::RwxMemoryWriteScope(const char\* comment) {
if (!v8\_flags.jitless) {
SetWritable();
diff --git a/src/common/code-memory-access.cc b/src/common/code-memory-access.cc
index be3b9741d2..b8c59c331f 100644
--- a/src/common/code-memory-access.cc
+++ b/src/common/code-memory-access.cc
@@ -4,6 +4,10 @@
#include "src/common/code-memory-access-inl.h"
#include "src/utils/allocation.h"
+#include "src/common/globals.h"
+#include "src/execution/isolate-inl.h"
+
+#include <sys/mman.h>
namespace v8 {
namespace internal {
@@ -11,6 +15,68 @@ namespace internal {
ThreadIsolation::TrustedData ThreadIsolation::trusted\_data\_;
ThreadIsolation::UntrustedData ThreadIsolation::untrusted\_data\_;
+thread\_local int RwMemoryWriteScope::nesting\_level\_ = 0;
+
+static void ProtectSpace(Space \*space, int prot) {
+ if (space->memory\_chunk\_list().Empty()) {
+ return;
+ }
+
+ MemoryChunk \*c = space->memory\_chunk\_list().front();
+ while (c) {
+ void \*addr = reinterpret\_cast<void \*>(c->address());
+ // printf("making %p read-only\n", addr);
+ CHECK(mprotect(addr, RoundDown(sizeof(MemoryChunk), 0x1000), prot) == 0);
+ c = c->list\_node().next();
+ }
+}
+
+// static
+void RwMemoryWriteScope::SetWritable() {
+ if (nesting\_level\_ == 0) {
+ int prot = PROT\_READ | PROT\_WRITE;
+ Isolate \*isolate = Isolate::Current();
+ for (int i = FIRST\_MUTABLE\_SPACE; i < LAST\_SPACE; i++) {
+ Space \*space = isolate->heap()->space(i);
+ if (space == nullptr) {
+ continue;
+ }
+
+ if (!v8\_flags.minor\_mc && i == NEW\_SPACE) {
+ SemiSpaceNewSpace\* semi\_space\_new\_space = SemiSpaceNewSpace::From(static\_cast<NewSpace \*>(space));
+ ProtectSpace(&semi\_space\_new\_space->from\_space(), prot);
+ ProtectSpace(&semi\_space\_new\_space->to\_space(), prot);
+ } else {
+ ProtectSpace(space, prot);
+ }
+ }
+ }
+ nesting\_level\_++;
+}
+
+// static
+void RwMemoryWriteScope::SetReadOnly() {
+ nesting\_level\_--;
+ if (nesting\_level\_ == 0) {
+ int prot = PROT\_READ;
+ Isolate \*isolate = Isolate::Current();
+ for (int i = FIRST\_MUTABLE\_SPACE; i < LAST\_SPACE; i++) {
+ Space \*space = isolate->heap()->space(i);
+ if (space == nullptr) {
+ continue;
+ }
+
+ if (!v8\_flags.minor\_mc && i == NEW\_SPACE) {
+ SemiSpaceNewSpace\* semi\_space\_new\_space = SemiSpaceNewSpace::From(static\_cast<NewSpace \*>(space));
+ ProtectSpace(&semi\_space\_new\_space->from\_space(), prot);
+ ProtectSpace(&semi\_space\_new\_space->to\_space(), prot);
+ } else {
+ ProtectSpace(space, prot);
+ }
+ }
+ }
+}
+
#if V8\_HAS\_PTHREAD\_JIT\_WRITE\_PROTECT || V8\_HAS\_PKU\_JIT\_WRITE\_PROTECT
thread\_local int RwxMemoryWriteScope::code\_space\_write\_nesting\_level\_ = 0;
#endif // V8\_HAS\_PTHREAD\_JIT\_WRITE\_PROTECT || V8\_HAS\_PKU\_JIT\_WRITE\_PROTECT
diff --git a/src/common/code-memory-access.h b/src/common/code-memory-access.h
index e90dcc9a64..4e835c87c7 100644
--- a/src/common/code-memory-access.h
+++ b/src/common/code-memory-access.h
@@ -331,6 +331,22 @@ class V8\_NODISCARD RwxMemoryWriteScope {
#endif // V8\_HAS\_PTHREAD\_JIT\_WRITE\_PROTECT || V8\_HAS\_PKU\_JIT\_WRITE\_PROTECT
};
+class V8\_NODISCARD RwMemoryWriteScope final {
+ public:
+ V8\_INLINE RwMemoryWriteScope();
+ V8\_INLINE explicit RwMemoryWriteScope(const char \*comment);
+ V8\_INLINE ~RwMemoryWriteScope();
+
+ RwMemoryWriteScope(const RwMemoryWriteScope&) = delete;
+ RwMemoryWriteScope& operator=(const RwMemoryWriteScope&) = delete;
+
+ private:
+ static void SetWritable();
+ static void SetReadOnly();
+
+ static thread\_local int nesting\_level\_;
+};
+
// This class is a no-op version of the RwxMemoryWriteScope class above.
// It's used as a target type for other scope type definitions when a no-op
// semantics is required.
@@ -346,7 +362,7 @@ using CodePageMemoryModificationScopeForPerf = RwxMemoryWriteScope;
#else
// Without per-thread write permissions, we only use permission switching for
// debugging and the perf impact of this doesn't matter.
-using CodePageMemoryModificationScopeForPerf = NopRwxMemoryWriteScope;
+using CodePageMemoryModificationScopeForPerf = RwMemoryWriteScope;
#endif
// Same as the RwxMemoryWriteScope but without inlining the code.
diff --git a/src/execution/isolate.cc b/src/execution/isolate.cc
index c935c8c5ca..2534b7bc2e 100644
--- a/src/execution/isolate.cc
+++ b/src/execution/isolate.cc
@@ -61,6 +61,7 @@
#include "src/handles/persistent-handles.h"
#include "src/heap/heap-inl.h"
#include "src/heap/heap-verifier.h"
+#include "src/heap/heap.h"
#include "src/heap/local-heap-inl.h"
#include "src/heap/parked-scope.h"
#include "src/heap/read-only-heap.h"
@@ -4236,6 +4237,8 @@ void Isolate::VerifyStaticRoots() {
bool Isolate::Init(SnapshotData\* startup\_snapshot\_data,
SnapshotData\* read\_only\_snapshot\_data,
SnapshotData\* shared\_heap\_snapshot\_data, bool can\_rehash) {
+ CodePageHeaderModificationScope scope{};
+
TRACE\_ISOLATE(init);
#ifdef V8\_COMPRESS\_POINTERS\_IN\_SHARED\_CAGE
diff --git a/src/heap/heap.cc b/src/heap/heap.cc
index 4d7c611dfd..d3027dd3b2 100644
--- a/src/heap/heap.cc
+++ b/src/heap/heap.cc
@@ -1748,6 +1748,8 @@ void Heap::CollectGarbage(AllocationSpace space,
DCHECK(AllowGarbageCollection::IsAllowed());
+ CodePageHeaderModificationScope hsm{};
+
const char\* collector\_reason = nullptr;
const GarbageCollector collector =
SelectGarbageCollector(space, gc\_reason, &collector\_reason);
diff --git a/src/heap/heap.h b/src/heap/heap.h
index 6675b09348..67772961fe 100644
--- a/src/heap/heap.h
+++ b/src/heap/heap.h
@@ -2537,7 +2537,10 @@ class V8\_NODISCARD AlwaysAllocateScopeForTesting {
// scope.
#if V8\_HEAP\_USE\_PTHREAD\_JIT\_WRITE\_PROTECT
using CodePageHeaderModificationScope = RwxMemoryWriteScope;
+#elif defined(V8\_JITLESS)
+using CodePageHeaderModificationScope = RwMemoryWriteScope;
#else
+#error "JIT not supported"
// When write protection of code page headers is not required the scope is
// a no-op.
using CodePageHeaderModificationScope = NopRwxMemoryWriteScope;
@@ -2560,6 +2563,7 @@ class V8\_NODISCARD CodePageMemoryModificationScope {
bool scope\_active\_;
base::Optional<base::MutexGuard> guard\_;
#endif
+ RwMemoryWriteScope header\_scope\_;
// Disallow any GCs inside this scope, as a relocation of the underlying
// object would change the {MemoryChunk} that this scope targets.
diff --git a/src/heap/memory-chunk.cc b/src/heap/memory-chunk.cc
index 0fc34c12a1..479993b219 100644
--- a/src/heap/memory-chunk.cc
+++ b/src/heap/memory-chunk.cc
@@ -10,6 +10,7 @@
#include "src/common/globals.h"
#include "src/heap/basic-memory-chunk.h"
#include "src/heap/code-object-registry.h"
+#include "src/heap/heap.h"
#include "src/heap/marking-state-inl.h"
#include "src/heap/memory-allocator.h"
#include "src/heap/memory-chunk-inl.h"
@@ -223,6 +224,7 @@ void MemoryChunk::ReleaseAllAllocatedMemory() {
}
SlotSet\* MemoryChunk::AllocateSlotSet(RememberedSetType type) {
+ CodePageHeaderModificationScope scope{};
SlotSet\* new\_slot\_set = SlotSet::Allocate(buckets());
SlotSet\* old\_slot\_set = base::AsAtomicPointer::AcquireRelease\_CompareAndSwap(
&slot\_set\_[type], nullptr, new\_slot\_set);
diff --git a/src/heap/paged-spaces.cc b/src/heap/paged-spaces.cc
index 6083c2a420..5b6d7c965e 100644
--- a/src/heap/paged-spaces.cc
+++ b/src/heap/paged-spaces.cc
@@ -311,6 +311,9 @@ void PagedSpaceBase::SetTopAndLimit(Address top, Address limit, Address end) {
DCHECK\_GE(end, limit);
DCHECK(top == limit ||
Page::FromAddress(top) == Page::FromAddress(limit - 1));
+
+ CodePageHeaderModificationScope scope{};
+
BasicMemoryChunk::UpdateHighWaterMark(allocation\_info\_.top());
allocation\_info\_.Reset(top, limit);
@@ -814,6 +817,7 @@ void PagedSpaceBase::UpdateInlineAllocationLimit() {
// OldSpace implementation
bool PagedSpaceBase::RefillLabMain(int size\_in\_bytes, AllocationOrigin origin) {
+ CodePageHeaderModificationScope scope("RefillLabMain");
VMState<GC> state(heap()->isolate());
RCS\_SCOPE(heap()->isolate(),
RuntimeCallCounterId::kGC\_Custom\_SlowAllocateRaw);
```
