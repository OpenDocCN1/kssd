
执行 `f(true, true, a, a)` 后，`a[0]` 就已经是 unboxed 的 `13.37` 的低 32 位了，所以访问 `a[0]` 就会导致 crash，这就是一个 object faking primitive。

## 0x05 利用

因为目前的 `V8` 都是有 `pointer compression` 的，而在这个模式下指针的低 32 位是不随机的，所以只需要一个 object faking 不需要任何泄露就能实现代码执行。具体思路如下：

1. 分配一个巨大 `Array`，`const faker = new Array(0x10000); faker.fill(13.37);`，此时 `V8` 的 GC 会为它的 Element 专门分配一些 pages，而这个地址是相对比较稳定的。
2. 通过调试获取 `<Map(HOLEY_DOUBLE_ELEMENTS)>` 的 `Array` 和 `ArrayBuffer` 的 `Map` 地址。
3. 在 `faker` 伪造一个 double array，其 element backing store 指向 `faker` 的 element 的 page 前面的 meta data，可以泄露出各种地址。
4. 在 `faker` 伪造一个 `ArrayBuffer`，实现任意读写。然后接下来的利用就见仁见智了，因为不是题目的重点也挺麻烦就不讲了（其实我也懒得写了）。还有就是这道题目 disable 了 `wasm`，有点恶心，但是最简单的方法我觉得就是通过把 `FLAG_expose_wasm` 改写回 `true` 然后再刷新，按照 `wasm` 的常规利用就行了。

