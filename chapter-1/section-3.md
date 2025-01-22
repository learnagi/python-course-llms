---
title: "虚拟环境设置"
slug: "virtual-environment"
sequence: 3
description: "学习如何创建和管理Python虚拟环境，实现项目依赖的隔离"
status: "published"
---

# 虚拟环境设置

Python虚拟环境允许您为不同的项目创建独立的Python环境，避免依赖冲突。本节将介绍如何创建和使用虚拟环境。

## 什么是虚拟环境？

虚拟环境是一个独立的Python环境，它：
- 有自己的Python解释器副本
- 有独立的包管理空间
- 不会影响其他项目的依赖
- 便于项目的部署和分发

## 创建虚拟环境

1. 打开终端（命令提示符）
2. 导航到项目目录
3. 运行以下命令创建虚拟环境：

```bash
# Windows
python -m venv llm-env

# macOS/Linux
python3 -m venv llm-env
```

## 激活虚拟环境

不同操作系统激活虚拟环境的命令不同：

### Windows
```bash
# CMD
llm-env\Scripts\activate.bat

# PowerShell
llm-env\Scripts\Activate.ps1
```

### macOS/Linux
```bash
source llm-env/bin/activate
```

激活后，终端提示符前会显示虚拟环境名称，如：`(llm-env)`

## 使用虚拟环境

1. 安装包：
```bash
pip install package_name
```

2. 查看已安装的包：
```bash
pip list
```

3. 导出依赖列表：
```bash
pip freeze > requirements.txt
```

4. 从依赖列表安装：
```bash
pip install -r requirements.txt
```

## 退出虚拟环境

在任何操作系统中，都可以使用以下命令退出虚拟环境：
```bash
deactivate
```

## VS Code中的虚拟环境

1. 打开VS Code命令面板（Ctrl+Shift+P 或 Cmd+Shift+P）
2. 输入"Python: Select Interpreter"
3. 选择虚拟环境中的Python解释器

## 最佳实践

1. 为每个项目创建独立的虚拟环境
2. 将`venv`目录添加到`.gitignore`
3. 始终保持`requirements.txt`更新
4. 在激活虚拟环境后再安装依赖

## 下一步

现在您已经掌握了Python开发环境的完整设置，包括Python安装、VS Code配置和虚拟环境管理。在下一章中，我们将开始学习Python的基础语法。
