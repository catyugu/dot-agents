# PHP — 落地细则

Source: https://www.php-fig.org/per/coding-style/
配套：PHP_CodeSniffer（`phpcs` / `phpcbf`）+ PHP-CS-Fixer + PHPStan 或 Psalm；先以仓库 `phpcs.xml`、`.php-cs-fixer.php`、`phpstan.neon` 为准。

## 1 适用范围与优先级

- 覆盖 `.php` 源文件与模板文件里的 PHP 区块；`vendor/`、`storage/` 与生成代码不检查、不评审。
- 规范层级：PHP-FIG 的 PER Coding Style（当前版本**取代并扩展** PSR-12，同时强制遵守 PSR-1）→ 框架自身风格（Laravel、Symfony 各有细则）→ 仓库的 ruleset 与静态分析配置。
- PHP-FIG 是社区互操作组织，不是 PHP 语言的官方标准机构：PER/PSR 只约束代码风格与自动加载，语言语义一律以 php.net 手册为准，两者冲突时手册优先。
- 工具默认值有约束力：PHP_CodeSniffer 在没有 `--standard` 且找不到 `.phpcs.xml` 时按 PSR12 检查，所以"没配就是 PSR-12"是真实行为，不是约定。
- 新项目按 PER 写；存量项目继续跑 PSR-12 也合法，但同一仓库内不得混用两套规则的换行与逗号约定。

## 2 缩进、行宽与排版

- 4 空格缩进，禁止制表符（PER 与 PSR-12 一致）。
- 行宽：**不设硬上限**；软上限 120 字符（超出即违规边界），并且行应不超过 80 字符、更长就拆成每行 ≤80 的多行。实践上把 phpcs 的 120 当门禁、80 当目标。
- 大括号分两类：类、接口、trait、enum、函数、方法、闭包的开括号**独占一行**；控制结构（`if`/`for`/`while`/`switch`/`try`）的开括号**与语句同行**。混用会被 phpcs 逐行报错。
- 控制结构必须有空格：`if (`、`) {`、`)` 前无空格；`elseif` 写成单个词，不写 `else if`。
- 声明关键字顺序固定：`abstract`/`final` → 可见性 → `static` → `readonly` → 类型 → 名字；全小写、单空格分隔。
- 尾随逗号是硬规则：单行列表**不得**有，跨多行列表**必须**有（函数参数、数组、`match` 分支、`use` 列表）。
- 运算符两侧至少一个空格；多行表达式把运算符放在**行首**；三元表达式必须占三行，不能压成两行。
- 文件用 LF 换行、结尾单个 LF、行末无尾随空格；纯 PHP 文件不写结束标签 `?>`，避免其后的空白被输出到响应体。
- 类型语法细节：`?T` 与 `T|null` 等价但同一个仓库只用一种；复合类型（union/intersection）的 `|` 与 `&` 两侧不留空格；`&` 引用符后不留空格。

## 3 命名

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| 命名空间段 | PascalCase，1:1 映射目录（PSR-4） | `Vendor\Package\Sub` |
| 类、接口、trait、enum | PascalCase（PSR-1 的 StudlyCaps） | `OrderRepository`、`PriceCalculator` |
| 方法 | camelCase（PSR-1 强制） | `findBySku()`、`isExpired()` |
| 函数（非方法） | 同作用域内统一，常用 snake_case（社区惯例，非官方规则） | `str_contains()`、`normalize_path()` |
| 类常量 | UPPER_SNAKE_CASE（PSR-1 强制） | `const VERSION = '1.0';` |
| 枚举 case | PascalCase（PER 规定） | `case Hearts = 'H';` |
| 属性、局部变量 | camelCase，且同一作用域内一致 | `$orderTotal`、`$retryCount` |
| 布尔方法/变量 | `is`/`has`/`can`/`should` 前缀（社区惯例，非官方规则） | `isExpired()`、`$hasDiscount` |
| 缩写与首字母缩写 | 当普通单词：首字母大写、其余小写 | `HttpClient`、`XmlParser` |
| 文件名 | 与主类型同名 + `.php` | `OrderRepository.php` |

- 禁止用单下划线前缀表示 protected/private：PER 明确"下划线前缀没有含义"，写了会被读成遗留约定。
- 属性命名 PSR-1 故意不作规定，但同一作用域内必须一致；混用 `$snake_case` 与 `$camelCase` 在同仓库视为违规。

