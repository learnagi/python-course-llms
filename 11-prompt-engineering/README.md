# 第十一章：提示词工程与优化

本章将介绍提示词工程的核心概念、最佳实践和优化技巧，帮助你更好地利用大语言模型。

## 1. 提示词工程基础

### 提示词结构
```python
class PromptTemplate:
    def __init__(self):
        self.templates = {
            "basic": "{instruction}",
            "role_based": "你是{role}。{instruction}",
            "context_based": "背景：{context}\n\n任务：{instruction}",
            "cot": "让我们一步一步地解决这个问题：\n\n问题：{problem}\n\n思考过程：",
            "few_shot": "示例：\n{examples}\n\n现在，请处理：{input}"
        }
    
    def format(self, template_name: str, **kwargs) -> str:
        """格式化提示词模板"""
        return self.templates[template_name].format(**kwargs)
```

### 基本提示策略
```python
class PromptStrategy:
    @staticmethod
    def direct_instruction(task: str) -> str:
        """直接指令"""
        return f"请{task}"
    
    @staticmethod
    def role_based(role: str, task: str) -> str:
        """基于角色"""
        return f"你是一位{role}。{task}"
    
    @staticmethod
    def step_by_step(task: str) -> str:
        """步骤分解"""
        return f"让我们一步一步来完成这个任务：\n\n任务：{task}\n\n1."
```

## 2. 提示词优化技术

### 提示词评估器
```python
class PromptEvaluator:
    def __init__(self):
        self.metrics = ["clarity", "specificity", "relevance"]
    
    def evaluate_prompt(self, prompt: str) -> Dict[str, float]:
        """评估提示词质量"""
        scores = {}
        
        # 清晰度评估
        scores["clarity"] = self._evaluate_clarity(prompt)
        
        # 具体性评估
        scores["specificity"] = self._evaluate_specificity(prompt)
        
        # 相关性评估
        scores["relevance"] = self._evaluate_relevance(prompt)
        
        return scores
    
    def _evaluate_clarity(self, prompt: str) -> float:
        """评估提示词清晰度"""
        # 实现评估逻辑
        return 0.0
    
    def _evaluate_specificity(self, prompt: str) -> float:
        """评估提示词具体性"""
        # 实现评估逻辑
        return 0.0
    
    def _evaluate_relevance(self, prompt: str) -> float:
        """评估提示词相关性"""
        # 实现评估逻辑
        return 0.0
```

### 提示词优化器
```python
class PromptOptimizer:
    def __init__(self):
        self.evaluator = PromptEvaluator()
    
    def optimize(self, prompt: str) -> str:
        """优化提示词"""
        # 添加具体说明
        if not self._has_specific_instruction(prompt):
            prompt = self._add_specific_instruction(prompt)
        
        # 添加示例（如果需要）
        if self._needs_examples(prompt):
            prompt = self._add_examples(prompt)
        
        # 添加输出格式说明
        if not self._has_output_format(prompt):
            prompt = self._add_output_format(prompt)
        
        return prompt
    
    def _has_specific_instruction(self, prompt: str) -> bool:
        """检查是否包含具体说明"""
        # 实现检查逻辑
        return False
    
    def _add_specific_instruction(self, prompt: str) -> str:
        """添加具体说明"""
        return f"{prompt}\n\n请提供详细的步骤和解释。"
    
    def _needs_examples(self, prompt: str) -> bool:
        """检查是否需要示例"""
        # 实现检查逻辑
        return False
    
    def _add_examples(self, prompt: str) -> str:
        """添加示例"""
        return f"{prompt}\n\n示例：\n[示例内容]"
    
    def _has_output_format(self, prompt: str) -> bool:
        """检查是否指定输出格式"""
        # 实现检查逻辑
        return False
    
    def _add_output_format(self, prompt: str) -> str:
        """添加输出格式说明"""
        return f"{prompt}\n\n请按以下格式输出：\n[格式说明]"
```

## 3. 高级提示技术

### Chain of Thought（思维链）
```python
class ChainOfThought:
    def __init__(self):
        self.template = """
让我们一步一步地思考这个问题：

问题：{question}

思考过程：
1. 首先，让我们理解问题的关键点：
   {key_points}

2. 然后，我们可以分析可能的解决方案：
   {analysis}

3. 最后，我们可以得出结论：
   {conclusion}
"""
    
    def generate_prompt(self, question: str) -> str:
        """生成思维链提示词"""
        return self.template.format(
            question=question,
            key_points="[关键点分析]",
            analysis="[解决方案分析]",
            conclusion="[结论]"
        )
```

