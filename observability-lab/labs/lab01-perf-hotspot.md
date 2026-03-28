# Lab01：perf 定位 CPU 热点

---

## 🎯 目标

通过本实验你将学会：

- 使用 perf 找到 CPU hotspot
- 理解“热点 ≠ 根因”
- 初步建立性能分析思维

---

## 🧪 实验 1：构造 CPU 热点

创建文件：

```c
#include <stdio.h>

void busy_loop() {
    for (volatile int i = 0; i < 100000000; i++);
}

int main() {
    while (1) {
        busy_loop();
    }
    return 0;
}
```

编译运行：

```bash
gcc test.c -o test
./test
```

---

## 🔍 实验 2：使用 perf 分析

在另一个终端执行：

```bash
perf record -p $(pidof test) -g -- sleep 10
perf report
```

---

## 🧠 你应该看到

- `busy_loop` 占用绝大多数 CPU
- 调用路径很简单（main → busy_loop）

---

## ❗ 关键理解

perf 不是在“计时”，而是在“采样”。

你看到的比例：

```text
busy_loop: 95%
```

意味着：

> 在采样时刻，大多数时间 CPU 正在执行它

而不是它精确执行了 95% 时间。

---

## 🧪 实验 3：增加复杂度

修改代码：

```c
void foo() { busy_loop(); }
void bar() { foo(); }

int main() {
    while (1) {
        bar();
    }
}
```

再次执行 perf。

---

## 🧠 观察变化

你会看到调用栈变成：

```text
main → bar → foo → busy_loop
```

---

## ❗ 思考重点

回答以下问题（写到你的记录中）：

### Q1
哪个函数最耗 CPU？

---

### Q2
为什么不是 `main` 或 `bar`？

---

### Q3
如果 `busy_loop` 很快，但被调用很多次，会发生什么？

---

### Q4（关键）

```text
如果热点在 busy_loop
是否说明问题一定在 busy_loop？
```

---

## 🔥 关键结论（你应该得出）

```text
热点 ≠ 根因
```

例如：

- busy_loop 热 → 可能只是被调用太多
- 根因可能在：
  - 上层循环
  - 业务逻辑
  - 锁竞争

---

## 📝 输出模板（必须填写）

```text
【现象】
CPU 100%

【猜想】
可能是死循环

【观察】
perf 显示 busy_loop 占 95%

【证据】
调用栈集中在 busy_loop

【结论】
当前热点在 busy_loop，但需要判断调用来源

【下一步】
引入 FlameGraph 看路径结构
```

---

## 🚀 进阶挑战（可选）

1. 引入 sleep，让 CPU 降下来，观察 perf 变化
2. 在多个函数之间分散调用，观察热点变化
3. 尝试多个线程，观察调度行为

---

## 🧠 本实验真正训练的能力

不是命令，而是：

> 从“看到热点”到“怀疑原因”的思维跳跃

如果你完成这个实验，并写出自己的解释，你已经进入性能分析的第一层了。