## 4 文件与模块组织

- 文件头块顺序固定，块之间恰好一个空行、块内不留空行：`<?php` → 文件级 docblock → `declare` → `namespace` → 类 `use` → 函数 `use` → 常量 `use` → 其余代码。
- `use` 不得以反斜杠开头（导入名总从命名空间根解析）；组合导入组内不得超过两层子命名空间。
- 一个文件只声明符号**或**只产生副作用，两者不混（PSR-1）：`ini_set`、`include`、输出、连接外部服务都算副作用。
- 一个文件一个公开类/接口/trait/enum，文件名与类名一致、目录与命名空间一致（PSR-4），否则自动加载与静态分析都会错位。
- `declare(strict_types=1);` 紧跟在 `<?php`（或文件级 docblock）之后、其余代码之前，逐文件书写，新文件一律带上。
- 目录按领域或层组织，不建 `Utils/`、`Helpers/` 大杂烩；一个文件放多个类只在辅助类型确实只服务本文件时成立。
- composer 的 `autoload` 段用 PSR-4 映射到 `src/`，测试用 `autoload-dev` 映射到 `tests/`；类名与路径的偏差靠脚本而非人工维护。

## 5 类型、值与不可变性

- `declare(strict_types=1);` 每个文件都写。理由是 strict 模式**按发起调用的文件**生效：漏写的文件调用严格文件里的函数时，仍然按宽松模式做隐式转换，一半严格等于没有严格。
- 类型声明能写就写：参数、返回、属性、类常量。静态分析的正确率完全取决于这些声明；缺类型等于放弃对应检查。
- 返回语义写准：无返回值用 `void`，永远抛出或终止用 `never`；返回 `static` 用于链式不可变 API。
- 可空用 `?T`；不要把 `null` 作为默认值来制造可空参数（子类改默认值时会触发类型兼容错误，手册明确不推荐）。
- 比较一律用 `===` / `!==`；不用 `==` 依赖隐式转换，也不用 `in_array()` 的宽松模式（第三参数传 `true`）。
- 值对象用 `final readonly class`，领域枚举用 backed `enum` 给显式标量；结构稳定但无类型的数据不要用裸数组承载。
- 数组用短语法 `[]`；数组形状写进 PHPDoc（`array{id: int, email: string}`、`list<string>`），让 PHPStan 能验证键与值的类型。
- 不使用 `mixed`：它让高等级检查全线失效；确实需要泛型时用 `@template`，需要异构集合时用接口或联合类型。

## 6 函数与 API 设计

- 参数顺序：必填在前、有默认值的在后（PER 硬规则）；参数超过 3~4 个，或相邻参数同为标量类型，用命名实参（`create(name: $name, email: $email)`）或引入 DTO。
- 命名实参写法固定：冒号紧跟参数名、冒号后一个空格（`somefunction($a, b: $b)`）。
- 可见性显式写在所有属性、常量、方法上；`var` 关键字不得用于声明属性（PER 明确禁止）。
- 默认 `final`（类与属性一起用，明确不可继承、不可变）；确实需要扩展点时才留抽象类或接口，且把扩展契约写进 PHPDoc。
- 闭包：单表达式用箭头函数 `fn($x) => $x * 2`；需要外部变量时用 `use (...)` 显式捕获，不要靠 `global` 或超全局变量隐式取。
- 引用传参只在必须写回时用 `&$param`；注意引用参数的类型只在进入时检查一次，返回时不再检查（手册明示），因此别在函数体里改变量类型。
- `new Foo()` 一律带括号，即使没有参数；`exit()`/`die()`/`clone()` 也带括号，避免与同名常量混淆。
- 链式调用折行时，`->` 放行首并与后续链对齐；第一段方法也要换到新行，不要留在赋值号后面。

## 7 错误处理

