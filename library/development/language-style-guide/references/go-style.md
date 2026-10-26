# Go（gofmt + Google Go Style）— 落地细则

Source: https://google.github.io/styleguide/go/guide ，细则见 https://google.github.io/styleguide/go/decisions ，评审高频意见见 https://go.dev/wiki/CodeReviewComments
配套：`gofmt`（格式唯一裁决者）、`goimports`、`go vet`、`staticcheck`；CI 跑 `gofmt -l` 与 `go vet ./...`。

## 1 适用范围与优先级

- Go 官方没有"风格规范"文档，只有 `gofmt` 这个格式化程序：**格式争议一律以 `gofmt` 输出结束**，不留人工排版空间。
- Google Go Style 分三份，层级不同，不要混读：
  - Guide（核心，规范性来源：命名、格式、行长、清晰度、一致性）；
  - Decisions（规范性但非权威：具体条目与理由，与 Guide 冲突时 Guide 优先）；
  - Best Practices（非规范：文档、错误、并发、测试的可操作建议）。
- `Effective Go` 与 `CodeReviewComments` 是长期参考，部分建议已过时（如接口设计、"表驱动测试写法"的细节），以当前工具与标准库用法为准。
- 优先级：仓库 `golangci-lint` 配置与既有代码 > 本文件 > Google Go Style。
- 语言语义只在 Go spec（https://go.dev/ref/spec）里找；`go.mod` 的 `go` 指令决定可用语言特性，不能用超出该版本的新语法。

## 2 格式化与布局

- 缩进 **Tab**，由 `gofmt` 决定；不要手工对齐，也不要为对齐写空格。
- 没有硬性行宽。可读性受"重新换行后是否出现难读片段"限制：长表达式靠重排（提前换行、提取变量）解决，不是靠数第 100 个字符。
- 注释行也没有固定上限；不确定时 80 或 100 列换行，同一文件保持一致。
- 括号：函数体与控制块必须有 `{}`，`{` 永不单独占行（这与 C/Java 相反，不要带入）。
- 空行分组表达结构：一个函数内按"取参数 → 校验 → 主逻辑 → 返回"分段。
- import 分组（`goimports` 自动处理，段间空行）：标准库 / 第三方 / 本模块。
- 构建约束 `//go:build` 放在文件最前，与 `package` 之间空一行；不要再用 `// +build`。
- **不手排**：提交前 `gofmt -w`，CI 用 `gofmt -l` 拒绝未格式化文件。

## 3 命名

| 对象                | 规则                                             | 示例                             |
| ------------------- | ------------------------------------------------ | -------------------------------- |
| 包                  | 短小、全小写、无下划线、无缩写歧义               | `http`, `tabwriter`, `oauth2`    |
| 导出标识            | `MixedCaps`（首字母大写）                        | `ComputeHash`, `MaxSize`         |
| 未导出标识          | `mixedCaps`（首字母小写）                        | `cachedValue`                    |
| 常量                | 同样 `MixedCaps`，**不用** `MAX_SIZE` 或 `kMax`  | `MaxPacketSize`, `ExecuteBit`    |
| 接收者              | 1~2 个字母、类型缩写、同类型所有方法一致         | `func (s *Scanner)`, `(w *Writer)` |
| 初始化缩写（URL/ID） | 同一串大小写一致                                 | `URLPony`, `xmlAPI`, `userID`    |

- 包名不要 `util`/`common`/`helper`/`model` 这类无信息名字；多词包名连写小写（`tabwriter` 而非 `tab_writer`）。
- 名字长度与作用域成反比：1~7 行的小作用域可以 `c`/`i`；跨页的大作用域用多词明确名（`userCount` 而非 `count`）。
- 不重复上下文：包已叫 `sqldb` 就别写 `DBConnection`（写 `Connection`）；单类型包构造函数就叫 `New`，不叫 `NewWidget`。
- 方法名不带 `Get` 前缀，直接用名词（`Counts()` 而非 `GetCounts()`）；耗时或可能失败时用 `Fetch`/`Compute` 表明代价。
- 不把类型写进变量名：`users` 而非 `usersSlice`，`users` 而非 `numUsers`；同作用域有两个形态时才加 `raw`/`parsed` 区分。
- 单字母仅限循环索引、坐标、`r`（`io.Reader`/`*http.Request`）、`w`（`io.Writer`）。
- 测试名可含下划线：`TestParse_EmptyInput`、`BenchmarkDecode`、`ExampleEval`。
- 接口命名：单方法用动词 + `-er`（`Reader`、`Stringer`）；不要给接口加 `I` 前缀。

## 4 文件与包组织

