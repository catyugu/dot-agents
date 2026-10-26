# Ruby — 落地细则

Source: <https://rubystyle.guide/>
配套：RuboCop（lint + 自动修正）、rbs/Steep 或 Sorbet（签名）、Bundler；先以仓库 .rubocop.yml 为准。

## 1 适用范围与优先级

- Ruby 语言与标准库文档定义语义；Ruby Style Guide 与 RuboCop 默认值是**社区惯例，非官方规则**——Ruby 没有官方格式规范。
- 裁决顺序：仓库 `.rubocop.yml` / `.editorconfig` → 邻近代码现状 → Ruby Style Guide → RuboCop 默认值。理由：每个 cop 都能被项目改写，把默认值当真理会在别人的仓库里产出无用 diff。
- 先读懂 `EnforcedStyle` 再动手：同一 cop 在不同项目里语义相反（字符串引号、`end` 对齐、`rescue` 是否显式写 `StandardError`）。
- 唯一硬约束是能解析、能运行、行为不变；风格改动不得顺手改变语义，也不得为了符合风格破坏向后兼容。
- 偏离规则必须收敛：`# rubocop:disable Cop/Name` 写在违规行上方或行尾，带 cop 全名与一行理由，禁止文件级 disable。

## 2 缩进、行宽与排版

- 2 空格缩进，禁止 Tab。理由：缩进宽度统一后，diff 只反映真实改动。
- 行宽：Ruby Style Guide 要求 80；RuboCop `Layout/LineLength` 默认 `Max: 120`。以 `.rubocop.yml` 的 `Max` 为准；新代码按 80 写，任何情况下不超过 120。理由：并排 diff 与终端宽度都按 80 设计。
- 二元运算符两侧留空格，逗号/冒号后留空格：`sum = 1 + 2`、`a, b = 1, 2`。
- 三个例外不加空格：指数 `M * c**2`、rational 字面量 `1/48r`、安全导航 `foo&.bar`。理由：这三种写法里空格会割裂单个语法单元。
- 括号内不留空格：`some(arg)`、`[1, 2, 3].each { |e| puts e }`；块花括号外侧各留一个空格。
- 字符串插值不填充：`"#{user.name}"`，不写 `"#{ user.name }"`。
- 区间字面量内部不留空格：`1..3`、`'a'...'z'`。
- 一行一个表达式，不用 `;` 分隔语句。
- 多行方法链用前导点（`Layout/DotPosition` 默认 `leading`）：`.` 出现在续行开头，读者不必回看上一行结尾就知道链还在继续。
- `case` 的 `when` 与 `case` 同缩进（`Layout/CaseIndentation` 默认 `case`）；把 `case` 的结果赋给变量时整块再缩进一级，避免 `when` 顶格后看不出归属。
- 方法之间空一行；`private`/`protected` 上下各空一行；连续空行最多 1 行；`class`/`def` 体内首尾不留空行。
- 方法定义无参数时省略括号（`def foo`），有参数时写括号（`def foo(a, b)`）。

## 3 命名

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| 文件 | snake_case，`.rb` 结尾 | `billing/invoice.rb` |
| 目录 | snake_case，与命名空间路径一致 | `billing/` |
| 类 / 模块 | CamelCase，术语缩写按 `Naming/FileName` 的 AllowedAcronyms 全大写 | `Invoice`, `HTTPClient` |
| 常量 | SCREAMING_SNAKE_CASE | `MAX_RETRIES = 3` |
| 方法 / 局部变量 / 符号 | snake_case | `total_amount`, `:paid` |
| 谓词方法 | `?` 结尾，只返回 `true`/`false` | `paid?`, `include?(x)` |
| 危险方法 | `!` 结尾，表示修改接收者或抛异常而非返回 `nil` | `sort!`, `save!` |
| 未使用参数 / 变量 | `_` 前缀，或单独一个 `_` | `\|_, value\|` |
| 全局变量 | `$` + snake_case，仅在可执行脚本顶层使用 | `$stdout` |

- `!` 不是"会失败"的标记，而是"存在对应非 `!` 版本、且这一版更危险"的约定；没有对应安全版本就不要加 `!`。
- 不用 `get_`/`set_` 前缀描述属性访问，直接用名词（`name`/`name=`）；`Naming/AccessorMethodName` 会判违规。
- 标识符只用 ASCII 英文；非 ASCII 标识符在工具链、grep 与编辑器里都会出问题（`Naming/AsciiIdentifiers`）。
- 名字带单位或口径：`timeout_seconds` 优于 `timeout`，`amount_cents` 优于 `amount`。

