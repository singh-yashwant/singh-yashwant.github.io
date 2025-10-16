---
title: "SSA form and PHI nodes in compilers"
date: 2023-04-09T20:52:34+05:30
draft: true
---

*Static Single Assignment (SSA) is a strict format to write code that enables a compiler to do all kinds of optimizations. Let's have a look at them in detail 
today.* 

---

## Introduction ##

Every compiler has a some form of interal representation for a high level languages such as Python/C++ which acts as an intermediate step 
between source and assembly. This intermediate represtation is usually in Static Single Assignment(SSA) form allowing the compiler to perform 
better saftey checks and optimizations, as we will see later in the blog. I'll try to base my explanation around aroud LLVM IR which is the 
intermediate represenation used by LLVM compiler but the theory itself can be easily extended to any compiler.   
<br>
## SSA
Consider this simple c++ function which does some arethmetic operations and returns the result.

```c++
int square_and_add(int x, int y){
    int base = x + 5;
    int offset = y - 10;
    return base * base + offset;
}
```
Lets look at the LLVM IR for above function 
```bash
yassingh@pc ~/ssa-phi $ clang -S -emit-llvm square-and-add.cpp -o -
```
- -emit-llvm: Dump IR
- -S : Emit IR in human redable form
- -o - : Don't redirect output to a file 
```bash
; ModuleID = 'square-and-add.cpp'
source_filename = "square-and-add.cpp"
define dso_local noundef i32 @_Z14square_and_addii(i32 noundef %x, i32 noundef %y) #0 {
entry:
  %x.addr = alloca i32, align 4
  %y.addr = alloca i32, align 4
  %base = alloca i32, align 4
  %offset = alloca i32, align 4
  store i32 %x, ptr %x.addr, align 4
  store i32 %y, ptr %y.addr, align 4
  %0 = load i32, ptr %x.addr, align 4
  %add = add nsw i32 %0, 5
  store i32 %add, ptr %base, align 4
  %1 = load i32, ptr %y.addr, align 4
  %sub = sub nsw i32 %1, 10
  store i32 %sub, ptr %offset, align 4
  %2 = load i32, ptr %base, align 4
  %3 = load i32, ptr %base, align 4
  %mul = mul nsw i32 %2, %3
  %4 = load i32, ptr %offset, align 4
  %add1 = add nsw i32 %mul, %4
  ret i32 %add1
}
```
This is how LLVM intermediate code representation looks like, it's not quite the assembly but also not very distinct from high level languages going around. This acts as command ground for multiple programming languages to use the same representation to be further lowered to multiple assembley files depending on the target we are compiling for. Since in this case we haven't explictly mentioned the target it will automatically pick up the host target. <br>
```
sequenceDiagram
    participant dotcom
    participant iframe
    participant viewscreen
    dotcom->>iframe: loads html w/ iframe url
    iframe->>viewscreen: request template
    viewscreen->>iframe: html & javascript
    iframe->>dotcom: iframe ready
    dotcom->>iframe: set mermaid data on iframe
    iframe->>iframe: render mermaid
```
Enough intro about IR, I'm sure you all know this.  

---

> Thanks for reading!!
