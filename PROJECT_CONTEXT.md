# Project Context: MongoDB Atlas RAG with HPE MLIS

**Purpose:** This document provides comprehensive context about the MongoDB Atlas RAG project for Large Language Models to understand the system architecture, configuration patterns, and implementation details.

## Project Overview

This is a Retrieval-Augmented Generation (RAG) system that combines MongoDB Atlas Vector Search with HPE Machine Learning Inference Service (MLIS) for predictive maintenance applications. The system ingests maintenance documentation, generates embeddings, stores them in MongoDB with vector search capabilities, and uses LLMs to answer maintenance-related questions with context retrieved from the vector database.

## Architecture

### High-Level Data Flow

```
1. Data Ingestion → MongoDB Atlas Collections
2. Text Extraction → Document Processing
3. Embedding Generation → HPE MLIS Embedding Model (OpenAI-compatible API)
4. Vector Storage → MongoDB with embeddings field
5. Vector Index Creation → Atlas Vector Search indexes
6. Query Processing:
   a. User Query → Embedding (via MLIS)
   b. Vector Search → MongoDB similarity search
   c. Context Retrieval → Top-k relevant documents
   d. LLM Generation → HPE MLIS Completion Model (via OpenAI-compatible API)
   e. Response → Contextualized answer
```

### Technology Stack

- **Python 3.8+** - Primary programming language
- **Jupyter Notebook** - Interactive development environment
- **MongoDB Atlas** - Cloud database with Vector Search capability
- **HPE MLIS** - Machine Learning Inference Service (OpenAI-compatible endpoints)
  - Embedding Models: NVIDIA, custom models
  - Completion Models: Meta Llama, custom models
- **OpenAI Python Client** - Used to interface with MLIS OpenAI-compatible endpoints
- **python-dotenv** - Environment variable management
- **httpx** - HTTP client with SSL configuration support

## API Integration Pattern

### HPE MLIS OpenAI Compatibility

HPE MLIS exposes OpenAI-compatible REST APIs, allowing use of the standard OpenAI Python client library. This project uses:

**Embedding API:**
```python
from openai import OpenAI

client = OpenAI(
    api_key=EMBEDDING_API_KEY,
    base_url=EMBEDDING_API_ENDPOINT,  # e.g., https://embed-model.namespace.domain/v1
    http_client=httpx.Client(verify=False)  # For self-signed certs
)

response = client.embeddings.create(
    input=["text to embed"],
    model="nvidia/nv-embedqa-e5-v5",
    extra_body={"input_type": "passage"}  # Voyage-specific parameter
)
embeddings = [e.embedding for e in response.data]
```

**Completion API:**
```python
response = llm_client.chat.completions.create(
    model="meta/llama-3.1-8b-instruct",
    messages=[
        {"role": "system", "content": "You are a helpful assistant..."},
        {"role": "user", "content": "What are maintenance procedures?"}
    ],
    temperature=0.7,
    max_tokens=500
)
answer = response.choices[0].message.content
```

### Key API Characteristics

1. **Endpoint Structure:** `https://<model>.<namespace>.<domain>/v1`
2. **Authentication:** Bearer tokens (Kubernetes service account tokens)
3. **SSL:** Often uses self-signed certificates requiring verification disable
4. **Extra Parameters:** Some models require additional parameters via `extra_body`
   - Voyage models require `input_type`: `"passage"` for documents, `"query"` for searches

## Configuration System

### Environment Variables (.env file)

All configuration is managed through environment variables loaded via `python-dotenv`:

```bash
# MongoDB Configuration
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/
DATABASE_NAME=pmd

# LLM Configuration
LLM_API_ENDPOINT=https://llama-endpoint.domain/v1
LLM_API_KEY=kubernetes_service_account_token
COMPLETION_MODEL=meta/llama-3.1-8b-instruct

# Embedding Configuration
USE_OPENAI_COMPATIBLE_EMBEDDINGS=true
EMBEDDING_API_ENDPOINT=https://embed-endpoint.domain/v1
EMBEDDING_API_KEY=kubernetes_service_account_token
EMBEDDING_MODEL=nvidia/nv-embedqa-e5-v5

# SSL Configuration
DISABLE_SSL_VERIFICATION=true
```

### Configuration Loading Pattern

```python
import os
from dotenv import load_dotenv

load_dotenv()

# Load all configuration
MONGODB_URI = os.getenv("MONGODB_URI")
COMPLETION_MODEL = os.getenv("COMPLETION_MODEL", "gpt-3.5-turbo")
USE_OPENAI_COMPATIBLE_EMBEDDINGS = os.getenv("USE_OPENAI_COMPATIBLE_EMBEDDINGS", "false").lower() in ['true', '1', 'yes']
DISABLE_SSL_VERIFICATION = os.getenv("DISABLE_SSL_VERIFICATION", "false").lower() in ['true', '1', 'yes']
```

**Important:** After modifying `.env`, the Jupyter kernel must be restarted for changes to take effect.

