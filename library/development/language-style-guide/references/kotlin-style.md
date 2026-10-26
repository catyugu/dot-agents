# Kotlin — 落地细则

Source: https://kotlinlang.org/docs/coding-conventions.html
配套：ktlint（lint + 内置 formatter）+ detekt（复杂度与坏味道）+ Android Kotlin Style Guide（Android 模块）；先以仓库 .editorconfig 的 `ktlint_code_style` 与 `max_line_length` 为准。

## 1 适用范围与优先级

- 覆盖 `.kt` 与 `.kts`（构建脚本沿用同一套排版规则，行宽可放宽）；生成的代码（`build/`、ksp/kapt 产物）不检查、不格式化。
- 三层规范：Kotlin 官方 Coding Conventions（语言与组织）→ Android Kotlin Style Guide（Android 模块的硬规则，含 100 列限制）→ 仓库 `.editorconfig` 与 ktlint 配置。
- ktlint 明确声明自己在若干点上比官方 convention 更严格；被 ktlint 钉住的取值按 ktlint 执行，不要与 formatter 对抗。
- `ktlint_code_style` 是默认值的总开关：`ktlint_official`、`android_studio`、`intellij_idea` 三套取值不同（见第 2 节行长），仓库必须显式钉一套。
- 语言语义以 Kotlin 规范与 kotlinx.coroutines 文档为准；风格偏好属于项目决策，凡属社区惯例的条目本文会标注，不作官方规则引用。

## 2 缩进、行宽与排版

- 4 空格缩进，禁止制表符：除行尾换行符外，源码里唯一允许的空白字符是 ASCII 空格（0x20），字符串里的其他空白必须转义。
- 行宽：Android 模块 100 列（Android Style Guide 硬规定）；ktlint 的 `max_line_length` 默认值按 code style 区分——`ktlint_official` 140、`android_studio` 100、`intellij_idea` 不设。非 Android 的通用库取 120，并写进 `.editorconfig`。
- 列宽对 `package` 与 `import` 不适用，且这两类语句永不折行（官方与 Android 一致）。
- 大括号用 K&R（Egyptian brackets）：`{` 跟在构造开始那行的末尾，`}` 独占一行与开头对齐。理由：Kotlin 分号可省，换行参与语义，语言设计本就按 K&R 大括号假设，换成 Allman 会踩到解析意外。
- `else`/`catch`/`finally`/`do-while` 的 `while` 与前面的 `}` 同行；空块把 `}` 换到下一行，不写 `{}`（Android Style Guide 明确 `catch (e: Exception) {}` 是错的）。
- 断行顺序按官方：在运算符或 infix 函数名**之后**断；在 `.`、`?.`、`::` **之前**断；逗号留在前一个 token 后；`->` 留在参数列表后。
- 续行一律 +4 空格；禁止任何横向对齐——重命名不应改变排版。
- 尾随逗号：声明处（类参数、函数参数）鼓励使用，调用处自便；理由是 diff 只落在变更项上，增删参数不必再补逗号。
- 分号能省则省；返回 `Unit` 时省略返回类型；`if`/`when`/`try` 用作表达式时优先表达式形态。

## 3 命名

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| 包 | 全小写，无下划线 | `org.example.project` |
| 类、接口、object、enum、注解 | UpperCamelCase | `DeclarationProcessor`、`EmptyDeclarationProcessor` |
| 函数、属性、局部变量 | lowerCamelCase，无下划线 | `processDeclarations()`、`declarationCount` |
| 常量（`const`、顶层/object 下深度不可变 `val`） | SCREAMING_SNAKE_CASE | `MAX_COUNT`、`USER_NAME_FIELD` |
| 顶层/object 下持有行为或可变数据的属性 | lowerCamelCase | `mutableCollection` |
| 背板属性（公开只读 + 私有可变同义） | 私有侧加 `_` 前缀 | `_elementList` / `elementList` |
| 枚举成员 | SCREAMING_SNAKE_CASE 或 UpperCamelCase | `RED` 或 `Red` |
| 类型参数 | 单大写字母，或 `T` 前缀描述名 | `T`、`TInput` |
| 测试方法 | 反引号句子或下划线（仅测试代码） | `` `ensure everything works` `` |
| 返回 `Unit` 的 `@Composable` | UpperCamelCase | `TabHeader` |
| 工厂函数 | 可与抽象返回类型同名 | `fun Foo(): Foo` |

