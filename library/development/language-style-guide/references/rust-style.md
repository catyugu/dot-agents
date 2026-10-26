# Rust（rustfmt + API Guidelines）— 落地细则

Source: https://doc.rust-lang.org/style-guide/（rustfmt 的规则来源）与 https://rust-lang.github.io/api-guidelines/
配套：`cargo fmt`（格式唯一裁决者）、`cargo clippy`、`rustdoc`；CI 跑 `cargo fmt --check` + `clippy -- -D warnings` + `test`。

## 1 适用范围与优先级

- 排版没有争议空间：`rustfmt` 输出即标准。手排只用于 rustfmt 覆盖不到的地方（宏内部、字符串、注释）。
- Rust Style Guide 是 rustfmt 的规范来源；Style Edition 与语言 Edition 解耦（2024 Edition 起可在 `rustfmt.toml` 写 `style_edition`），项目必须显式声明，否则编辑器 format-on-save 与 `cargo fmt` 输出会不一致。
- Rust API Guidelines 只约束**公开 crate API** 的设计（命名、转换、错误、可扩展性），不影响内部实现风格。
- Clippy 覆盖"惯用法"层面：正确性（correctness）与可疑（suspicious）级别必须清零；风格（style）级别按仓库配置。
- 优先级：仓库 `rustfmt.toml`/`clippy.toml`/`Cargo.toml`（`edition`、`rust-version`）> 本文件 > Style Guide / API Guidelines。
- 语义来源是 Rust Reference 与标准库文档，不是博客。

## 2 缩进与排版

- 缩进 4 空格；`max_width` 默认 100（`rustfmt.toml` 可改，内核态/嵌入式项目常改 80）。
- 函数签名放不下时一个参数一行，尾随逗号；rustfmt 的"魔术逗号"决定了合并还是展开，别手工对抗。
- 方法链换行由 rustfmt 处理：链式调用断行时把 `.method()` 放在行首。
- 块状结构：`fn`/`struct`/`impl`/`match` 用块；`match` 每个臂末尾逗号，短臂可单行。
- 单行 `if` 表达式用 `if cond { x } else { y }`（rustfmt 只在能保持单行时保留）。
- `use` 顺序由 rustfmt（`reorder_imports`）排序；分组习惯：`std` → 外部 crate → `crate::` → `super::`/`self::`。禁用 glob 导入（`prelude`、测试模块除外）。
- 类型标注：`let x: Vec<String> = Vec::new();` 里 `:` 前不留空格、后一空格。

## 3 命名

| 对象                              | 规则                        | 示例                              |
| --------------------------------- | --------------------------- | --------------------------------- |
| 函数 / 方法 / 局部变量 / 模块     | `snake_case`                | `calc_speed`, `item_count`        |
| 类型 / trait / enum variant       | `UpperCamelCase`            | `struct HttpRequest`, `BadGateway`|
| 常量 / `static`                   | `SCREAMING_SNAKE_CASE`      | `const DIAL_TIMEOUT_MS`           |
| crate 名                          | `snake_case`                | `serde_json`                      |
| 泛型参数                          | 短大写                      | `<T>`, `<K, V>`, `<E: Error>`     |
| 生命周期                          | 短小写                      | `'a`, `'de`                       |
| 布尔 / 谓词                       | `is_`/`has_`/`can_` 或动词  | `is_empty`, `has_next`            |

- 转换方法名按 API Guidelines 的约定固定（`C-CONV`）：
  - `as_*`：廉价、借用 → 借用（`as_str`, `as_bytes`）；
  - `to_*`：需要计算/分配的转换（`to_owned`, `to_string`）；
  - `into_*`：消耗 self（`into_iter`, `into_inner`）。
- getter 不加 `get_` 前缀（`fn name(&self) -> &str`）；setter 用 `set_*`。
- 迭代器方法不带 `iter` 以外的花样：`iter()`/`iter_mut()`/`into_iter()` 固定；返回 `impl Iterator<Item = T>`。
- 错误类型叫 `XxxError`，trait 用形容词/能力命名（`Read`, `Encode`）；密封 trait 用 `sealed` 子模块。
- 不用缩写除非是领域公认（`ctx`, `cfg`, `buf`, `rx`/`tx`）；不为了短而丢字母。

## 4 模块与 crate 结构

