# tedit-plugin-example

模块·插件·规范·示例

*EN: tedit-plugin-example — the reference implementation of the tedit v1 plugin spec.*

## 目录

* [定位](#定位)
* [插件规范 v1](#插件规范-v1)
* [文件](#文件)
* [构建与运行](#构建与运行)
* [License](#license)

## 定位

tedit 插件规范 v1 的参考实现：最小示例插件 `example.greet`，演示清单声明、
钩子约定与宿主静态装配的完整回路。tedit 核心见同级 `tedit` 仓。

*EN: the reference implementation of the tedit v1 plugin spec — a minimal
example plugin (`example.greet`) demonstrating the manifest, hook conventions
and the host's static-assembly loop. Core lives in the sibling `tedit` repo.*

## 插件规范 v1

* **装配形态**：静态装配式 —— 宿主入口 `import` 插件模块（tie 静态 import
  文本内联，tiec 编译期按装配裁剪未用模块，零运行时依赖）。动态加载（v2）
  待纯 tie 动态加载方案定案后另行开放。
* **清单（manifest）**：插件仓根 `plugin.data.tie`（tie:data 明文，文件角色
  `tie<data>`，正文即 tie 表字面量；规范基准 tie-spec 20262 §17.1）。字段：
  `id` / `name` / `version` / `license` / `entry`。分发时可经
  `tiec --compress-data plugin.data.tie -o plugin.zd` 单向编译为 zd（语义不变）。
* **钩子**（固定命名导出）：
  * `plugin_on_load(ctx: i64) -> i64` —— 装载；`ctx` 预留，0 为成功
  * `plugin_on_unload() -> i64` —— 卸载；0 为成功
  * `plugin_commands() -> table<string>` —— 注册命令名表
  * 命令处理器约定命名 `cmd_<命令名>(args: table<string>) -> string`
* **注意**：tie:data 正文即 tie 表字面量，**不支持尾逗号**（`..., ]` 报
  E00000，与 JSONC 风格不同）。

*EN: v1 static assembly (host imports plugin modules; unused modules trimmed
at compile time); manifest = `plugin.data.tie` in tie:data plaintext
(tie-spec 20262 §17.1; compressible to binary zd with identical semantics);
fixed-name hooks + `cmd_<name>` handlers.*

## 文件

```
plugin.data.tie        插件清单（tie:data；含 extern 原生依赖声明字段）
build.tsh.tie          构建驱动（tsh 角色：产物落 build/demo 与 build/release）
build/demo/            演示包（demo_host.exe + plugin.data.tie）
build/release/         分发包（demo_host.exe + plugin.zd + 清单 + LICENSE）
test/                  测试（占位）
src/greet_plugin.tie   插件模块（钩子 + 命令处理器）
src/demo_host.tie      v1 静态装配演示宿主（内核子集 + 插件回路）
src/api|class|config|extern|script|ui|assets/   包布局占位（规范见核心仓
                       tedit/docs/plugin-layout.md）
```

## 插件包布局

目录语义、extern 原生依赖规则、config 分层与 script 加载约定的规范基准在
核心仓：`tedit/docs/plugin-layout.md`。

## 构建与运行

需要同级克隆：`../tiec/`（编译器）与 `../tedit/`（内核）。

```sh
<tshell>/src/tsh_main.exe -f build.tsh.tie
```

产物（**不进 src/**）：

* `build/demo/` —— 演示包：`demo_host.exe` + `plugin.data.tie`
* `build/release/` —— 分发包：`demo_host.exe` + `plugin.zd`（清单二进制形态）
  + `plugin.data.tie` + `LICENSE`

或直编：

```sh
../tiec/compiler/tiec.exe src/demo_host.tie -o build/demo/demo_host.exe
build/demo/demo_host.exe
```

`plugin.zd` 为清单的二进制分发形态（`tiec --compress-data` 产出，语义不变，
gitignored，可随时从 `plugin.data.tie` 重建）。构建驱动内 `exec_code` 为异步
返回，产物读取前以 `sleep_ms` 轮询（tsh p.9216 §4）。

## License

以 [Tie Public License 2.2](https://github.com/tie-lang/TPL/blob/main/tpl.txt)
（TPL 2.2）开源。
