---
title: "Using git worktrees to manage large codebases"
date: 2023-03-01T00:01:46+05:30
---

*As the first blog in the series let's keep it short and informative.* 

---

## Introduction ##
If you work on compilers you will always come across the need to have 
multiple compiler versions as a part of your development arsenal. One might be the latest one(in sync with upstream), one might be for a brand new feature you are working on and the other might be some compiler crash issue that you are exploring. 

When I started out I thought about 2 ways of managing it:

1) Keep separate repositories for all, it was easy to use but very difficult to manage (eg moving around commits in between isn't straightforward). When you have the source of size LLVM it will quickly eat up all your memory and also looks very redundant, surely we can do better.
2) The other option was to create separate branches. This can work but has major cons:
   - Working on multiple branches at once is a nightmare, to say the least, we all know what happens when you try to edit files on different branches at once in an IDE.
   - Switching can be expensive, especially for large scale projects developing rapidly and branches diverge really quickly.
   - It's a hassle to make sure you build to a designated directory for a branch, a bit of carelessness and you'll end up losing a lot of time.

To summarize we would like to:
- Have git branch like functionality.
- Easy to work on multiple features simultaneously. 
- Clean folder structure, separate build folder associated with source.

This is exactly where [git worktree](https://git-scm.com/docs/git-worktree) shines.

Let's understand it using LLVM source code as example.

You have the main source inside llvm-project. Let's say you have to start working on a new compiler feature and very much like to keep both sources and build-directories/binaries separate. 

## Creating Worktrees

Run the command 
```
cd llvm-project
git worktree add ../new-feature
```
The above command will create a new worktree named new-feature inside new-feature folder. This new-feaure is effectively a git branch but with its own copy of source code and hence can have its own build folders. Additionally providing all functionality you have between 2 git branches (cherry-pick, merge, rebase, diff, etc).

Below is how the folder structure will look like after adding new-feature worktree.

```
yassingh@pc ~/workstation $ ls
llvm-project  new-feature
yassingh@pc ~/workstation $ cd llvm-project; git branch
* main
+ new-feature
```
You can keep going, let's say you create another worktree for some register-allocation issue you are working on.

```
yassingh@pc ~/workstation/llvm-project (main) $ git worktree add ../regalloc-issue
Preparing worktree (new branch 'regalloc-issue')
Updating files: 100% (127092/127092), done.
HEAD is now at 003078b62d8d [Clang][Driver] Add -mcpu=help and -mtune=help to clang
```

Folder structure now:
```
yassingh@pc ~/workstation $ ls
llvm-project  new-feature  regalloc-issue
yassingh@pc ~/workstation $ cd llvm-project; git branch 
* main
+ new-feature
+ regalloc-issue
```

Once you are done, it only makes sense to get rid of them

## Deleting worktrees

Getting rid of regalloc worktree
```
yassingh@pc ~/workstation/llvm-project (main) $ git worktree remove ../regalloc-issue/
yassingh@pc ~/workstation/llvm-project (main) $ git branch -D regalloc-issue 
Deleted branch regalloc-issue (was 003078b62d8d).
yassingh@pc ~/workstation/llvm-project (main) $ git branch
* main
+ new-feature
yassingh@pc ~/workstation/llvm-project (main) $ ls ../
llvm-project  new-feature
```
## Bonus

To maintain multiple compiler binaries associated with each worktree you can follow below folder structure.

```
yassingh@pc ~/workstation $ tree -L 2 . 
.
├── llvm-project
│   ├── bolt
│   ├── build-dbg
│   ├── build-rel
│   ├── clang
│   ├── clang-tools-extra
│   ├── cmake
│   ├── compiler-rt
│   ├── CONTRIBUTING.md
│   ├── cross-project-tests
│   ├── flang
│   ├── libc
│   ├── libclc
│   ├── libcxx
│   ├── libcxxabi
│   ├── libunwind
│   ├── LICENSE.TXT
│   ├── lld
│   ├── lldb
│   ├── llvm
│   ├── llvm-libgcc
│   ├── mlir
│   ├── openmp
│   ├── polly
│   ├── pstl
│   ├── README.md
│   ├── runtimes
│   ├── SECURITY.md
│   ├── third-party
│   └── utils
└── new-feature
    ├── bolt
    ├── build-dbg
    ├── build-rel
    ├── clang
    ├── clang-tools-extra
    ├── cmake
    ├── compiler-rt
    ├── CONTRIBUTING.md
    ├── cross-project-tests
    ├── flang
    ├── libc
    ├── libclc
    ├── libcxx
    ├── libcxxabi
    ├── libunwind
    ├── LICENSE.TXT
    ├── lld
    ├── lldb
    ├── llvm
    ├── llvm-libgcc
    ├── mlir
    ├── openmp
    ├── polly
    ├── pstl
    ├── README.md
    ├── runtimes
    ├── SECURITY.md
    ├── third-party
    └── utils
```
As you can see we are maintaining separate release and debug builds(build-rel, build-dbg) for both the worktrees which will be a lot more difficult to maintain if we had followed branch-like development. 

---

> Thanks for reading!!