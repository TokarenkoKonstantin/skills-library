# 🐍 Python

## Уровень владения: ⭐⭐⭐⭐⭐ Expert

## Опыт: 5+ лет

---

## 📝 Описание

Python - мой основной язык программирования. Использую для разработки AI-приложений, Telegram ботов, REST API и автоматизации.

## 🎯 Специализация

### Backend разработка
- **FastAPI** - создание REST API
- **aiogram** - Telegram боты
- **asyncio** - асинхронное программирование

### AI/ML
- **LangChain** - LLM приложения
- **OpenAI API** - интеграция GPT
- **Pydantic** - валидация данных

### Работа с данными
- **SQLAlchemy** - ORM для БД
- **Pandas** - анализ данных
- **asyncpg** - асинхронный PostgreSQL

## 💼 Проекты

### 1. Второй Мозг (vtoroi-mozg)
**Описание:** Telegram бот для управления задачами с AI

**Использованные технологии:**
- Python 3.12+
- aiogram 3.x - Telegram bot framework
- FastAPI - REST API
- asyncio - асинхронная обработка
- Pydantic - валидация данных
- SQLAlchemy - работа с БД

**Ключевые фичи:**
```python
# Асинхронный обработчик голосовых сообщений
@router.message(F.voice)
async def handle_voice(message: Message):
    # Скачивание файла
    file = await bot.get_file(message.voice.file_id)
    audio_bytes = await bot.download_file(file.file_path)
    
    # Транскрипция через Deepgram
    text = await deepgram_service.transcribe(audio_bytes)
    
    # AI обработка через Groq
    result = await ai_service.process(text)
    
    # Создание задачи в Todoist
    await todoist_service.create_task(result)
```

### 2. RAG Module (neuroforge)
**Описание:** Модуль для семантического поиска

**Код:**
```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma

class RAGModule:
    def __init__(self):
        self.embeddings = OpenAIEmbeddings()
        self.vectorstore = Chroma(embedding_function=self.embeddings)
    
    async def add_documents(self, documents: list[str]):
        """Добавление документов в vector store"""
        await self.vectorstore.aadd_texts(documents)
    
    async def search(self, query: str, k: int = 5):
        """Семантический поиск"""
        results = await self.vectorstore.asimilarity_search(query, k=k)
        return results
```

## 🛠️ Инструменты и библиотеки

### Основные
- **Python 3.12+** - последняя версия
- **uv** - быстрый package manager
- **Poetry** - управление зависимостями
- **pytest** - тестирование
- **black** - форматирование кода
- **mypy** - type checking

### Frameworks
- **FastAPI** - REST API
- **aiogram** - Telegram bots
- **LangChain** - LLM applications
- **Pydantic** - data validation

### Async
- **asyncio** - асинхронное программирование
- **aiohttp** - async HTTP клиент
- **asyncpg** - async PostgreSQL
- **aioboto3** - async AWS SDK

## 📚 Best Practices

### Type Hints
```python
from typing import Optional, List, Dict

async def process_data(
    items: List[str],
    config: Dict[str, any],
    timeout: Optional[int] = None
) -> List[Dict[str, str]]:
    """Обработка данных с type hints"""
    results = []
    for item in items:
        result = await process_item(item, config)
        results.append(result)
    return results
```

### Error Handling
```python
from typing import Union

async def safe_api_call(url: str) -> Union[dict, None]:
    """Безопасный API вызов с обработкой ошибок"""
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(url, timeout=10) as response:
                response.raise_for_status()
                return await response.json()
    except aiohttp.ClientError as e:
        logger.error(f"API call failed: {e}")
        return None
    except asyncio.TimeoutError:
        logger.error("Request timeout")
        return None
```

### Async Context Managers
```python
class DatabaseConnection:
    async def __aenter__(self):
        self.conn = await asyncpg.connect(DATABASE_URL)
        return self.conn
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()

# Использование
async with DatabaseConnection() as conn:
    result = await conn.fetch("SELECT * FROM users")
```

## 🎓 Сертификаты

- Python для профессионалов
- Async Python Programming
- FastAPI Best Practices

## 📖 Ресурсы

- [Python Documentation](https://docs.python.org/3/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Real Python](https://realpython.com/)

---

**Последнее обновление:** 2026-03-23