## 4 文件与模块组织

- 一个文件一个主类，路径映射命名空间：`lib/billing/invoice.rb` 定义 `Billing::Invoice`。理由：自动加载与 `require` 路径可预测。
- 命名空间用显式嵌套 `module Billing; class Invoice`，不写 `class Billing::Invoice`。理由：`Module.nesting` 决定常量查找顺序，作用域解析写法少一层词法作用域，常量会查到别处。
- `require` 分三段、段内字母序：标准库 → gem → 项目内文件；项目内用 `require_relative`，它不依赖 `$LOAD_PATH`，加载路径变化会立刻报错而不是静默取到同名文件。
- `require` 不写 `.rb` 扩展名。
- gem 入口 `lib/<gem_name>.rb` 只做加载与常量声明，不放业务逻辑。
- 类体顺序固定：`extend`/`include`/`prepend` → 内部类与异常 → 常量 → `attr_*` → `initialize` → 公共方法 → `protected` → `private`；与 `Layout/ClassStructure` 的 `ExpectedOrder` 一致。
- 多个 mixin 分多行写（`include Foo` 换行 `include Bar`），不写 `include Foo, Bar`，方便逐个增删并保留注释。
- 禁止类变量 `@@`：它在继承链上共享，子类赋值会污染父类；改用类实例变量加 `class << self` 访问器，或直接用常量。

## 5 类型、值与不可变性

- 一切皆对象且 `nil` 也是对象；`nil` 只表示"没有值"，不用它表示 `false`、`0` 或空集合。
- 只有 `false` 和 `nil` 为假，`0`、`""`、`[]` 都为真。理由：`if items.length` 判断空数组会永真，这类 bug 在 Ruby 里极常见。
- 文件首行写 `# frozen_string_literal: true`（`Style/FrozenStringLiteralComment` 默认 `always`）。理由：字符串字面量不再每次分配，且共享常量无法被就地修改。
- 默认参数每次调用求值，`def f(items = [])` 安全；但 `Hash.new([])` 让所有键共享同一个数组，要写 `Hash.new { |h, k| h[k] = [] }`。
- 哈希键用符号与 1.9 语法 `{a: 1}`；禁止同一哈希里混用 `=>` 与 `:` 键。
- 取可选键用 `fetch(key, default)` 或 `dig`，不用 `h[:k] || default`。理由：`||` 会把合法的 `false` 当成缺失。
- 值对象优先不可变：`Data.define(:x, :y)` 的实例默认冻结；`Struct.new` 生成的类型可写，只在确需就地更新时使用。
- 数字：Integer 自动扩展精度，不会溢出；`1 / 2` 是整除得 `0`，需要小数写 `1.0 / 2` 或 `1.fdiv(2)`；浮点相等比较用 `Float::EPSILON` 容差或改用 `Rational`。
- 外部输入转符号前做白名单校验；不要把任意字符串直接 `to_sym`，也不要依赖"符号很廉价"的假设。

## 6 函数与 API 设计

- 方法 ≤10 行（Ruby Style Guide 与 `Metrics/MethodLength` 默认 10），理想 ≤5 行；空行不计入。
- 参数 ≤4 个（Ruby Style Guide 的 3~4 个上限，`Metrics/ParameterLists` 默认 `Max: 5`）；超出改为关键字参数，调用方不必记住位置。
- 关键字参数优于 options 哈希：`def create(name:, email:)` 而非 `def create(options = {})`；缺参在调用点就抛 `ArgumentError`，不必等到方法体深处。
- 布尔入参必须用关键字：`def render(pretty: false)` 而非 `def render(pretty = false)`，调用点自带标签。
- 可选位置参数放在参数表末尾；Ruby 对前置可选参数的匹配规则反直觉，容易把实参绑错位置。
- 无参方法调用不带括号（`obj.save`），有参调用带括号（`obj.save(force: true)`）。
- guard clause 挡在最前，非法输入立即返回或抛异常，主流程保持 0~1 层缩进；嵌套超过 2 层的条件分支是重构信号。
- 省略多余的 `return`（`Style/RedundantReturn`）：方法体最后一个表达式的值就是返回值。
- 禁用 `and`/`or`：优先级低于 `=`，`x = a or b` 实际是 `(x = a) or b`；统一用 `&&`/`||`。
- `attr_reader` 优于手写 getter；对外返回内部可变集合时给 `dup.freeze`，否则调用方能改坏对象不变量。
- 集合变换用块：`map`/`select`/`reject` 返回新对象，`map!` 原地改；方法内部不修改入参。
- 面向用户或调试的类要实现 `to_s`，调试输出实现 `inspect`；两者默认的 `#<Foo:0x...>` 没有信息量。

