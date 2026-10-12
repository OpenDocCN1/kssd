
# IDA SDK 合并 jmp 乱序插件代码示例阅读

> 原文：[`www.pediy.com/kssd/pediy12/131016.html`](https://www.pediy.com/kssd/pediy12/131016.html)

看了下 [`bbs.pediy.com/showthread.php?t=60798`](http://bbs.pediy.com/showthread.php?t=60798)这篇文章和[`www.openrce.org/downloads/det...x86grph\_for\_x8`](http://www.openrce.org/downloads/details/40/x86grph_for_x8)这个插件。

功能主要是:

`A small utility that will generate a flowgraph from x86 code -- similar to IDA's built-in funtionality -- which is capable of graphing non-contigous functions (as created by Microsoft's internal optimization tools).`

这样可以指定地址和结束地址，来显示这段代码的流程图，但有的变形代码里采用直接跳转来打乱代码显示次序，显得很乱和难以分析，所以我们要合并他们。

很巧 IDA 里有份合并乱序 jmp 的文章，这里只合并的绝对跳转，由于 IDA 资料好少，搞了半天，终于明白了合并 jmp 的步骤，分享出来，希望能抛砖引玉。

代码:



### C++ 代码示例

```cpp
#include <ida.hpp>
#include <idp.hpp>
#include <graph.hpp>
#include <loader.hpp>
#include <kernwin.hpp>

typedef std::map<int, areavec_t> cmbnodes_t;
static cmbnodes_t cmbnodes; // for each combined node: ranges that it represents

//--------------------------------------------------------------------------
static void removed_block(intseq_t &seq, int m)
{
  for ( int i=0; i < seq.size(); i++ )
    if ( seq[i] >= m )
      seq[i]--;
}

//--------------------------------------------------------------------------
static void combine_blocks(qflow_chart_t &fc, int n, int m)
{
  // copy successors of m to successors of n
  qbasic_block_t &bn = fc.blocks[n];
  qbasic_block_t &bm = fc.blocks[m];
  bn.succ = bm.succ;                //因为 bn.succ 只有一个成员，所以不用考虑其他成员替换会乱的情况
  // remember that n includes m
  areavec_t &vn = cmbnodes[n];
  if ( vn.empty() )
    vn.push_back(bn);
  cmbnodes_t::iterator pm = cmbnodes.find(m);  // 把 m 处的 bb 拷贝到 n 处的 areavec_t 里， 这样所有已经合并的 bb 都在一起了，这样下面生成 txt 时候，可以把所有已经合并的 bb 的 txt 一起输出 
  if ( pm == cmbnodes.end() )
  {
    vn.push_back(bm);  //n 处的所有合并 BB 的 vector
  }
  else
  {
    vn.insert(vn.end(), pm->second.begin(), pm->second.end());
    cmbnodes.erase(pm);
  }
  // update the end address
  bn.endEA = bm.endEA;
  // correct the predecessors of successors of m to be n:
  for ( int j=0; j < bn.succ.size(); j++ ) // 这里应该用 bm 好理解，但是应为上面   bn.succ = bm.succ; 所以用 bn 也没问题
  {
    int p = bn.succ[j];                    //主要把 bm 的 succ 中每个元素代表的 BB， 更新每个 BB 的 pred,
    intseq_t &bp = fc.blocks[p].pred;
    int idx = bp.index(m);
    QASSERT(idx != -1);
    bp[idx] = n;
  }
  // remove block m
  fc.nproper--;
  fc.blocks.erase(fc.blocks.begin()+m);  // 移除 m 代表的 bb
  // renumber blocks >= m
  for ( int i=0; i < fc.size(); i++ )
  {
    removed_block(fc.blocks[i].pred, m);            //因为移除了 index=m 的 node ， 所以 m 后的所有 succ pred 里的 node 的 index 要减 1
    removed_block(fc.blocks[i].succ, m);
  }
  cmbnodes_t ninc; // updated ranges
  for ( cmbnodes_t::iterator p=cmbnodes.begin(); p != cmbnodes.end(); )
  {
    int n = p->first;
    areavec_t &vec = p->second;                  //因为移除了 index=m 的 node，所以 在 cmbnodes 这个 map 里，所有 node index 对应已合并 bb 的 vec,这里 index 要减 1
    if ( n >= m )
    {
      ninc[n-1] = vec;
      cmbnodes.erase(p++);
    }
    else
    {
      ++p;
    }
  }
  cmbnodes.insert(ninc.begin(), ninc.end()); //把 index-1 后生成的 map 插入到全局变量 map 里。
}

//--------------------------------------------------------------------------
static void combine_sequential_nodes(qflow_chart_t &fc)
{
  msg("combine_sequential_nodes");msg("\n");
  char tmp[20];
  // calculate predecessors
      itoa(fc.size(), tmp, 10);
      msg("fc.size(): "); msg(tmp);msg("\n");

  for ( int n=0; n < fc.size(); n++ )
  {
    int ns = (int)fc.nsucc(n);                           // 第 N 个 node 一共有多少个后继者
  itoa(ns, tmp,10);
  msg("ns: "); msg(tmp);msg("\n");
    for ( int j=0; j < ns; j++ )
  {
    itoa(fc.succ(n,j), tmp,10);
     msg("fc.succ(n,j): "); msg(tmp);msg("\n");
      fc.blocks[fc.succ(n,j)].pred.push_back(n);
    // fc.succ(n,j)是 第 N 个 node 第 j 个后继者
    //然后设置这个继承者的 pred(qvector),加进去


  }
  }
  // n -> m, n&m can be combined if
  //    nsucc(n) == 1      //如果 n 的继承者只有一个 ，m 的父辈只有一个，且 m 的父辈是 n，那么这两个 node 可以合并
  //    npred(m) == 1
  cmbnodes.clear();
  for ( int n=0; n < fc.size(); n++ )
  {
    if ( fc.nsucc(n) != 1 )
      continue;
    int m = fc.succ(n, 0);
    if ( fc.npred(m) != 1 )
      continue;
    if ( n == m )
      continue;
    // ok, found a sequence, combine the blocks
    combine_blocks(fc, n, m);
    n--; // check once more        //bug 如果第一个就合并了 这里变成-1 了。。
  }
}

//--------------------------------------------------------------------------
static bool generate_combined_node_text(int n, text_t &text)
{
  cmbnodes_t::iterator p = cmbnodes.find(n);
  if ( p == cmbnodes.end() )
    return false; // this node has not been combined

  // generate combine node text by generating text for all nodes in it
  areavec_t &vec = p->second;
  for ( int i=0; i < vec.size(); i++ )
  {
      char tmp[20];

      itoa(vec[i].startEA, tmp, 16);
      msg("vec[i].startEA: "); msg(tmp);msg("\n");
    ea_t ea = vec[i].startEA;
    gen_disasm_text(ea, vec[i].endEA, text, false);
      itoa(vec[i].endEA, tmp, 16);
      msg("vec[i].endEA: "); msg(tmp);msg("\n");
  //msg("gen_disasm_text: %s", text);
  }
  return true;
}

//--------------------------------------------------------------------------
static int idaapi idp_cb(void *, int code, va_list va)
{
  switch ( code )
  {
    case processor_t::preprocess_chart:
                              // gui has retrieved a function flow chart
                              // in: qflow_chart_t *fc
                              // returns: none
                              // Plugins may modify the flow chart in this callback
      {
        qflow_chart_t *fc = va_arg(va, qflow_chart_t *);
        combine_sequential_nodes(*fc);
      }
      break;
  }
  return 0;
}

//--------------------------------------------------------------------------
static int idaapi ui_cb(void *, int code, va_list va)
{
  switch ( code )
  {
    case ui_gen_idanode_text:   // cb: generate disassembly text for a node
                              // qflow_chart_t *fc
                              // int node
                              // text_t *text
                              // Plugins may intercept this event and provide
                              // custom text for an IDA graph node
                              // They may use gen_disasm_text() for that.
                              // Returns: bool text_has_been_generated
      {
        /*qflow_chart_t *fc =*/ va_arg(va, qflow_chart_t *);
        int node = va_arg(va, int);
        text_t *text = va_arg(va, text_t *);
        return generate_combined_node_text(node, *text);
      }
  }
  return 0;
}

//--------------------------------------------------------------------------
int idaapi init(void)
{
  // unload us if text mode, no graph are there

//   warning("我加载了");
  if ( callui(ui_get_hwnd).vptr == NULL )
    return PLUGIN_SKIP;

  hook_to_notification_point(HT_IDP, idp_cb, NULL);
  hook_to_notification_point(HT_UI, ui_cb, NULL);
  
  return PLUGIN_KEEP;
}

//--------------------------------------------------------------------------
void idaapi term(void)
{
  unhook_from_notification_point(HT_IDP, idp_cb, NULL);
  unhook_from_notification_point(HT_UI, ui_cb, NULL);
}

//--------------------------------------------------------------------------
void idaapi run(int /*arg*/)
{
  info("This plugin is fully automatic");
}
char wanted_name[] = "graph2";
//--------------------------------------------------------------------------
//
//       PLUGIN DESCRIPTION BLOCK
//
//--------------------------------------------------------------------------
plugin_t PLUGIN =
{
  IDP_INTERFACE_VERSION,
  0,        // plugin flags
  init,                // initialize
  term,                // terminate. this pointer may be NULL.
  run,                 // invoke plugin
  NULL,
  NULL,
  wanted_name,
  NULL
};
```
