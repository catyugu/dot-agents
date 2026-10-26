# Bash — 落地细则

Source: <https://google.github.io/styleguide/shellguide.html>
配套：ShellCheck、shfmt、bats-core；先以仓库 .shellcheckrc / .editorconfig 为准。

## 1 适用范围与优先级

- Bash 参考手册定义语义；Bash 没有官方格式规范。Google Shell Style Guide 是公司/社区规范（**社区惯例，非官方规则**），ShellCheck 是静态分析器，shfmt 是格式化器。
- 脚本必须声明自己是 Bash 还是 POSIX `sh`。用 Bash 特性就写 `#!/bin/bash`（Google 指南的规定，路径固定）或 `#!/usr/bin/env bash`（按 PATH 取 bash），项目内选一种。禁止在 `#!/bin/sh` 下写数组、`[[ ]]`、`function` 等 bashism。
- 适用边界：只调用其他工具、数据搬运量小的脚本才用 shell。超过 100 行或控制流不直白时，改用结构化语言——脚本只会变长，早改成本更低。
- 裁决顺序：仓库 `.shellcheckrc` / `.editorconfig` / CI → 邻近脚本 → Google Shell Style Guide → ShellCheck 默认严重级别 → shfmt 默认格式。
- 禁止 SUID/SGID 脚本：shell 无法可靠地清洗环境变量、IFS 与 PATH，需要提权就用 `sudo`。

## 2 缩进、行宽与排版

- 2 空格缩进，禁止 Tab（唯一例外：`<<-` heredoc 的缩进体）。Google 指南规定 2 空格；shfmt 默认输出 tab，所以必须显式 `shfmt -i 2`。
- 行宽 ≤80。超长字面量用 heredoc 或内嵌换行；无法拆的长路径、URL 单独成行或先赋值给短变量。
- 管道与逻辑链超过一行时，每行一个 `|` / `&&`，反斜杠续行，续行缩进 2：

  ```bash
  command1 \
    | command2 \
    | command3
  ```

- `; then` / `; do` 与 `if`/`for`/`while` 同行；`else` 独占一行；`fi`/`done` 与开头对齐。
- `case`：模式相对 `case` 缩进一级，多行动作再缩进一级，`;;` 独占一行；单行分支写成 `a) cmd ;;`（模式右括号后一个空格）；模式前不加 `(`；禁用 `;&` 与 `;;&`。
- 函数定义：`name() {`，大括号与函数名同行，名字与 `()` 之间不留空格；`function` 关键字可省略，但项目内必须统一。
- 重定向符紧贴目标（`>file`），除非项目用 `shfmt -sr` 统一加空格。
- 变量展开优先 `"${var}"`；单字符特殊参数 `$1`、`$?`、`$#`、`$$`、`$!`、`$@` 不加花括号。
- 命令替换一律 `$(command)`，禁止反引号：嵌套时反引号要转义，`$( )` 不用。
- 一行一条命令，不用 `;` 把多条命令挤在一行。

## 3 命名

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| 函数 | 小写下划线；库内加 `pkg::` 前缀 | `cleanup()`, `db::connect()` |
| 局部变量 | 小写下划线，必须 `local` | `local file_count` |
| 常量 / 只读变量 | UPPER_SNAKE_CASE，声明在文件顶部 | `readonly MAX_RETRY=3` |
| 环境变量 | UPPER_SNAKE_CASE + `export` | `export APP_HOME` |
| 脚本文件 | 小写下划线；库文件用 `.sh` 且不可执行 | `backup_db.sh` |
| 循环变量 | 取被遍历对象的单数名 | `for zone in "${zones[@]}"` |
| 未使用参数 | `_` 占位 | `for _ in {1..3}` |

- 名字带单位与来源：`timeout_secs`、`src_dir`、`backup_root`。
- 禁止把 `PATH`、`IFS`、`LANG`、`PS1`、`UID`、`PWD` 等已有特殊含义的名字用作局部变量，覆盖它们会破坏后续所有命令。
- 库函数统一前缀，避免 `source` 后和调用方脚本的函数重名覆盖。

## 4 文件与模块组织

- 文件骨架固定顺序：shebang → 文件头注释 → `set` 选项 → 常量 → 全部函数 → `main` → 最后一行 `main "$@"`。
- 函数集中放在常量之后、`main` 之前，中间不夹可执行语句；函数之间藏着代码会让断点与阅读顺序错乱。
- 含至少一个其他函数的脚本必须有 `main`，且 `main` 是最后一个函数。理由：`main` 里的变量可以声明为 `local`，顶层代码做不到。
- 库文件被 `source` 时不得有副作用：不调用 `main`、不 `set -e`、不 `cd`、不 `exit`。
- 定位脚本自身目录用 `readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"`，不依赖调用者的 cwd。
- 外部依赖在使用点检测：`command -v jq >/dev/null || { err "jq not found"; exit 1; }`，不要在文件头一次性检查所有工具。
- 一个脚本一个职责；共享函数抽到 `lib/*.sh`，用 `source` 显式引入。