## Data Model

### MongoDB Collections

Three primary collections store maintenance data:

1. **manuals** - Equipment technical documentation
   - Fields: `title`, `text`, `embeddings`
   
2. **interviews** - Historical maintenance interviews
   - Fields: `title`, `text`, `embeddings`
   
3. **workorders** - Work order records
   - Fields: `title`, `observations`, `embeddings`

### Document Structure

```json
{
  "_id": ObjectId("..."),
  "title": "Equipment Manual Section",
  "text": "Full text content...",
  "embeddings": [0.123, -0.456, ...],  // Vector embedding (e.g., 1024 dimensions)
  "metadata": { ... }
}
```

### Vector Index Configuration

```json
{
  "fields": [{
    "type": "vector",
    "path": "embeddings",
    "numDimensions": 1024,  // Model-specific
    "similarity": "cosine"
  }]
}
```

## Core Functions

### Text Extraction

```python
def extract_text_for_embedding(document: Dict[str, Any], text_fields: List[str] = None) -> str:
    """Extract text from document for embedding generation"""
    if text_fields is None:
        text_fields = ['text', 'title', 'observations']
    
    texts = []
    for field in text_fields:
        if field in document and document[field]:
            value = document[field]
            if isinstance(value, str):
                texts.append(value)
            elif isinstance(value, list):
                texts.extend([str(v) for v in value if v])
    
    return " ".join(texts)
```

### Embedding Generation

```python
def generate_embeddings_batch(texts: List[str], model: str = EMBEDDING_MODEL, input_type: str = "passage") -> List[List[float]]:
    """
    Generate embeddings using configured API
    
    Args:
        texts: List of texts to embed
        model: Model name from environment variable
        input_type: "passage" for documents, "query" for searches
    """
    if USE_OPENAI_COMPATIBLE_EMBEDDINGS:
        response = embedding_client.embeddings.create(
            input=texts,
            model=model,
            extra_body={"input_type": input_type}
        )
        embeddings = [e.embedding for e in response.data]
    else:
        # Voyage AI native API fallback
        voyage_input_type = "document" if input_type == "passage" else "query"
        response = embedding_client.embed(
            texts=texts,
            model=model,
            input_type=voyage_input_type
        )
        embeddings = [e for e in response.embeddings]
    
    return embeddings
```

### Vector Search

```python
def vector_search_mongodb(db, collection_name: str, query_vector: List[float], num_results: int = 5) -> List[Dict]:
    """Perform vector similarity search on MongoDB collection"""
    collection = db[collection_name]
    
    pipeline = [
        {
            "$vectorSearch": {
                "index": "vector_index",
                'queryVector': query_vector,
                'numCandidates': 10,
                'limit': num_results
            }
        },
        {
            "$project": {
                "'score": {"$meta": "vectorSearchScore"},
                "document": "$$ROOT"
            }
        },
        {
            "$limit": num_results
        }
    ]
    
    results = list(collection.aggregate(pipeline))
    return results
```

### RAG Answer Generation

```python
class MongoDBOpenAIRAG:
    """RAG system using MongoDB Atlas and OpenAI-compatible LLMs"""
    
    def __init__(self, db, model: str = None, temperature: float = 0.7):
        self.db = db
        self.model = model if model else COMPLETION_MODEL
        self.temperature = temperature
    
    def answer_question(self, query: str, num_context_docs: int = 3) -> Dict[str, Any]:
        # 1. Generate query embedding
        query_embedding = generate_embeddings_batch([query], input_type="query")
        query_vector = query_embedding[0]
        
        # 2. Search collections for relevant context
        manual_results = vector_search_mongodb(self.db, "manuals", query_vector, num_context_docs)
        interview_results = vector_search_mongodb(self.db, "interviews", query_vector, num_context_docs)
        workorder_results = vector_search_mongodb(self.db, "workorders", query_vector, num_context_docs)
        
        # 3. Format context
        context = format_retrieved_context(manual_results, interview_results, workorder_results)
        
        # 4. Generate answer with LLM
        response = llm_client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful assistant..."},
                {"role": "user", "content": f"Context: {context}\n\nQuestion: {query}"}
            ],
            temperature=self.temperature,
            max_tokens=500
        )
        
        return {
            'success': True,
            'answer': response.choices[0].message.content,
            'context': context,
            'model': self.model
        }
```

## Common Patterns and Idioms

### Error Handling

```python
try:
    # Operation
    result = operation()
    print(f"✓ Success message")
    return result
except SpecificException as e:
    print(f"✗ Error message: {e}")
    return None
```

### Batch Processing

```python
for i in range(0, len(documents), batch_size):
    batch = documents[i:i + batch_size]
    
    # Process batch
    texts = [extract_text_for_embedding(doc) for doc in batch]
    embeddings = generate_embeddings_batch(texts)
    
    # Update documents
    for doc, embedding in zip(batch, embeddings):
        collection.update_one(
            {"_id": doc["_id"]},
            {"$set": {"embeddings": embedding}}
        )
```

