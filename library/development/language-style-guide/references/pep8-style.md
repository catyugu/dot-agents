# Python（PEP 8 + 类型注解 + 工具链）— 落地细则

Source: https://peps.python.org/pep-0008/ ，配套 https://peps.python.org/pep-0484/ 、https://google.github.io/styleguide/pyguide.html
配套：`ruff format`（或 `black`）+ `ruff check` + `mypy`/`pyright`；先以仓库 `pyproject.toml`、`.editorconfig` 为准。

## 1 适用范围与优先级

- PEP 8 是**风格建议**，原文明确"规则可以被知情地打破"；但"可读性"不是任意例外的通行证，例外要写理由。
- 格式化没有争议空间：`black`/`ruff format` 的输出即标准；手写排版与工具打架时改配置，不改代码。
- 类型检查（PEP 484 起）与风格是两个维度，都要满足；`from __future__ import annotations` 行为按项目 Python 版本决定。
- 优先级：仓库 `pyproject.toml`/`.editorconfig`/`pre-commit` 配置 > 本文件 > PEP 8 > Google Python Style Guide（仅在其更严格且不冲突时参考）。
- 语义来源是 Python 语言参考与标准库文档；第三方风格文章不作依据。

## 2 缩进、行宽与排版

- 一级缩进 4 空格，**禁用 Tab**（混用会直接 `TabError`）。
- 行宽：PEP 8 默认 79；`black` 默认 88；现代项目常在 88~100。以 formatter 配置为准，别在同一仓库两种限长。
- 括号内隐式续行优先；反斜杠续行只在无法加括号时用（`with` 多上下文、长字符串拼接除外）。
- 运算符换行时放在**行首**（`black` 风格），与被断开的操作数对齐关系由工具决定。
- 悬挂缩进要与被缩进内容有区分（多一层），否则视觉上像同级语句。
- 空行：顶层函数/类之间 2 个空行，类内方法之间 1 个；函数内部用单个空行分隔逻辑段，不要堆空行。
- 不在行尾写分号，不在 `if`/`while`/`def` 后写单行语句体（`if cond: do()` 不可读且与调试器不友好）。
- 文件以单个换行结尾，UTF-8 编码，不加 `# -*- coding: utf-8 -*-`（默认已是 UTF-8）。
- 尾随逗号在多行容器/调用中一律保留（`black` 的 magic trailing comma 决定是否展开），便于 diff。

## 3 命名

| 对象                    | 规则                    | 示例                                     |
| ----------------------- | ----------------------- | ---------------------------------------- |
| 包 / 模块               | `lower_with_underscore` | `my_package`, `user_store.py`            |
| 类 / 异常 / 类型别名    | `CapWords`              | `HTMLElement`, `class NotFoundError`     |
| 函数 / 方法 / 变量      | `lower_with_underscore` | `get_data()`, `item_count`               |
| 常量                    | `UPPER_SNAKE_CASE`      | `MAX_TIMEOUT_S`                          |
| 受保护                  | `_single_leading`       | `_internal()`, `self._cache`             |
| 私有（name mangling）   | `__double_leading`      | `self.__secret`                          |
| 魔术方法                | `__dunder__`            | `__init__`, `__len__`                    |
| 类型变量                | `CapWords` 或短大写     | `T = TypeVar("T")`, `K`, `V`             |
| 泛型参数（PEP 695 语法）| `CapWords`              | `def first[T](xs: list[T]) -> T: ...`    |

- 布尔/谓词用 `is_`/`has_`/`can_`/`should_`；异常类名以 `Error` 结尾。
- 名字带单位（`timeout_s`、`size_bytes`）；不用 `list`/`dict`/`str`/`type` 等内建名，也不用 `l`、`O`、`I` 这类易混单字母。
- 单字母只用于推导式与极短作用域的索引（`for i, x in enumerate(xs)`）。
- 模块级"常量"若语义上不是常量（可配置、可注入），用小写名字并在文档说明，别硬写成 `UPPER`。

## 4 import 与模块组织

