
``` 
static bool IsMatch(String input, String pattern, RegexOptions options, TimeSpan matchTimeout) {
    return new Regex(pattern, options, matchTimeout, true).IsMatch(input);
}
```

该函数会在内部构造一个临时的 `Regex` 对象，并且构造函数的最后一个参数 `useCaChe` 的值是 `true`，表示使用缓存。

**疑问5** 和 **疑问6** 的答案在 `Regex` 的构造函数中，先看看 `Regex` 的构造函数。

## `Regex` 构造函数

`Regex` 有很多个构造函数，列举如下：

``` 
public Regex(String pattern)
    : this(pattern, RegexOptions.None, DefaultMatchTimeout, false) {}
public Regex(String pattern, RegexOptions options)
    : this(pattern, options, DefaultMatchTimeout, false) {}
Regex(String pattern, RegexOptions options, TimeSpan matchTimeout)
    : this(pattern, options, matchTimeout, false) {}
```

> **注意：** 以上构造函数的最后一个参数都是 `false`，表示不使用缓存。

这些构造函数最后都会调用下面的**私有**构造函数（代码有所精简调整）：

``` 
private Regex(String pattern, RegexOptions options, TimeSpan matchTimeout, bool useCache) {
    string cultureKey = null;
    if ((options & RegexOptions.CultureInvariant) != 0)
        cultureKey = CultureInfo.InvariantCulture.ToString(); // "English (United States)"
    else
        cultureKey = CultureInfo.CurrentCulture.ToString();

    // 构造缓存用到的 key，包含 options，culture 和 pattern
    String key = ((int) options).ToString(NumberFormatInfo.InvariantInfo) + ":" + cultureKey + ":" + pattern;
    CachedCodeEntry cached = LookupCachedAndUpdate(key);

    this.pattern = pattern;
    this.roptions = options;

    if (cached == null) {
        // 如果没找到缓存就生成类型为 RegexCodes 的 code，包含了字节码等信息
        RegexTree tree = RegexParser.Parse(pattern, roptions);
        code = RegexWriter.Write(tree);
        // 如果指定了 useCache 参数就缓存起来，下次就能在缓存中找到了
        if (useCache)
            cached = CacheCode(key);
    } else {
        // 如果找到了缓存就使用缓存中的信息
        code = cached._code;
        factory = cached._factory;
        runnerref = cached._runnerref;
    }

    // 如果指定了 Compiled 选项，并且 factory 是空（没使用缓存，或者缓存中的 _factory 是空）
    if (UseOptionC() && factory == null) {
        // 根据 code 和 roptions 生成 factory
        factory = Compile(code, roptions);

        // 需要缓存就缓存起来
        if (useCache && cached != null)
            cached.AddCompiled(factory);
    }
}
```

> **注意：** 带 `bool useCache` 标记的构造函数是私有的，也就是说不能直接使用此构造函数实例化 `Regex`。

首先会根据 `option + culture + pattern` 到缓存中查找。如果没找到缓存就生成类型为 `RegexCodes` 的 `code`（包含了字节码等信息），如果找到了缓存就使用缓存中的信息。 如果指定了 `Compiled` 选项（`UseOptionC()` 会返回 `true`），并且 `factory` 是空（没使用缓存或者缓存中的 `_factory` 是空），就会执行 `Compile()` 函数，并把返回值保存到 `factory` 成员中。

至此，可以回答第 `5 6` 两个疑问了。

1. **直接实例化的 `Regex` 对象会使用正则表达式引擎内部的缓存吗？**

   会优先根据 `option + culture + pattern` 到缓存中**查找**，但是否**更新缓存**是由最后一个参数 `useCache` 决定的，与是否指定 `Compiled` 选项无关。

1. **正则表达式引擎内部根据什么缓存的？**

   根据 `option + culture + pattern` 缓存。

**疑问7** 与由 **疑问1** 引申出来的 `JIT` 问题是一个问题。之所以会 `JIT`，是因为有需要 `JIT` 的代码，如果不断有新的动态方法产生出来并执行，那么就需要不断地 `JIT`。由于此问题涉及到的代码量比较大，逻辑比较复杂，需要深入 `.NET` 源码进行查看。为了更好的理解整个过程，我简单梳理了 `IsSpitter()` 函数中涉及到的关键类以及类之间的关系，整理成下图，供参考。

## 流程 & 类关系梳理

