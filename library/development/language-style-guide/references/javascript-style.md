# JavaScript / TypeScript（Prettier + ESLint + strict TS）— 落地细则

Source: https://google.github.io/styleguide/tsguide.html 、https://google.github.io/styleguide/jsguide.html 、https://www.typescriptlang.org/tsconfig
配套：`prettier`（格式唯一裁决者）+ `eslint`（含 typescript-eslint）+ `tsc --noEmit`；先以仓库 `.prettierrc`、`eslint.config.js`、`tsconfig.json` 为准。

## 1 适用范围与优先级

- ECMAScript 只规定语义，不规定缩进、引号、换行；**格式一律交给 Prettier**，不要手排、不要在 ESLint 里配 stylistic 规则（ESLint 已废弃格式化规则）。
- TypeScript 的类型系统由 TS 规范与 `tsconfig` 决定；"类型安全"靠编译器选项，不靠约定。
- Google JS/TS Style Guide 与 Airbnb 都是**社区/公司规范**，可作参考但不能当语言要求；二者冲突处以仓库配置为准。
- 优先级：仓库 Prettier/ESLint/tsconfig > 本文件 > Google TS Style Guide > Airbnb。
- 语义来源：ECMAScript 规范（https://tc39.es/ecma262/）、MDN、TypeScript Handbook。

## 2 缩进、行宽与排版

- 缩进 2 空格（社区与 Prettier 默认），禁 Tab。
- 行宽由 Prettier `printWidth` 决定（默认 80）；改宽窄改配置，不改代码。长表达式自动重排，不要手工对齐参数。
- 分号：要么统一加分号，要么统一不加（Prettier `semi`）。同仓只能一种，别靠"ASI 恰好成立"。
- 引号：`singleQuote: true` 或 `false` 二选一，全仓一致；模板字符串只用于插值或多行文本，不用于普通字符串。
- 尾随逗号 `trailingComma: "all"`（Prettier 默认），多行对象/数组/参数保留尾逗号，便于 diff。
- 箭头函数参数括号 `arrowParens: "always"`（默认）；对象字面量属性的引号仅在必要时加。
- 大括号：单语句块也加括号；`else`/`catch` 与前一个 `}` 同行。
- 空行用来分段：import 之后一行、逻辑段落之间一行；不要连续两行以上空行。

## 3 命名

| 对象                          | 规则                                  | 示例                                  |
| ----------------------------- | ------------------------------------- | ------------------------------------- |
| 变量 / 函数 / 参数 / 属性     | `lowerCamelCase`                      | `itemCount`, `loadUser()`             |
| 类 / 接口 / 类型别名 / 枚举   | `UpperCamelCase`                      | `class HttpClient`, `type UserId`     |
| 模块级真常量                  | `UPPER_SNAKE_CASE`（或项目统一 camel）| `MAX_RETRY`, `DEFAULT_TIMEOUT_MS`     |
| 布尔                          | `is/has/can/should` 前缀              | `isReady`, `hasAccess`                |
| 事件处理函数                  | `handleX`；事件回调属性 `onX`         | `handleSubmit`, `onClick`             |
| 私有类成员                    | `#field`（原生私有）                  | `#cache`                              |
| 文件名                        | `kebab-case.ts` 或框架约定 `PascalCase.tsx` | `user-profile.ts`, `UserCard.tsx` |
| React 组件 / hooks            | 组件 `UpperCamelCase`，hook `useX`    | `UserCard`, `useUser()`               |

- 不要 `I` 前缀（`IUser`）、不要 `T` 前缀类型别名；接口与类型别名都不加前缀。
- 名字带单位与语义（`timeoutMs`、`sizeBytes`）；不用 `data`、`obj`、`temp`、`res` 这类无信息名，除极短作用域的回调参数。
- 单字母仅用于极短箭头函数（`xs.map(x => x.id)`）；不要为省字丢字母（`usr`、`btn`）。
- 缩写按英文习惯大小写：`userId`、`htmlParser`、`urlPath`（不用 `userID`、`HTMLParser`）。

## 4 模块组织

- 前端与 Node 新代码统一 ESM（`import`/`export`）；CommonJS 只在工具链要求时用，且不混用两种模块系统。
- import 顺序（由 ESLint `import/order` 或简单分组落地，段间空行）：Node 内建（`node:` 前缀）→ 第三方 → 内部绝对路径别名 → 相对路径 → 类型导入。
- 类型专用导入写 `import type { User } from "./types"`，并开 `verbatimModuleSyntax`，避免类型被当成运行时导入。
- 不写无扩展名的深层相对路径穿透（`../../../..`），改用 tsconfig `paths` 别名。
- `index.ts` 桶文件只用于包的公开入口；内部目录不要层层 barrel（会造成循环依赖与打包膨胀）。
- 循环依赖是结构问题：把共享类型/常量下沉到叶子模块，不要用延迟 import 或动态 `require` 掩盖。
- 一个文件一个主要导出；工具函数按领域命名文件，禁止 `utils.ts` 无限膨胀。