- 可预期失败抛具体异常（`InvalidArgumentException`、领域异常），不用 `false`/`null`/`0` 混合表示失败原因。
- 禁用 `@` 抑制符、`error_reporting(0)`、空 `catch {}`；它们把故障变成静默的默认值，排障时没有入口。
- 只捕获能处理或能补上下文的异常；捕获后要么重抛并带上 `$previous`，要么返回明确的结果对象并记录原因。
- 边界处理用 `Throwable`，业务分支用 `Exception`：现代 PHP 的 `Error`（类型错误、未定义函数）与 `Exception` 都实现 `Throwable`，只 `catch (Exception)` 会漏掉引擎级错误；`catch (Throwable)` 只出现在入口、中间件、队列 worker 这类最外层。
- 抛什么异常写进 PHPDoc 的 `@throws`，PHPStan 会顺着它检查是否漏捕获；不写等于让调用方猜。
- 异常消息与日志里不放密码、token、身份证/邮箱等个人数据；需要上下文时用 ID 而不是原文。
- 事务与资源在异常路径的收尾见第 8 节；`catch` 里不得把异常转成成功返回值。

## 8 并发、资源与生命周期

- 默认模型是"每请求一个进程、请求间不共享内存"：请求作用域的状态放局部变量或容器，不放 `static`、不放全局变量。
- 长生命周期进程（常驻 worker、Swoole/RoadRunner 类运行时）里，`static` 缓存与单例会跨请求存活；必须写明是"进程级缓存"并处理失效，否则就是内存泄漏与脏数据源。
- 资源用 `try/finally` 或库提供的作用域 API 释放：`fopen` 配 `fclose`、锁配 `unlock`、游标配 `close`，都放在 `finally`。
- 数据库事务写成 `begin` 在 `try` 之外、`commit` 在 `try` 末尾、`rollBack` 在 `catch` 里并重抛；漏掉异常路径的回滚会让连接带着未提交事务回到池里。
- 并发原语取决于运行时：Fiber 是协作式切换，不能当并行加速用；Swoole 协程与 `pcntl_fork` 各有独立的共享内存规则。不要假设"全局变量在并发下安全"。
- 队列 worker 必须幂等：投递语义是至少一次，同一个 job 会重跑；外部副作用（发信、扣款）要有幂等键。
- 连接池/客户端（PDO、Redis、HTTP client）按进程复用，不要在循环里反复 `new`；也不要跨请求复用带状态的连接。

## 9 注释与文档

- 公共类与公共方法写 PHPDoc：摘要、`@param`、`@return`、`@throws`；泛型与数组形状用 `@template`、`array{...}`、`list<...>`。
- 签名已经表达的类型不要在 PHPDoc 里重复叙述；PHPDoc 只补签名写不出的信息（泛型参数、数组形状、不可变契约、`@deprecated` 的替代物）。
- 文件级 docblock 只放版权与许可，类说明放类的 docblock，两者不要混在一处。
- 注释解释"为什么"：为什么这里要 `REPEATABLE READ`、为什么这个分支必须重抛、为什么这段代码看起来多余但拆不得。
- `TODO`/`FIXME` 必须带 issue 编号或负责人；无主的 TODO 会被当成已完成项，评审时按缺陷处理。

## 10 工具链与验证

- `phpcs` 默认标准是 PSR12：无 ruleset 时按 PSR-12 检查；项目自定义在 `phpcs.xml` 里 `rule ref="PSR12"` 叠加或收紧。
- `phpcbf` 自动修可修项；`phpcs -s` 输出 sniff code（如 `Squiz.WhiteSpace.SuperfluousWhitespace`），用它写行级 ignore 比整条规则关掉精确得多。
- PHP-CS-Fixer 与 `phpcs` 二选一负责格式：两套 ruleset 有重叠，同时跑会互相改回去。PHP-CS-Fixer 用 `@PSR12` 或 `@PER-CS` 规则集，并在 CI 里跑 `--dry-run --diff`。
- PHPStan 分级推进：`vendor/bin/phpstan analyse -l 6 src tests` 起步，新项目直接 `--level max`；等级从 0 起逐级收紧，`--level max` 永远取当前工具支持的最高级，升级不降级。
- 存量迁移用基线（`includes: [phpstan-baseline.neon]`），但基线只许变小：新增代码不得进基线，否则等于把错误冻结成规范。
- 命令：`vendor/bin/phpcs`、`vendor/bin/phpcbf`、`vendor/bin/phpstan analyse`、`vendor/bin/phpunit`、`composer validate --strict`。
- CI 顺序：`phpcs`（只检查）→ `phpstan` → 测试；格式修复在本地用 `phpcbf` 完成，CI 不自动改代码。
- 分析范围排除 `vendor/`、`storage/`、缓存与生成代码；第三方代码的问题不归本项目修。

