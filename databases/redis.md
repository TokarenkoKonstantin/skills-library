# ⚡ Redis

## Уровень владения: ⭐⭐⭐⭐ Advanced

## Опыт: 3+ года

---

## 📝 Описание

Redis - in-memory база данных для кеширования, очередей, сессий и real-time данных.

## 🎯 Специализация

### Основные навыки
- **Caching** - кеширование данных
- **Sessions** - хранение сессий
- **Pub/Sub** - real-time сообщения
- **Queues** - очереди задач
- **Rate limiting** - ограничение запросов

### Структуры данных
- **Strings** - простые значения
- **Hashes** - объекты
- **Lists** - списки
- **Sets** - множества
- **Sorted Sets** - отсортированные множества

## 💼 Проекты

### 1. Кеширование в Telegram боте
**Описание:** Кеш для частых запросов

**Код:**
```python
import redis.asyncio as redis
from typing import Optional
import json

class CacheService:
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url)
    
    async def get(self, key: str) -> Optional[dict]:
        """Получение из кеша"""
        value = await self.redis.get(key)
        if value:
            return json.loads(value)
        return None
    
    async def set(
        self,
        key: str,
        value: dict,
        ttl: int = 3600
    ):
        """Сохранение в кеш с TTL"""
        await self.redis.setex(
            key,
            ttl,
            json.dumps(value)
        )
    
    async def delete(self, key: str):
        """Удаление из кеша"""
        await self.redis.delete(key)

# Использование
cache = CacheService("redis://localhost:6379")

# Кеширование результата API
async def get_user_data(user_id: int):
    cache_key = f"user:{user_id}"
    
    # Проверка кеша
    cached = await cache.get(cache_key)
    if cached:
        return cached
    
    # Запрос к API
    data = await api.get_user(user_id)
    
    # Сохранение в кеш на 1 час
    await cache.set(cache_key, data, ttl=3600)
    
    return data
```

### 2. Rate Limiting
**Описание:** Ограничение частоты запросов

**Код:**
```python
class RateLimiter:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    async def is_allowed(
        self,
        user_id: int,
        limit: int = 10,
        window: int = 60
    ) -> bool:
        """Проверка rate limit"""
        key = f"rate_limit:{user_id}"
        
        # Инкремент счётчика
        count = await self.redis.incr(key)
        
        # Установка TTL при первом запросе
        if count == 1:
            await self.redis.expire(key, window)
        
        return count <= limit

# Использование в Telegram боте
@router.message(F.text)
async def handle_message(message: Message):
    # Проверка rate limit
    if not await rate_limiter.is_allowed(message.from_user.id):
        await message.answer("Слишком много запросов. Подождите минуту.")
        return
    
    # Обработка сообщения
    await process_message(message)
```

### 3. Pub/Sub для real-time уведомлений
**Описание:** Real-time коммуникация между сервисами

**Код:**
```python
class PubSubService:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.pubsub = self.redis.pubsub()
    
    async def publish(self, channel: str, message: dict):
        """Публикация сообщения"""
        await self.redis.publish(
            channel,
            json.dumps(message)
        )
    
    async def subscribe(self, channel: str):
        """Подписка на канал"""
        await self.pubsub.subscribe(channel)
        
        async for message in self.pubsub.listen():
            if message['type'] == 'message':
                data = json.loads(message['data'])
                yield data

# Publisher (сервис 1)
await pubsub.publish('notifications', {
    'user_id': 123,
    'type': 'new_task',
    'data': {...}
})

# Subscriber (сервис 2)
async for notification in pubsub.subscribe('notifications'):
    await send_telegram_notification(notification)
```

### 4. Очереди задач
**Описание:** Background jobs с Redis