### SSL Configuration for Internal Deployments

```python
import httpx

if DISABLE_SSL_VERIFICATION:
    http_client = httpx.Client(verify=False)
else:
    http_client = None

client = OpenAI(
    api_key=API_KEY,
    base_url=ENDPOINT,
    http_client=http_client
)
```

## Troubleshooting Knowledge

### Common Issues

1. **404 Model Not Found**
   - Cause: Incorrect model name or endpoint
   - Solution: Verify model name exactly matches MLIS deployment
   - Check: No extra quotes in `.env` values

2. **400 Bad Request - input_type Required**
   - Cause: Voyage models require `input_type` parameter
   - Solution: Use `extra_body={"input_type": "passage"}` for documents or `"query"` for searches

3. **SSL Verification Failed**
   - Cause: Self-signed certificates on MLIS
   - Solution: Set `DISABLE_SSL_VERIFICATION=true`

4. **Environment Variables Not Loading**
   - Cause: Jupyter kernel cache
   - Solution: Restart kernel after modifying `.env`

5. **Token Expiration**
   - Cause: Kubernetes tokens have expiration dates
   - Solution: Generate new token and update `.env`

## Performance Considerations

### Embedding Generation
- **Batch Size:** 5-10 documents optimal for balance between speed and memory
- **Rate Limiting:** Add delays between batches if hitting API limits
- **Dimensions:** Different models produce different dimensions (check with test embeddings)

### Vector Search
- **Index Creation:** Takes time for large collections, happens asynchronously in Atlas
- **Query Performance:** Cosine similarity with proper indexes is very fast
- **Fallback:** Notebook includes numpy-based fallback for when Atlas search isn't ready

### LLM Generation
- **Temperature:** 0.7 default, lower for more deterministic responses
- **Max Tokens:** 500 default, adjust based on desired response length
- **Context Size:** Balance between relevance (more docs) and token limits

## Development Workflow

1. **Modify `.env`** - Update configuration
2. **Restart Kernel** - Critical step to load new env vars
3. **Run Cells Sequentially** - Top to bottom execution order matters
4. **Verify at Each Step** - Check for ✓ success messages
5. **Debug with Print Statements** - Use debug cells to inspect values

## Extension Points

To extend this system:

1. **Add New Collections:** Follow the same pattern (ingest → embed → index)
2. **Custom Models:** Update model names in `.env`, ensure API compatibility
3. **Different Embedding Models:** May require index recreation with new dimensions
4. **Additional Metadata:** Add fields to documents, update extraction function
5. **Advanced Search:** Modify vector search pipeline for filters, scoring

## Important Constants

- Default embedding batch size: 5-10
- Default vector search candidates: 10
- Default result limit: 5
- Default LLM temperature: 0.7
- Default max tokens: 500
- MongoDB timeout: 5000ms

## File Structure

```
demo-rag-pm/
├── .env                          # Configuration (not in git)
├── .env.example                  # Configuration template
├── rag_mongodb_voyage_openai.ipynb  # Main RAG notebook
├── data/
│   ├── manuals.json             # Technical documentation
│   ├── interviews.json          # Historical interviews
│   └── workorders.json          # Work order records
├── SETUP_GUIDE.md               # Step-by-step setup instructions
├── PROJECT_CONTEXT.md           # This file - comprehensive context for LLMs
├── EMBEDDING_API_GUIDE.md       # Embedding API configuration reference
└── README.md                    # Project overview
```

## Code Style and Conventions

- **Print Messages:** Use ✓ for success, ✗ for errors, ⚠ for warnings
- **Type Hints:** Use where helpful (Dict, List, Any, Optional)
- **Docstrings:** Triple-quoted strings with Args/Returns sections
- **Error Messages:** Include context and actionable information
- **Variable Names:** Descriptive, follow Python conventions (snake_case)
- **Constants:** ALL_CAPS for environment-derived constants

## Security Considerations

- **API Keys:** Never commit `.env` to version control
- **SSL:** Only disable verification for development/internal deployments
- **MongoDB:** Use connection strings with authentication
- **Token Expiration:** MLIS tokens expire, plan for rotation
- **Network Access:** Restrict MongoDB Atlas to specific IPs in production

## Version Compatibility

- Python: 3.8+ (tested with 3.10, 3.13)
- MongoDB: Atlas with Vector Search (M10+ tier for production)
- OpenAI Client: Latest version (1.x+)
- HPE MLIS: OpenAI API compatible version

## Testing Strategy

1. **Test Embedding Endpoint:** curl with sample text
2. **Test Completion Endpoint:** curl with simple prompt
3. **Test MongoDB Connection:** Ping command
4. **Test Embeddings:** Generate test embeddings, check dimensions
5. **Test Vector Search:** Query with sample vector
6. **End-to-End Test:** Full RAG query with known answer

This context provides a comprehensive foundation for understanding, maintaining, and extending the MongoDB Atlas RAG system with HPE MLIS.
