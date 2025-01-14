# 第七章：文件操作与数据处理基础

本章将介绍在LLM应用开发中常见的文件操作和数据处理方法，重点关注文本处理和数据预处理。

## 1. 文本文件处理

### 读取文本文件
```python
# 基本文本文件读取
def read_text_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        return file.read()

# 按行读取
def read_lines(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        return file.readlines()

# 大文件处理（适用于处理大量提示词或语料）
def process_large_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        for line in file:  # 逐行读取，节省内存
            yield line.strip()
```

### 写入文本文件
```python
# 保存对话历史
def save_conversation(conversation, file_path):
    with open(file_path, 'w', encoding='utf-8') as file:
        for message in conversation:
            file.write(f"{message['role']}: {message['content']}\n")

# 追加写入（适用于日志记录）
def log_interaction(prompt, response, log_file):
    with open(log_file, 'a', encoding='utf-8') as file:
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        file.write(f"[{timestamp}]\n")
        file.write(f"Prompt: {prompt}\n")
        file.write(f"Response: {response}\n\n")
```

## 2. 文本预处理

### 基本文本清理
```python
import re

def clean_text(text):
    """基本文本清理"""
    # 移除多余的空白字符
    text = ' '.join(text.split())
    # 移除特殊字符
    text = re.sub(r'[^\w\s]', '', text)
    return text.strip()

def normalize_text(text):
    """文本标准化"""
    # 转换为小写
    text = text.lower()
    # 移除数字
    text = re.sub(r'\d+', '', text)
    # 移除多余空格
    text = ' '.join(text.split())
    return text
```

### 文本分段
```python
def split_into_chunks(text, max_length=2000):
    """将长文本分割成适合LLM处理的片段"""
    # 按句子分割
    sentences = re.split(r'(?<=[.!?])\s+', text)
    chunks = []
    current_chunk = ""
    
    for sentence in sentences:
        if len(current_chunk) + len(sentence) < max_length:
            current_chunk += sentence + " "
        else:
            chunks.append(current_chunk.strip())
            current_chunk = sentence + " "
    
    if current_chunk:
        chunks.append(current_chunk.strip())
    
    return chunks
```

## 3. 数据格式转换

### CSV处理
```python
import csv

def read_prompts_from_csv(file_path):
    """从CSV文件读取提示词"""
    prompts = []
    with open(file_path, 'r', encoding='utf-8') as file:
        reader = csv.DictReader(file)
        for row in reader:
            prompts.append({
                'category': row['category'],
                'prompt': row['prompt'],
                'parameters': row.get('parameters', '')
            })
    return prompts

def save_responses_to_csv(responses, file_path):
    """将API响应保存到CSV"""
    with open(file_path, 'w', encoding='utf-8', newline='') as file:
        writer = csv.DictWriter(file, fieldnames=['prompt', 'response', 'timestamp'])
        writer.writeheader()
        for resp in responses:
            writer.writerow(resp)
```

### Excel处理
```python
import pandas as pd

def process_excel_data(file_path):
    """处理Excel中的数据"""
    # 读取Excel文件
    df = pd.read_excel(file_path)
    
    # 基本数据清理
    df['text'] = df['text'].fillna('')  # 填充空值
    df['text'] = df['text'].str.strip() # 清理空白
    
    # 转换为字典列表
    records = df.to_dict('records')
    return records
```

## 4. 提示词模板管理

### 模板系统
```python
class PromptTemplateManager:
    def __init__(self, template_dir):
        self.template_dir = template_dir
        self.templates = self.load_templates()
    
    def load_templates(self):
        """加载所有模板文件"""
        templates = {}
        template_files = glob.glob(f"{self.template_dir}/*.txt")
        
        for file_path in template_files:
            name = os.path.basename(file_path).replace('.txt', '')
            with open(file_path, 'r', encoding='utf-8') as f:
                templates[name] = f.read()
        
        return templates
    
    def get_prompt(self, template_name, **kwargs):
        """使用模板生成提示词"""
        if template_name not in self.templates:
            raise KeyError(f"Template '{template_name}' not found")
        
        template = self.templates[template_name]
        return template.format(**kwargs)
```