**Код:**
```python
class TaskQueue:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.queue_name = "tasks"
    
    async def enqueue(self, task: dict):
        """Добавление задачи в очередь"""
        await self.redis.rpush(
            self.queue_name,
            json.dumps(task)
        )
    
    async def dequeue(self) -> Optional[dict]:
        """Получение задачи из очереди"""
        # BLPOP - блокирующее получение
        result = await self.redis.blpop(self.queue_name, timeout=5)
        if result:
            _, task_data = result
            return json.loads(task_data)
        return None
    
    async def get_queue_size(self) -> int:
        """Размер очереди"""
        return await self.redis.llen(self.queue_name)

# Producer
await queue.enqueue({
    'type': 'send_email',
    'to': 'user@example.com',
    'subject': 'Hello'
})

# Worker
while True:
    task = await queue.dequeue()
    if task:
        await process_task(task)
```

## 🛠️ Инструменты

### Python библиотеки
- **redis-py** - синхронный клиент
- **redis.asyncio** - асинхронный клиент
- **aioredis** - альтернативный async клиент

### CLI
```bash
# Подключение
redis-cli

# Основные команды
SET key value
GET key
DEL key
EXISTS key
EXPIRE key 60

# Списки
LPUSH mylist value
RPUSH mylist value
LRANGE mylist 0 -1

# Хеши
HSET user:1 name "Konstantin"
HGET user:1 name
HGETALL user:1

# Мониторинг
INFO
MONITOR
SLOWLOG GET 10
```

## 📚 Best Practices

### 1. Connection pooling
```python
# Плохо: новое подключение каждый раз
async def get_data(key):
    redis = await redis.from_url("redis://localhost")
    return await redis.get(key)

# Хорошо: connection pool
redis_pool = redis.ConnectionPool.from_url("redis://localhost")
redis_client = redis.Redis(connection_pool=redis_pool)

async def get_data(key):
    return await redis_client.get(key)
```

### 2. Naming conventions
```python
# Используй префиксы и разделители
user_cache = f"cache:user:{user_id}"
session_key = f"session:{session_id}"
rate_limit = f"rate_limit:{user_id}:{endpoint}"
```

### 3. TTL для всех ключей
```python
# Всегда устанавливай TTL
await redis.setex("key", 3600, "value")  # 1 час

# Или после создания
await redis.set("key", "value")
await redis.expire("key", 3600)
```

### 4. Pipeline для batch операций
```python
# Плохо: много round-trips
for i in range(100):
    await redis.set(f"key:{i}", i)

# Хорошо: pipeline
pipe = redis.pipeline()
for i in range(100):
    pipe.set(f"key:{i}", i)
await pipe.execute()
```

## 🎓 Продвинутые техники

### 1. Distributed locks
```python
from redis.lock import Lock

async def process_with_lock(resource_id: str):
    """Обработка с distributed lock"""
    lock = Lock(redis, f"lock:{resource_id}", timeout=10)
    
    if await lock.acquire(blocking=False):
        try:
            # Критическая секция
            await process_resource(resource_id)
        finally:
            await lock.release()
    else:
        print("Resource is locked by another process")
```

### 2. Sorted sets для leaderboards
```python
class Leaderboard:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.key = "leaderboard"
    
    async def add_score(self, user_id: int, score: int):
        """Добавление очков"""
        await self.redis.zadd(self.key, {user_id: score})
    
    async def get_top(self, n: int = 10):
        """Топ N пользователей"""
        return await self.redis.zrevrange(
            self.key, 0, n-1, withscores=True
        )
    
    async def get_rank(self, user_id: int):
        """Позиция пользователя"""
        return await self.redis.zrevrank(self.key, user_id)
```

### 3. Bloom filters
```python
# Для проверки существования без false negatives
from redis.commands.bf import BFCommands

bf = BFCommands(redis)

# Создание bloom filter
await bf.create("users:seen", 0.01, 10000)

# Добавление
await bf.add("users:seen", "user:123")

# Проверка
exists = await bf.exists("users:seen", "user:123")
```

## 📖 Ресурсы

- [Redis Documentation](https://redis.io/docs/)
- [Redis Best Practices](https://redis.io/docs/manual/patterns/)
- [redis-py Documentation](https://redis-py.readthedocs.io/)

---

**Последнее обновление:** 2026-03-23