## 7 错误处理

- 自定义异常继承 `StandardError` 或其更具体的子类（`ArgumentError`、`IOError`、`KeyError`）；禁止继承 `Exception`。
- 抛异常用两参形式 `raise MyError, "msg"`，不用 `raise MyError.new("msg")`。理由：两参形式还能再传 backtrace，实例形式不行。
- `rescue => e` 等价于 `rescue StandardError => e`；优先复用标准库已有异常类，少造新类。
- 禁止 `rescue Exception`：它连 `Interrupt`、`SignalException`、`SystemExit` 一起吞掉，进程只能用 `kill -9` 结束。
- 只 rescue 当前层能处理的异常；多条 `rescue` 从具体到宽泛排列，宽泛的写在前面会让后面的分支永远不执行（`Lint/ShadowedException`）。
- 不用异常做常规控制流；rescue 后不得返回 `nil` 掩盖失败，要么修复并返回有效值，要么带上下文重新抛出。
- `ensure` 负责释放文件、锁、连接、临时目录；禁止在 `ensure` 里 `return`，那会丢弃正在传播的异常。
- 块 API 自动释放资源：`File.open(path) { |f| f.read }` 优于手动 `close`；一次性读全文用 `File.read`/`File.binread`。
- 重新抛出当前异常用不带参数的 `raise`，保留原 backtrace；包装成新异常时把原始消息写进新消息，`cause` 会自动串联。
- 异常消息必须自足：带上路径、入参、状态码或期望值，只看日志这一行就能定位。

## 8 并发、资源与生命周期

- CRuby 有 GVL：同一时刻只有一个线程执行 Ruby 代码（官方 Thread 文档提到"executed without GVL"的操作）。CPU 密集任务靠 `Thread` 不会提速，要用多进程（`Process.fork`/`spawn`）或 Ractor。
- IO 密集任务用 `Thread`，线程间用 `Queue` 传结果；Ruby 官方 Thread 文档要求线程通信走 `Queue` 这类机制，而不是共享变量。
- 共享可变状态必须加 `Mutex#synchronize`；`@counter += 1` 不是原子操作，读-改-写在 GVL 下仍能被打断。
- 子线程异常默认会在 `join` 时被报告（`Thread.report_on_exception` 默认开启）；用 `join` 或 `value` 收回结果与异常，不要让线程无声消失。
- Ractor 之间只传 shareable 对象：`Ractor.make_shareable(obj)` 会深冻结；非 shareable 对象默认深拷贝，`move: true` 转移所有权并让原引用失效。
- 文件、锁、连接、子进程一律用块 API 或 `ensure` 关闭；`at_exit` 只做最后兜底，不承载业务清理。
- 测试期间改动的全局状态（`ENV`、`$stdout`、类实例变量、`Dir.chdir`）必须在 `ensure` 或 teardown 里恢复。

## 9 注释与文档

- 注释解释 why——约束、权衡、不变量、线程与资源语义、兼容性原因；不复述代码在做什么。
- 公共 API 用 YARD 与 `@param`、`@return`、`@raise`、`@yield`、`@example` 标记；Ruby Style Guide 指定 YARD 为文档工具，`yard doc` 直接消费这些标记。
- 公共类上方写一行职责说明（`Style/Documentation` 要求）；内部类与显然的 DTO 不必写。
- 注释标记写在被标记代码的上一行：`# TODO(name):`、`# FIXME(name):`、`# HACK:`、`# OPTIMIZE:`、`# REVIEW:`，附负责人或来源链接。
- 禁止 `=begin`/`=end` 块注释；它不能被缩进，块内内容也无法被编辑器折叠识别。
- magic comment（`# frozen_string_literal: true`、`# encoding:`、shebang）置文件首行，每条一行，之后空一行再接代码（`Layout/EmptyLineAfterMagicComment`）。
- 注释语言全仓一致：要么全英文，要么与项目文档语言一致（中文注释可接受）；标识符、字符串字面量、日志文本、异常消息里不得出现中文。

## 10 工具链与验证

