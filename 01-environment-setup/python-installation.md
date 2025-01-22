---
title: "Python安装指南"
slug: "python-installation"
sequence: 1
description: "在Windows和macOS系统上安装Python解释器的详细步骤"
status: "published"
---

# Python安装指南

在开始Python编程之前，我们需要先在电脑上安装Python解释器。本节将指导您在不同操作系统上完成Python的安装。

## Windows系统安装步骤

1. 访问Python官网：https://www.python.org/downloads/
2. 点击"Download Python 3.x.x"（选择最新的稳定版本）
3. 下载安装程序后双击运行
4. **重要：** 勾选"Add Python to PATH"选项
5. 点击"Install Now"开始安装

## macOS系统安装步骤

1. 访问Python官网：https://www.python.org/downloads/
2. 下载macOS版本的安装程序
3. 双击下载的.pkg文件
4. 按照安装向导完成安装

## 验证安装

安装完成后，需要验证Python是否正确安装：

1. 打开终端（命令提示符）
2. 输入以下命令：
   ```bash
   python --version
   ```
3. 如果看到类似"Python 3.x.x"的输出，说明安装成功

## 常见问题解决

1. Windows系统提示"python不是内部或外部命令"
   - 检查是否在安装时勾选了"Add Python to PATH"
   - 或手动将Python添加到系统环境变量

2. macOS系统显示"命令未找到"
   - 尝试使用`python3`命令代替`python`
   - 确认安装包是否正确安装

## 下一步

完成Python安装后，我们将在下一节学习如何安装和配置代码编辑器VS Code，它将帮助我们更高效地编写Python代码。
