---
title: ✨IDEA 常用快捷键
published: 2026-05-09
pinned: false
description: "整理 IntelliJ IDEA 日常开发常用快捷键，覆盖搜索、编辑、导航、重构、运行调试、Git 与窗口操作。"
tags: [IDEA, 开发工具]
category: 技术分享
draft: false
image: ../uploads/20260110/yy.png
---

# IDEA 常用快捷键

> 本文以 IntelliJ IDEA 默认 Windows/Linux Keymap 为主，macOS 可以按常见映射理解：`Ctrl` 对应 `Command`，`Alt` 对应 `Option`，部分组合以 IDEA 实际 Keymap 为准。

## 快速入口

| 功能 | Windows / Linux | macOS | 使用场景 |
| --- | --- | --- | --- |
| 查找动作 | `Ctrl + Shift + A` | `Command + Shift + A` | 忘记快捷键时，直接搜索功能名称 |
| 全局搜索 | `Shift` 连按两次 | `Shift` 连按两次 | 搜索类、文件、符号、设置和动作 |
| 打开设置 | `Ctrl + Alt + S` | `Command + ,` | 修改插件、主题、Keymap、代码样式 |
| 打开项目结构 | `Ctrl + Alt + Shift + S` | `Command + ;` | 配置 SDK、模块、依赖 |

## 搜索与定位

| 功能 | Windows / Linux | macOS |
| --- | --- | --- |
| 查找文件 | `Ctrl + Shift + N` | `Command + Shift + O` |
| 查找类 | `Ctrl + N` | `Command + O` |
| 查找符号 | `Ctrl + Alt + Shift + N` | `Command + Option + O` |
| 当前文件查找 | `Ctrl + F` | `Command + F` |
| 全局文本查找 | `Ctrl + Shift + F` | `Command + Shift + F` |
| 当前文件替换 | `Ctrl + R` | `Command + R` |
| 全局文本替换 | `Ctrl + Shift + R` | `Command + Shift + R` |
| 跳转到指定行 | `Ctrl + G` | `Command + L` |
| 最近文件 | `Ctrl + E` | `Command + E` |
| 最近修改位置 | `Ctrl + Shift + E` | `Command + Shift + E` |
| 返回上一个位置 | `Ctrl + Alt + Left` | `Command + Option + Left` |
| 前进到下一个位置 | `Ctrl + Alt + Right` | `Command + Option + Right` |

## 代码编辑

| 功能 | Windows / Linux | macOS |
| --- | --- | --- |
| 基础补全 | `Ctrl + Space` | `Control + Space` |
| 智能补全 | `Ctrl + Shift + Space` | `Control + Shift + Space` |
| 生成代码 | `Alt + Insert` | `Command + N` |
| 快速修复 | `Alt + Enter` | `Option + Enter` |
| 格式化代码 | `Ctrl + Alt + L` | `Command + Option + L` |
| 优化导入 | `Ctrl + Alt + O` | `Control + Option + O` |
| 复制当前行 | `Ctrl + D` | `Command + D` |
| 删除当前行 | `Ctrl + Y` | `Command + Delete` |
| 上移当前行 | `Shift + Alt + Up` | `Option + Shift + Up` |
| 下移当前行 | `Shift + Alt + Down` | `Option + Shift + Down` |
| 注释当前行 | `Ctrl + /` | `Command + /` |
| 块注释 | `Ctrl + Shift + /` | `Command + Option + /` |
| 包围代码 | `Ctrl + Alt + T` | `Command + Option + T` |
| 展开选择 | `Ctrl + W` | `Option + Up` |
| 缩小选择 | `Ctrl + Shift + W` | `Option + Down` |
| 多光标选择下一个匹配项 | `Alt + J` | `Control + G` |
| 选择所有匹配项 | `Ctrl + Alt + Shift + J` | `Control + Command + G` |

## 代码导航

| 功能 | Windows / Linux | macOS |
| --- | --- | --- |
| 跳转到声明 | `Ctrl + B` | `Command + B` |
| 跳转到实现 | `Ctrl + Alt + B` | `Command + Option + B` |
| 查看继承结构 | `Ctrl + H` | `Control + H` |
| 查看使用位置 | `Alt + F7` | `Option + F7` |
| 快速查看定义 | `Ctrl + Shift + I` | `Command + Y` |
| 查看文档 | `Ctrl + Q` | `F1` |
| 文件结构 | `Ctrl + F12` | `Command + F12` |
| 跳转到上一个方法 | `Alt + Up` | `Control + Up` |
| 跳转到下一个方法 | `Alt + Down` | `Control + Down` |
| 跳转到匹配括号 | `Ctrl + Shift + M` | `Control + M` |

## 重构

| 功能 | Windows / Linux | macOS |
| --- | --- | --- |
| 重命名 | `Shift + F6` | `Shift + F6` |
| 修改方法签名 | `Ctrl + F6` | `Command + F6` |
| 提取变量 | `Ctrl + Alt + V` | `Command + Option + V` |
| 提取常量 | `Ctrl + Alt + C` | `Command + Option + C` |
| 提取字段 | `Ctrl + Alt + F` | `Command + Option + F` |
| 提取方法 | `Ctrl + Alt + M` | `Command + Option + M` |
| 内联变量或方法 | `Ctrl + Alt + N` | `Command + Option + N` |
| 打开重构菜单 | `Ctrl + Alt + Shift + T` | `Control + T` |

## 运行与调试

