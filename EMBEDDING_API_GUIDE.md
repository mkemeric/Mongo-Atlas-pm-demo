# Embedding API Configuration Guide

This notebook supports both **Voyage AI native API** and **OpenAI-compatible embedding APIs** (such as HPE MLIS, vLLM, or any other OpenAI-compatible hosting platform).

## Configuration

The embedding API is controlled by a single environment variable:

```bash
USE_OPENAI_COMPATIBLE_EMBEDDINGS="true"  # or "false"
```

## Quick Start

### Option 1: Voyage AI Native API (Default)

```bash
# .env file
USE_OPENAI_COMPATIBLE_EMBEDDINGS="false"
VOYAGE_API_KEY="pa-r-your-voyage-api-key"
EMBEDDING_MODEL="voyage-3-large"
```

### Option 2: OpenAI-Compatible API (e.g., HPE MLIS with Qwen)

```bash
# .env file
USE_OPENAI_COMPATIBLE_EMBEDDINGS="true"
EMBEDDING_API_ENDPOINT="https://your-mlis-endpoint.com/v1"
EMBEDDING_API_KEY="your-api-key"
EMBEDDING_MODEL="qwen2-embedding"
```

### Option 3: Voyage Model on OpenAI-Compatible Platform

```bash
# .env file
USE_OPENAI_COMPATIBLE_EMBEDDINGS="true"
EMBEDDING_API_ENDPOINT="https://your-mlis-endpoint.com/v1"
EMBEDDING_API_KEY="your-api-key"
EMBEDDING_MODEL="voyage-3-large"
```

## Environment Variables

### Required for All Configurations
- `MONGODB_URI` - MongoDB Atlas connection string
- `DATABASE_NAME` - Database name
- `LLM_API_KEY` - API key for LLM (answer generation)
- `EMBEDDING_MODEL` - Name of the embedding model to use

### Embedding API Selection
- `USE_OPENAI_COMPATIBLE_EMBEDDINGS` - Set to `"true"`, `"1"`, or `"yes"` to use OpenAI-compatible API; otherwise uses Voyage AI native API

### For OpenAI-Compatible APIs
- `EMBEDDING_API_ENDPOINT` - Base URL for the OpenAI-compatible endpoint (e.g., `https://mlis.example.com/v1`)
- `EMBEDDING_API_KEY` - API key for the embedding service

### For Voyage AI Native API
- `VOYAGE_API_KEY` - Voyage AI API key (can also use `EMBEDDING_API_KEY`)
- `VOYAGE_API_ENDPOINT` - Voyage API endpoint (optional, defaults to Voyage's service)

## How It Works

The notebook automatically selects the appropriate client library based on `USE_OPENAI_COMPATIBLE_EMBEDDINGS`:

### Voyage AI Native API (`USE_OPENAI_COMPATIBLE_EMBEDDINGS=false`)
```python
# Uses voyageai.Client
response = embedding_client.embed(
    texts=texts,
    model=model,
    input_type="document"
)
embeddings = [e for e in response.embeddings]
```

### OpenAI-Compatible API (`USE_OPENAI_COMPATIBLE_EMBEDDINGS=true`)
```python
# Uses openai.OpenAI
response = embedding_client.embeddings.create(
    input=texts,
    model=model
)
embeddings = [e.embedding for e in response.data]
```

## Key Insight

**The embedding API selection is about the hosting platform's API protocol, not the model itself.**

- If your model is hosted on an **OpenAI-compatible endpoint** (like HPE MLIS), set `USE_OPENAI_COMPATIBLE_EMBEDDINGS="true"` regardless of whether you're using Voyage, Qwen, or any other embedding model
- If you're using **Voyage's native API service**, set `USE_OPENAI_COMPATIBLE_EMBEDDINGS="false"`

## Backward Compatibility

The notebook maintains backward compatibility with the original configuration:
- If `USE_OPENAI_COMPATIBLE_EMBEDDINGS` is not set, it defaults to `"false"` (Voyage AI native)
- If `EMBEDDING_API_KEY` is not set and using Voyage native API, it will fall back to `VOYAGE_API_KEY`

## Common Scenarios

### Scenario 1: Using Voyage AI's Cloud Service
```bash
USE_OPENAI_COMPATIBLE_EMBEDDINGS="false"
VOYAGE_API_KEY="pa-r-..."
EMBEDDING_MODEL="voyage-3-large"
```

### Scenario 2: Using Qwen on HPE MLIS
```bash
USE_OPENAI_COMPATIBLE_EMBEDDINGS="true"
EMBEDDING_API_ENDPOINT="https://mlis.hpe.com/v1"
EMBEDDING_API_KEY="mlis-key-..."
EMBEDDING_MODEL="qwen2-embedding"
```

### Scenario 3: Using Voyage Model Hosted on MLIS
```bash
USE_OPENAI_COMPATIBLE_EMBEDDINGS="true"
EMBEDDING_API_ENDPOINT="https://mlis.hpe.com/v1"
EMBEDDING_API_KEY="mlis-key-..."
EMBEDDING_MODEL="voyage-3-large"
```

## Troubleshooting

### Embeddings Not Generating
1. Check that `embedding_client` is initialized (should see confirmation message)
2. Verify API key is correct
3. For OpenAI-compatible APIs, ensure `EMBEDDING_API_ENDPOINT` is set correctly
4. Check the model name matches what's available on your endpoint

### Wrong API Being Used
- Verify `USE_OPENAI_COMPATIBLE_EMBEDDINGS` is set correctly (must be string `"true"` or `"false"`)
- Check the initialization output to confirm which API type is being used

### Dimension Mismatch Errors
- Different models may produce embeddings with different dimensions
- Update the MongoDB vector index if you switch models with different dimensions
- Check test output for actual embedding dimension