- 缩写规则：两字母缩写全大写（`IOStream`）；超过两字母只首字母大写（`XmlFormatter`、`HttpInputStream`）。
- 布尔属性/函数用 `is`/`has`/`can`/`should` 前缀（`isSyncing`）——社区惯例，非官方规则，但能让调用点读成自然语句。
- 名字要说明用途：`Manager`、`Wrapper`、`Helper` 这类无语义词不用；文件名里也不许出现 `Util`。
- 动作语义分清单次与返回新值：原地排序用 `sort`，返回副本用 `sorted`（官方明确要求名字体现是否修改接收者）。

## 4 文件与模块组织

- 目录结构跟包结构一致，省略公共根包：根包下的文件直接放源码根，子包放对应子目录。JVM 混合项目里 Kotlin 源码与既有源码同源根、同目录结构。
- 文件名：单类型文件用 `类型名.kt`；多声明或仅顶层声明用描述性 UpperCamelCase（`ProcessDeclarations.kt`）。
- 文件头顺序固定，每块之间恰好一个空行：版权/许可（可选）→ file 注解（`@file:JvmName`）→ `package` → import → 顶层声明。
- import 合成单列表、ASCII 排序、**禁止任何通配符 import**；组内顺序由 ktlint 的 `ij_kotlin_imports_layout` 决定，不手工干预。
- 类成员顺序：属性与 init 块 → 次构造函数 → 方法 → companion object。不按字母序或可见性排，把相关的东西放一起，让从上往下读能跟上逻辑。
- 重载必须相邻；实现接口时成员顺序与接口声明一致；嵌套类贴近使用者，仅供外部使用的放类末尾。
- 相关声明可同文件（扩展函数跟被扩展类型放一起），但文件不应到几百行以上；不要建一个只装某类扩展的"扩展仓库"文件。
- 多平台项目中，平台源集的顶层声明文件带源集后缀（`Platform.jvm.kt`、`Platform.ios.kt`），`commonMain` 下不加后缀。理由是 JVM 的 file facade 全限定名由文件名决定，同名文件会产生 "Duplicate JVM classes"。

## 5 类型、值与不可变性

- 默认 `val`，只有确实重新赋值才 `var`；先把不可变性写进类型，再谈其他。
- 集合用不可变接口声明（`Collection`、`List`、`Set`、`Map`），工厂用 `listOf`/`setOf`/`mapOf`；`arrayListOf`/`mutableListOf` 只在真的需要修改时出现。
- 值对象用 `data class`；封闭状态族用 `sealed interface`/`sealed class`，让 `when` 能穷尽检查；领域上"没有值"用可空类型表达，不用哨兵对象。
- 可空性就是契约：`String?` 必须显式处理；不用 `!!`，改用 `?:` 早返回、`requireNotNull`、`checkNotNull`，并在失败信息里带上能定位问题的值。
- 平台类型（JVM 互操作）必须收敛：公开函数返回平台类型时显式写 Kotlin 类型（`fun apiCall(): String = LegacyApi.getProperty("name")`），属性同理；局部变量可以省略。
- 常用函数类型或带类型参数的复合类型，定义 `typealias`（`typealias PersonIndex = Map<String, Person>`）；仅为避重名用 `import ... as ...`，不要造私有别名。
- 集合类型不要让 JVM 侧的 `HashSet`/`ArrayList` 出现在签名里；那会把可变实现细节固化成 API。

## 6 函数与 API 设计

- 单表达式函数用表达式体（`fun foo() = 1`），多语句或多分支用块体；官方把"能一行写完却写成 `{ return x }`"当作冗余。
- 默认参数值优先于重载函数：一组重载能压成一个带默认值的签名时，压成一个。
- 命名实参：多个同类型原始参数、或任何 `Boolean` 参数，调用点必须写参数名（`drawSquare(x = 10, y = 10, fill = true)`）。
- 扩展函数：接收者是"主操作对象"时才用；可见性收到最小（`private` 顶层、成员扩展、局部扩展），否则等于往全局补丁区扔 API。
- `infix` 只在两个对等角色之间用（`to`、`zip`、`and`）；会修改接收者的函数不得声明为 `infix`。
- 工厂函数不要与类同名，除非确实没有任何额外语义：`Point.fromPolar(angle, radius)` 优于 `Point(angle, radius)` 这个名字打架的版本。
- 属性 vs 无参函数：不抛异常、开销小或已缓存、状态不变时返回同值 → 用属性；否则用函数，让调用方预期到代价。
- 作用域函数（`let`/`run`/`with`/`apply`/`also`）一次调用最多嵌一层；每个调用的接收者与返回值必须一眼可辨，读不出返回什么就不要用。
- 编写库时叠加官方额外规则：显式写可见性、显式写返回类型与属性类型、所有公共成员有 KDoc（重写且无需新文档的除外）。

