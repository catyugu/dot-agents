# C（ISO C + 工程实践）— 落地细则

Source: https://www.iso.org/standard/74528.html（C17）与 https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf（C23 草案 N3220）
配套：clang-format + clang-tidy（或 cppcheck）+ sanitizer；先以仓库 `.clang-format`、`.clang-tidy`、编译告警集为准。

## 1 适用范围与优先级

- ISO C 只规定语言语义、库行为和实现下限，**不规定缩进、命名、行宽**；因此本文件是工程约定，不是标准。
- 三类常见外部参考，目标不同，不能互相搬运：
  - GNU Coding Standards：GNU 项目自身风格（长名字、两空格、`{` 换行长）。
  - Linux kernel `CodingStyle`：内核风格（Tab = 8 列、`goto` 统一清理、禁止 typedef 指针）。
  - SEI CERT C：安全编码规则，讲的是"别写出漏洞"，与排版无关。
- 优先级：仓库 `.clang-format`/`.clang-tidy`/CI 告警集 > 本文件 > 上述外部参考。
- 先在代码里明确目标标准：`__STDC_VERSION__` 的 C11 = `201112L`、C17 = `201710L`、C23 = `202311L`。用 C23 语法前先确认工具链真的支持（GCC 完整，Clang 部分支持）。
- C 与 C++ 不混用：`restrict`、隐式 `void*` 转换、`enum` 底层类型、结构体标签规则、`static_assert` 宏形态都不同。C 头文件若被 C++ 使用，加 `extern "C"` 保护，但内部仍按 C 规则写。

## 2 缩进、行宽与排版

- 缩进二选一并全仓统一：空格 4（多数用户态项目）或 Tab（内核风格，显示宽度 8）。同一文件禁止混用。
- 行宽 80 列（内核与多数嵌入式）或 100 列（部分用户态项目）；以 `.clang-format` 的 `ColumnLimit` 为准。
- 大括号用 K&R 变体：函数定义的 `{` 另起一行，控制语句的 `{` 与语句同行；`else`/`while` 与前一个 `}` 同行。
- **单语句也加括号**：`if (x) y();` 不加括号会在后续扩展时引入悬垂 `else`，也是 `goto fail` 这类真实漏洞的成因。
- 指针声明星号位置二选一并统一：`char *p`（内核）或 `char* p`（部分用户态）；同一仓库不要两种都有。
- `switch` 的 `case` 与 `switch` 同缩进，`case` 内部语句再缩进一级；故意的贯穿写 `/* fall through */`，与 C23 的 `[[fallthrough]]` 择一。
- 运算符优先级存疑就加括号；不要靠读者回忆 `&&` 与 `||` 的次序。
- 长条件换行时把运算符放行尾（`&&` 结尾），便于看出仍未结束。
- 不手排：跑 `clang-format -i`，格式差异只应来自工具的配置版本。

## 3 命名

| 对象                      | 规则                                      | 示例                          |
| ------------------------- | ----------------------------------------- | ----------------------------- |
| 文件                      | `snake_case.c/.h`                         | `ring_buffer.c`               |
| 函数 / 变量 / 结构体字段  | `snake_case`                              | `ring_push`, `write_index`    |
| 类型（struct/union/enum） | `snake_case_t` 或 `PascalCase`，仓库统一  | `struct ring_buffer`          |
| 宏 / 枚举常量             | `UPPER_SNAKE_CASE`                        | `RING_CAPACITY`, `ERR_TIMEOUT`|
| 内部（文件私有）符号      | `static` + 模块前缀                       | `static size_t rb_next(...)`  |
| 模块级公开符号            | 模块前缀 + `snake_case`                   | `rb_init`, `net_send`         |

