---
title: "Tmux"
date: 2023-06-10T14:40:28+05:30
draft: true
---

Lets have a look at tmux today which is known to make using terminal/shell comfortable to use and navigate. As a programmer most of our time is spent on navigating the the n terminal sessions open and jumping back and forth executing commands. Which as much fun as it is sometimes can be frustating if you don't have your setup configured properly. Now there are a lot of ways to achieve this today we are gonna explore tmux.

### Motivation
My reason to learn was majorly my frustating experience with navigating active session and since I majorly work on remote machine preserving the session state even after connection terminates is very important. 

After getting some experirnce with tmux I can list 3 major reason to switch to tmux and how it handles them.

First things first, start by installing tmux on your machine and launch a new tmux session with 

```bash
yassingh@pc ~ $ tmux
```
This will launch and attach you to a new tmux session(we will see what these are later).
You will find yourself 
### 1\) Panes
