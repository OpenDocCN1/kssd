
# 玩玩 IDA Graph View

> 原文：[`www.pediy.com/kssd/pediy10/60798.html`](https://www.pediy.com/kssd/pediy10/60798.html)

IDA 自 5.0 版加入了 `graph view` 功能，按空格键可在文本与图形模式间切换，看起来很直观（我觉得比 `wingraph` 好用），不过只能处理函数。如果代码不能归属到某个函数，会报错：

![](img/5abb17657f4ad6ac6e58c96c43457ccc.jpg)

我们自己写插件来玩玩 `Graph View`。要显示的代码是 `Themida 1930 Demo` 的 VM 引擎。升级到 19xx 后，`Themida` 的变形代码有所增强，一直想再玩玩，不过断断续续搞了一段时间，效果不好。本文不涉及变形代码处理（能搞出可接受的结果再说）。最初的打算是以图形方式显示代码，加入一定的交互能力，对程序无法清理的代码手工处理。实际证明不可行，一是 `Basic Block` 量太大，手工操作不现实。另外 `IDA` 在处理复杂的图时似乎有点问题。最终可能还是只有先压缩代码，`graph` 只能看看。

原来也考虑过 `uDraw` 之类的，感觉那个更麻烦。找了本讲图算法的书，翻了几页就不行了，头晕恶心，到底是土八路啊。

`Dump VM` 的 6 个区段，贴到一个小程序后面，以免 `IDB` 过于庞大。`IDA` 对 `Themida VM` 的初始分析似乎有点问题，始终停不下来。将其停下来，自己处理一下 `unexploered` 部分。修改 `IDA` 设置，把 `graph` 允许的最大节点数设大点。

![](img/8f0d28532f676bd695f5337e8ec1124f.jpg)

`IDA v5.2 SDK` 带了个例子（`ugraph`），直接在例子上写。

## 收集 Basic Block

`graph` 内的 `node` 对应代码基本块（`Basic Block`）。`BB` 指可作为整体对待的代码片段，如果 `BB` 的第 1 条指令被执行，则该块内的所有指令会被顺序执行。`IDA` 自己使用了 `Basic Block` 的概念，若选择如下设置，会在 `BB` 结束处分界。

![](img/7d3fcf4316bc767c21adf2bac8648cd6.jpg)

![](img/ddfb56cc9f667f7dbd28704e29e72e34.jpg)

显然，`IDA` 是把 `call` 指令也作为 `BB` 结束的，因为 `call` 可被用作变形的 `jmp` 而不再返回。`SDK` 并没有直接提供 `Basic Block` 信息，相关的 `API` 只有一个。

```
idaman bool ida_export is_basic_block_end(bool call_insn_stops_block); // in:cmd
```

参数指示 `call` 是否视为 `BB` 结束。另一个参数为全局的 `cmd` 结构，即应先用 `ua_ana0` 分析指令以填写 `cmd`。

对 `BB` 的收集可有 2 种方式。一是对指定的地址范围线性搜索，这样数据量较大。这里用递归方式，从指定地址（某个 `handler` 的入口）开始分析。

定义 `CBasicBlock` 类：

```
class CBasicBlock

{
public:
  int n;        // graph 内的索引(add_node 的返回)
  ea_t first;  // bb 第 1 条指令的机器码地址,这个成员用于代表该 bb，在压缩变形代码时不会改变
              // (实际的指令可能被清理)
vector<PCode>   codes;    // basic-block 包含的指令(反汇编为 pcode),pcode 的 eip 成员
// 为原机器码地址
vector<DWORD>   cref_from;  // 参考该 bb 的 bb。收集 bb 时为其出口指令地址（此时该 bb
// 可能还未创建）,完成收集后替换为其 first 成员
  vector<DWORD>   cref_to;    // 被参考的 bb 入口地址(即被参考 bb 的 first)

public:
  CBasicBlock();         //没有用 oop,用类只为代码好管理
  ~CBasicBlock();
};
```

使用的数据结构：

```
std::set<ea_t>    path_mark;    // 已分析 bb 的 first
std::set<ea_t>    future_label;    // 未分析 bb 的 first
std::vector<ea_t> nodes;      // vector 的下标=add_node 的返回,用于从给定的 graph 内
// index 搜索对应 bb 数据
std::map<ea_t,ea_t> exit2entry;  // 从 bb 的出口查找入口地址
std::map<ea_t,CBasicBlock> bbs;  // basic blocks
```

直接列代码，本来想用文字描述，发现那更麻烦。抄袭 `MetaPHOR` 的做法。

```
#define flow_ordinary   0x15   // ordinary flow 即“自然落下”的那个地址

static void gather_basic_block(ea_t entry)
{
  // 从 entry 开始分析 basic block,结果在 bbs
  ea_t  start = entry;
  ea_t  curr, next;
  ea_t  next_entry; //下个分析地址
  PCode pcode;
  CBasicBlock bb;
  if (!isCode(get_flags_novalue(start)))
  {
    msg("invalid entry\n"); //entry != 代码
    return;
  }
  show_wait_box("Searching basic blocks...\n\n\nJust a Moment");
  bbs.clear();
  exit2entry.clear();
  path_mark.clear();
  future_label.clear();
  bb_num = 0;

__Analysis:
  if (wasBreak())
goto __Exit; //用户取消
  if (path_mark.count(start)) //已分析
    goto __FutureLabel;
  else
    path_mark.insert(start); //置已分析标记
  bb.codes.clear(); //初始化 bb 成员
  bb.cref_from.clear();
  bb.cref_to.clear();
  //如果存在对应的 future label 则删除
  if (future_label.count(start)) future_label.erase(start);
  curr = start;
  while (true) //搜索当前 bb 边界
  {
    if (!ua_ana0(curr)) goto __Exit; //output->cmd
    memset(&pcode, 0, sizeof(pcode));
    pcode.eip = curr; //保存机器码地址
    bb.codes.push_back(pcode);
    next = get_item_end(curr);
    if (is_basic_block_end(true)) break; //bb 结束?
    curr = next;
  }
  bb.first = start;      // bb.first=入口代码地址
  exit2entry[curr] = start;  // 保存 exit->entry 映射
  //分析交叉参考
  xrefblk_t xb;
  // IDA SDK:
  // The following functions return: 1-ok, 0-no (more) xrefs
  // They first return code references, then data references. <<<< 首先返回代码参考
  // If you need only code references, you need to check 'iscode' after each call.
  // If you need only data references, use XREF_DATA bit.
  // 参考当前 bb 的 bb
  // cref_from=参考当前 bb 的 bb 的出口地址,这里无法得知对应 bb 的 first,bb 可能还未生成
  if (xb.first_to(start, XREF_ALL) && xb.iscode)
  {
    bb.cref_from.push_back(xb.from);
    while(xb.next_to() && xb.iscode) bb.cref_from.push_back(xb.from);
  }
  //被当前 bb 参考的 bb
  next_entry = 0; //下个分析地址预置为 0
  //////////////////////////////////////////////////////////////////////////
  //
  //  bb 的 cref_to 最多包含 2 个分枝:
  //  1. size=0,无分枝(如 jmp esi, ret)
  //  2. size=1,1 个分枝(如 jmp, ordinary flow),使用 cref_to[0]
  //  3. size=2,2 个分枝(jcc),约定 ordinary flow=cref_to[0], dst=cref_to[1]
  //
  //////////////////////////////////////////////////////////////////////////
  if ( ! (xb.first_from(bb.codes.back().eip, XREF_ALL) && xb.iscode) )
    goto __SaveBasicBlock; //无 cref,next_entry=0
  else
  {
    //1 条指令最多存在 2 个代码分枝,IDA 先返回代码参考,只需要调用 1 次 next_from
    ea_t  first_to    = xb.to;
    uchar first_type = xb.type;
    if (xb.next_from() && xb.iscode) //存在第 2 个分枝?
    {
      ea_t second_to = xb.to;
      //不依赖 first_to|next_to 返回地址的顺序(按大小排列?)来确定哪个是
//ordinary flow，自己处理一下
      if (first_type == flow_ordinary)
      {
        next_entry = first_to; //继续分析 ordinary flow
        bb.cref_to.push_back(first_to);  //cref_to[0]=ordinary flow
        bb.cref_to.push_back(second_to); //cref_to[1]=dst
        //在 future_label 插入 jcc dst
        if ( !path_mark.count(second_to) &&   // 未分析
            !future_label.count(second_to))  // 不存在 future_label
        {
          future_label.insert(second_to);
        }
      }
      else
      {
        next_entry = second_to;
        bb.cref_to.push_back(second_to); //cref_to[0]=ordinary flow
        bb.cref_to.push_back(first_to);  //cref_to[1]=dst
        //在 future_label 插入 jcc dst
        if ( !path_mark.count(first_to) &&
          !future_label.count(first_to))           )
        {
          future_label.insert(first_to);
        }
      }
    }
    else //只有 1 个分枝
    {
      bb.cref_to.push_back(first_to);
      next_entry = first_to;
    }
  }

__SaveBasicBlock:
  bbs[start] = bb;
  start = next_entry;
  if ( start==0 ) //已到了叶节点
    goto __FutureLabel;
  if ( path_mark.count(start) )
    goto __FutureLabel; //已处理
  goto __Analysis; //分析下 1 个 bb

__FutureLabel:
  if (future_label.empty()) //没有 future label 结束分析
    goto __Exit;
  else
  {
//取第 1 项
    start = *future_label.begin();
    goto __Analysis;
  }

__Exit:
  hide_wait_box();
  msg("basic blocks=%d\n", bb_num);
}
```

此时可进行一些预处理，如删除 `bb` 的外来参考（参考者不在收集的 `basic block` 内），合并由 `jmp` 链接的块等。

完成 `Basic Block` 收集后，用 `exit2entry` 将所有 `bb` 的 `cref_from`（参考者）替换为对应 `bb` 的 `first`。压缩变形代码时可能会删除原来的出口代码。

下面将所有 `bb` 包含的代码反汇编为 `pcode`（麻烦，但对压缩变形代码有用）。逐个压缩 `bb`，删除 `fake jcc`（如果可能则合并 `basic block`，压缩合并结果）。后 2 步要反复循环。我还有无数没解决的问题及无数的 `bug`，这里就不多说了。

## 编写 IDA 回调函数

这里只列出改写的代码。完整代码请参考 `SDK` 的例子。

### grcode_user_refresh: 创建 node 和 edge

```
case grcode_user_refresh:    // refresh user-defined graph nodes and edges
                // in:  mutable_graph_t *g
                // out: success
{
    //生成 node 和 edge
    mutable_graph_t *g = va_arg(va, mutable_graph_t *);
    if(!g->empty()) //清空 graph
      g->clear();
    nodes.clear();
```



`// 为 bbs 内所有 item 加入 node`

```cpp
    map<ea_t,CBasicBlock>::iterator it=bbs.begin();

    for (; it != bbs.end(); ++it)
    {
      int n = g->add_node(NULL);

      it->second.n = n;  // 保存当前 bb 的 index
      nodes.push_back(it->second.first);
      if (it->second.first == entry) //保存 handler entry 对应 node
      {                //后面要将这个 node 用红色显示
        root = n;
        msg("root=%d\n", root);
      }
    }
    // 加 edge
    for (it=bbs.begin(); it != bbs.end(); ++it)
    {
      if (!it->second.cref_to.empty()) //存在 cref_to
      {
        vector<ulong>::size_type size = it->second.cref_to.size();
        if (size==1) //jmp 或 ordinay flow
        {

          //edge 用缺省色
          map<ea_t,CBasicBlock>::iterator cref =
bbs.find(it->second.cref_to[0]);
          if(cref != bbs.end())
          {
            g->add_edge(it->second.n, cref->second.n, NULL);
            //msg("add_edge: %d->%d\n", it->second.n, cref->second.n);
          }
        }
        else //size=2(jcc),只可能有 2 个分枝
        {
          for (ulong i=0; i<size; ++i)
          {
            map<ea_t,CBasicBlock>::iterator cref =
bbs.find(it->second.cref_to[i]);
            if (cref != bbs.end()) //被参考 bb 存在则加边
            {
              edge_info_t edge_info;
              if (i==0)
                edge_info.color = RGB(255,0,0);  //ordinary flow=red(约
//定为 cref_to[0])
              else
                edge_info.color = RGB(30,150,30); //jcc dst=green
              g->add_edge(it->second.n, cref->second.n, &edge_info);

              //msg("add_edge: %d->%d\n", it->second.n, cref->second.n);
            }
          }
        }
      }
    }
    msg("add nodes and edges ok\n");
    result = 1;
}
break;
```

`grcode_user_text`：为每个 node 生成显示的文本

`case grcode_user_text:  // retrieve text for user-defined graph node`
`// in:  mutable_graph_t *g`
`//      int node`
`//      const char **result`
`//      bgcolor_t *bg_color (maybe NULL)`
`// out: must return 0, result must be filled`
`// NB: do not use anything calling GDI!`

```cpp
{
  //IDA 为每个 node 调用 callback
  mutable_graph_t *g = va_arg(va, mutable_graph_t *);
  int n               = va_arg(va, int);
  const char **text  = va_arg(va, const char **);
  bgcolor_t *bgcolor = va_arg(va, bgcolor_t *);
  char curr[256];
  memset(buf, 0, sizeof(buf));
  //n 为当前 node 在图内的 index
  map<ea_t,CBasicBlock>::iterator it=bbs.find(nodes[n]);
  if(it != bbs.end())
  {
    for (int i=0; i<it->second.codes.size(); ++i)
    {
      memset(curr,0, sizeof(curr));
      //将 pcode 转换为助记符,这里的类名不大合适呵呵
CAsm::DisasmMnem(it->second.codes[i], curr);
      if ((strlen(buf)+strlen(curr)+10) > MAXSTR)
      {
        //text 太长,在后面显示”…”
        qstrncat(buf, COLSTR("...",SCOLOR_INSN), MAXSTR-1);
        break;
      }
      else
      {
        qstrncat(buf, curr, MAXSTR-1);
        qstrncat(buf, "\n", MAXSTR-1);
      }
    }
    *text=buf;
  }
  else
  {
    //一般情况下不应该,但 IDA 插入了 dummy node…
    msg("bb=%d addr=%08X not existed\n", n, nodes[n]);
    qstrncpy(buf, "error", sizeof(buf));
    return result; //直接返回 0
  }
  //对 node 设置背景色
  bool vm_exit = (it->second.codes[back].opcode==Op_JMP_Reg ||
it->second.codes[back].opcode==Op_Ret)?
true:false;
  if(bgcolor != NULL )
  {
    if(n == root)
      *bgcolor = RGB(255,120,140); //handler entry=红色
    else if(vm_exit)
      *bgcolor = RGB(150,150,150); //handler exit=灰色
    else
      *bgcolor = DEFCOLOR;
  }
  // IDA SDK 原例子的注释有错,这里必须返回 1
  result = 1;
  qnotused(g);
}
break;
```

`grcode_user_title`：为 node 绘制 title

`case grcode_user_title:  // render node title of a user-defined graph`
`// in:  mutable_graph_t *g`
`//      int node`
`//      rect_t *title_rect`
`//      int title_bg_color`
`//      HDC dc`
`// out: 0-did not render, ida will fill it with title_bg_color`
`//      1-rendered node title`
`// ida will draw the node title itself`

```cpp
{
  mutable_graph_t *g = va_arg(va, mutable_graph_t *);
  int n               = va_arg(va, int);
  rect_t *rect        = va_arg(va, rect_t *);
  bgcolor_t bgcolor   = va_arg(va, bgcolor_t);
  HDC dc              = va_arg(va, HDC);
  //在 title bar 显示 bb 起始指令地址
  map<ea_t,CBasicBlock>::iterator it=bbs.find(nodes[n]);
  if (it != bbs.end())
  {
    char title[32];
    qsnprintf(title, sizeof(title)-1, "loc_%X", it->second.first);
    SetTextColor(dc, RGB(255,255,255));
    SetBkMode(dc, TRANSPARENT);
    FillRect(dc, (LPRECT)rect, (HBRUSH)GetStockObject(GRAY_BRUSH));
    LOGFONT logfont;
    HFONT font = (HFONT)GetCurrentObject(dc, OBJ_FONT);
    GetObject(font, sizeof(LOGFONT), &logfont);
logfont.lfHeight = rect->height();
    logfont.lfWidth  = 0;
    logfont.lfWeight = 500;
    qstrncpy(logfont.lfFaceName, "Courier New", sizeof(logfont.lfFaceName)-1);
    HFONT bigfont = CreateFontIndirect(&logfont);
    HFONT oldfont = (HFONT)SelectObject(dc, bigfont);
    DrawText(dc, title, strlen(title), (LPRECT)rect,
DT_SINGLELINE |DT_LEFT|DT_VCENTER);
    SelectObject(dc, oldfont);
    DeleteObject(bigfont);
  }
}
result = 1;
break;
```

`void idaapi run(int /*arg*/)`

```cpp
void idaapi run(int /*arg*/)
{
  HWND hwnd = NULL;
  entry = 0x01DC4C7B; //硬编码的 handler 地址，实际应用 askaddr
// 反汇编前的预处理
  gather_basic_block(entry);      // 收集 bb
  CShrink::Preprocess();         // 预处理
  CShrink::DeletePermutation();  // 删除 permutation-jmp
  CShrink::UpdateCrefFrom();     // 更新所有 cref_from 为对应 bb 的 first
  exit2entry.clear();            // 不再需要
  CShrink::Shrink();            // Fuck oreans!
  TForm *form = create_tform("Deobfuscator", &hwnd);
  if(hwnd != NULL)
  {
    // get a unique graph id
    netnode id;
    id.create();
//要显示 title,height 不能为 0
    graph_viewer_t *gv = create_graph_viewer(form,  id, callback, NULL, 14);
    gview = gv; //保存到全局变量
    open_tform(form, FORM_MDI|FORM_TAB|FORM_MENU);
    if ( gv != NULL )
    {
      viewer_fit_window(gv);
      viewer_add_menu_item(gv, "Layout", menu_layout, gv, NULL, 0);
      //增加一个搜索功能
      viewer_add_menu_item(gv, "Search...", menu_search, gv, NULL, 0);
    }
  }
  else
  {
    close_tform(form, 0);
  }
}

bool idaapi menu_search(void *ud)
{
  //在 graph 中搜索指定 entry 对应的 node
  ea_t addr=BADADDR;
  if (!askaddr(&addr, "basic block entry")) return false;
  mutable_graph_t *g = get_viewer_graph(gview);
  bool found=false;
for (int n=0; n<nodes.size(); ++n)
  {
    if (nodes[n] == addr)
    {
      found=true;
      break;
    }
  }
  if(!found)
  {
    msg("node not found\n");
    return false;
  }
  viewer_center_on(gview, n); //居中显示 node
  return true;
}
```

基本就是这些，看看效果。

`![](img/bcbb797cfdacecb281199301eff8be43.jpg)`

`![](img/ec9e471e400bdf7d6d4d347adab30ca0.jpg)`

还有很多变形代码没有处理。第 2 块出口 fake jcc 的 dst 没有出现，无法合并（限制了 basic block 数量）。

IDA 的问题`____________________________________________`



# 问题描述

不敢随便质疑这样的软件，必须打个问号。在 `graph` 比较复杂的时候，`IDA` 似乎有问题。当前 `handler` 若不压缩有 68000 个 `bb`（因为 `fake jcc` 即加壳时的 `MultiBranch Technology` 选项）。限制收集的 `bb` 数量为 8000（当前压缩结果为 500 多 `bb`，水份还很大），可以正常显示。

当设置更大的数量时，会导致 `IDA` 出错退出。

![](img/d4eea791558bc59325f0d581defac1f1.jpg)

# 代码分析

似乎是这个函数有问题：

```
size_t idaapi sort_layer_nodes(const row_info_t &r1,
                               const intmap_t &lpi1,
                               row_info_t &r2,
                               intmap_t &lpi2,
                               bool ispred) const;
```

另外，当 `nodes` 数量较大时会出另外的错，不能正常画图。加了些调试信息后发现，`IDA` 在计算 `layout` 时增加了 `node`，并会用这些 `node` 的 `n` 来调用 `grcode_user_text`。当然收集的 `bb` 内没有对应项，在代码内返回的 `text` 为 `error`。

# 参考资料

在网上搜到一篇文章《`Graph Layout for Applications in Compiler Construction`》，增加 `node` 是正常的。我不翻译了，直接贴在这里。

![](img/629a93dc90695919b4b842f020bb8e1a.jpg)

![](img/f36e01d978c8da121dc6195087703e15.jpg)

![](img/3fe8c8851b294e622fcbf6fdb2829c00.jpg)

# 备注

可惜 `hex-rays` 的论坛似乎要正版用户才能注册，麻烦哪位能登录上去的帮我问问。我被这玩艺郁闷得不行，只好考虑先尽可能压缩，减少 `bb` 数量。再抱怨一下 `IDA SDK`，严重缺乏文档，经常得到头文件里找东西。