![](https://bbs.kanxue.com/upload/attach/202112/873494_MH9HAX3HTQB2N6G.png)

看完上图后，可以继续看剩下的 `JIT` 问题了。因为大多数 `JIT` 都出现在 `MatchCollection.Count` 中，可以由此切入。

## `MatchCollection.Count`

实现代码如下：

``` 
public int Count {
    get {
        if (_done)
            return _matches.Count;
        GetMatch(infinite);
        return _matches.Count;
    }
}
```

`Count` 会调用 `GetMatch()` 函数，而 `GetMatch()` 函数会不断调用 `_regex.Run()` 函数。

`_regex` 是哪来的呢？在构造 `MatchCollection` 实例时传过来的。

`MatchCollection` 是由 `Regex.Matches()` 实例化的，代码如下（去掉了判空逻辑）：

``` 
public MatchCollection Matches(String input, int startat) {
    return new MatchCollection(this, input, 0, input.Length, startat);
}
```

该函数会实例化一个 `MatchCollection` 对象，并把当前 `Regex` 实例作为第一个参数传给 `MatchCollection` 的构造函数。该参数会被保存到 `MatchCollection` 实例的 `_regex` 成员中。

接下来继续查看 `Regex.Run` 函数的实现。

## `Regex.Run()`

具体实现代码如下（代码有精简）：

``` 
internal Match Run(bool quick, int prevlen, String input, int beginning, int length, int startat) {
    Match match;
    // 使用缓存的时候，可能从缓存中拿到一个有效的 runner，其它情况下都是 null。
    RegexRunner runner = (RegexRunner)runnerref.Get();

    // 不使用缓存的时候 runner是 null
    if (runner == null) {
        // 如果 factory 不为空就通过 factory 创建一个 runner。
        // 使用了 Compiled 标志创建的 Regex 实例的 factory 不为空
        if (factory != null)
            runner = factory.CreateInstance();
        else
            runner = new RegexInterpreter(code, UseOptionInvariant() ? CultureInfo.InvariantCulture : CultureInfo.CurrentCulture);
    }

    try {
        // 调用 RegexRunner.Scan 扫描匹配项。
        match = runner.Scan(this, input, beginning, beginning + length, startat, prevlen, quick, internalMatchTimeout);
    } finally {
        runnerref.Release(runner);
    }

    return match;
}
```

逻辑还是非常清晰的，先找到或者创建（通过 `factory.CreateInstance()` 或者直接 `new`）一个类型为 `RegexRunner` 实例 `runner`，然后调用 `runner->Scan()` 进行匹配。

对于使用 `Compiled` 选项创建的 `Regex`，其 `factory` 成员变量会在 `Regex` 构造函数中赋值，对应的语句是 `factory = Compile(code, roptions);` ，类型是 `CompiledRegexRunnerFactory`。

我们先来看看 `CompiledRegexRunnerFactory.CreateInstance()` 的实现。

## `CompiledRegexRunnerFactory.CreateInstance()`

代码如下：

``` 
protected internal override RegexRunner CreateInstance() {
    CompiledRegexRunner runner = new CompiledRegexRunner();

    new ReflectionPermission(PermissionState.Unrestricted).Assert();
    // 设置关键的动态函数，这三个函数是在 RegexLWCGCompiler
    // 类的 FactoryInstanceFromCode() 中生成的。
    runner.SetDelegates(
        (NoParamDelegate) goMethod.CreateDelegate(typeof(NoParamDelegate)),
        (FindFirstCharDelegate) findFirstCharMethod.CreateDelegate(typeof(FindFirstCharDelegate)),
        (NoParamDelegate) initTrackCountMethod.CreateDelegate(typeof(NoParamDelegate))
    );

    return runner;
}
```

该函数返回的是 `CompiledRegexRunner` 类型的 `runner`。在返回之前会先调用 `runner.SetDelegates` 为对应的关键函数（`Go`, `FindFirstChar`, `InitTrackCount`）赋值。参数中的 `goMethod, findFirstCharMethod, initTrackCountMethod` 是在哪里赋值的呢？在 `Regex.Compile()` 函数中赋值的。

`Regex.Compile()` 会直接转调 `RegexCompiler` 的静态函数 `Compile()`，相关代码如下（有调整）:

``` 
internal static RegexRunnerFactory Compile(RegexCode code, RegexOptions options) {
    RegexLWCGCompiler c = new RegexLWCGCompiler();
    return c.FactoryInstanceFromCode(code, options);
}
```

该函数直接调用了 `RegexLWCGCompiler` 类的 `FactoryInstanceFromCode()` 成员函数。相关代码如下（有删减）：

``` 
internal RegexRunnerFactory FactoryInstanceFromCode(RegexCode code, RegexOptions options) {

    // 获取唯一标识符，也就是FindFirstChar后面的数字
    int regexnum = Interlocked.Increment(ref _regexCount);
    string regexnumString = regexnum.ToString(CultureInfo.InvariantCulture);

    // 生成动态函数Go
    DynamicMethod goMethod = DefineDynamicMethod("Go" + regexnumString, null, typeof(CompiledRegexRunner));
    GenerateGo();

    // 生成动态函数FindFirstChar
    DynamicMethod firstCharMethod = DefineDynamicMethod("FindFirstChar" + regexnumString, typeof(bool), typeof(CompiledRegexRunner));
    GenerateFindFirstChar();

    // 生成动态函数InitTrackCount
    DynamicMethod trackCountMethod = DefineDynamicMethod("InitTrackCount" + regexnumString, null, typeof(CompiledRegexRunner));
    GenerateInitTrackCount();

    return new CompiledRegexRunnerFactory(goMethod, firstCharMethod, trackCountMethod);
}
```