- 检查：`bundle exec rubocop`。自动修正分两档：`rubocop -a` 只做安全修正；`rubocop -A` 含不安全修正（会改语义），必须逐条 review 后提交。
- 配置：`.rubocop.yml` 用 `inherit_gem` 或 `plugins` 引入 rubocop-rails、rubocop-rspec、rubocop-performance；不要复制默认配置再局部改。
- 排查 cop 来源：`rubocop -d` 显示每条违规来自哪个配置文件；`rubocop --show-cops Layout/LineLength` 查当前生效参数。
- 语法检查：`ruby -c path/to/file.rb`；编译期警告用 `ruby -w`。
- 类型签名：RBS 放 `sig/` 目录，与原文件同名同路径（`lib/a/b.rb` → `sig/a/b.rbs`）；`rbs prototype rb lib/a/b.rb` 生成骨架，`rbs validate` 校验签名合法性，`rbs -I sig validate` 在指定路径下校验。
- 依赖：`bundle install` 后提交 `Gemfile.lock`；一切工具用 `bundle exec` 运行，避免落到全局 gem。
- 测试：`bundle exec rake test` 或 `bundle exec rspec`；新行为与新异常分支都要有测试，测试名描述行为而非方法名。
- 退出条件：rubocop 0 offense、测试全绿、`rbs validate` 通过（仓库启用 RBS 时）、CI 与本地跑同一条命令。
- 参考文档：[Ruby Style Guide](https://rubystyle.guide/)、[RuboCop 文档](https://docs.rubocop.org/rubocop/)、[RBS](https://github.com/ruby/rbs)、[Ruby 官方文档](https://docs.ruby-lang.org/en/master/)。

## 11 反例（Good / Bad）

```ruby
# frozen_string_literal: true

# Bad：4 空格缩进；rescue Exception 连中断信号一起吞；失败返回 nil，
# 调用方无法区分"配置为空"与"没读到配置"。
def load_config(path)
    begin
        YAML.load_file(path)
    rescue Exception
        nil
    end
end

config = load_config("/etc/app.yml")
puts config["name"] # 读失败时这里才炸，且看不出根因
```

```ruby
# frozen_string_literal: true

# Good：窄异常类、guard clause 提前失败、包装异常时带路径与原始消息，
# 调用方必须显式处理 ConfigError。
class ConfigError < StandardError; end

def load_config(path)
  raise ConfigError, "config not found: #{path}" unless File.exist?(path)

  YAML.load_file(path)
rescue Psych::SyntaxError => e
  raise ConfigError, "invalid yaml in #{path}: #{e.message}"
end
```

```ruby
# Bad：类变量在继承链上共享；add! 没有对应的安全版本；has? 返回对象而非布尔值；
# entries 直接把内部数组交出去。
class Registry
  @@entries = []

  def add!(name)
    @@entries << name
    @@entries
  end

  def has?(name)
    @@entries.find { |entry| entry == name }
  end
end
```

```ruby
# Good：实例状态、谓词只返回布尔、只读访问器返回冻结副本、没有多余的危险标记。
class Registry
  def initialize
    @entries = []
  end

  def add(name)
    @entries << name
    self
  end

  def include?(name)
    @entries.include?(name)
  end

  def entries
    @entries.dup.freeze
  end
end
```

## 12 评审清单

- [ ] 缩进 2 空格、无 Tab；行宽不超过仓库 `Layout/LineLength` 的 `Max`，新代码按 80 写
- [ ] 文件首行是 `# frozen_string_literal: true`，编码 UTF-8、行尾 LF、文件末尾单个换行
- [ ] 命名符合 snake_case / CamelCase / SCREAMING_SNAKE_CASE，`?` 只返回布尔，`!` 只在有安全版本时出现
- [ ] 每个方法 ≤10 行、参数 ≤4 个，超出部分改成关键字参数或拆成两个方法
- [ ] guard clause 挡在最前，主流程缩进不超过 2 层
- [ ] 没有 `rescue Exception`，没有 rescue 后返回 `nil`，rescue 子句从具体到宽泛排列
- [ ] 文件、锁、连接、临时目录都在 `ensure` 或块 API 内释放，`ensure` 内没有 `return`
- [ ] 共享可变状态有 `Mutex#synchronize` 或改为 `Queue` 消息传递，后台线程被 `join` 且结果被检查
- [ ] 类体内没有 `@@` 变量；对外暴露的集合是 `dup.freeze` 副本
- [ ] 公共 API 有 YARD 标记，写明参数、返回值与抛出异常的触发条件
- [ ] `bundle exec rubocop` 0 offense，`ruby -c` 通过，新增行为有对应测试
- [ ] 每个 `# rubocop:disable` 都是行级、带 cop 全名与一行理由

> 冲突时的裁决顺序：仓库 `.rubocop.yml` / `.editorconfig` → 邻近代码现状 → Ruby Style Guide → RuboCop 默认值；语义问题以 Ruby 官方文档为准，风格问题以仓库配置为准。