### Few-shot Learning（少样本学习）
```python
class FewShotLearning:
    def __init__(self):
        self.template = """
以下是一些示例：

{examples}

现在，请处理这个新案例：
{input}

请按照示例的格式输出结果。
"""
    
    def format_example(self, input_text: str, output_text: str) -> str:
        """格式化示例"""
        return f"输入：{input_text}\n输出：{output_text}\n"
    
    def generate_prompt(self, examples: List[Tuple[str, str]], input_text: str) -> str:
        """生成少样本学习提示词"""
        formatted_examples = "\n".join(
            self.format_example(input_text, output_text)
            for input_text, output_text in examples
        )
        
        return self.template.format(
            examples=formatted_examples,
            input=input_text
        )
```

## 4. 特定任务提示模板

### 文本分类
```python
class TextClassificationPrompts:
    @staticmethod
    def binary_classification(text: str) -> str:
        return f"""
请判断以下文本的情感倾向（正面/负面）：

文本：{text}

请只输出"正面"或"负面"。
"""
    
    @staticmethod
    def multi_class_classification(text: str, classes: List[str]) -> str:
        return f"""
请将以下文本分类到这些类别之一：{', '.join(classes)}

文本：{text}

请只输出对应的类别名称。
"""
```

### 信息提取
```python
class InformationExtractionPrompts:
    @staticmethod
    def named_entity_recognition(text: str) -> str:
        return f"""
请从以下文本中提取人名、地名、组织名：

文本：{text}

请按以下格式输出：
人名：[名称1, 名称2, ...]
地名：[名称1, 名称2, ...]
组织名：[名称1, 名称2, ...]
"""
    
    @staticmethod
    def key_information_extraction(text: str, keys: List[str]) -> str:
        return f"""
请从以下文本中提取这些关键信息：{', '.join(keys)}

文本：{text}

请按照给定的关键信息字段逐一输出。
"""
```

## 5. 提示词测试与调试

### 提示词测试器
```python
class PromptTester:
    def __init__(self):
        self.test_cases = []
        self.results = []
    
    def add_test_case(self, prompt: str, expected_output: str):
        """添加测试用例"""
        self.test_cases.append({
            "prompt": prompt,
            "expected": expected_output
        })
    
    async def run_tests(self) -> Dict:
        """运行测试"""
        for case in self.test_cases:
            try:
                response = await openai.ChatCompletion.acreate(
                    model="gpt-3.5-turbo",
                    messages=[{"role": "user", "content": case["prompt"]}]
                )
                
                actual_output = response.choices[0].message.content
                success = self._compare_outputs(actual_output, case["expected"])
                
                self.results.append({
                    "prompt": case["prompt"],
                    "expected": case["expected"],
                    "actual": actual_output,
                    "success": success
                })
                
            except Exception as e:
                self.results.append({
                    "prompt": case["prompt"],
                    "error": str(e)
                })
        
        return self._generate_report()
    
    def _compare_outputs(self, actual: str, expected: str) -> bool:
        """比较输出结果"""
        # 可以实现更复杂的比较逻辑
        return actual.strip() == expected.strip()
    
    def _generate_report(self) -> Dict:
        """生成测试报告"""
        total_tests = len(self.results)
        successful_tests = sum(1 for r in self.results if r.get("success", False))
        
        return {
            "total_tests": total_tests,
            "successful_tests": successful_tests,
            "success_rate": successful_tests / total_tests if total_tests > 0 else 0,
            "detailed_results": self.results
        }
```