## 5 数据类型与变量作用域

- Bash 只有字符串和数组（索引数组、关联数组）。数字是字符串，只有 `(( ))` 里才按整数运算。
- 一切变量默认全局，函数内必须 `local`。理由：泄漏的变量会覆盖调用方同名变量，且跨函数隐式耦合。
- `local x="$(cmd)"` 丢弃 `cmd` 的退出码（`local` 自身返回 0），`set -e` 也拦不住。必须拆两行：

  ```bash
  local out
  out="$(cmd)" || return 1
  ```

  同样的坑适用于 `export`、`declare`、`typeset`（ShellCheck SC2155）。
- 索引数组展开必须 `"${arr[@]}"`；`${arr[*]}` 只在拼日志或消息时用。赋值用 `arr=(a b c)`，追加用 `arr+=(d)`。
- 关联数组用 `declare -A`，下标必须加引号：`"${map[$key]}"`。理由：`${map[$key]}` 里未加引号的 `$key` 会发生分词与通配展开，含空格或 `*` 的键直接匹配失败或取到错误元素。
- 整数变量用 `local -i` / `declare -i`，避免 `08` 被当作非法八进制。
- 常量用 `readonly`；需要条件赋值的常量在赋值完成后立刻 `readonly VAR`。
- 区分"未设置"与"空值"：`${var:-default}` 两者都替换，`${var-default}` 仅在未设置时替换；`set -u` 下二者必用其一。
- 禁止 `eval`；间接引用用 `${!name}`，字符串替换用 `${var/pat/rep}`，数字用 `(( ))`。
- 环境变量在脚本内改写前先备份，退出时恢复，否则会影响后续 `source` 该脚本的进程。

## 6 函数与 API 设计

- 函数控制在一屏内（约 20 行）；超出就拆，或说明它其实是主流程。
- 入口先校验参数个数：`(( $# == 2 )) || { err "usage: $0 SRC DST"; return 2; }`，再 `local src=$1 dst=$2` 接住。
- 输出契约必须明确：结果写 stdout，进度与诊断写 stderr，返回码只表示成功/失败（0 / 非 0）；返回值语义写进函数头注释。
- 输出可变内容用 `printf '%s\n'`，不用 `echo`：`echo` 对 `-n`、`-e` 以及反斜杠的处理随实现与 shell 选项变化。
- 返回结构化数据用 stdout 加调用方命令替换，或返回码加输出变量名；不要用全局变量在函数之间传值。
- 选项解析用 `getopts`（短选项）或 `while (( $# )); case $1 in ...`；不要手写 `$1 $2 $3` 位移链。
- 引用规则：除刻意分词或通配，所有展开都加双引号（SC2086）；`[[ ]]` 里模式或正则的右侧不加引号——加了就变成字面量匹配。
- 条件判断用 `[[ ]]`（无分词与通配展开）：字符串相等用 `==`，正则用 `=~`；数值比较用 `(( ))` 或 `-lt`/`-gt`，`[[ $a > 7 ]]` 是字典序比较。
- 判断空串用 `-z` / `-n`，不写 `[[ "${var}" ]]`，也不写 `[[ "${var}X" == "valueX" ]]` 这类填充技巧。
- 布尔结果用返回码传递，不要 `printf 'true'` 再让调用方比较字符串。

## 7 错误处理

