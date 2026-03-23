# 🐘 PostgreSQL

## Уровень владения: ⭐⭐⭐⭐ Advanced

## Опыт: 3+ года

---

## 📝 Описание

PostgreSQL - моя основная реляционная база данных. Использую для хранения данных приложений, пользователей, транзакций.

## 🎯 Специализация

### Основные навыки
- **SQL запросы** - SELECT, JOIN, GROUP BY, CTE
- **Индексы** - B-tree, GiST, GIN
- **Транзакции** - ACID, изоляция
- **Оптимизация** - EXPLAIN, query planning
- **Репликация** - master-slave, streaming

### Продвинутые
- **JSON/JSONB** - работа с JSON данными
- **Full-text search** - поиск по тексту
- **Partitioning** - разделение таблиц
- **Extensions** - pg_trgm, pgvector

## 💼 Проекты

### 1. База данных для Telegram бота
**Описание:** Хранение пользователей, задач, подписок

**Схема:**
```sql
-- Пользователи
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    telegram_id BIGINT UNIQUE NOT NULL,
    username VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    subscription_tier VARCHAR(50) DEFAULT 'free'
);

-- Задачи
CREATE TABLE tasks (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP
);

-- Индексы для быстрого поиска
CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_created_at ON tasks(created_at DESC);
```

### 2. Работа с JSON данными
**Описание:** Хранение метаданных в JSONB

**Код:**
```sql
-- Таблица с JSONB
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(100),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Индекс для JSONB
CREATE INDEX idx_events_metadata ON events USING GIN (metadata);

-- Запросы к JSONB
-- Поиск по ключу
SELECT * FROM events 
WHERE metadata->>'user_id' = '12345';

-- Поиск по вложенному ключу
SELECT * FROM events 
WHERE metadata->'user'->>'name' = 'Konstantin';

-- Проверка существования ключа
SELECT * FROM events 
WHERE metadata ? 'error';
```

### 3. Full-text search
**Описание:** Поиск по тексту с ранжированием

**Код:**
```sql
-- Добавление tsvector колонки
ALTER TABLE articles 
ADD COLUMN search_vector tsvector;

-- Обновление search_vector
UPDATE articles 
SET search_vector = 
    setweight(to_tsvector('russian', title), 'A') ||
    setweight(to_tsvector('russian', content), 'B');

-- Индекс для full-text search
CREATE INDEX idx_articles_search 
ON articles USING GIN (search_vector);

-- Поиск с ранжированием
SELECT 
    title,
    ts_rank(search_vector, query) AS rank
FROM articles, 
     to_tsquery('russian', 'python & программирование') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

## 🛠️ Инструменты

### Python библиотеки
- **asyncpg** - асинхронный драйвер
- **SQLAlchemy** - ORM
- **psycopg2** - синхронный драйвер
- **alembic** - миграции

### Администрирование
- **pgAdmin** - GUI для управления
- **pg_dump/pg_restore** - бэкапы
- **pg_stat_statements** - мониторинг запросов

## 📚 Best Practices

### 1. Использование asyncpg
```python
import asyncpg

class Database:
    def __init__(self, dsn: str):
        self.dsn = dsn
        self.pool = None
    
    async def connect(self):
        """Создание connection pool"""
        self.pool = await asyncpg.create_pool(
            self.dsn,
            min_size=5,
            max_size=20,
            command_timeout=60
        )
    
    async def fetch_users(self, limit: int = 100):
        """Получение пользователей"""
        async with self.pool.acquire() as conn:
            rows = await conn.fetch(
                "SELECT * FROM users LIMIT $1",
                limit
            )
            return [dict(row) for row in rows]
    
    async def create_task(self, user_id: int, title: str):
        """Создание задачи"""
        async with self.pool.acquire() as conn:
            return await conn.fetchrow(
                """
                INSERT INTO tasks (user_id, title)
                VALUES ($1, $2)
                RETURNING id, created_at
                """,
                user_id, title
            )
```

### 2. Транзакции
```python
async def transfer_money(from_user: int, to_user: int, amount: float):
    """Перевод денег с транзакцией"""
    async with pool.acquire() as conn:
        async with conn.transaction():
            # Снятие со счёта
            await conn.execute(
                "UPDATE accounts SET balance = balance - $1 WHERE user_id = $2",
                amount, from_user
            )
            
            # Пополнение счёта
            await conn.execute(
                "UPDATE accounts SET balance = balance + $1 WHERE user_id = $2",
                amount, to_user
            )
            
            # Если ошибка - автоматический rollback
```

### 3. Оптимизация запросов
```sql
-- Плохо: N+1 запросов
SELECT * FROM users;
-- Для каждого user: SELECT * FROM tasks WHERE user_id = ?

-- Хорошо: JOIN
SELECT 
    u.*,
    json_agg(t.*) as tasks
FROM users u
LEFT JOIN tasks t ON t.user_id = u.id
GROUP BY u.id;

-- Использование EXPLAIN для анализа
EXPLAIN ANALYZE
SELECT * FROM tasks 
WHERE user_id = 123 
AND status = 'pending';
```

## 🎓 Продвинутые техники

### 1. CTE (Common Table Expressions)
```sql
WITH active_users AS (
    SELECT user_id, COUNT(*) as task_count
    FROM tasks
    WHERE created_at > NOW() - INTERVAL '30 days'
    GROUP BY user_id
    HAVING COUNT(*) > 5
)
SELECT u.*, au.task_count
FROM users u
JOIN active_users au ON u.id = au.user_id;
```

### 2. Window Functions
```sql
-- Ранжирование пользователей по количеству задач
SELECT 
    user_id,
    COUNT(*) as task_count,
    RANK() OVER (ORDER BY COUNT(*) DESC) as rank
FROM tasks
GROUP BY user_id;
```

### 3. Partitioning
```sql
-- Партиционирование по дате
CREATE TABLE events (
    id BIGSERIAL,
    event_type VARCHAR(100),
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Партиции по месяцам
CREATE TABLE events_2026_03 PARTITION OF events
FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');
```

## 📖 Ресурсы

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Use The Index, Luke](https://use-the-index-luke.com/)
- [PostgreSQL Exercises](https://pgexercises.com/)

---

**Последнее обновление:** 2026-03-23