- 名字带单位与量纲：`timeout_ms`、`buf_len`、`freq_hz`；带所有权或方向：`out_`、`src`、`dst`、`owned_`。
- 单字母只用于循环索引 `i`/`j`、坐标 `x`/`y`，且作用域要短。
- **不要使用保留标识符**：下划线加大写字母开头、双下划线、`str*`/`mem*`/`is*`/`to*`（`<string.h>`、`<ctype.h>` 保留）、`E*`（`<errno.h>` 宏）。这些在别的平台上会与实现冲突。
- 宏不够用时优先 `enum`、`static const`、`static inline` 函数：宏没有作用域、没有类型检查、调试信息差。

## 4 文件与模块组织

- `.h` 声明、`.c` 实现，成对存在；**头文件必须自包含**，能被单独编译（`gcc -fsyntax-only foo.h` 通过），不要依赖调用方先包含别的头。
- 包含保护二选一：`#pragma once`（主流编译器都支持）或 `#ifndef MODULE_FILE_H_ / #define ... / #endif`。同仓统一。
- `#include` 顺序（段间空行、段内字母序）：
  1. 本模块同名头（顺便证明自包含）
  2. C 标准库（`<stdio.h>` ...）
  3. 系统/平台库（`<pthread.h>` ...）
  4. 第三方库
  5. 项目内其他头
- 头文件只放：函数声明、`extern` 变量声明、宏、`typedef`、`static inline`、结构体/枚举定义；**不放定义**（会多重定义）与实现。
- 不 `#include "foo.c"`；不做 `#include` 循环；公共头不包含私有头。
- 编译单元内部辅助函数用 `static`，同时降低符号冲突与优化阻碍（同 TU 内联）。
- C23 的 `#embed` 用于内嵌二进制资源，替代"生成十六进制数组"的脚本。

## 5 类型、整数与内存

- 类型选择有明确默认：
  - 对象大小、数组下标、长度 → `size_t`；
  - 指针差 → `ptrdiff_t`；
  - 协议字段、二进制布局、序列化 → `<stdint.h>` 的 `int32_t`/`uint64_t` 等固定宽度；
  - 一般算术 → `int`。
- 有符号/无符号混比是高频缺陷：`if (len > 0)` 配 `size_t` 无意义；反向遍历用 `for (size_t i = n; i-- > 0;)`，不要写 `i >= 0`。
- 窄化转换显式写出并说明范围前提：`(uint16_t)value`，注释写"调用方保证 ≤ 65535"，或先用 `<limits.h>` 检查、用 C23 的 `ckd_add`/`ckd_mul` 做带检查运算。
- 结构体字段按对齐从大到小排（在意 `sizeof` 时）；不要 `typedef` 隐藏指针或数组（`typedef char *str_t` 会让 const 语义失真）。
- 位域与 `#pragma pack` 只用于协议/寄存器映射，并在注释写出 ABI 前提；不要用它省内存。
- UB 高发点，评审必看：越界读写、读未初始化变量、有符号溢出（`INT_MAX + 1`）、严格别名违规（用 `memcpy` 做类型双关）、`free` 后使用、`memcpy` 重叠（用 `memmove`）、解引用 `NULL`、`char` 的符号性。
- C23 可用语法（工具链支持时）：`nullptr`（替代 `NULL`）、`bool`/`true`/`false`（替代 `_Bool`）、`static_assert`（替代 `_Static_assert`）、`constexpr`（只用于对象）、`typeof`、`{}` 零初始化、`[[nodiscard]]`/`[[maybe_unused]]`/`[[fallthrough]]`、`__VA_OPT__`、`#elifdef`、`%b` 打印二进制。旧编译器回退到 C11 拼写，**不要两种拼写混用**。

## 6 函数与 API 设计

