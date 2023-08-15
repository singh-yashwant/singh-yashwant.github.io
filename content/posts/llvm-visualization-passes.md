---
title: "Graph based data structures in LLVM"
date: 2023-07-30T23:35:06+05:30
draft: false
---

Compilers make heavy use of graph data structures as all the code being compiled is essentially stored as a graph. Most compilers also provide ways for visualizing these complex data structures which might come in handy while debugging, and help understanding certain optimizations. Let's have a look at some of these data structures and how to visualize them using graphviz and LLVM passes.


*Note:* All the graphs we'll produce are generated in `.dot` format. If you want to visualize them make sure you have support for graphviz in your IDE or use an online [tool](https://dreampuf.github.io/GraphvizOnline/).
## Control Flow Graph (CFG)
A Control flow graph(CFG) is the unit of code to represent a function in LLVM. It's essentially a graph where each node represents a smaller code block. This graph is passed around in the compilation pipeline and undergoes various rearrangements that we call optimizations. Before we investigate what are all the components and their properties are it's better to have an example that will help map the concepts easily. 
Consider below C++ function.

```c++
int loopSum() {
    int ans = 0;
    for(int i = 0; i < 5; i++){
        ans += (20 % i);
    }
    if(ans < 10)
        ans += 1;
    else 
        ans += 2;
    return ans;
}
```

Whose IR and CFG look like this (I have generated IR at -O0 to prevent obvious optimizations)
![img1](/images/graphs1.jpg)

The individual boxes are *Control Flow Blocks* connected by *Control Flow Edges*.

Some interpretations from the graph:
- Each CFG has an `entry` block that contains the first instruction.
- There is a single `exit` block that contains `ret` instruction. Block `if.end` in above example.
- Each node is the smallest consecutive set of instructions that cannot be further split into smaller blocks.
- These nodes are connected to each other by edges. Which mark which block of code should execute
after this has done executing.

So how does a sequential IR code gets converted to a graph? Turns out it is straightforward provided we follow the below rules.

1) Starting from the top first instruction goes into the entry block.
2) Each consecutive instruction goes into the same block until we see a [treminator](https://llvm.org/docs/LangRef.html#terminator-instructions) instruction. eg `br, switch, ret, indirectbr`. 
3) In that case, depending on conditional (`for.cond`) or un-conditional (`entry`) branch create  new block(s) connected to this block via directed edge(s).
4) Follow rules 2-4 until you run out of instructions.

You can generate a `.dot` graph for any IR code in LLVM using `-dot-cfg` pass.
```bash
$ opt -passes=dot-cfg test.ll -cfg-dot-filename-prefix=temp -o /dev/null
Writing 'temp._Z7loopSumv.dot'...
```
`-cfg-dot-filename-prefix` will add a prefix, so the output file is not hidden. 
This pass support multiple other options which you can explore on official documentation. One
such option is `-dot-cfg-only` which will only print blocks names and edges getting rid of all the code.

One lesser-known feature of LLVM is that it also supports dumping dot graphs for MIR after [D133709](https://reviews.llvm.org/D133709). 
```bash
$ llc -mtriple=amdgcn-- -run-pass=dot-machine-cfg  -mcfg-dot-filename-prefix=temp temp.mir -o /dev/null
# Don't forget to give -mtriple as MIR is heavily target dependent
```

Having understood the working of a CFG understanding the graphs next will be easy.

## Dominator Tree
A node (block) `X` is said to dominate node `Y` if each path in the CFG from `X` to exit goes via `Y`. Eg in the above example
- Every node dominates itself.
- `entry` dominates everything.
- `for.end` dominates `if.end` but not `if.then` and `if.else`.
- Exit (`if.end`) doesn't dominate anything

Before we create a dominator tree we need 3 more definitions.
- A node `X` **strictly dominates** a node `Y` if `X` dominates `Y` and `X` does not equal `Y` .
- The **immediate dominator** of a node `Y` is the unique node that strictly dominates `Y`  but does not strictly dominate any other node that strictly dominates `Y`. In simpler terms, each node has a closest unique dominator which is called its *immediate dominator*. Every node, except the entry node, has an immediate dominator.
- A **dominator tree** is a tree where each node's children are those nodes it immediately dominates. The entry node is the root of the tree.

Hence starting from the entry node if we keep on adding directed edges to the nodes it immediately dominates, we  will get a dominator tree for that CFG.
The dominator tree of the example above will look like this 
![img2](/images/graphs2.jpg)

You can use the `dot-dom` pass to achieve this
```bash
$ opt -passes=[dot-dom|dot-dom-only] test.ll  -disable-output
```

## Post Dominator tree
A post dominator tree is very similar but exactly opposite to a dominator tree.
A node `Y` is said to post dominate a node `X` if all paths to exit node starting at `X` must go through `Y`. Eg
- Every node except exit(`if.end`) has post dominator.
- Exit (`if.end`) post dominates everything.
- `for.end` (strictly)post dominates `entry`, `for.cond`, `for.body` and `for.inc`.
- `for.end` is immediate post dominator of `for.cond`.

To create a post-dominator tree start with the exit node and keep adding its immediate post dominators.

For the above example, you can see the post-dom tree by
```bash
$ opt -passes=[dot-post-dom|dot-post-dom-only] test.ll  -disable-output
```
![img2](/images/graphs3.jpg)

Seems like LLVM has added a pseudo root node to our post-dom tree might be interesting to see why it was done for this and not for the dom-tree.
## Call Graph

A call graph is the graphical representation of all functions in your code calling each other. Each directed edge from A to B represents that there was a call from `func A` to `func B`.
For example consider below C++ code:

```llvm
define void @temp1() {
  ret void
}

define void @temp2(){
  ret void
}

define void @bar() {
  call void @temp2()
  ret void
}

define void @foo() {
  call void @bar()
  call void @temp1()
  call void @temp2()
  ret void
}
```
Whose call graph looks like this:
```bash
$ opt test2.ll -passes=dot-callgraph -callgraph-dot-filename-prefix=callgraph -disable-output
Writing 'callgraph.callgraph.dot'...
```
![img2](/images/graphs4.jpg)

This graph is necessary as many optimizations need to traverse the code bottom-up i.e. callees before callers.

### Conclusion

These were the few important graphs used inside a compiler to optimize/process the code. I feel the article itself is just an introductory one and each of the subtopics can have its own detailed explanation, but I still hope you came across a trick or 2 you didn't know before:)

Thanks for reading!!

{{< subscribe-convertkit >}}