## 5. 实际应用示例

### 文档处理器
```python
class DocumentProcessor:
    def __init__(self, model="gpt-3.5-turbo"):
        self.model = model
        self.chunk_size = 2000
    
    def process_document(self, file_path):
        """处理文档并生成摘要"""
        # 读取文档
        text = self.read_document(file_path)
        
        # 分割文本
        chunks = split_into_chunks(text, self.chunk_size)
        
        # 处理每个片段
        summaries = []
        for chunk in chunks:
            summary = self.get_chunk_summary(chunk)
            summaries.append(summary)
        
        # 合并摘要
        final_summary = self.combine_summaries(summaries)
        return final_summary
    
    def read_document(self, file_path):
        """根据文件类型读取文档"""
        ext = file_path.lower().split('.')[-1]
        
        if ext == 'txt':
            with open(file_path, 'r', encoding='utf-8') as f:
                return f.read()
        elif ext == 'pdf':
            # 需要安装pdfplumber库
            import pdfplumber
            with pdfplumber.open(file_path) as pdf:
                return '\n'.join(page.extract_text() for page in pdf.pages)
        else:
            raise ValueError(f"Unsupported file type: {ext}")
    
    def get_chunk_summary(self, chunk):
        """获取文本片段的摘要"""
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "请对以下文本生成简洁的摘要。"},
                {"role": "user", "content": chunk}
            ]
        )
        return response.choices[0].message.content
    
    def combine_summaries(self, summaries):
        """合并多个摘要"""
        combined_text = "\n".join(summaries)
        response = openai.ChatCompletion.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "请将以下多个摘要合并为一个连贯的总结。"},
                {"role": "user", "content": combined_text}
            ]
        )
        return response.choices[0].message.content
```

## 6. 练习题

1. 创建一个提示词管理系统：
```python
"""
创建一个系统，可以：
1. 从文件加载提示词模板
2. 管理不同类型的提示词
3. 记录提示词使用情况
"""

class PromptManager:
    def __init__(self):
        self.templates = {}
        self.usage_log = []
    
    def add_template(self, name, template_file):
        """添加新模板"""
        with open(template_file, 'r', encoding='utf-8') as f:
            self.templates[name] = f.read()
    
    def get_prompt(self, template_name, **kwargs):
        """获取格式化的提示词"""
        if template_name not in self.templates:
            raise KeyError(f"Template not found: {template_name}")
        
        prompt = self.templates[template_name].format(**kwargs)
        self.log_usage(template_name)
        return prompt
    
    def log_usage(self, template_name):
        """记录使用情况"""
        self.usage_log.append({
            'template': template_name,
            'timestamp': datetime.now().isoformat()
        })
    
    def save_usage_stats(self, output_file):
        """保存使用统计"""
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump(self.usage_log, f, indent=2)

# 使用示例
manager = PromptManager()
manager.add_template('summary', 'templates/summary.txt')
prompt = manager.get_prompt('summary', text="需要总结的文本")
```

## 小结

本章我们学习了：
- 文本文件的读写操作
- 文本预处理技术
- 数据格式转换
- 提示词模板管理
- 文档处理实践

这些技能对于处理和准备LLM输入数据、管理提示词模板以及处理API响应至关重要。

## 下一步

在下一章中，我们将学习简单Web应用开发，这将帮助我们创建基于LLM的Web服务。

## 额外资源
- [Python文件操作文档](https://docs.python.org/zh-cn/3/tutorial/inputoutput.html#reading-and-writing-files)
- [Pandas文档](https://pandas.pydata.org/docs/)
- [正则表达式教程](https://docs.python.org/zh-cn/3/howto/regex.html)
