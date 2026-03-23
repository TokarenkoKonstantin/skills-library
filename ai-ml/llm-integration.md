# 🤖 LLM Integration

## Уровень владения: ⭐⭐⭐⭐⭐ Expert

## Опыт: 2+ года

---

## 📝 Описание

Интеграция больших языковых моделей (LLM) в приложения. Работа с GPT-4, Claude, Llama и другими моделями через API.

## 🎯 Специализация

### Провайдеры
- **OpenAI** - GPT-4, GPT-3.5
- **Anthropic** - Claude 3
- **Groq** - Llama 3, Mixtral
- **Local LLMs** - Ollama

### Техники
- **Prompt Engineering** - оптимизация промптов
- **Function Calling** - вызов функций через LLM
- **Streaming** - потоковые ответы
- **Context Management** - управление контекстом

## 💼 Проекты

### 1. AI Telegram Bot
**Описание:** Бот с интеграцией нескольких LLM

**Код:**
```python
from openai import AsyncOpenAI
from anthropic import AsyncAnthropic

class LLMService:
    def __init__(self):
        self.openai = AsyncOpenAI()
        self.anthropic = AsyncAnthropic()
    
    async def generate_response(
        self,
        messages: list[dict],
        model: str = "gpt-4"
    ) -> str:
        """Генерация ответа через LLM"""
        if model.startswith("gpt"):
            response = await self.openai.chat.completions.create(
                model=model,
                messages=messages,
                temperature=0.7
            )
            return response.choices[0].message.content
        
        elif model.startswith("claude"):
            response = await self.anthropic.messages.create(
                model=model,
                messages=messages,
                max_tokens=1024
            )
            return response.content[0].text
```

### 2. Function Calling
**Описание:** LLM вызывает функции для выполнения действий

**Код:**
```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Получить погоду для города",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "Название города"
                    }
                },
                "required": ["city"]
            }
        }
    }
]

async def process_with_tools(user_message: str):
    """Обработка с использованием tools"""
    response = await client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": user_message}],
        tools=tools,
        tool_choice="auto"
    )
    
    # Если LLM хочет вызвать функцию
    if response.choices[0].message.tool_calls:
        tool_call = response.choices[0].message.tool_calls[0]
        function_name = tool_call.function.name
        arguments = json.loads(tool_call.function.arguments)
        
        # Вызываем функцию
        result = await execute_function(function_name, arguments)
        
        # Отправляем результат обратно в LLM
        final_response = await client.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "user", "content": user_message},
                response.choices[0].message,
                {
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": str(result)
                }
            ]
        )
        return final_response.choices[0].message.content
```

### 3. Streaming Responses
**Описание:** Потоковая генерация для лучшего UX

**Код:**
```python
async def stream_response(prompt: str):
    """Потоковая генерация ответа"""
    stream = await client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    
    async for chunk in stream:
        if chunk.choices[0].delta.content:
            yield chunk.choices[0].delta.content

# Использование в Telegram
@router.message(F.text)
async def handle_message(message: Message):
    response_message = await message.answer("Думаю...")
    
    full_response = ""
    async for chunk in stream_response(message.text):
        full_response += chunk
        # Обновляем сообщение каждые 50 символов
        if len(full_response) % 50 == 0:
            await response_message.edit_text(full_response)
    
    await response_message.edit_text(full_response)
```

## 🛠️ Инструменты

### API Клиенты
- **openai** - OpenAI API
- **anthropic** - Claude API
- **groq** - Groq API
- **litellm** - Unified API для всех LLM

### Frameworks
- **LangChain** - LLM orchestration
- **LlamaIndex** - Data framework
- **Instructor** - Structured outputs

## 📚 Best Practices

### 1. Управление токенами
```python
import tiktoken

def count_tokens(text: str, model: str = "gpt-4") -> int:
    """Подсчёт токенов"""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

def truncate_to_tokens(text: str, max_tokens: int) -> str:
    """Обрезка текста до max_tokens"""
    encoding = tiktoken.encoding_for_model("gpt-4")
    tokens = encoding.encode(text)
    if len(tokens) <= max_tokens:
        return text
    return encoding.decode(tokens[:max_tokens])
```

### 2. Retry Logic
```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
async def call_llm_with_retry(prompt: str) -> str:
    """LLM вызов с автоматическими повторами"""
    response = await client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content
```

### 3. Cost Tracking
```python
class CostTracker:
    PRICES = {
        "gpt-4": {"input": 0.03, "output": 0.06},  # per 1K tokens
        "gpt-3.5-turbo": {"input": 0.0015, "output": 0.002}
    }
    
    def calculate_cost(
        self,
        model: str,
        input_tokens: int,
        output_tokens: int
    ) -> float:
        """Расчёт стоимости запроса"""
        prices = self.PRICES.get(model, {"input": 0, "output": 0})
        input_cost = (input_tokens / 1000) * prices["input"]
        output_cost = (output_tokens / 1000) * prices["output"]
        return input_cost + output_cost
```

## 🎓 Техники

### Chain of Thought
```python
COT_PROMPT = """
Реши задачу пошагово:

Задача: {problem}

Давай рассуждать шаг за шагом:
1. [Первый шаг]
2. [Второй шаг]
...

Ответ: [финальный ответ]
"""
```

### Few-Shot Learning
```python
FEW_SHOT_PROMPT = """
Классифицируй отзыв как позитивный или негативный.

Примеры:
Отзыв: "Отличный продукт!"
Класс: Позитивный

Отзыв: "Ужасное качество"
Класс: Негативный

Теперь классифицируй:
Отзыв: {review}
Класс:
"""
```

## 📖 Ресурсы

- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Anthropic Claude Documentation](https://docs.anthropic.com/)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)

---

**Последнее обновление:** 2026-03-23