- `set -e` 不是错误处理方案：管道、`if`/`while` 的条件、`&&`/`||` 列表、以及"函数被当作条件调用"这几种场景都会让它失效，规则还随 Bash 版本与 POSIX 模式变化。结论是别把正确性押在它上面，逐处检查。
- 采用哪种严格模式必须在文件头显式写出，并让所有协作者知道：最少 `set -u`；用管道就加 `set -o pipefail`；是否加 `-e` 由脚本契约决定，加了就要接受它的例外。
- `set -u` 下所有可为空的展开写成 `"${var:-}"`；空数组展开在旧 Bash 里需写成 `${arr[@]+"${arr[@]}"}` 才不报错。
- `set -o pipefail` 让管道返回第一个失败命令的状态；只看 `$?` 的最后一条命令会漏掉上游失败。
- 检查返回值：`if ! cmd; then ...` 或 `cmd || { err "..."; exit 1; }`。禁止先执行再查 `$?`（SC2181）——中间插入任何命令都会覆盖它。
- 禁止 `|| true` 掩盖失败；确实可忽略时必须写出为什么可忽略，以及失败后的后果。
- 错误一律写 stderr：`printf '%s: %s\n' "${0##*/}" "$*" >&2`，统一封装成 `err()`，带脚本名便于在 CI 日志里定位。
- `cd` 必须带失败处理：`cd "${dir}" || exit 1`（SC2164）。理由：`cd` 失败后后续命令会在错误目录里执行，破坏范围更大。
- 退出码要有语义：0 成功、1 一般错误、2 用法错误；退出前打印可行动的提示，而不是只打印栈式描述。
- `trap` 在资源创建之前安装，处理 `EXIT INT TERM`；`EXIT` 的处理函数里先 `local rc=$?` 再清理，最后 `exit "${rc}"` 保留原退出码。清理失败不得改变退出码。

## 8 并发、资源与生命周期

- 后台任务保存 PID：`cmd & pid=$!`，随后 `wait "${pid}"` 收退出码。`wait` 不带参数会等待全部后台任务但返回 0，拿不到各自状态；多个任务要逐个 `wait` 并分别记录。
- 并发写同一文件必须加锁：`flock` 或写临时文件后原子 `mv`。不要假设"不会同时跑"。
- 管道会创建子 shell，`while read` 里赋的变量在父 shell 中不可见；改用进程替换 `done < <(cmd)` 或 `readarray -t lines < <(cmd)`。
- 临时文件与目录用 `mktemp -d`，路径放进变量并在 `trap` 里删除；不要用 `$$` 拼固定名，也不要写进 `/tmp` 的可预测路径。
- 通配展开用 `./*` 而不是 `*`，避免以 `-` 开头的文件名被当成选项；传给外部命令的参数前加 `--`。
- 长任务响应信号：`trap 'exit 130' INT`、`trap 'exit 143' TERM`，让清理走同一条退出路径。
- `exec 3>file` 打开的 fd 要 `exec 3>&-` 关闭；脚本退出时内核会回收，但库函数不应依赖这一点。
- 不要用 `kill -0` 轮询代替 `wait`：轮询既消耗 CPU，也无法拿到退出码。

## 9 注释与文档

- 文件头注释写：用途、输入、输出、退出码、依赖的外部命令、需要的权限。shebang 之后紧跟。
- 函数头注释按 Google 的四段式：Description、Globals（读写了哪些全局变量）、Arguments、Outputs / Returns。
- 注释解释 why：为什么这里必须加引号、为什么这个失败可以忽略、为什么用子 shell。
- `# TODO(name):` 带负责人或 issue 链接；裸 `# TODO` 无法追踪。
- 注释与代码同步失效比没有注释更糟：改行为必须同时改注释。
- 复杂 `sed`/`awk`/正则上方给一行输入输出示例，证明它做什么。
- 危险命令（`rm -rf`、`dd`、覆盖写）在同一行或上一行注释说明前置条件与影响范围。

## 10 工具链与验证