## 5 类型系统（TypeScript）

- `strict: true` 是底线，另外建议开启：`noUncheckedIndexedAccess`、`exactOptionalPropertyTypes`、`noImplicitOverride`、`noFallthroughCasesInSwitch`、`noUnusedLocals`。
- 禁止 `any`；未知输入用 `unknown` 并在边界收窄（类型守卫 `function isUser(v: unknown): v is User`）。
- 用 `satisfies` 校验字面量满足类型而保留窄化推断；`as` 断言只用于确实比运行时检查更可靠的场景，并注释依据。
- 判别联合表达状态机：`type State = { kind: "idle" } | { kind: "loading" } | { kind: "error"; error: Error }`，`switch` 后加穷尽检查（`const _never: never = state`）。
- `interface` 用于可扩展的对象契约，`type` 用于联合/交叉/工具类型；不要为同一形状同时存在两种。
- 不用 `enum`（运行时代码 + 与 `const` 断言冲突），改用 `as const` 对象 + 派生联合类型。
- 集合默认 `readonly T[]`/`ReadonlyArray<T>` 与 `readonly` 属性；需要可变时在边界处复制。
- 泛型要窄：`<T extends { id: string }>` 而不是无约束 `<T>`；返回类型显式写，不靠推断泄漏内部结构。
- 品牌类型隔离 ID：`type UserId = string & { readonly __brand: "UserId" }`，避免把 `OrderId` 传给用户接口。
- 外部数据（HTTP、localStorage、环境变量）在边界做运行时校验（zod/valibot/手写守卫），校验结果才是可信类型。

## 6 函数与 API 设计

- 公开函数显式写返回类型（否则实现改动会悄悄改变契约）；内部短函数可依赖推断。
- 参数超过 3 个用对象参数（`function createUser({ name, email }: CreateUserInput)`），便于扩展与命名调用。
- 默认值用参数默认值或解构默认值，不用 `x = x || fallback`（会把 `0`/`""` 吃掉）；需要区分 `undefined` 时用 `x ?? fallback`。
- 可选属性 `?` 与 `| undefined` 语义不同（`exactOptionalPropertyTypes` 下尤其明显），同一接口内不要混用表示同一含义。
- 函数式优先：纯函数 + 不可变数据；需要状态时用闭包或显式类，不滥用类继承。
- 避免重载签名；用联合类型 + 收窄，或用两个命名清楚的函数。
- 异步函数返回 `Promise<T>`，不要返回 `Promise<T> | T`；不要在 async 函数里同时用回调风格。
- 导出的 API 在 JSDoc 里写清失败、取消、副作用与线程/并发假设（浏览器单线程但共享可变状态仍会踩）。

## 7 错误处理

- 抛 `Error` 子类（`class UserLoadError extends Error`）并设置 `name`；不要抛字符串或普通对象。
- `catch (e: unknown)` 后用 `instanceof Error` 收窄；不加注解的 `catch (e)` 在 TS 下是 `any`/`unknown` 混用，必须显式。
- 保留原因链：`new Error("load user failed", { cause: err })`。
- 禁止空 `catch {}`；确实可忽略时写注释说明为什么（如埋点失败不影响主流程）。
- Promise 的拒绝路径必须处理：`await` 放 `try`，或用 `.catch()`；`floating promise` 用 ESLint `no-floating-promises` 拦截。
- 不用异常做常规控制流；"没有值"用 `undefined`/`null` 或结果联合类型（`{ ok: true; value: T } | { ok: false; error: E }`），并在模块内统一一种。

## 8 并发、资源与副作用

- `async`/`await` 优先；不要在 `forEach`/`map` 里跑 async 回调（`return` 的 promise 被丢弃），改用 `for...of` 顺序执行或 `Promise.all` 并行。
- `Promise.all` 首错即停；需要每项结果时用 `Promise.allSettled`，并显式处理拒绝项——不要吞掉。
- 并发写共享状态要串行化（队列/锁/单写者）；前端避免多个请求回写同一份 store 造成竞态。
- 取消用 `AbortController`/`AbortSignal` 传到 fetch 与自定义异步函数；组件卸载/请求超时要真的 abort。
- 定时器、事件监听、订阅必须有对称清理（`clearInterval`、`removeEventListener`、`unsubscribe`），或使用框架提供的生命周期。
- 支持显式资源管理（`using`）的工具链下，优先 `using` 替代手写 `try/finally`。
- 浏览器与 Node 差异（`document`、`process`、`Buffer`）只在边界层出现；共享代码不写环境判断散落各处。
- 避免模块级可变单例状态（测试会互相污染）；需要时通过依赖注入传入。