## 7 错误处理

- 抛具体异常类型并带上下文：`require(limit > 0) { "limit must be positive, was $limit" }`，不写 `throw Exception("error")` 这类无法分流的宽类型。
- `require`/`requireNotNull` 表达调用方违约（抛 `IllegalArgumentException`）；`check`/`checkNotNull` 表达自身状态违约（抛 `IllegalStateException`）；两者语义不能互换。
- 可预期失败用返回类型表达：`sealed` 结果类型或 `Result<T>`；返回 `null` 表示失败时必须同时给出原因通道（日志/错误码），只用 `null` 会让调用方无法区分"没有"和"出错"。
- `try` 只包住会抛的那几行，捕获具体异常；需要兜底时记录并重抛，不要 catch 后返回一个无法区分的默认值。
- 协程的取消不能被吞：`catch (e: Exception)` 会把 `CancellationException` 一起吞掉，必须显式重抛（`catch (e: CancellationException) { throw e }`），否则取消语义失效、结构化并发失去意义。
- 资源清理放 `finally`，但能用 `use { }` 就不要手写；`finally` 里不得再抛新异常覆盖原始失败。

## 8 并发、资源与生命周期

- 结构化并发是硬要求：所有挂起调用都必须挂在某个 `CoroutineScope` 的作业树下；禁止 `GlobalScope`，它没有父作业，失败与生命周期都无从约束。
- 并行分解成对使用 `async` 与 `awaitAll`：`coroutineScope { ids.map { async { load(it) } }.awaitAll() }`。丢下未 `await` 的 `Deferred` 会把异常静默丢进 scope。
- `coroutineScope` 任子任务失败即取消全体（快速失败）；`supervisorScope` 让兄弟任务互不牵连（一个失败其余继续）。选择依据是失败是否需要整体放弃。
- 长循环与长计算里插入取消点（`ensureActive()`、`yield()`）；阻塞调用放 `Dispatchers.IO`，CPU 密集放 `Dispatchers.Default`，`Dispatchers.Main` 只做 UI 更新。
- 共享可变状态用 `Mutex.withLock { }`、原子类型或 `Channel`/`Flow`；`@Volatile` 只保证可见性，不保证读-改-写原子性。
- 资源用 `use { }`（`Closeable`/`AutoCloseable`）保证释放；`Flow` 的清理放在 `onCompletion`，`callbackFlow` 必须 `awaitClose`，否则回调式订阅永远不注销。
- 生命周期感知的 scope（Android 的 `viewModelScope`/`lifecycleScope`）优先于自建 scope；自建 scope 必须在 `close()`/`cancel()` 处调用，且写明谁负责关。

## 9 注释与文档

- 公共契约用 KDoc（`/** */`）：做什么、参数、返回、抛出的异常、线程与取消语义；实现细节用普通 `//`。
- 官方不鼓励 `@param`/`@return`：把参数与返回值直接写进主文本，用 `[name]` 链接到参数；只有描述太长、塞不进正文时才用这些 tag。
- 多行 KDoc 的 `/**` 独占一行、后续行以 `*` 开头；单行 KDoc 可压成 `/** ... */`；注释里的可见性修饰与代码一致。
- 注释解释"为什么"：为什么这里必须切到 `Dispatchers.IO`、为什么这个 `catch` 要重抛、为什么这个 `@Suppress` 是安全的。不复述类型——编译器与 IDE 已经提供。
- 抑制警告必须收口：`@Suppress("...")` 只包住真正需要的声明，并在同一处写清原因；文件级抑制等同删除检查。

## 10 工具链与验证

