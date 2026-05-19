# uart-prompt-console

[English](README.md) | [中文](README.zh-CN.md)

`uart-prompt-console` 是一个面向 `tio` 风格 UART 调试流程的 prompt-aware 包装层，适合设备 UART 一直刷后台日志、但你又需要输入 shell 命令的调试场景。

它的核心目标是：在看到设备 prompt 后暂停后台刷屏，让你稳定输入命令；发送命令前先把暂停期间缓存的旧日志刷出来，再把命令发给设备，避免旧日志混到命令响应后面。

## 和 tio 的关系

这个工具的定位是对 `tio` 常见使用流程做一层小包装，不是要替代 `tio` 的完整能力。

普通串口会话直接用 `tio` 就很好；当设备持续刷后台日志、你又需要围绕 shell prompt 输入命令时，用 `uart-prompt-console` 在熟悉的串口终端流程上补一层 prompt-aware 的交互状态机。快捷键也刻意保留了 `tio` 用户熟悉的 `Ctrl-T` 前缀风格。

## 安装

```bash
cargo install --path .
```

或者只构建 release 版本：

```bash
cargo build --release
```

## 使用

```bash
uart-prompt-console /dev/cu.usbmodem01234567895 -b 3000000
```

也可以通过环境变量指定默认串口设备：

```bash
export UART_PROMPT_DEVICE=/dev/cu.usbmodem01234567895
uart-prompt-console -b 3000000
```

默认日志会写到 `/tmp` 下，例如：

```text
/tmp/uart-prompt-console-1779081234.log
```

日志文件保存 UART 原始字节。屏幕显示层做的换行整理不会改动日志内容。

## 交互模型

正常模式：

```text
设备日志实时刷屏
```

按一次空 `Enter`：

```text
给设备发送一个行结束符
等待看到 '$' prompt
显示 prompt
暂停后续设备输出
```

输入命令后再按 `Enter`：

```text
清掉本地输入行
先把暂停期间缓存的设备输出刷到屏幕
再把你的命令发送给设备
恢复实时输出
```

这样旧的后台日志不会出现在命令响应后面。

## 快捷键

```text
空 Enter     发送换行，等待 '$'，然后停在 prompt
Enter        刷出暂停输出，然后发送当前输入行
Ctrl-U       清空当前输入
Backspace    删除一个输入字符
Ctrl-C       向设备发送 Ctrl-C
Ctrl-T r     恢复实时输出
Ctrl-T q     退出
Ctrl-T l     清屏
Ctrl-T ?     显示帮助
```

## 参数

```text
-d <device>              串口设备。也可以作为位置参数传入。
-b <baud>                波特率。默认：3000000。
-l <logfile>             日志文件路径。默认：/tmp/uart-prompt-console-*.log。
--newline cr|lf|crlf     命令行结束符。默认：cr。
```

## 说明

- 当前 prompt 检测使用 `$` 字符。
- 默认命令行结束符是 carriage return，也就是 `cr` / `\r`，这是很多嵌入式 UART shell 的常见输入方式。
- 如果你的 shell 需要 LF 或 CRLF，可以使用 `--newline lf` 或 `--newline crlf`。