原文：[f34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3@1J5x3o6p5&6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6B7k6h3E0&6L8r3I4Q4x3V1k6#2M7r3c8S2N6r3g2Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6W2Q4x3V1j5I4z5g2)9J5c8W2q4i4b7W2)9J5k6p5N6G2L8@1g2^5k6h3y4Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@d13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3@1J5x3o6p5&6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6B7k6h3E0&6L8r3I4Q4x3V1k6#2M7r3c8S2N6r3g2Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6W2Q4x3V1j5I4z5g2)9J5c8W2q4i4b7W2)9J5k6p5N6G2L8@1g2^5k6h3y4Q4x3X3g2Z5N6r3#2D9)

## 0x00 前言

前言简单的吐个槽，跟赛题无关。这次单人参加了在郑州的强网杯线下，是第一次但应该也是最后一次了。总体的来讲无论是从赛题质量还是赛制设计都是不错的。比赛一共有两个部分：传统 Attack Defense 和 Real World Jeopardy 题目。我在第一天几乎一直在看 AD 的题，然后从第二天早上开始就几乎完全投入到了这道 `GooExec` 的 `V8` 题目上面来了，导致最后一道 AD 我连附件下都没下载。而更可惜的是这道题目我在 `15:40` 拿到了一个 object faking primitive。我原以为是 `17:00` 之前都可以上台演示，那么就还有一个多小时写利用（这完全够了），但是 16 点就不能申请演示了，我在 `15:59` 申请了一次结果 `16:04` 就把我叫上去了，而这样的话我就完全没准备好，于是这道题就没有做出来。换句话说他这个本质上 RW 项目是 `16:00` 就结束了，并不是 `17:00` 结束。而我则等于第二天所花的精力完全没有拿到分，还不如做第四道 AD 题。

## 0x01 摘要

这道题修改了 `load-elimination.cc` 中 `Reduce` 两个 `Node` 的代码，把 `KillMaps` 函数删除了。这导致一些 `Node` 的 `Map` 会被错误估计，从而导致 `CheckMaps` 会被错误地删除掉。通过这个我们可以把一个 unboxed double 写到一个 `FixedArray` 里面去，于是再访问那个 `array` 就可以 fake 任意 object 了。由于开启 `pointer compression` 的 `V8` 并没有把存在 GC 堆中的低 32 位地址随机化，我们可以通过分配大 `Array` 来 fake object，从而获得任意读写。

## 0x02 前置知识

这道题目的 patch 点涉及到几个 Load Elimination Phase 的类，所以在分析 patch 前我会先分析这几个相关的类。

### AbstractState

此类代表在某个 node 的 effect 之后的状态，在这里我们只关心他的 `AbstractMaps const* maps_` 成员。其中被删除的 `KillMaps` 函数代码如下：

```cpp
LoadElimination::AbstractState const* LoadElimination::AbstractState::KillMaps(
  const AliasStateInfo& alias_info, Zone* zone) const {
  if (this->maps_) {
    AbstractMaps const* that_maps = this->maps_->Kill(alias_info, zone);
    // 本质上就是调用 maps_ 的 Kill 函数
    if (this->maps_ != that_maps) {
      AbstractState* that = zone->New<AbstractState>(*this);
      that->maps_ = that_maps;
      return that;
      // 如果不一样才返回一个新的
    }
  }
  return this;
}
LoadElimination::AbstractState const* LoadElimination::AbstractState::KillMaps(
  Node* object, Zone* zone) const {
  AliasStateInfo alias_info(this, object);
  return KillMaps(alias_info, zone);
}
```

### AbstractMaps

此类代表在某个 effect 状态时所有 node 所可能有的 maps。很明显其 `ZoneMap<Node*, ZoneHandleSet<Map>> info_for_node_` 成员变量就是用来存储这个信息的：把 `Node` 映射到一个 `Map` 的集合。这边注意，假如某个 Node 没有对应的映射，那么代表这个 Node 的 Map 信息 JIT 是不知道的；假如某个 Node 存在对应映射，那么这个 Node 在当前状态的 Map 必然是这个 set 中的其中一个。换句话说，假如某个可能的 Map* 没有被包含在这个 set 里面，那么就有可能造成类型混淆。

接着再来看看 `AbstractMaps::Kill` 的实现，即 `KillMaps` 所调用的那个函数：

```cpp
LoadElimination::AbstractMaps const* LoadElimination::AbstractMaps::Kill(
  const AliasStateInfo& alias_info, Zone* zone) const {
  for (auto pair : this->info_for_node_) {
    if (alias_info.MayAlias(pair.first)) {
      // if one of nodes may alias
      AbstractMaps* that = zone->New<AbstractMaps>(zone);
      for (auto pair : this->info_for_node_) {
        if (!alias_info.MayAlias(pair.first)) that->info_for_node_.insert(pair);
      }
      // keep all except the ones that may alias
      return that;
    }
  }
  return this;
}
```

### AliasStateInfo and MayAlias

简单地说，`MayAlias` 函数返回是否两个 Node 可能 refer 到同一个对象。而 `AliasStateInfo` 则代表用来比较的其中一个 Node。

```cpp
class LoadElimination::AliasStateInfo {
public:
  AliasStateInfo(const AbstractState* state, Node* object, Handle<Map> map)
    : state_(state), object_(object), map_(map) {}
  AliasStateInfo(const AbstractState* state, Node* object)
    : state_(state), object_(object) {}

  bool MayAlias(Node* other) const;

private:
  const AbstractState* state_;
  Node* object_; // 用来比较的 Node
  MaybeHandle<Map> map_;
};
```

`MayAlias` 具体的实现我就不放这里了，稍微有点长，只要知道它在两个 Node 一定不是同一个对象的时候返回 `false` 就好了。

### TransitionElementsKind

在分析漏洞之前，可能还是得先看看相关函数所 reduce 的 `TransitionElementsKind` 是个什么东西。这是当 Element 类型改变的时候，用来改变 Array 的 element 存储方式的操作，所以同时也会改变其 map。比如 `arr = [1.1, 2.2]` 的 element 会是 `FixedDoubleArray`，而经过 `arr[0] = {}` 操作后，element 会变成一个 `FixedArray`，这个时候就需要 `TransitionElementsKind` 这个操作来做这个转换。

## 0x03 漏洞

接下来可以看看他的这个 patch 了，虽然他 patch 了两个函数，但实际上似乎我的利用只用到了其中一个函数，不知道这是不是非预期解。所以我这篇 Writeup 只讲解那个 patch，虽然另一个也是一个 `KillMaps` 的删除，可能也差不多。但是那个更难触发到，需要通过 `Array.prototype.map` 来触发。

```cpp
// LoadElimination::ReduceTransitionElementsKind
if (object_maps.contains(ZoneHandleSet<Map>(source_map))) {
  object_maps.remove(source_map, zone());
  object_maps.insert(target_map, zone());
  // AliasStateInfo alias_info(state, object, source_map);
  // state = state->KillMaps(alias_info, zone());
  state = state->SetMaps(object, object_maps, zone());
}
```

很明显，`KillMaps` 的删除导致某些本应该没有 map 信息的一些 node 仍然保留了信息，那么假如所保留的 Map 信息是个错误估计，即某些可能存在的 Map 没有被存储在其中，那么就可能造成类型混淆。比如说 `ReduceCheckMaps` 这里，就有可能导致错误地删除 `CheckMaps`：

```cpp
Reduction LoadElimination::ReduceCheckMaps(Node* node) {
  ZoneHandleSet<Map> const& maps = CheckMapsParametersOf(node->op()).maps();
  Node* const object = NodeProperties::GetValueInput(node, 0);
  Node* const effect = NodeProperties::GetEffectInput(node);
  AbstractState const* state = node_states_.Get(effect);
  if (state == nullptr) return NoChange();
  ZoneHandleSet<Map> object_maps;
  // 假如 object_maps 的 Map 信息并不完整，可能导致 maps.contains 错误地返回 true
  if (state->LookupMaps(object, &object_maps)) {
    if (maps.contains(object_maps)) return Replace(effect);
    // TODO(turbofan): Compute the intersection.
  }
  state = state->SetMaps(object, maps, zone());
  return UpdateState(node, state);
}
```

那么如何让 `state` 去保留一个错误估计呢？在 `TransitionElementsKind` 后 `object` 的 Map 插入了一个新的 `target_map`，那么假如有另一个 Node 和 `object` 指向同一个对象，它的 state 就即不会被 `KillMaps` 删除也不可能被插入这个新的 `target_map`，于是就可以导致一个错误估计。

## 0x04 PoC 构造

接着就需要尝试构造 PoC 了。首先需要解决的问题是如何构造两个 Node 指向同一个 object 的情况，我尝试了使用一些 JIT 的方法让 sea of nodes 去生成一些 aliasing nodes，但是好像特别难构造。最后我发现了一个最简单的方法，就是给两个不同的函数参数传入同一个对象，那么当 JIT 访问不同的 Parameter 时，就可以构成不同 Node 指向同一个对象的条件。

写成 JavaScript 大概是这样：声明 `function test(arr, arr2) {...}`，调用 `arr = Array(0); test(arr, arr);`

关于 PoC 的 JIT 函数，大概思路如下：

1. 首先让 `state` 认为 `arr` 和 `arr2` 都是 `FixedDoubleArray` map。
2. 在 `arr` 调用 `TransitionElementsKind`，此时 `state` 中，`arr` 的 Map 会变成 `FixedArray` Map，但是 `arr2` 的 Map 却仍然是 `DoubleFixedArray` Map，这是错误的。因为此时 `arr2` 的 Map 跟着 `arr` 一起被 transition 了（因为指向同一个 object）
3. 执行 `arr2[0] = 1.1`，此时因为 `state` 仍然认为 `arr2` 是 `FixedDoubleArray` 的 Map，所以会把的 unboxed array 的 store 操作前面的 `CheckMaps` 给删除掉。但是此时 `arr2` 的 Element 已经是 `FixedArray` 了，那么等于把一个 unboxed double 写入了一个存放指针的 slot，即再访问 `arr[0]` 就可以产生类型混淆了。但实际上这一步遇到了一些问题卡了我很久，待会再详细说明。

接着就按照上面的思路写这么一个函数：