- 一目录一包；`package` 名与目录名一致；内部实现放 `internal/` 阻止外部导入。
- 包注释（`// Package foo ...`）只写一处，放 `doc.go` 或主要文件；包注释以包名开头，说明意图与不变量。
- 文件按职责命名（`decode.go`、`server_test.go`），不按类型堆砌（禁止 `utils.go`、`types.go` 这类倾倒文件）。
- 导出面从调用方出发：只导出必须跨包使用的东西；导出即承诺兼容性。
- 避免循环依赖：用接口由使用方定义、把共享类型下沉到更小的包，而不是延迟初始化或全局变量绕过。
- `init()` 尽量不用；必须有时，函数内不做事以外的副作用，并在注释写明依赖顺序。包级可变状态只在确实需要缓存时使用，并保证并发安全。
- 每个包提供一个 `_test.go`；需要黑盒测试时用 `package foo_test`。

## 5 类型、值与零值

- 设计让**零值可用**：`var buf bytes.Buffer` 直接就能写，`sync.Mutex` 零值即可锁；需要构造函数就提供 `New`，但不要让零值崩。
- 值 vs 指针：小且不可变的结构体传值；需要表示"缺失/未设置"、需要共享修改、或尺寸大时用指针。不要让同一类型在 API 上时值时指针。
- 接口要小，且由使用方（消费方）声明所需最小接口；实现方返回具体类型。不要为"将来可能扩展"提前抽接口。
- 避免 `interface{}`/`any` 承载领域数据；需要泛型时用类型参数，并约束得尽量窄（`constraints.Ordered` 等）。
- 枚举用自定义类型 + `iota` + `String()` 方法；不要用裸 `int` 常量集合。
- 时间一律 `time.Duration`/`time.Time`，不要用秒数 `int` 或字符串表示时长。
- 尽量不做"隐式布尔"API：`SetFlag(bool)` 调用点难读，改用两个语义化方法或选项结构。
- 生成代码（protobuf 等）不改；在边界包一层适配。

## 6 函数与 API 设计

- `error` 永远是最后一个返回值；`(T, error)` 顺序固定，不要交换。
- 函数签名在调用点可读：`func Open(name string) (*File, error)`；参数超过 3~4 个时用 `Options` 结构 + 函数式选项（`WithTimeout(...)`）。
- `context.Context` 是第一个参数，命名 `ctx`；**不要存进结构体**，也不要用 `context.TODO()` 长期留在生产路径。
- 不返回 `nil` 表示"空集合"与"未设置"两种含义：返回空 slice 时用 `[]T{}` 明确，或在文档写明 `nil` 语义。
- 构造 API 用 `New`/`NewFromX`；要校验的构造返回 `(*T, error)`，不要"构造后另调 `Validate`"。
- 兼容性：不删导出字段/方法，不改语义；扩展用新方法或选项；废弃写 `// Deprecated: 用 X 代替`。
- 每个导出标识都有以该名开头的文档注释（完整句子，句号结尾），说明做什么、返回什么、失败条件；不用 `// 注释` 复述签名。
- 不要在库代码里打印日志或 `os.Exit`；把错误返回给调用方，由 `main` 决定退出码。

## 7 错误处理

- error 字符串**小写开头、无结尾标点**（专有名词与首字母缩写除外），因为会被拼进上层上下文：`fmt.Errorf("open config: %w", err)`。
- 用 `%w` 包装且只包一次；分类判断用 `errors.Is`/`errors.As`，**不要字符串匹配错误文本**。
- 自定义错误类型实现 `Error() string`，必要时实现 `Unwrap() error`；哨兵错误用 `errors.New` 并导出为 `ErrXxx`。
- 多个错误一起返回用 `errors.Join`；不要把多个错误拼成字符串。
- `panic` 只用于程序员错误与不可能状态；库对外不 panic，必须 panic 时在文档写 `// Panics if ...`。`recover` 只在进程边界（HTTP 中间件、goroutine 顶层）使用并把堆栈记入日志。
- 错误要带上下文但不要重复：`fmt.Errorf("load users: %w", err)`，不要每层都写 `failed to`。
- 判断"没有结果"用哨兵错误（如 `sql.ErrNoRows`）或 `(T, bool)`，不要返回 `(nil, nil)` 又让调用方猜。

## 8 并发与资源

