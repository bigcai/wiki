# 观测技术演练场（Observability Lab）

这是一个面向 **perf / FlameGraph / eBPF / bcc / bpftrace** 的系统化练习场。

目标不是背命令，而是训练一种能力：

> 从现象出发，逐步构建“观察系统、解释系统、定位根因”的能力。

---

## 1. 为什么要建这个演练场

很多人学习性能分析时，会陷入两个误区：

- 把 perf 当成命令大全来背
- 把 eBPF 当成炫技工具来学

这样会导致：

- 会敲命令，但不知道为什么用
- 看到了数据，但不会解释
- 工具越学越多，问题却定位不下来

这个演练场的核心思路是：

```text
现象 → 热点 → 调用路径 → 观测设计 → 根因解释
```

也就是把学习路径统一成这条链：

```text
perf → flamegraph → eBPF → bcc → bpftrace
```

---

## 2. 这条进阶路线为什么成立

### perf
你第一次“看到系统在干嘛”。

它擅长回答：
- CPU 花在哪些函数上
- 哪些路径是热点
- 当前系统大概在忙什么

它的特点是：
- 视角高
- 上手快
- 但本质是采样，不是精确计时

所以 perf 更像：

> 用望远镜看系统热点

---

### FlameGraph
你第一次“看懂结构”。

FlameGraph 不是新的采样器，而是把 perf 的堆栈数据可视化。

它擅长回答：
- 热点主要堆在哪条调用链上
- 哪一层宽，哪一层是放大器
- 这个问题是叶子函数热，还是上游路径放大

所以 FlameGraph 更像：

> 把散乱数据变成结构认知

---

### eBPF
你第一次“可以自己定义怎么观察”。

当 perf 只能告诉你“哪里热”，但你还想知道：
- 为什么 syscall 变慢
- 为什么 IO 延迟抖动
- 为什么调度异常
- 为什么锁竞争严重

这时候就需要 eBPF。

eBPF 的本质不是一个命令，而是：

> 在内核观测点上编写可编程的观测逻辑

所以：

```text
perf = 观察现象
eBPF = 定义观察方式
```

---

### bcc
bcc 是把 eBPF 工程化。

如果直接写底层 eBPF，门槛很高；bcc 提供了一套更可用的开发方式：
- 内核态逻辑用 BPF 程序描述
- 用户态通常用 Python 组织
- 能写成长期可复用的工具

它更适合：
- 写复杂 tracing 工具
- 做统计聚合
- 做长期观测工具

所以你可以把它理解成：

> bcc = 造工具的工具箱

---

### bpftrace
bpftrace 是把 eBPF 脚本化。

它适合：
- 临时排查
- 快速验证猜想
- 一句话挂探针看现场

例如：

```bash
bpftrace -e 'tracepoint:syscalls:sys_enter_execve { printf("%s\n", comm); }'
```

它非常适合把“我怀疑这里有问题”快速变成“我看到证据了”。

所以你可以把它理解成：

> bpftrace = 随手观测的脚本语言

---

## 3. bcc 和 bpftrace 的关系

它们都建立在 eBPF 之上，但定位不同：

```text
eBPF（底层能力）
 ├── bcc（偏工程化，适合做工具）
 └── bpftrace（偏脚本化，适合快速排查）
```

一个非常实用的记忆方式：

```text
bcc = 造武器
bpftrace = 用武器
```

或者：

```text
bcc = C++ SDK
bpftrace = Python 脚本
```

一般来说：
- 先用 bpftrace 建立直觉
- 再用 bcc 做复杂工具

---

## 4. 学习这个方向的正确方法

不要一开始就死记硬背参数。

正确方法是：

1. 写一个简单程序或找一个明确现象
2. 用 perf 看热点
3. 用 FlameGraph 看路径结构
4. 当问题解释不动时，引入 eBPF 观测
5. 用 bpftrace 快速验证猜想
6. 用 bcc 沉淀成长期工具

所以学习过程不是：

```text
记命令 → 背参数 → 收藏教程
```

而应该是：

```text
现象 → 观察 → 假设 → 验证 → 解释 → 沉淀
```

---

## 5. 演练场目录结构

```text
observability-lab/
├── README.md
├── notes/
│   ├── 01-roadmap.md
│   ├── 02-perf-basics.md
│   ├── 03-flamegraph-basics.md
│   ├── 04-ebpf-overview.md
│   ├── 05-bcc-vs-bpftrace.md
│   └── 06-learning-plan.md
├── labs/
│   ├── lab01-perf-hotspot.md
│   ├── lab02-flamegraph-cpu.md
│   ├── lab03-bpftrace-syscall.md
│   └── lab04-bcc-tool-thinking.md
└── templates/
    └── issue-template.md
```

---

## 6. 你将训练出的能力

做完这一套后，你应该逐步具备这些能力：

- 能区分“热点”与“根因”
- 能理解采样与追踪的边界
- 能从 CPU、syscall、调度、IO 这些维度组织排查路径
- 能把一次排查沉淀成文档或工具
- 能把现象翻译成系统机制解释

---

## 7. 学习原则

### 原则一：先建立系统直觉，再追求工具熟练
不是先背参数，而是先知道每个工具回答哪类问题。

### 原则二：每次练习都要写“现象—猜想—证据—结论”
不要只保存命令输出，要保存思考过程。

### 原则三：一个工具学到“能解释”才算过关
比如 perf 不是会敲 `perf record` 就结束，而是能解释：
- 为什么这里是热点
- 为什么这个热点不一定是根因
- 为什么需要下一步观测

---

## 8. 接下来的建议

建议从以下顺序开始：

1. 先做 `lab01-perf-hotspot`
2. 再做 `lab02-flamegraph-cpu`
3. 然后做 `lab03-bpftrace-syscall`
4. 最后进入 `lab04-bcc-tool-thinking`

如果你能持续记录每次演练的结果，这个仓库就会逐步变成你自己的“性能排查知识库”。