### 提示词调试器
```python
class PromptDebugger:
    def __init__(self):
        self.history = []
    
    async def debug_prompt(self, prompt: str) -> Dict:
        """调试提示词"""
        # 记录原始提示词
        self.history.append({
            "type": "original",
            "prompt": prompt
        })
        
        # 测试不同的变体
        variants = self._generate_variants(prompt)
        results = []
        
        for variant in variants:
            try:
                response = await openai.ChatCompletion.acreate(
                    model="gpt-3.5-turbo",
                    messages=[{"role": "user", "content": variant}]
                )
                
                results.append({
                    "variant": variant,
                    "response": response.choices[0].message.content
                })
                
                self.history.append({
                    "type": "variant",
                    "prompt": variant,
                    "response": response.choices[0].message.content
                })
                
            except Exception as e:
                results.append({
                    "variant": variant,
                    "error": str(e)
                })
        
        return {
            "original_prompt": prompt,
            "variants": results,
            "recommendations": self._generate_recommendations(results)
        }
    
    def _generate_variants(self, prompt: str) -> List[str]:
        """生成提示词变体"""
        variants = []
        
        # 添加更多具体说明
        variants.append(f"{prompt}\n请提供详细的步骤和解释。")
        
        # 添加角色设定
        variants.append(f"作为一个专家，{prompt}")
        
        # 添加输出格式要求
        variants.append(f"{prompt}\n请按照以下格式输出：[格式说明]")
        
        return variants
    
    def _generate_recommendations(self, results: List[Dict]) -> List[str]:
        """生成改进建议"""
        recommendations = []
        
        # 分析结果并生成建议
        # 这里可以实现更复杂的分析逻辑
        
        return recommendations
```

## 6. 实践应用

### 智能助手提示系统
```python
class AssistantPromptSystem:
    def __init__(self):
        self.template = PromptTemplate()
        self.optimizer = PromptOptimizer()
        self.debugger = PromptDebugger()
    
    async def generate_response(self, user_input: str, context: Dict = None) -> str:
        """生成助手响应"""
        # 构建提示词
        prompt = self._build_prompt(user_input, context)
        
        # 优化提示词
        optimized_prompt = self.optimizer.optimize(prompt)
        
        try:
            # 生成响应
            response = await openai.ChatCompletion.acreate(
                model="gpt-3.5-turbo",
                messages=[{"role": "user", "content": optimized_prompt}]
            )
            
            return response.choices[0].message.content
            
        except Exception as e:
            return f"生成响应时出错: {str(e)}"
    
    def _build_prompt(self, user_input: str, context: Dict = None) -> str:
        """构建提示词"""
        if context and context.get("role"):
            return self.template.format(
                "role_based",
                role=context["role"],
                instruction=user_input
            )
        
        return self.template.format("basic", instruction=user_input)
```

### 代码生成提示系统
```python
class CodeGenerationPromptSystem:
    def __init__(self):
        self.template = """
请根据以下需求生成Python代码：

需求：{requirement}

要求：
1. 代码应该清晰易懂
2. 包含必要的注释
3. 遵循PEP 8规范
4. 包含适当的错误处理

请生成代码：
"""
    
    async def generate_code(self, requirement: str) -> str:
        """生成代码"""
        prompt = self.template.format(requirement=requirement)
        
        try:
            response = await openai.ChatCompletion.acreate(
                model="gpt-3.5-turbo",
                messages=[{"role": "user", "content": prompt}]
            )
            
            return response.choices[0].message.content
            
        except Exception as e:
            return f"生成代码时出错: {str(e)}"
```

## 7. 练习题

1. 实现一个提示词优化系统：
```python
async def exercise_1():
    """
    实现一个提示词优化系统，要求：
    1. 能够评估提示词质量
    2. 能够提供改进建议
    3. 能够生成优化后的提示词
    """
    optimizer = PromptOptimizer()
    prompt = "讲解Python装饰器"
    
    optimized_prompt = optimizer.optimize(prompt)
    return optimized_prompt
```

2. 实现特定任务的提示词生成器：
```python
async def exercise_2():
    """
    实现一个特定任务的提示词生成器，例如：
    1. 文本分类提示词
    2. 信息提取提示词
    3. 代码生成提示词
    """
    # 实现你的代码
    pass
```

## 小结

本章我们学习了：
- 提示词工程的基本概念
- 提示词优化技术
- 高级提示技术（CoT、Few-shot）
- 特定任务提示模板
- 提示词测试与调试
- 实际应用案例

这些知识将帮助你更好地设计和优化提示词，提高模型输出质量。

## 下一步

在下一章中，我们将学习LangChain应用开发，探索如何使用这个强大的框架来构建复杂的LLM应用。

## 额外资源
- [OpenAI GPT最佳实践](https://platform.openai.com/docs/guides/gpt-best-practices)
- [提示词工程指南](https://github.com/dair-ai/Prompt-Engineering-Guide)
- [LangChain提示词模板](https://python.langchain.com/docs/modules/model_io/prompts/prompt_templates/)