- 小型 crate：单 `lib.rs`；变大后拆 `mod`，用 `foo/mod.rs` 或 `foo.rs + foo/`（同仓统一，Rust 2018+ 推荐后者）。
- 公开 API 收敛在 `lib.rs` 的 `pub use`：内部模块 `pub(crate)`，对外只暴露有意设计的名字。
- 模块私有性与可见性最小化：默认私有，按需 `pub(crate)`，最后才 `pub`。
- feature 是兼容性契约：不随意改语义、不把 feature 组合成"未测试状态"；CI 至少跑 `--no-default-features` 与全 feature。
- 二进制与库分离：`src/main.rs` 只做参数解析和错误输出，逻辑在库内以便测试。
- 不把逻辑写进 `build.rs`；生成代码与手写代码分模块，生成物标注来源。
- `unsafe` 集中到最小模块并统一封装安全 API，外层不散落 `unsafe`。

## 5 类型、所有权与封装

- 参数用借用类型：`&str` 而非 `&String`、`&[T]` 而非 `&Vec<T>`；需要"任意可转换"时用 `impl AsRef<Path>`/`impl Into<String>`，但不要为泛而泛。
- 返回借用还是拥有：能返回 `&str` 就不要返回 `String`；确实需要拥有时返回 `String`/`Cow<'_, str>`（条件拥有）。
- 用枚举表达互斥状态（`enum Phase { Idle, Running(JobId), Done }`），不要用"多个 `Option` 字段互相约束"。
- newtype 隔离单位与语义（`struct Meters(f64)`），避免"两个 `f64` 相加"这类静默 bug。
- 公开 struct/enum 用 `#[non_exhaustive]` 为将来加字段/变体留余地；只读数据用 `#[derive(Debug, Clone, PartialEq, Eq)]`，按需而非堆砌。
- `Default` 只在零值合理时实现；`Copy` 只在类型确实廉价且语义是值复制时派生。
- `Deref` 不用于"继承"复用方法（会破坏封装与可发现性）。
- 内部可变性（`Cell`/`RefCell`/`Mutex`）要能说清为什么需要；`Rc<RefCell<T>>` 不能跨线程共享。

## 6 trait 与 API 设计

- 参数位置用 `impl Trait`（单态，签名更短），需要调用方显式指定类型参数时用泛型；trait 对象（`dyn Trait`）用于运行时多态，注意对象安全。
- 错误类型：库导出具体错误 enum 并实现 `std::error::Error + Display`（`source()` 返回底层错误）；应用层可以用 `anyhow::Error`，**不要把 `anyhow` 泄漏到库的公开 API**。
- `Display` 给人看，`Debug` 给开发者看；两者都要实现错误类型。
- 不在公开 API 里用具体容器类型，除非语义需要：接受 `impl IntoIterator<Item = T>`，返回 `impl Iterator`。
- 为公开类型实现 `From`/`TryFrom`（而不是自定义 `from_*` 构造函数）；`From` 只能是无损转换。
- 公开 trait 若要封闭，用私有 `sealed` 模式；公开 trait 一旦发布加方法就是破坏性变更，必要时给默认实现。
- 文档里的示例就是测试：`///` 代码块由 `cargo test` 编译运行。

## 7 错误处理

- 可恢复错误一律 `Result<T, E>` + `?`；`Option<T>` 只表达"缺失"，不表达"失败原因"。
- 库路径禁止 `unwrap()`/`expect()`（除已证明的不变量，且 `expect` 文案要写清前提）。
- `panic!` 只用于违反不变量的编程错误；线程边界用 `catch_unwind` 只在必须时，并在文档写清 `# Panics`。
- 加错误上下文用 `thiserror` 派生枚举或 `anyhow::Context`（`map_err(|e| Error::Load { path, source: e })`），不要丢掉底层原因。
- 不用 `Box<dyn Error>` 作为库公开错误类型（不可匹配）；不要用字符串当错误类型。
- 提前返回保持主路径扁平：`let Some(x) = opt else { return Err(...) };`（let-else）。
- 不把 `Result` 里的错误静默 `ok()` 掉；需要忽略时写 `let _ = ...` 并注释原因。

## 8 并发、unsafe 与资源

- 线程间共享用 `Arc<Mutex<T>>`/`Arc<RwLock<T>>`；明确锁粒度与锁顺序，**不要跨 `await` 持锁**（改用 `tokio::sync::Mutex` 或缩小临界区）。
- 优先消息传递（`mpsc`/`crossbeam` channel）而不是共享加锁可变状态。
- `Send`/`Sync` 是类型系统承诺：不要为绕过编译错误手写 `unsafe impl Send`；确需时写 `// SAFETY:` 说明全部前提。
- `unsafe` 规则：最小化块范围、每处写 `// SAFETY:` 解释满足的不变量（指针有效性、别名、初始化、线程）、提交前跑 Miri（`cargo +nightly miri test`）。
- 资源释放靠 RAII（`Drop`、`BufReader`、`File`）；不手写 `close` 语义；异步资源用 `Drop` 或显式 `shutdown()` 并文档化。
- 避免 `mem::forget`/`ManuallyDrop` 除非与 `unsafe` 代码配套并注释原因。