- ShellCheck：`shellcheck -x -s bash -S warning script.sh`。`-x` 跟随 `source`，`-s bash` 明确方言，`-S` 设定最低严重级别；CI 中要求 0 告警。
- 每处 `# shellcheck disable=SC2086` 必须限定到单行并写明理由；不允许文件级 disable。
- 格式化：`shfmt -i 2 -ci -bn -d .` 检查，`shfmt -w` 写入。`-i 2` 覆盖默认 tab；`-ci` 缩进 case、`-bn` 让 `&&`/`|` 起行，这两项与 Google 风格一致。参数可固化到 `.editorconfig`（`indent_style`、`indent_size`、`case_indent`、`binary_next_line`、`language_dialect`）。
- 语法检查：`bash -n script.sh`；shfmt 是静态全量解析、要求合法 UTF-8，比 `bash -n` 更严格，两者一起用。
- 测试：bats-core 写行为测试，断言 stdout、stderr 与退出码；CI 固定 bash 版本，避免本地与 CI 行为不一致。
- 真实执行一遍：lint 不检查逻辑，脚本必须在目标环境跑通并留下可核对的输出。
- 退出条件：`shellcheck` 0 告警、`shfmt -d` 无 diff、`bash -n` 通过、bats 全绿。
- 参考文档：[Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)、[Bash 参考手册](https://www.gnu.org/software/bash/manual/bash.html)、[ShellCheck Wiki](https://github.com/koalaman/shellcheck/wiki)、[shfmt 手册](https://github.com/mvdan/sh)。
- 已知争议：[BashFAQ 105 关于 set -e](https://mywiki.wooledge.org/BashFAQ/105)、[BashPitfalls](https://mywiki.wooledge.org/BashPitfalls)。

## 11 反例（Good / Bad）

```bash
#!/bin/bash
# Bad：展开未加引号；$(ls ...) 按空白分词；eval 拼接命令；
# local 吞掉 mktemp 的退出码；先执行再查 $?；失败信息写 stdout。
collect_logs() {
  local target=$(mktemp -d)
  for f in $(ls /var/log/*.log); do
    eval "gzip $f"
  done
  tar -cf $target/logs.tar /var/log
  if [ $? -ne 0 ]; then
    echo "archive failed"
  fi
  echo $target/logs.tar
}
```

```bash
#!/usr/bin/env bash
#
# collect_logs: gzip the given log files into one tarball.
# Globals:   LOG_DIR (default /var/log)
# Arguments: $1 output tarball path
# Outputs:   tarball path on stdout, diagnostics on stderr
# Returns:   0 on success, 1 on failure, 2 on usage error

set -u -o pipefail

err() { printf '%s: %s\n' "${0##*/}" "$*" >&2; }

collect_logs() {
  local out=$1 dir=${LOG_DIR:-/var/log}
  [[ -n "${out}" ]] || { err "usage: collect_logs OUT"; return 2; }

  tmp="$(mktemp -d)" || { err "mktemp failed"; return 1; }
  # EXIT trap 在函数返回后才执行，tmp 必须是脚本级变量，不能用 local
  trap 'rm -rf -- "${tmp}"' EXIT

  find "${dir}" -maxdepth 1 -name '*.log' -print0 \
    | tar --null -czf "${tmp}/logs.tar.gz" -T - \
    || { err "archive failed under ${dir}"; return 1; }

  mv "${tmp}/logs.tar.gz" "${out}" || { err "cannot write ${out}"; return 1; }
  printf '%s\n' "${out}"
}
```

```bash
#!/usr/bin/env bash
# Bad：管道创建子 shell，循环里对 count 的修改在父 shell 中丢失，结果永远是 0；
# 聚合还依赖 grep -c 的输出恰好一行，参数也没有校验。
count_failures() {
  local count=0
  grep -c "ERROR" "${1}" | while read -r line; do
    count=$((count + line))
  done
  printf 'failures=%d\n' "${count}"
}
```

```bash
#!/usr/bin/env bash
# Good：readarray 在父 shell 里读入，参数先校验，聚合后一次输出。
count_failures() {
  local -i count=0
  local line
  [[ -n "${1:-}" ]] || { printf '%s: usage: count_failures LOGFILE\n' "${0##*/}" >&2; return 2; }
  readarray -t lines < <(grep -c "ERROR" "${1}")
  for line in "${lines[@]}"; do
    count+=line
  done
  printf 'failures=%d\n' "${count}"
}
```

## 12 评审清单

- [ ] shebang 明确是 bash，且没有任何 `#!/bin/sh` 文件里使用 bashism
- [ ] 缩进 2 空格、无 Tab（除 `<<-` heredoc），行宽 ≤80，`shfmt -i 2 -ci -bn -d .` 无 diff
- [ ] 所有展开加了双引号，数组用 `"${arr[@]}"`，`[[ ]]` 里模式/正则右侧未加引号
- [ ] 函数内变量全部 `local`，`local`/`export`/`declare` 与命令替换分开两行写
- [ ] `set` 选项在文件头显式声明，管道失败在 `pipefail` 下能被检测到
- [ ] 没有 `eval`，没有 `|| true` 掩盖失败，没有先执行再查 `$?`
- [ ] 每个 `cd` 都带 `|| exit`，通配用 `./*` 或补 `--`，临时路径来自 `mktemp`
- [ ] 后台任务被 `wait` 收退出码，并发写同一文件有锁
- [ ] 不依赖管道子 shell 里的变量赋值（改用进程替换或 `readarray`）
- [ ] 脚本有 `main` 且末行是 `main "$@"`，函数头注释含 Globals / Arguments / Outputs / Returns
- [ ] `shellcheck -x -s bash` 0 告警，`bash -n` 通过，bats 全绿

> 冲突时的裁决顺序：仓库 `.shellcheckrc` / `.editorconfig` / CI → 邻近脚本 → Google Shell Style Guide → ShellCheck 默认严重级别 → shfmt 默认格式；语义问题以 Bash 参考手册为准，可移植性以目标环境实际 `/bin/sh` 为准。
