# 🔍 RAG Systems (Retrieval Augmented Generation)

## Уровень владения: ⭐⭐⭐⭐⭐ Expert

## Опыт: 2+ года

---

## 📝 Описание

RAG (Retrieval Augmented Generation) - техника, которая позволяет LLM получать доступ к внешним документам и базам знаний для генерации более точных ответов.

## 🎯 Специализация

### Компоненты RAG
- **Document Loading** - загрузка документов
- **Text Splitting** - разбиение на чанки
- **Embeddings** - векторизация текста
- **Vector Store** - хранение векторов
- **Retrieval** - поиск релевантных документов
- **Generation** - генерация ответа

### Технологии
- **LangChain** - RAG framework
- **LlamaIndex** - data framework
- **ChromaDB** - vector database
- **OpenAI Embeddings** - векторизация

## 💼 Проекты

### 1. RAG Module для чат-бота
**Описание:** Модуль для семантического поиска по документам

**Архитектура:**
```
User Query
    ↓
Embedding (OpenAI)
    ↓
Vector Search (ChromaDB)
    ↓
Top K Documents
    ↓
Context + Query → LLM
    ↓
Response
```

**Код:**
```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

class RAGSystem:
    def __init__(self):
        self.embeddings = OpenAIEmbeddings()
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200
        )
        self.vectorstore = None
        self.qa_chain = None
    
    async def load_documents(self, documents: list[str]):
        """Загрузка и индексация документов"""
        # Разбиение на чанки
        chunks = self.text_splitter.create_documents(documents)
        
        # Создание vector store
        self.vectorstore = await Chroma.afrom_documents(
            documents=chunks,
            embedding=self.embeddings,
            persist_directory="./chroma_db"
        )
        
        # Создание QA chain
        self.qa_chain = RetrievalQA.from_chain_type(
            llm=OpenAI(temperature=0),
            chain_type="stuff",
            retriever=self.vectorstore.as_retriever(
                search_kwargs={"k": 3}
            )
        )
    
    async def query(self, question: str) -> dict:
        """Запрос к RAG системе"""
        if not self.qa_chain:
            raise ValueError("Documents not loaded")
        
        # Получение ответа
        result = await self.qa_chain.ainvoke({"query": question})
        
        # Получение источников
        docs = await self.vectorstore.asimilarity_search(question, k=3)
        sources = [doc.metadata.get("source", "Unknown") for doc in docs]
        
        return {
            "answer": result["result"],
            "sources": sources,
            "context": [doc.page_content for doc in docs]
        }
```

### 2. Hybrid Search (Semantic + Keyword)
**Описание:** Комбинация семантического и keyword поиска

**Код:**
```python
from langchain.retrievers import BM25Retriever, EnsembleRetriever

class HybridRAG:
    def __init__(self, documents: list):
        # Semantic retriever
        self.vector_retriever = Chroma.from_documents(
            documents,
            OpenAIEmbeddings()
        ).as_retriever(search_kwargs={"k": 5})
        
        # Keyword retriever
        self.bm25_retriever = BM25Retriever.from_documents(documents)
        self.bm25_retriever.k = 5
        
        # Ensemble retriever (комбинация)
        self.ensemble_retriever = EnsembleRetriever(
            retrievers=[self.vector_retriever, self.bm25_retriever],
            weights=[0.5, 0.5]  # 50/50 semantic и keyword
        )
    
    async def search(self, query: str) -> list:
        """Гибридный поиск"""
        results = await self.ensemble_retriever.aget_relevant_documents(query)
        return results
```

### 3. RAG с Reranking
**Описание:** Улучшение качества через reranking

**Код:**
```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CohereRerank

class RAGWithReranking:
    def __init__(self):
        # Base retriever
        base_retriever = Chroma.from_documents(
            documents,
            OpenAIEmbeddings()
        ).as_retriever(search_kwargs={"k": 10})
        
        # Reranker
        compressor = CohereRerank(top_n=3)
        
        # Compression retriever
        self.retriever = ContextualCompressionRetriever(
            base_compressor=compressor,
            base_retriever=base_retriever
        )
    
    async def retrieve(self, query: str):
        """Поиск с reranking"""
        # Получаем 10 документов, reranker выбирает топ-3
        docs = await self.retriever.aget_relevant_documents(query)
        return docs
```

