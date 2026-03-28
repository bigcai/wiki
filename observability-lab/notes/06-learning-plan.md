# 观测技术学习提纲（4 周训练计划）

这份提纲用于指引后续练习，目标不是背命令，而是建立一条稳定的排查路径：

```text
现象 → 观察 → 假设 → 验证 → 解释 → 沉淀
```

---

## 总目标

通过 4 周练习，逐步掌握：

- 用 `perf` 找 CPU 热点
- 用 FlameGraph 理解调用路径结构
- 用 `bpftrace` 快速验证观测假设
- 理解 `bcc` 和 `bpftrace` 的分工
- 把一次排查沉淀成文档和工具意识

---

## 第 1 周：perf 基础与热点意识

### 目标

建立“热点不等于根因”的直觉，理解 perf 采样的基本工作方式。

### 练习内容

1. 写一个简单的 CPU 消耗程序
2. 使用 `perf record` + `perf report` 分析热点
3. 对照源码解释热点为什么出现

### 必会命令

```bash
perf top
perf record ./a.out
perf report
perf stat ./a.out
```

### 必答问题

- 哪个函数最耗 CPU？
- perf 看到的是时间还是采样概率？
- 为什么热点不一定就是最终根因？

### 产出物

- `lab01-perf-hotspot.md` 练习记录
- 1 份自己的现象—猜想—证据—结论文档

---

## 第 2 周：FlameGraph 与调用路径结构

### 目标

建立“函数热点属于一条路径”的认知，而不是只盯着单个函数名。

### 练习内容

1. 对同一个 CPU 程序抓带调用栈的 perf 数据
2. 生成 FlameGraph
3. 观察哪一层最宽，判断是 leaf 热还是路径放大

### 必会命令

```bash
perf record -F 99 -g ./a.out
perf script > out.perf
stackcollapse-perf.pl out.perf > out.folded
flamegraph.pl out.folded > flame.svg
```

### 必答问题

- FlameGraph 的“宽度”表示什么？
- 它和 perf report 的差异是什么？
- 什么情况下 FlameGraph 比 report 更直观？

### 产出物

- 1 张 flame.svg
- 1 份结构分析记录

---

## 第 3 周：eBPF 入门与 bpftrace 快速观测

### 目标

从“看热点”进入“定义观察什么”，理解 trace 的价值。

### 练习内容

1. 用 `bpftrace` 观察 syscall 进入情况
2. 对 read / open / execve 等事件做简单统计
3. 尝试根据现象写一个最小观测脚本

### 示例命令

```bash
bpftrace -e 'tracepoint:syscalls:sys_enter_execve { printf("%s\n", comm); }'
bpftrace -e 'tracepoint:syscalls:sys_enter_openat { @cnt[comm] = count(); }'
```

### 必答问题

- bpftrace 能看到什么是 perf 不擅长看到的？
- 什么是 tracepoint？
- 为什么 eBPF 更像“定义观察方式”？

### 产出物

- 2 个自己的 bpftrace 脚本
- 1 份 syscall 观察记录

---

## 第 4 周：bcc 与工具化思维

### 目标

理解临时排查与长期工具化之间的差别。

### 练习内容

1. 运行常见 bcc 工具，如：
   - `execsnoop`
   - `opensnoop`
   - `biolatency`
2. 观察输出格式和统计方式
3. 思考如果自己做一个工具，应该观测什么

### 必答问题

- bcc 与 bpftrace 的本质差异是什么？
- 为什么 bcc 更适合复杂和长期观测？
- 哪些观测问题适合做成长期工具？

### 产出物

- 1 份 bcc 工具使用记录
- 1 份“我想做的观测工具”草案

---

## 加分练习：完整链路排查

目标是走通下面这条链：

```text
CPU 高
→ perf 找热点
→ FlameGraph 看路径
→ bpftrace 看 syscall / tracepoint
→ 解释是锁、IO、调度还是代码本身问题
```

做到这一步时，你就不是在学命令，而是在训练系统级诊断能力。

---

## 每次练习都要记录的模板

```text
【现象】
发生了什么？

【猜想】
我怀疑是什么原因？

【观察】
我用了哪些工具？看到了什么？

【证据】
哪些输出支撑了判断？

【结论】
真实原因是什么？

【下一步】
还需要补什么观测？
```

---

## 学习原则

### 原则 1：每次只解决一个明确问题
不要一上来同时研究 CPU、内存、IO、网络。先围绕一个现象把链路走通。

### 原则 2：输出比收藏更重要
宁可少看三篇文章，也要多写一份自己的排查记录。

### 原则 3：从“工具使用”过渡到“系统解释”
真正的进步不在于会敲命令，而在于你能说清楚：

- 这个输出意味着什么
- 为什么会这样
- 下一步为什么这么做

---

## 推荐起点

从下面顺序开始：

1. `labs/lab01-perf-hotspot.md`
2. `labs/lab02-flamegraph-cpu.md`
3. `labs/lab03-bpftrace-syscall.md`
4. `labs/lab04-bcc-tool-thinking.md`

当这些内容逐步积累起来，这个仓库就会成为你自己的“观测技术知识库”。