## 11 反例（Good / Bad）

```php
<?php

// Bad：无 strict_types、松散比较、返回 false 表示失败
function parseLimit($raw)
{
    if ($raw == '') {
        return 0;
    }

    return $raw;
}
```

```php
<?php

declare(strict_types=1);

// Good：严格模式 + 声明类型 + 明确异常
function parseLimit(string $raw): int
{
    $limit = filter_var($raw, FILTER_VALIDATE_INT);
    if ($limit === false || $limit < 1) {
        throw new InvalidArgumentException(
            sprintf('limit must be a positive integer, got "%s"', $raw),
        );
    }

    return $limit;
}
```

```php
<?php

declare(strict_types=1);

// Bad：异常路径漏回滚，句柄靠 GC 收尾
function importCsv(PDO $pdo, string $path): void
{
    $handle = fopen($path, 'rb');
    $pdo->beginTransaction();
    while (($line = fgets($handle)) !== false) {
        insertRow($pdo, $line);
    }
    $pdo->commit();
    fclose($handle);
}

// Good：回滚与关闭都覆盖异常路径，异常补上下文后重抛
function importCsvSafe(PDO $pdo, string $path): void
{
    $handle = fopen($path, 'rb');
    if ($handle === false) {
        throw new RuntimeException(sprintf('cannot open %s', $path));
    }

    $pdo->beginTransaction();
    try {
        while (($line = fgets($handle)) !== false) {
            insertRow($pdo, $line);
        }
        $pdo->commit();
    } catch (Throwable $e) {
        $pdo->rollBack();

        throw new RuntimeException(sprintf('import failed for %s', $path), 0, $e);
    } finally {
        fclose($handle);
    }
}

function insertRow(PDO $pdo, string $line): void
{
    $pdo->prepare('INSERT INTO import_rows (payload) VALUES (:payload)')
        ->execute(['payload' => $line]);
}
```

```php
<?php

declare(strict_types=1);

// Bad：裸 array 让静态分析无法推理，键名靠猜
/** @return array */
function loadUsers(PDO $pdo): array
{
    return $pdo->query('SELECT id, email FROM users')->fetchAll();
}

// Good：写出返回形状，调用方与静态分析都拿到契约
/** @return list<array{id: int, email: string}> */
function loadUsersTyped(PDO $pdo): array
{
    /** @var list<array{id: int, email: string}> $rows */
    $rows = $pdo->query('SELECT id, email FROM users')->fetchAll(PDO::FETCH_ASSOC);

    return $rows;
}
```

## 12 评审清单

- [ ] 每个新文件都有 `declare(strict_types=1);`，且位于其余代码之前。
- [ ] 参数、返回、属性、类常量都写了类型声明，没有 `mixed`。
- [ ] 所有比较用 `===`/`!==`，没有松散的 `==` 与未开严格模式的 `in_array()`。
- [ ] 单行列表无尾随逗号，多行列表有尾随逗号。
- [ ] 类与函数开括号独占一行，控制结构开括号与语句同行。
- [ ] 无 `@` 抑制符、无空 `catch {}`；捕获后要么处理、要么带 `$previous` 重抛。
- [ ] 无 `var` 声明属性，所有属性/常量/方法都显式写了可见性。
- [ ] 资源释放与事务回滚在 `finally`/`catch` 路径上，`begin` 在 `try` 之外。
- [ ] 没有跨请求的 `static`/全局可变状态；长生命周期进程里的缓存写了失效策略。
- [ ] 公共 API 有 PHPDoc，含泛型或数组形状，且 `@throws` 与实际抛出相符。
- [ ] `phpcs`（只检查模式）与 `phpstan`（不降级、基线未增长）在 CI 通过。
- [ ] 文件用 LF、结尾单个 LF、纯 PHP 文件无 `?>`、行末无尾随空格。

> 冲突时的裁决顺序：仓库 `phpcs.xml` / `.php-cs-fixer.php` / `phpstan.neon` > 本文 > PER Coding Style（取代 PSR-12）> PSR-1；语言语义一律以 php.net 手册为准。