- ktlint 同时提供 lint 与 formatter：`ktlint` 只检查，`ktlint -F`（Gradle 的 `ktlintFormat`）改写文件；CI 只跑检查模式，避免 CI 反向改代码。
- 先钉住 `ktlint_code_style`：换 style 会连带改 `max_line_length`、类签名换行、空行规则等一批默认值，不钉住会让升级 ktlint 时 CI 无故抖动。
- 值得显式配置的键：`max_line_length`、`insert_final_newline`、`ij_kotlin_imports_layout`、`ktlint_chain_method_rule_force_multiline_when_chain_operator_count_greater_or_equal_than`。
- 分工：ktlint 管格式（缩进、空格、换行、import 顺序），detekt 管语义坏味道（长函数、深嵌套、`TooGenericExceptionCaught`、`GlobalScope` 相关规则）；两边都开，不要互相覆盖。
- 命令：`./gradlew ktlintCheck`、`./gradlew ktlintFormat`、`./gradlew detekt`、`./gradlew test`。
- CI 顺序：格式检查 → detekt → 编译（`allWarningsAsErrors = true` 让警告阻断）→ 测试。警告不清零就不算通过。
- 生成代码与 `build/` 目录排除在 ktlint 与 detekt 之外；第三方源码不纳入检查。

## 11 反例（Good / Bad）

```kotlin
// Bad：可变集合接口 + 可变工厂，只读契约形同虚设
fun validate(actualValue: String, allowedValues: HashSet<String>) {
    require(actualValue in allowedValues) { "unsupported value: $actualValue" }
}

val allowedValues = arrayListOf("a", "b", "c")

// Good：接口与工厂都选不可变，调用方无法改穿数据
fun validateImmutable(actualValue: String, allowedValues: Set<String>) {
    require(actualValue in allowedValues) { "unsupported value: $actualValue" }
}

val allowedValuesImmutable = listOf("a", "b", "c")
```

```kotlin
// Bad：GlobalScope 无父作业，catch(Exception) 吞掉取消，失败无人知晓
fun refresh(ids: List<String>) {
    GlobalScope.launch {
        ids.forEach { id ->
            try {
                cache[id] = api.load(id)
            } catch (e: Exception) {
                log.warn("load failed", e)
            }
        }
    }
}

// Good：结构化并发，失败传播到调用方，取消不被吞
suspend fun refreshAll(ids: List<String>) = coroutineScope {
    ids.map { id -> async { id to api.load(id) } }
        .awaitAll()
        .forEach { (id, user) -> cache[id] = user }
}
```

```kotlin
// Bad：!! 把契约失败变成 NPE，堆栈里看不出是哪个 id 出的问题
fun load(id: String): User = repository.find(id)!!.also { log.info("loaded $id") }

// Good：显式失败并带上定位信息
fun loadOrThrow(id: String): User =
    requireNotNull(repository.find(id)) { "user $id not found" }
```

```kotlin
// Bad：表达式体里塞嵌套三元与作用域函数，返回值要看两层才知道
fun label(order: Order) = order.coupon?.let { if (it.active) it.code else "none" } ?: "none"

// Good：把分支摊平成早返回，接收者与返回值都一眼可见
fun formatLabel(order: Order): String {
    val coupon = order.coupon ?: return "none"

    return if (coupon.active) coupon.code else "none"
}
```

## 12 评审清单

- [ ] 文件头顺序正确（file 注解 → `package` → import → 顶层声明），块间恰好一个空行。
- [ ] 无通配符 import，import 列表由 formatter 排序，未手工改动。
- [ ] 变量与属性默认 `val`，`var` 都有明确的重赋值理由。
- [ ] 集合签名用不可变接口（`List`/`Set`/`Map`），没有 `ArrayList`/`HashSet` 泄露到 API。
- [ ] 没有 `!!`；可空处理用 `?:`、早返回或 `requireNotNull`/`checkNotNull`。
- [ ] 每个挂起调用都在有父作业的 scope 内，没有 `GlobalScope`。
- [ ] `catch (e: Exception)` 处显式重抛了 `CancellationException`。
- [ ] `use { }` 覆盖所有 `Closeable`；`callbackFlow` 配了 `awaitClose`。
- [ ] 公共成员有 KDoc，且描述的是契约（参数/返回/异常/取消语义），不是复述签名。
- [ ] 单表达式函数用表达式体，多分支用块体，作用域函数未嵌套超过一层。
- [ ] `./gradlew ktlintCheck` 与 `./gradlew detekt` 均通过，无新增文件级 `@Suppress`。
- [ ] 编译警告已清零（`allWarningsAsErrors`），没有用 `@Suppress` 掩盖的弃用调用。

> 冲突时的裁决顺序：仓库 `.editorconfig`（含 `ktlint_code_style`、`max_line_length`）与工具配置 > 本文 > Kotlin Coding Conventions 与 Android Kotlin Style Guide 原文；协程行为以 kotlinx.coroutines 文档为准。