## 9 注释与文档

- 公开项写 `///` 文档注释；模块用 `//!`（放文件顶部）；crate 级说明放 `lib.rs` 的 `//!`。
- 标准段落（rustdoc 识别）：`# Examples`、`# Errors`、`# Panics`、`# Safety`（`unsafe fn` 必备）。
- 文档注释是完整句子，首行一句话摘要；公开 API 要说明复杂度、阻塞行为、分配行为与失败条件。
- `#[doc(hidden)]` 只用于生成宏/内部细节，不作为"半个公开 API"的借口。
- 普通注释讲 why：为什么这样加锁、为什么这里要 `unsafe`、为什么这个 clone 不可避免。
- `#[allow(...)]` 只在最小作用域加，并注明触发原因；禁止在 crate 根部全局静默。
- README 与 crate 文档保持一致，示例代码要能编译。

## 10 工具链与验证

- `cargo fmt --check`（配 `rustfmt.toml` 的 `style_edition`/`max_width`）必须通过。
- `cargo clippy --all-targets --all-features -- -D warnings`：correctness/suspicious 清零，其余按需 allow 并注释。
- `cargo test`（含 doc test）、`cargo test --all-features`、必要时 `--release` 跑一遍。
- `cargo doc --no-deps` 检查文档告警（`#![deny(rustdoc::broken_intra_doc_links)]` 可入库）。
- MSRV 用 `rust-version` 声明，CI 至少跑一次 MSRV 构建；依赖审计用 `cargo deny`/`cargo audit`（按仓库配置）。
- 只认工具输出：改完报告实际命令与结果，不用"应该能编译"代替验证。

## 11 反例（Good / Bad）

```rust
// Bad：库代码里 unwrap、借用类型用错、错误上下文丢失
pub fn load(path: &String) -> Config {
    let raw = std::fs::read_to_string(path).unwrap();
    serde_json::from_str(&raw).unwrap()
}

// Good：借用 &Path、错误带上下文、不 panic
pub fn load(path: impl AsRef<Path>) -> Result<Config, LoadError> {
    let raw = std::fs::read_to_string(path.as_ref()).map_err(|source| LoadError::Read {
        path: path.as_ref().to_path_buf(),
        source,
    })?;
    serde_json::from_str(&raw).map_err(LoadError::Parse)
}
```

```rust
// Bad：unsafe 没有前置条件说明、借用检查绕道
unsafe { *(ptr as *mut u8) = 0 };

// Good：最小 unsafe 块 + SAFETY 说明不变量
// SAFETY: `ptr` 来自 Box::into_raw，且本函数是该指针唯一所有者，
// 调用前后没有别的别名存在。
unsafe { *ptr = 0 };
```

```rust
// Bad：用 Deref 伪装继承、共享可变状态绕过检查
impl Deref for Service { type Target = Inner; fn deref(&self) -> &Inner { &self.inner } }

// Good：显式委托，语义清楚
impl Service { pub fn status(&self) -> Status { self.inner.status() } }
```

## 12 评审清单

- [ ] `cargo fmt --check` 通过，`rustfmt.toml` 声明了 `style_edition`
- [ ] 命名符合 snake_case / UpperCamelCase / SCREAMING_SNAKE_CASE，转换为 `as_`/`to_`/`into_`
- [ ] 公开 API 参数用借用类型，返回 `impl Iterator`/`Cow` 等而非过度拥有
- [ ] 库路径无 `unwrap`/`expect`；错误类型实现 `Error + Display`，无 `anyhow` 泄漏
- [ ] 每处 `unsafe` 都有 `// SAFETY:`，范围最小；Miri 跑过（如适用）
- [ ] 无跨 `await` 持锁；`Send`/`Sync` 没有被 `unsafe impl` 硬绕
- [ ] 公开项有 `///`，含 `# Examples`；`unsafe fn` 有 `# Safety`
- [ ] 公开 struct/enum 考虑 `#[non_exhaustive]`，trait 有封闭或扩展策略
- [ ] `clippy -D warnings`、`cargo test`、`cargo doc` 全绿（真实运行）
- [ ] MSRV 与 feature 组合至少各验证一次

> 冲突时裁决顺序：`rustfmt` 输出与 `rustfmt.toml` > 仓库 clippy 配置 > 本文件 > Style Guide / API Guidelines > 社区惯例。
