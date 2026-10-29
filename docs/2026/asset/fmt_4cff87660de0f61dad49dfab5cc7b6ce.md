
```
extern template class PassManager<Module>;
/// Convenience typedef for a pass manager over modules.
using ModulePassManager = PassManager<Module>;
```

而 `PassManager` 也是一个模板，由 `PassInfoMixin` 提供一个 `name`。

`IRUnitT` 就是传入的 `llvm::Module`，然后推断出 `AnalysisManagerT` 的类型为 `AnalysisManager<IRUnitT>`，也就是 `AnalysisManager<Module>`，恰好是 `ModuleAnalysisManager`。

`AnalysisManager` 也是一个模板类，就不展开了。

```
template <typename IRUnitT,
          typename AnalysisManagerT = AnalysisManager<IRUnitT>,
          typename... ExtraArgTs>
class PassManager : public PassInfoMixin<
    PassManager<IRUnitT, AnalysisManagerT, ExtraArgTs...>> {
  //xxxxxxxx
}

/////////////////////

template <typename DerivedT> struct PassInfoMixin {
  /// Gets the name of the pass we are mixed into.
  static StringRef name() {
    static_assert(std::is_base_of<PassInfoMixin, DerivedT>::value,
                  "Must pass the derived type as the template argument!");
    StringRef Name = getTypeName<DerivedT>();
    if (Name.startswith("llvm::"))
      Name = Name.drop_front(strlen("llvm::"));
    return Name;
  }
};

/////////////////////

using ModuleAnalysisManager = AnalysisManager<Module>;
```

继续看 `addPass` 的实现，有两个重载，第一个是传入 pass 时，将其加入到列表中；第二个是传入另一个 `PassManager` 时，将其内部的 `passes` 全部加入到列表中。

```
template <typename IRUnitT,
          typename AnalysisManagerT = AnalysisManager<IRUnitT>,
          typename... ExtraArgTs>
class PassManager : public PassInfoMixin<
    PassManager<IRUnitT, AnalysisManagerT, ExtraArgTs...>> {

  template <typename PassT>
  std::enable_if_t<!std::is_same<PassT, PassManager>::value>
  addPass(PassT &&Pass) {
    using PassModelT =
        detail::PassModel<IRUnitT, PassT, PreservedAnalyses, AnalysisManagerT,
                          ExtraArgTs...>;

    Passes.emplace_back(new PassModelT(std::forward<PassT>(Pass)));
  }

  template <typename PassT>
  std::enable_if_t<std::is_same<PassT, PassManager>::value>
  addPass(PassT &&Pass) {
    for (auto &P : Pass.Passes)
      Passes.emplace_back(std::move(P));
  }
}
```

重点关注第一个，`detail::PassModel` 位于 `llvm/include/llvm/IR/PassManagerInternal.h`，是一个模板类。`IRUnitT` 是传入的 `llvm::Module`，`PassT` 是传入的用户自定义的结构体，`PreservedAnalysesT` 是传入的 `PreservedAnalyses`，`AnalysisManagerT` 是传入的 `ModuleAnalysisManager`。

它的构造方法拿到用户自定义的结构体，`run` 方法调用传入的 pass 的 `run` 方法，需要满足 `IRUnitT` 和 `AnalysisManagerT` 的类型约束，这和上文提到的

> `ModulePass` 需要这里是 `llvm::Module` 和 `ModuleAnalysisManager`，`FunctionPass` 需要这里是 `Function` 和 `FunctionAnalysisManager`。

相互印证。它的 `name` 方法调用传入的结构体的 `static` `name` 方法。它的 `passIsRequiredImpl` 会根据传入的结构体的 `isRequired` 是否声明而定，未声明时默认为 `false`，已声明则调用传入的结构体的 `static` `isRequired`。

```
template <typename IRUnitT, typename PassT, typename PreservedAnalysesT,
          typename AnalysisManagerT, typename... ExtraArgTs>
struct PassModel : PassConcept<IRUnitT, AnalysisManagerT, ExtraArgTs...> {
  explicit PassModel(PassT Pass) : Pass(std::move(Pass)) {}
  // We have to explicitly define all the special member functions because MSVC
  // refuses to generate them.
  PassModel(const PassModel &Arg) : Pass(Arg.Pass) {}
  PassModel(PassModel &&Arg) : Pass(std::move(Arg.Pass)) {}

  //xxxxxxxxxxx
  PreservedAnalysesT run(IRUnitT &IR, AnalysisManagerT &AM,
                         ExtraArgTs... ExtraArgs) override {
    return Pass.run(IR, AM, ExtraArgs...);
  }

  StringRef name() const override { return PassT::name(); }

  template <typename T>
  using has_required_t = decltype(std::declval<T &>().isRequired());

  template <typename T>
  static std::enable_if_t<is_detected<has_required_t, T>::value, bool>
  passIsRequiredImpl() {
    return T::isRequired();
  }
  template <typename T>
  static std::enable_if_t<!is_detected<has_required_t, T>::value, bool>
  passIsRequiredImpl() {
    return false;
  }

  bool isRequired() const override { return passIsRequiredImpl<PassT>(); }

  PassT Pass;
};
```