- 返回值表达失败，成功路径尽量用返回值而不是出参。固定模式：`int op(const in_t *in, out_t *out, size_t out_size)`——输入在前、输出居中、大小在后。
- `const` 正确性：只读指针用 `const T *`；小尺寸结构体传值，大结构体传 `const T *`。
- 无参函数写 `(void)`；C23 中 `()` 已表示无参，但跨标准兼容时 `(void)` 更稳妥。
- 参数超过 5~6 个改用配置结构体传递，字段只增不改语义。
- 公开函数在头文件注释里写清契约：可空性、所有权（谁分配谁释放）、线程安全性、失败返回值与 `errno` 语义。
- `[[nodiscard]]` 标在"忽略返回值一定是 bug"的函数上（分配、`read`/`write` 包装）。
- 不导出没被使用的符号；内部 API 用 `static` 或私有头隔离。

## 7 错误处理

- 每个可能失败的调用都要检查：`malloc`/`calloc`、`open`/`fopen`、`read`/`write`、`fread`/`fwrite`、`close`/`fclose`、`pthread_*`、以及 `snprintf` 返回是否被截断。
- 资源清理用单一出口 `goto cleanup`：标签逆序释放，且 `goto` 只能跳到已初始化资源之后（否则会释放未初始化指针）。多出口写法只适用于极短且无资源的函数。
- 错误码用私有枚举 + 返回 `int`（`0` 成功，负值失败）或返回 `enum err`；不要用 `-1`/`NULL` 哨兵与合法值混叠。
- 保留 `errno`：`int saved = errno; ... errno = saved;`，多步清理后仍能报告原始错误；日志里不要只打 `strerror(errno)` 而丢掉上下文。
- `assert` 只表达内部不变量，**不能用于校验外部输入**（`NDEBUG` 下会消失）；外部输入校验写成真代码并返回错误。
- 不静默吞错：`(void)close(fd)` 只在确实不关心时用，并注释原因。
- 编译告警零容忍（见 §10）；`-Wno-*` 必须有局部理由与注释。

## 8 资源、并发与安全

- 所有权约定写在头文件：谁分配谁释放、是否允许传 `NULL`、失配时是否已释放。成对操作（`lock`/`unlock`、`open`/`close`、`alloc`/`free`）保持在同一函数或同一 `goto` 链上。
- 线程：
  - 共享可变状态必须有 `pthread_mutex_t` 或 C11 `<stdatomic.h>` 保护；
  - **`volatile` 不是同步原语**，不提供原子性与内存序；
  - 条件变量等待写 `while (!pred) pthread_cond_wait(...)`，处理伪唤醒；
  - 锁顺序写进注释，避免死锁。
- 信号处理器里只调用 async-signal-safe 函数（`write`、`_exit`），不要 `printf`、`malloc`、`pthread_mutex_lock`。
- 安全编码（CERT C 核心项，不是可选项）：
  - 禁用 `strcpy`/`strcat`/`sprintf`/`gets`/`scanf("%s")`；用 `snprintf`、显式长度检查、`memcpy` + 人工 NUL 终止；
  - 外部长度先校验再分配，并检查整数溢出：`if (n > SIZE_MAX / sizeof(T)) return ERR_TOO_LARGE;`
  - 敏感数据（密钥、口令）用完显式擦除，用 C23 `memset_explicit`；普通 `memset` 可能被优化掉；
  - Annex K 的 `_s` 函数（如 `memcpy_s`）实现支持参差不齐，不可当可移植方案；团队自备带长度检查的封装。
- `setjmp`/`longjmp` 只在明确需要非局部跳转时使用（如解析器错误恢复），并保证被跳过的栈帧不持有需清理的资源。

## 9 注释与文档

- 头文件中的公开 API 注释是契约文档：参数含义与可空性、返回值、`errno`、所有权、线程安全、副作用。
- 注释讲 why：硬件寄存器约束、协议边界、锁顺序、规避的 UB、性能取舍。不要复述代码。
- Doxygen 风格 `/** ... */`（或统一 `/*! ... */`），只对公开符号生成文档；不要生成与代码不符的文档。
- 待办写 `TODO(name): 具体动作`，避免无主 `TODO`。
- 魔数一律具名：`#define`/`enum`/`static const`，并把单位与依据写进注释。

