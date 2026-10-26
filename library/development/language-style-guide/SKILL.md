---
name: language-style-guide
description: Apply the official style and engineering conventions of a language when writing or reviewing code — per-language numbered references covering formatting, naming, file/module layout, types, API design, error handling, concurrency, docs, tooling, examples, and a review checklist, for C, C++, Python, Rust, Go, JavaScript/TypeScript, C#, Kotlin, PHP, Ruby, Bash, and Markdown.
version: "1.2"
author: "catyugu"
license: "MIT"
---

## When to Use

写或审某个语言的代码时加载此技能。先匹配仓库现有配置，再读对应语言的参考文件；不要把不同生态的偏好混用。

## 通用原则

- **项目约定优先**：先读 `.editorconfig`、formatter、linter、CI 配置和邻近代码。
- **自动化优先**：格式争议回到 formatter 输出（gofmt/rustfmt/black/prettier/clang-format）；不手工制造格式差异。
- **命名表意**：名称表达领域概念、单位、所有权、生命周期和副作用。
- **函数专注**：保持短小；拆分应降低认知负担，而不是追求固定行数。
- **错误显式**：区分可恢复错误、缺失值、异常和不可达状态；不静默吞错。
- **注释说 why**：解释约束、权衡、不变量、线程/资源/安全语义，不复述代码。
- **验证闭环**：格式化、静态检查、测试和构建均以真实工具输出为准。

## 语言参考

| 语言                      | 文件                                  |
| ------------------------- | ------------------------------------- |
| C++                       | `references/google-cpp-style.md`      |
| C                         | `references/c-style.md`               |
| Python                    | `references/pep8-style.md`            |
| Rust                      | `references/rust-style.md`            |
| Go                        | `references/go-style.md`              |
| JavaScript / TypeScript   | `references/javascript-style.md`      |
| C#                        | `references/csharp-style.md`          |
| Kotlin                    | `references/kotlin-style.md`          |
| PHP                       | `references/php-style.md`             |
| Ruby                      | `references/ruby-style.md`            |
| Bash                      | `references/bash-style.md`            |
| Markdown（文档写作）      | `references/markdown-style.md`        |

每个参考文件结构相同：适用范围与优先级 → 缩进/行宽/排版 → 命名表 → 文件与模块组织 → 类型与值 → 函数与 API 设计 → 错误处理 → 并发与资源 → 注释与文档 → 工具链与验证 → Good/Bad 反例 → 评审清单。末尾一句给出冲突时的裁决顺序。

参考文件记录稳定的命名、格式、API 设计和错误处理原则；具体版本、项目规则和工具参数以官方文档及仓库配置为准。

## 执行顺序

1. 识别语言、版本、框架和仓库级工具配置。
2. 读取对应参考文件，只取与目标代码相关的规则；跳过不适用的章节，不要为了"完整"堆规则。
3. 遵循现有代码的局部约定；冲突时以明确的仓库配置和编译器/formatter 行为为准。
4. 让改动保持局部、可读、可测试；不为风格清理无关代码。
5. 按参考文件末尾的评审清单自检，运行 formatter、linter、测试或构建，并报告真实结果。

## 边界

- 语言语义问题（未定义行为、生命周期、类型系统限制）以官方规范为准，本技能只管风格与工程约定。
- 对抗"过度规范化"：规则与可读性冲突时，按裁决顺序降级并写清理由，而不是机械套用。
- 不引入项目里不存在的工具链；仓库没有 formatter 就按邻近代码写，并在报告里说明。

## 行为原则

加载 `code-writing-and-style` 获取更高层的 Karpathy 风格原则：先思考、保持简单、外科手术式变更、目标驱动。