- 每个 goroutine 都要有明确的所有者与退出条件；启动时写清谁负责、何时返回。
- 取消与超时通过 `context` 传播，所有阻塞调用都接受 `ctx`；不要在库内部造 `context.Background()`。
- channel 所有权规则写下来：**发送方 close**，接收方不 close；不要 close 一个可能被多处发送的 channel。
- 不要用 `time.Sleep` 做同步；用 `sync.WaitGroup`、`chan struct{}` 或 `errgroup.Group`。测试里也不要靠 sleep，用 `chan`/`Eventually` 语义。
- 共享可变状态优先用 channel 或 `sync.Mutex`；`sync.Mutex` 不与值拷贝共存（含结构体内嵌时注意复制）。
- 资源用 `defer` 关闭紧跟获取点；循环内大量打开文件时把"打开+关闭"抽成函数，避免 defer 堆积。
- 并发正确性用 `go test -race ./...` 验证；报告失败时给出竞态栈，不重跑碰运气。

## 9 注释与文档

- 文档注释是 godoc 的正文：以标识名开头，第一句是摘要（独立成行），空行后写细节；段落之间空行。
- 列表与代码片段用缩进表示，godoc 不解析复杂 markdown；避免装饰性分隔线、表格、标题符号。
- 可执行示例写成 `ExampleXxx` 函数（放 `_test.go`），godoc 会展示且 `go test` 会验证输出。
- 注释讲 why：并发假设、兼容性约束、性能取舍、协议细节。不要写 `// increment i` 这类复述。
- 导出 API 的注释要说明：失败情形、是否可并发调用、是否阻塞、是否持有资源。

## 10 工具链与验证

- `gofmt -l .`（或 `gofmt -d`）必须无输出；`goimports` 负责分组与去未用导入。
- `go vet ./...` 必跑；`staticcheck ./...` 覆盖 vet 之外的可疑写法（死代码、错用 API）。
- `golangci-lint run`（配置入库）作为聚合门禁；禁用一组 linter 时在配置里写原因，而不是在代码里到处 `//nolint`。
- 测试：`go test -race -count=1 ./...`；覆盖率只作参考，不作为目标。
- 兼容性：`go build ./...` + 目标平台交叉编译（`GOOS=linux GOARCH=arm64`）验证构建约束。
- 依赖卫生：`go mod tidy` 后无 diff；`go mod verify`。
- 只认工具输出：报告里给真实命令与结果，不说"应该没问题"。

## 11 反例（Good / Bad）

```go
// Bad：error 字符串大写带标点、字符串匹配错误、丢掉包装链
if strings.Contains(err.Error(), "not found") {
    return fmt.Errorf("Failed to load user.")
}

// Good：小写无标点、%w 包装、errors.Is 判断
if errors.Is(err, sql.ErrNoRows) {
    return fmt.Errorf("load user %d: %w", id, ErrUserMissing)
}
```

```go
// Bad：常量全大写、接收者名不一致、Get 前缀
const MAX_SIZE = 1024

func (this *ReportWriter) GetSize() int { return 0 }
func (w *ReportWriter) setLimit(n int)  {}

// Good：MixedCaps、接收者一致、无 Get 前缀
const MaxSize = 1024

func (w *ReportWriter) Size() int  { return 0 }
func (w *ReportWriter) SetLimit(n int) {}
```

```go
// Bad：goroutine 无退出路径、无 context、sleep 同步
func refresh() {
    go func() {
        for {
            time.Sleep(time.Second)
            cache = load()
        }
    }()
}

// Good：显式所有者与退出条件，ctx 取消，锁保护共享状态
func (r *Refresher) Run(ctx context.Context) error {
    ticker := time.NewTicker(r.interval)
    defer ticker.Stop()
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-ticker.C:
            if err := r.reload(); err != nil {
                return fmt.Errorf("reload: %w", err)
            }
        }
    }
}
```

## 12 评审清单

- [ ] `gofmt -l` 无输出，import 分组由 `goimports` 处理
- [ ] 包名短小无下划线，无 `util`/`common` 类包
- [ ] 常量用 MixedCaps；初始化缩写大小写一致
- [ ] 接收者 1~2 字母且同类型统一
- [ ] error 最后返回，字符串小写无标点，`%w` 只包一次，判断用 `errors.Is/As`
- [ ] 没有库内 `log.Fatal`/`os.Exit`/裸 `panic`
- [ ] `context.Context` 是首参且不进结构体；阻塞调用可取消
- [ ] goroutine 有退出路径，channel 由发送方 close，无 sleep 同步
- [ ] 导出标识有以名字开头的文档注释；示例写成 `ExampleXxx`
- [ ] `go vet`、`staticcheck`、`go test -race` 全绿（真实运行）

> 冲突时裁决顺序：`gofmt` 输出 > 仓库 lint 配置与既有代码 > 本文件 > Google Go Style（Guide 优先于 Decisions）> CodeReviewComments。