## 10 工具链与验证

- 格式：`clang-format`（`.clang-format` 入库）；CI 用 `clang-format --dry-run --Werror` 拒绝未格式化代码。
- 静态检查：`clang-tidy` 或 `cppcheck`；`gcc -fanalyzer` 对资源与空指针路径有帮助。
- 告警集起步：`-Wall -Wextra -Wpedantic -Wshadow -Wconversion -Wcast-qual -Wstrict-prototypes`；`-Wconversion` 若历史代码太吵，按目录开关而不是全局关掉。
- 动态检查：`-fsanitize=address,undefined`（ASan/UBSan）、`-fsanitize=thread`（TSan，与 ASan 互斥，分 job 跑）、`valgrind --leak-check=full`。
- 多标准编译：C11 / C17 / C23 各跑一次，暴露隐式函数声明与旧语法依赖；GCC 与 Clang 双编译器能暴露对 UB 的依赖。
- 只认工具输出：改完跑格式检查、`-Werror` 构建、sanitizer 测试，如实报告命令与结果。

## 11 反例（Good / Bad）

```c
/* Bad：无界复制、不检查 NULL、忽略部分写 */
void save(const char *src) {
    char buf[64];
    strcpy(buf, src);              /* 溢出；src 可能为 NULL */
    write(fd, buf, strlen(buf));   /* 部分写与错误都被忽略 */
}

/* Good：长度检查、显式 NUL、返回值可判 */
int save(const char *src, size_t src_len) {
    char buf[64];
    if (src == NULL || src_len >= sizeof(buf)) return ERR_INVALID;
    memcpy(buf, src, src_len);
    buf[src_len] = '\0';
    return write_all(fd, buf, src_len);
}
```

```c
/* Bad：多出口手写清理，第二条错误路径泄漏 fd */
int load(FILE **out) {
    int fd = open(path, O_RDONLY);
    if (fd < 0) return -1;
    FILE *f = fdopen(fd, "r");
    if (!f) return -1;             /* fd 泄漏 */
    *out = f;
    return 0;
}

/* Good：单一出口、逆序清理、只跳已初始化资源 */
int load(FILE **out) {
    int rc = -1;
    int fd = open(path, O_RDONLY);
    if (fd < 0) return -1;
    FILE *f = fdopen(fd, "r");
    if (f == NULL) goto cleanup_fd;
    *out = f;
    rc = 0;
    return rc;
cleanup_fd:
    close(fd);
    return rc;
}
```

```c
/* Bad：用 assert 校验外部长度，Release 构建下失效 */
assert(len < MAX);

/* Good：真代码校验，返回可传播的错误 */
if (len >= MAX) return ERR_TOO_LONG;
```

## 12 评审清单

- [ ] 每个可能失败的调用都检查了返回值与 `errno` 语义
- [ ] 资源成对释放，`goto cleanup` 只跳到已初始化资源之后
- [ ] 没有 `strcpy`/`strcat`/`sprintf`/`gets`；长度先校验再分配，且检查整数溢出
- [ ] 头文件自包含、有包含保护、不含定义、无循环包含
- [ ] 索引与长度用 `size_t`，无有符号/无符号混比告警
- [ ] 共享可变状态有互斥或原子保护，锁顺序已注释；无 `volatile` 冒充同步
- [ ] 无读未初始化、无严格别名违规、无有符号溢出、无 `free` 后使用
- [ ] 公开 API 的契约（可空、所有权、线程安全）写在头文件
- [ ] 宏没有被用来替代 `enum`/`static const`/`inline` 函数
- [ ] `clang-format --dry-run`、`-Werror` 构建、ASan/UBSan 测试全绿（真实运行）

> 冲突时裁决顺序：仓库 `.clang-format`/`.clang-tidy`/CI 告警集 > 本文件 > ISO C 语义（不可违背）> 外部参考（GNU / 内核 / CERT）。