- 每行一个 import；分组顺序（段间空行）：`__future__` → 标准库 → 第三方 → 本项目/本地，段内字母序。
- 绝对导入优先（`from mypkg.util import parse`）；相对导入仅在包内部且项目风格要求时使用，避免 `from . import *`。
- 不使用通配导入（`from x import *`），它会静默覆盖名字且让静态检查失效。
- `__all__` 显式声明公开接口；未在 `__all__` 且以下划线开头的名字视为内部。
- `if __name__ == "__main__":` 只放入口调用；可执行逻辑写成 `main()` 便于测试。
- 避免循环导入：把共享抽象下沉到更低层模块，或用 `TYPE_CHECKING` 下的类型导入（`if TYPE_CHECKING: from .models import User`），而不是函数内 import 遮掩结构问题。
- 包结构按职责划分，禁止无边界的 `utils.py`/`helpers.py` 倾倒文件。

## 5 类型注解

- 公开函数与公开属性写注解；局部变量可依赖推断，只在推断不成立或需要固定类型时标注。
- 用现代拼写：`list[int]`、`dict[str, float]`（PEP 585）而非 `List[int]`；可选类型用 `X | None`（PEP 604）而非 `Optional[X]`。
- 结构化子类型用 `typing.Protocol`（`class Reader(Protocol): def read(self, n: int) -> bytes: ...`），避免为测试硬造基类。
- 字典形状固定时用 `TypedDict`；有限取值用 `Literal`；只赋值一次的模块常量用 `Final`。
- `Any` 只出现在与无类型第三方库的边界，并尽量在此后立刻收窄；不要用 `Any` 图省事。
- `cast()` 只用于类型系统无法表达但运行期确定的情形，并注释依据；不要用它掩盖真实类型错误。
- 禁止用可变默认值（`def f(xs=[])`）——用 `None` 哨兵或 `field(default_factory=list)`。
- 类型检查器用 `mypy --strict` 或 `pyright` 严格模式；`# type: ignore[代码]` 必须写具体错误码并注释原因。

## 6 函数与 API 设计

- 函数职责单一；参数超过 4~5 个改用 `dataclass`/`NamedTuple` 或关键字参数（`*` 之后的 keyword-only），调用点可读性优先。
- 用返回值而不是修改传入的可变参数；确需输出时命名 `out_*` 并写文档。
- 默认参数值必须不可变；`None` 表示"未提供"时在函数开头归一化。
- 生成器用于流式处理大数据；一次性小集合直接返回 `list`。
- 资源使用 `with` 与 `contextlib.contextmanager`；不要手写 `try/finally` 关文件。
- 公开 API 用 `@dataclass(frozen=True)` 或 `NamedTuple` 表达值对象；可变状态封装在类内并显式提供方法。
- 避免"返回 `None` 与抛出异常"混用来表达同一种失败；在文档字符串里固定一种契约。
- `@property` 只用于无副作用、廉价的属性访问；耗时操作写成方法名（`fetch_*`/`compute_*`）。

## 7 错误处理

- 只捕获能处理的异常，且尽量具体（`except (OSError, ValueError) as exc`）；禁止裸 `except:` 与 `except Exception: pass`。
- 重抛时保留原因链：`raise ConfigError(path) from exc`；不要在 `except` 里 `raise` 一个新的无上下文异常。
- 自定义异常继承 `Exception`，放在模块级，命名以 `Error` 结尾；不继承 `BaseException`，不定义空异常当控制流。
- EAFP 优先（`try: v = d[k] except KeyError: ...`），但简单可读的 `d.get(k, default)` 更清楚时用后者。
- `assert` 只用于内部不变量与测试，不用于校验外部输入（`-O` 下被移除）。
- 不用异常做常规分支；`StopIteration`、`KeyError` 不要作为正常流程的跳转手段。
- 在进程边界（CLI 入口、worker）捕获顶层异常并记录 `exc_info`，然后以非零码退出。

## 8 并发与资源