## 🛠️ Инструменты

### Frameworks
- **LangChain** - RAG orchestration
- **LlamaIndex** - data framework
- **Haystack** - NLP framework

### Vector Databases
- **ChromaDB** - локальная vector DB
- **Pinecone** - облачная vector DB
- **Weaviate** - open-source vector DB
- **Qdrant** - vector search engine

### Embeddings
- **OpenAI** - text-embedding-ada-002
- **Cohere** - embed-multilingual
- **Sentence Transformers** - open-source

## 📚 Best Practices

### 1. Оптимальный размер чанков
```python
# Для разных типов контента
CHUNK_SIZES = {
    "code": 500,           # Код - маленькие чанки
    "documentation": 1000, # Документация - средние
    "articles": 1500,      # Статьи - большие
}

def get_text_splitter(content_type: str):
    chunk_size = CHUNK_SIZES.get(content_type, 1000)
    return RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_size // 5  # 20% overlap
    )
```

### 2. Метаданные для фильтрации
```python
# Добавление метаданных к документам
documents = [
    Document(
        page_content="...",
        metadata={
            "source": "docs/api.md",
            "category": "api",
            "date": "2026-03-23",
            "language": "ru"
        }
    )
]

# Поиск с фильтрацией
results = vectorstore.similarity_search(
    query="API authentication",
    filter={"category": "api", "language": "ru"}
)
```

### 3. Кеширование embeddings
```python
import hashlib
from functools import lru_cache

class CachedEmbeddings:
    def __init__(self, base_embeddings):
        self.base = base_embeddings
        self.cache = {}
    
    def embed_query(self, text: str) -> list[float]:
        # Хеш текста для кеша
        text_hash = hashlib.md5(text.encode()).hexdigest()
        
        if text_hash not in self.cache:
            self.cache[text_hash] = self.base.embed_query(text)
        
        return self.cache[text_hash]
```

## 🎓 Техники

### 1. Multi-Query RAG
Генерация нескольких вариантов запроса для лучшего поиска:
```python
from langchain.retrievers.multi_query import MultiQueryRetriever

retriever = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=OpenAI()
)

# LLM сгенерирует 3-5 вариантов запроса
docs = retriever.get_relevant_documents("Как настроить API?")
```

### 2. Parent Document Retriever
Поиск по маленьким чанкам, возврат больших:
```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.storage import InMemoryStore

# Маленькие чанки для поиска
child_splitter = RecursiveCharacterTextSplitter(chunk_size=200)

# Большие чанки для контекста
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=1000)

retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=InMemoryStore(),
    child_splitter=child_splitter,
    parent_splitter=parent_splitter
)
```

### 3. Self-Query Retriever
LLM сам генерирует фильтры:
```python
from langchain.retrievers.self_query.base import SelfQueryRetriever

retriever = SelfQueryRetriever.from_llm(
    llm=OpenAI(),
    vectorstore=vectorstore,
    document_contents="Technical documentation",
    metadata_field_info=[
        {"name": "category", "type": "string"},
        {"name": "date", "type": "date"}
    ]
)

# LLM сам поймёт что нужен фильтр
docs = retriever.get_relevant_documents(
    "API документация за последний месяц"
)
```

## 📊 Метрики качества

### Оценка RAG системы:
```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_recall,
    context_precision
)

# Оценка качества
results = evaluate(
    dataset=test_dataset,
    metrics=[
        faithfulness,        # Соответствие ответа контексту
        answer_relevancy,    # Релевантность ответа
        context_recall,      # Полнота контекста
        context_precision    # Точность контекста
    ]
)
```

## 📖 Ресурсы

- [LangChain RAG Tutorial](https://python.langchain.com/docs/use_cases/question_answering/)
- [LlamaIndex Documentation](https://docs.llamaindex.ai/)
- [RAG Papers](https://arxiv.org/abs/2005.11401)

---

**Последнее обновление:** 2026-03-23
