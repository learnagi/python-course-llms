---
title: "代码编辑器配置"
slug: "vscode-setup"
sequence: 2
description: "安装和配置Visual Studio Code作为Python开发环境"
status: "published"
---

# 代码编辑器配置

Visual Studio Code（VS Code）是一个功能强大、轻量级的代码编辑器，它提供了丰富的Python开发支持。本节将指导您完成VS Code的安装和配置。

## 安装VS Code

1. 访问VS Code官网：https://code.visualstudio.com/
2. 下载适合您操作系统的版本
3. 运行安装程序，按照提示完成安装

## 安装Python扩展

VS Code需要安装Python扩展来支持Python开发：

1. 打开VS Code
2. 点击左侧扩展图标（或使用快捷键）：
   - Windows：`Ctrl+Shift+X`
   - macOS：`Cmd+Shift+X`
3. 在搜索框中输入"Python"
4. 找到并安装Microsoft的Python扩展（通常是第一个搜索结果）

## 配置Python解释器

1. 打开VS Code
2. 按下快捷键打开命令面板：
   - Windows：`Ctrl+Shift+P`
   - macOS：`Cmd+Shift+P`
3. 输入"Python: Select Interpreter"
4. 选择之前安装的Python版本

## 推荐的VS Code扩展

除了Python扩展外，以下扩展也能提升开发效率：

1. **Pylance**: Python语言服务器，提供更好的代码补全和类型检查
2. **Python Indent**: 自动处理Python缩进
3. **Python Docstring Generator**: 自动生成文档字符串
4. **GitLens**: Git集成增强

## VS Code常用快捷键

| 功能 | Windows | macOS |
|------|---------|--------|
| 运行Python文件 | F5 | F5 |
| 打开终端 | Ctrl+` | Cmd+` |
| 代码格式化 | Shift+Alt+F | Shift+Option+F |
| 快速修复 | Ctrl+. | Cmd+. |

## 下一步

完成VS Code的配置后，我们将在下一节学习如何创建和管理Python虚拟环境，这对于项目依赖管理非常重要。
