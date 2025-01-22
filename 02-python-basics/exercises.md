---
title: "练习实战"
slug: "exercises"
sequence: 5
description: "通过实践项目巩固Python基础知识，包括简单计算器和温度转换器的实现"
status: "published"
---

# 练习实战

通过实践是掌握编程的最好方式。本节将通过几个简单的项目来巩固前面学习的Python基础知识。

## 项目一：简单计算器

创建一个基础的计算器程序，支持加减乘除运算：

```python
# calculator.py

def get_number(prompt):
    """获取用户输入的数字"""
    while True:
        try:
            return float(input(prompt))
        except ValueError:
            print("请输入有效的数字！")

def get_operator():
    """获取用户输入的运算符"""
    valid_operators = ['+', '-', '*', '/']
    while True:
        operator = input("请输入运算符（+、-、*、/）：")
        if operator in valid_operators:
            return operator
        print("无效的运算符！请重新输入")

def calculate(num1, operator, num2):
    """执行计算"""
    if operator == '+':
        return num1 + num2
    elif operator == '-':
        return num1 - num2
    elif operator == '*':
        return num1 * num2
    elif operator == '/':
        if num2 == 0:
            return "错误：除数不能为0"
        return num1 / num2

def main():
    print("==== 简单计算器 ====")
    
    # 获取第一个数字
    num1 = get_number("请输入第一个数字：")
    
    # 获取运算符
    operator = get_operator()
    
    # 获取第二个数字
    num2 = get_number("请输入第二个数字：")
    
    # 执行计算并显示结果
    result = calculate(num1, operator, num2)
    print(f"\n{num1} {operator} {num2} = {result}")

if __name__ == "__main__":
    main()
```

## 项目二：温度转换器

创建一个温度转换程序，支持摄氏度和华氏度的相互转换：

```python
# temperature_converter.py

def celsius_to_fahrenheit(celsius):
    """将摄氏度转换为华氏度"""
    return (celsius * 9/5) + 32

def fahrenheit_to_celsius(fahrenheit):
    """将华氏度转换为摄氏度"""
    return (fahrenheit - 32) * 5/9

def get_temperature(prompt):
    """获取用户输入的温度值"""
    while True:
        try:
            return float(input(prompt))
        except ValueError:
            print("请输入有效的数字！")

def get_conversion_choice():
    """获取用户的转换选择"""
    while True:
        choice = input("选择转换类型（1：摄氏度到华氏度，2：华氏度到摄氏度）：")
        if choice in ['1', '2']:
            return choice
        print("无效的选择！请输入1或2")

def main():
    print("==== 温度转换器 ====")
    
    # 获取用户选择
    choice = get_conversion_choice()
    
    if choice == '1':
        # 摄氏度到华氏度
        celsius = get_temperature("请输入摄氏度：")
        fahrenheit = celsius_to_fahrenheit(celsius)
        print(f"\n{celsius}°C = {fahrenheit:.2f}°F")
    else:
        # 华氏度到摄氏度
        fahrenheit = get_temperature("请输入华氏度：")
        celsius = fahrenheit_to_celsius(fahrenheit)
        print(f"\n{fahrenheit}°F = {celsius:.2f}°C")

if __name__ == "__main__":
    main()
```

## 项目三：个人信息卡片生成器

创建一个程序，收集用户信息并生成格式化的信息卡片：

```python
# info_card_generator.py

def get_valid_input(prompt, validator, error_message):
    """获取并验证用户输入"""
    while True:
        value = input(prompt).strip()
        if validator(value):
            return value
        print(error_message)

def validate_name(name):
    """验证姓名（2-20个字符）"""
    return 2 <= len(name) <= 20

def validate_age(age):
    """验证年龄（0-150的数字）"""
    try:
        age_num = int(age)
        return 0 <= age_num <= 150
    except ValueError:
        return False

def validate_email(email):
    """简单的邮箱格式验证"""
    return '@' in email and '.' in email

def generate_card(name, age, email, hobbies):
    """生成信息卡片"""
    card = f"""
╔══════════════════════════
║ 个人信息卡
╠══════════════════════════
║ 姓名：{name}
║ 年龄：{age}岁
║ 邮箱：{email}
║ 兴趣：{', '.join(hobbies)}
╚══════════════════════════
"""
    return card

def main():
    print("==== 个人信息卡片生成器 ====")
    
    # 收集信息
    name = get_valid_input(
        "请输入姓名：",
        validate_name,
        "姓名长度必须在2-20个字符之间"
    )
    
    age = get_valid_input(
        "请输入年龄：",
        validate_age,
        "请输入有效的年龄（0-150）"
    )
    
    email = get_valid_input(
        "请输入邮箱：",
        validate_email,
        "请输入有效的邮箱地址"
    )
    
    # 收集兴趣爱好
    print("请输入兴趣爱好（每行一个，输入空行结束）：")
    hobbies = []
    while True:
        hobby = input().strip()
        if not hobby:
            break
        hobbies.append(hobby)
    
    # 生成并显示信息卡片
    card = generate_card(name, age, email, hobbies)
    print("\n生成的信息卡片：")
    print(card)

if __name__ == "__main__":
    main()
```

## 运行和测试

1. 将每个项目的代码保存到单独的`.py`文件中
2. 在终端中运行程序：
   ```bash
   python calculator.py
   python temperature_converter.py
   python info_card_generator.py
   ```
3. 按照提示输入数据，测试程序功能
4. 尝试输入无效数据，观察程序的错误处理

## 扩展练习

1. 为计算器添加更多功能：
   - 支持幂运算
   - 添加历史记录功能
   - 支持连续计算

2. 增强温度转换器：
   - 添加开尔文温度的转换
   - 支持批量转换
   - 添加温度范围验证

3. 改进信息卡片生成器：
   - 添加更多个人信息字段
   - 支持保存到文件
   - 添加卡片样式选择

## 最佳实践

1. 代码组织：
   - 将相关功能封装到函数中
   - 使用有意义的变量名
   - 添加适当的注释

2. 输入验证：
   - 始终验证用户输入
   - 提供清晰的错误信息
   - 允许用户重试

3. 用户体验：
   - 提供清晰的使用说明
   - 格式化输出结果
   - 添加适当的空行和分隔符

## 下一步

完成这些练习后，您应该已经掌握了Python的基础知识。接下来，我们将进入下一章，学习更多Python的高级特性。