| 功能 | Windows / Linux | macOS |
| --- | --- | --- |
| 运行当前配置 | `Shift + F10` | `Control + R` |
| 调试当前配置 | `Shift + F9` | `Control + D` |
| 停止运行 | `Ctrl + F2` | `Command + F2` |
| 重新运行 | `Ctrl + F5` | `Command + R` |
| 单步跳过 | `F8` | `F8` |
| 单步进入 | `F7` | `F7` |
| 强制单步进入 | `Alt + Shift + F7` | `Option + Shift + F7` |
| 单步跳出 | `Shift + F8` | `Shift + F8` |
| 运行到光标处 | `Alt + F9` | `Option + F9` |
| 查看表达式 | `Alt + F8` | `Option + F8` |
| 打断点 | `Ctrl + F8` | `Command + F8` |

## Git 常用操作

| 功能 | Windows / Linux | macOS |
| --- | --- | --- |
| 提交代码 | `Ctrl + K` | `Command + K` |
| 更新项目 | `Ctrl + T` | `Command + T` |
| 推送代码 | `Ctrl + Shift + K` | `Command + Shift + K` |
| 查看版本控制窗口 | `Alt + 9` | `Command + 9` |
| 查看文件历史 | `Alt + BackQuote` 后选择 History | `Control + V` 后选择 History |
| 回滚当前文件改动 | `Ctrl + Alt + Z` | `Command + Option + Z` |

## 窗口与工具栏

| 功能 | Windows / Linux | macOS |
| --- | --- | --- |
| Project 窗口 | `Alt + 1` | `Command + 1` |
| Commit 窗口 | `Alt + 0` | `Command + 0` |
| Run 窗口 | `Alt + 4` | `Command + 4` |
| Debug 窗口 | `Alt + 5` | `Command + 5` |
| Terminal 窗口 | `Alt + F12` | `Option + F12` |
| 隐藏所有工具窗口 | `Ctrl + Shift + F12` | `Command + Shift + F12` |
| 最大化编辑器 | `Ctrl + Shift + F12` | `Command + Shift + F12` |
| 切换标签页 | `Alt + Left / Right` | `Command + Shift + [ / ]` |

## 进阶常用快捷键

| 功能 | Windows / Linux | macOS | 使用场景 |
| --- | --- | --- | --- |
| 构建项目 | `Ctrl + F9` | `Command + F9` | 修改代码后快速编译当前项目 |
| 重新构建 | `Ctrl + Shift + F9` | `Command + Shift + F9` | 重新编译当前文件、包或模块 |
| 完成当前语句 | `Ctrl + Shift + Enter` | `Command + Shift + Enter` | 自动补齐分号、括号、代码块 |
| 查看参数信息 | `Ctrl + P` | `Command + P` | 调用方法时查看参数列表 |
| 跳转到测试 | `Ctrl + Shift + T` | `Command + Shift + T` | 在源码和测试类之间切换 |
| 下一个错误或警告 | `F2` | `F2` | 快速定位代码检查问题 |
| 上一个错误或警告 | `Shift + F2` | `Shift + F2` | 回到上一个代码检查问题 |
| 剪贴板历史 | `Ctrl + Shift + V` | `Command + Shift + V` | 从历史复制内容中选择粘贴 |
| 合并多行 | `Ctrl + Shift + J` | `Control + Shift + J` | 将多行代码合并为一行 |
| 折叠代码块 | `Ctrl + NumPad -` | `Command + NumPad -` | 收起当前代码块 |
| 展开代码块 | `Ctrl + NumPad +` | `Command + NumPad +` | 展开当前代码块 |
| 添加书签 | `F11` | `F3` | 标记需要回看的代码位置 |
| 查看所有书签 | `Shift + F11` | `Option + F3` | 打开书签列表并快速跳转 |
| Run Anything | `Ctrl` 连按两次 | `Control` 连按两次 | 快速运行配置、命令或 Gradle/Maven 任务 |
| 查看断点列表 | `Ctrl + Shift + F8` | `Command + Shift + F8` | 管理全部断点和断点条件 |

## 实用组合

### 快速定位代码

```text
Shift 双击
输入类名、文件名或方法名
Enter 打开目标
Ctrl + B 跳转声明
Alt + F7 查看使用位置
```

### 快速整理代码

```text
Alt + Enter 快速修复
Ctrl + Alt + O 优化导入
Ctrl + Alt + L 格式化代码
Ctrl + Alt + V 提取变量
Ctrl + Alt + M 提取方法
```

### 快速调试问题

```text
Ctrl + F8 添加断点
Shift + F9 启动调试
F8 单步跳过
F7 单步进入
Alt + F8 查看表达式
Alt + F9 运行到光标处
```

## 记忆建议

高频快捷键不需要一次性全部记住，优先掌握下面这一组就足够覆盖大多数日常开发：

| 快捷键 | 功能 |
| --- | --- |
| `Shift` 双击 | 全局搜索 |
| `Ctrl + Shift + A` | 查找动作 |
| `Alt + Enter` | 快速修复 |
| `Ctrl + Alt + L` | 格式化代码 |
| `Ctrl + B` | 跳转声明 |
| `Alt + F7` | 查看使用位置 |
| `Shift + F6` | 重命名 |
| `Ctrl + Alt + V` | 提取变量 |
| `Ctrl + Alt + M` | 提取方法 |
| `Shift + F9` | 调试 |

## 内容来源

本文快捷键内容参考 IntelliJ IDEA 默认 Keymap、JetBrains 官方快捷键文档及日常开发使用习惯整理，由 ChatGPT 根据本文写作需求编写并补充说明。