## 9 注释与文档

- 公开 API 用 JSDoc/TSDoc：`@param`、`@returns`、`@throws`、`@deprecated`、`@example`；类型已写在签名里就不必在注释重复。
- 注释讲 why：为什么这里需要 `as`、为什么这个请求要串行、为什么忽略这个 lint。
- 魔法数字与正则必须具名常量并注释来源/依据。
- `TODO(name): 动作` 带责任人与触发条件；不使用无主 `TODO` 与 `FIXME` 洪水。
- `@ts-expect-error` 优于 `@ts-ignore`（它会在此行不再报错时反过来报错），并必须在同一行/上一行写原因。
- README/示例要与当前类型定义同步；示例代码应可编译。

## 10 工具链与验证

- 格式：`prettier --check .`（CI），不要用 ESLint 管格式。
- 静态检查：`eslint . --max-warnings=0`；typescript-eslint 的 `recommended-type-checked` 起步，`no-floating-promises`、`no-misused-promises` 必须开。
- 类型：`tsc --noEmit`（或项目 `build` 的 typecheck 目标）作为独立门禁，不依赖打包器偶尔的类型检查。
- 测试：`vitest`/`jest` + 类型级测试（`expectTypeOf`/`tsd`）对公开 API 的形状做断言。
- 依赖：锁文件入库；`npm audit`/`pnpm audit` 按项目政策，不要自动大版本升级。
- 只认工具输出：报告真实命令与结果；"本地能跑"不等于 CI 通过。

## 11 反例（Good / Bad）

```ts
// Bad：any、吞错、丢弃 promise、隐式字符串拼接 URL
async function load(id: any) {
  try {
    return await fetch("/users/" + id).then((r) => r.json());
  } catch (_) {
    return null;
  }
}

// Good：unknown 收窄、错误带状态与原因、编码参数、显式返回类型
async function loadUser(id: UserId, signal: AbortSignal): Promise<User> {
  const res = await fetch(`/users/${encodeURIComponent(id)}`, { signal });
  if (!res.ok) throw new UserLoadError(id, res.status);
  const body: unknown = await res.json();
  return parseUser(body);
}
```

```ts
// Bad：enum、无约束泛型、断言掩盖问题
enum Status { Idle, Loading, Done }
function pick<T>(items: T[], key: string): T { return items[0] as T; }

// Good：as const 联合、窄约束、显式可能缺失
const STATUS = { idle: "idle", loading: "loading", done: "done" } as const;
type Status = (typeof STATUS)[keyof typeof STATUS];
function first<T extends { id: string }>(items: readonly T[]): T | undefined {
  return items[0];
}
```

```ts
// Bad：forEach + async，返回的 promise 被丢弃
items.forEach(async (item) => { await save(item); });

// Good：明确顺序或明确并行，失败会传播
for (const item of items) await save(item);
await Promise.all(items.map((item) => save(item)));
```

## 12 评审清单

- [ ] Prettier 决定格式，分号/引号/尾逗号全仓一致
- [ ] 命名符合 §3，无 `I`/`T` 前缀，布尔用 `is/has/can/should`
- [ ] import 分组、类型导入用 `import type`，无深层相对路径穿透与循环依赖
- [ ] `strict` 全开，无 `any`，外部输入在边界运行时校验后收窄
- [ ] 判别联合 + 穷尽检查表达状态；集合与属性默认可变/只读选择明确
- [ ] 公开函数有显式返回类型，多参数用对象，`??` 不用 `||` 处理默认值
- [ ] 无空 catch，错误是 `Error` 子类且有 `cause`；无 floating promise
- [ ] 无 `forEach + async`；取消用 `AbortSignal`；监听器/定时器有对称清理
- [ ] `@ts-expect-error` 带原因，`as` 有依据，无 lint 全局禁用
- [ ] `prettier --check`、`eslint --max-warnings=0`、`tsc --noEmit`、测试全绿（真实运行）

> 冲突时裁决顺序：仓库 Prettier/ESLint/tsconfig > 本文件 > Google TS/JS Style Guide > Airbnb；格式争议一律回到 Prettier 输出。