- 明确并发模型：I/O 密集用 `asyncio` 或 `threading`，CPU 密集用 `multiprocessing`；GIL 不保证复合操作原子性。
- `asyncio`：`create_task` 的返回值要保留引用，否则任务可能被 GC；`asyncio.TaskGroup`（或 `gather`）里区分"首错即停"和"等全部完成"。
- 所有阻塞调用不能出现在事件循环里；CPU/阻塞 I/O 用 `asyncio.to_thread`/`run_in_executor`。
- 线程间共享状态用 `queue.Queue` 或显式锁（`threading.Lock`），并保持锁内不调用可能阻塞的外部代码。
- 超时一律显式传递（`asyncio.timeout`、`socket.settimeout`、HTTP 客户端 timeout），不要依赖默认无限等待。
- 资源清理用 `with`/`async with`；进程池、会话、客户端在应用关闭时显式释放。

## 9 注释与文档

- 块注释 `# ` 后一个空格，与代码同缩进；行内注释与语句至少隔两个空格。
- 注释讲 why：约束、权衡、协议、性能、平台差异；不复述代码。
- 文档字符串用 `"""`，首行一句话摘要（句号结尾），空行后写细节；公开 API 写明参数含义、返回、异常、副作用。
- 文档字符串风格按仓库统一（Google 的 `Args:`/`Returns:`/`Raises:` 或 NumPy 风格），不要混用两种。
- 类型注解已表达的不要再在文档里复述类型；文档写语义（单位、可空、所有权、线程安全）。
- `TODO(name): 动作`、`FIXME(name): 条件` 必须带人名与触发条件。

## 10 工具链与验证

- 格式化：`ruff format`（或 `black`）；检查：`ruff check`（含 `I` 导入排序、`UP` 现代语法、`B` bugbear、`SIM`）；CI 用 `--check` 而非自动改写。
- 类型：`mypy --strict` 或 `pyright`；新代码不允许新增错误，历史错误按文件加豁免而不是全局关。
- 测试：`pytest`（`-q`、`--strict-markers`、`-W error` 暴露告警）；覆盖率只作参考。
- 依赖与虚拟环境：`pyproject.toml` + 锁文件；不用 `pip install` 到系统环境。
- 提交前钩子：`pre-commit` 串起 format、lint、type-check。
- 只认工具输出：报告实际命令与结果，不用"应该没问题"代替验证。

## 11 反例（Good / Bad）

```python
# Bad：可变默认值、裸 except、吞错
def add(item, items=[]):
    try:
        items.append(item)
    except:
        pass
    return items

# Good：None 哨兵、具体异常、错误不吞
def add(item: Item, items: list[Item] | None = None) -> list[Item]:
    if items is None:
        items = []
    items.append(item)
    return items
```

```python
# Bad：用睡眠与轮询做同步、隐式 Any
def wait_ready(flag):
    while not flag.ready:
        time.sleep(0.1)

# Good：显式超时与等待原语，类型明确
def wait_ready(flag: threading.Event, timeout_s: float) -> None:
    if not flag.wait(timeout=timeout_s):
        raise TimeoutError(f"not ready after {timeout_s}s")
```

```python
# Bad：布尔比较、len 判空、类型比较
if flag == True: ...
if len(items) > 0: ...
if type(x) is int: ...

# Good
if flag: ...
if items: ...
if isinstance(x, int): ...
```

## 12 评审清单

- [ ] 4 空格缩进，行宽与 formatter 配置一致，无手写排版
- [ ] 命名符合 §3 表格，异常以 `Error` 结尾，无内建名遮蔽
- [ ] import 分组且字母序，无通配导入，公开接口有 `__all__`
- [ ] 公开函数有类型注解，用 `list[int]`/`X | None` 现代拼写，无裸 `Any`
- [ ] 无可变默认值，参数过多时改为关键字参数或数据类
- [ ] 无裸 `except`，异常链用 `raise ... from ...`，`assert` 不校验外部输入
- [ ] 并发模型明确，有超时，`with`/`async with` 管理资源
- [ ] 文档字符串有摘要与语义说明，注释讲 why
- [ ] `ruff format --check`、`ruff check`、`mypy`/`pyright`、`pytest` 全绿（真实运行）
- [ ] 无新增类型检查错误，豁免按文件而非全局

> 冲突时裁决顺序：仓库 formatter/linter 配置 > 本文件 > PEP 8 > Google Python Style Guide；PEP 8 允许知情例外，但例外必须在代码注释里给出理由。
