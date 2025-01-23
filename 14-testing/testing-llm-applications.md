# Testing LLM Applications

## Introduction

Testing LLM applications requires a unique approach due to their probabilistic nature and complex interactions. This guide covers testing strategies and best practices specific to LLM applications.

## Testing Fundamentals

### 1. Unit Testing LLM Components

```python
import pytest
from unittest.mock import Mock, patch

def test_prompt_generation():
    prompt_generator = PromptGenerator()
    prompt = prompt_generator.generate(
        template="Answer the question: {question}",
        question="What is Python?"
    )
    assert "What is Python?" in prompt

@pytest.mark.asyncio
async def test_llm_client():
    with patch('openai.ChatCompletion.acreate') as mock_create:
        mock_create.return_value = {
            'choices': [{'message': {'content': 'Test response'}}]
        }
        client = LLMClient()
        response = await client.generate_response("Test prompt")
        assert response == "Test response"
```

### 2. Integration Testing

```python
class TestLLMIntegration:
    @pytest.fixture
    def llm_app(self):
        return LLMApplication(
            model="gpt-3.5-turbo",
            temperature=0.7
        )
    
    def test_end_to_end_flow(self, llm_app):
        input_text = "Translate to French: Hello, world!"
        response = llm_app.process_request(input_text)
        assert isinstance(response, str)
        assert len(response) > 0
```

## Testing Strategies

### 1. Response Validation

```python
class ResponseValidator:
    def validate_format(self, response, expected_format):
        """Validate response format against schema"""
        try:
            jsonschema.validate(response, expected_format)
            return True
        except jsonschema.exceptions.ValidationError:
            return False

    def validate_content(self, response, criteria):
        """Validate response content against specific criteria"""
        return all(criterion(response) for criterion in criteria)

def test_response_validation():
    validator = ResponseValidator()
    response = {"text": "Test response", "confidence": 0.95}
    schema = {
        "type": "object",
        "properties": {
            "text": {"type": "string"},
            "confidence": {"type": "number"}
        }
    }
    assert validator.validate_format(response, schema)
```

### 2. Prompt Testing

```python
class PromptTester:
    def test_prompt_variations(self, prompt_template, test_cases):
        """Test prompt template with different inputs"""
        results = []
        for case in test_cases:
            prompt = prompt_template.format(**case['inputs'])
            response = self.get_llm_response(prompt)
            results.append({
                'case': case,
                'response': response,
                'valid': self.validate_response(response, case['expected'])
            })
        return results
```

## Performance Testing

### 1. Response Time Testing

```python
import time

class PerformanceTester:
    def measure_response_time(self, llm_client, prompts, iterations=10):
        """Measure average response time for a set of prompts"""
        times = []
        for _ in range(iterations):
            for prompt in prompts:
                start_time = time.time()
                _ = llm_client.generate_response(prompt)
                end_time = time.time()
                times.append(end_time - start_time)
        return {
            'avg_time': sum(times) / len(times),
            'max_time': max(times),
            'min_time': min(times)
        }
```

### 2. Token Usage Monitoring

```python
class TokenUsageMonitor:
    def track_token_usage(self, llm_client, test_suite):
        """Monitor token usage across test cases"""
        usage_stats = {
            'prompt_tokens': 0,
            'completion_tokens': 0,
            'total_tokens': 0
        }
        for test_case in test_suite:
            response = llm_client.generate_response(
                test_case['prompt'],
                return_usage=True
            )
            usage_stats['prompt_tokens'] += response['usage']['prompt_tokens']
            usage_stats['completion_tokens'] += response['usage']['completion_tokens']
            usage_stats['total_tokens'] += response['usage']['total_tokens']
        return usage_stats
```

## Error Handling Testing

### 1. API Error Simulation

```python
@pytest.mark.parametrize("error_type,expected_handling", [
    (openai.error.RateLimitError, "rate_limit_backoff"),
    (openai.error.InvalidRequestError, "invalid_request_handler"),
    (openai.error.APIError, "api_error_handler")
])
def test_error_handling(error_type, expected_handling):
    with patch('openai.ChatCompletion.acreate') as mock_create:
        mock_create.side_effect = error_type("Test error")
        llm_client = LLMClient()
        response = llm_client.generate_response("Test prompt")
        assert response['error_handling'] == expected_handling
```

## Best Practices

1. Always mock LLM API calls in unit tests
2. Use deterministic test cases when possible
3. Implement comprehensive error handling tests
4. Monitor and log token usage in tests
5. Use parameterized testing for different scenarios
6. Implement response validation for all test cases
7. Regular performance benchmark testing

## Common Testing Patterns

### 1. Test Data Management

```python
class TestDataManager:
    def __init__(self, data_path):
        self.data_path = data_path
        self.test_cases = self.load_test_cases()
    
    def load_test_cases(self):
        """Load test cases from JSON file"""
        with open(self.data_path) as f:
            return json.load(f)
    
    def get_test_suite(self, suite_name):
        """Get specific test suite"""
        return self.test_cases.get(suite_name, [])
```

### 2. Test Result Analysis

```python
class TestResultAnalyzer:
    def analyze_results(self, test_results):
        """Analyze test results and generate report"""
        return {
            'total_tests': len(test_results),
            'passed_tests': sum(1 for r in test_results if r['passed']),
            'failed_tests': sum(1 for r in test_results if not r['passed']),
            'average_response_time': sum(r['response_time'] for r in test_results) / len(test_results),
            'total_token_usage': sum(r['token_usage'] for r in test_results)
        }
```

## Conclusion

Testing LLM applications requires a combination of traditional testing practices and specialized approaches for handling the unique characteristics of language models. Regular testing and monitoring help ensure reliable and efficient LLM applications.