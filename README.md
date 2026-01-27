# MongoDB Atlas RAG with HPE AI Essentials

A production-ready Retrieval-Augmented Generation (RAG) system combining MongoDB Atlas Vector Search with HPE Machine Learning Inference Service (MLIS) for intelligent maintenance support. This system enables semantic search across maintenance documentation and generates contextual answers using enterprise AI infrastructure.

This solution provides:

- **Enterprise AI Integration:** Seamless integration with HPE MLIS (OpenAI-compatible API)
- **Flexible Model Support:** Works with NVIDIA embeddings, Meta Llama, and custom models
- **Vector Search at Scale:** MongoDB Atlas Vector Search for fast semantic similarity matching
- **Production-Ready Configuration:** SSL handling, token authentication, and environment-based config

## Architecture

**Data Flow:**

1. **Data Ingestion:** Maintenance documents (manuals, interviews, work orders) loaded into MongoDB Atlas
2. **Embedding Generation:** Text converted to vectors using HPE MLIS embedding models
3. **Vector Storage:** Embeddings stored in MongoDB collections with vector search indexes
4. **Query Processing:** User questions converted to embeddings and used for similarity search
5. **Context Retrieval:** Top-k relevant documents retrieved from MongoDB
6. **Answer Generation:** HPE MLIS LLM generates contextual responses using retrieved documents

## Getting Started

### Prerequisites

- Python 3.8+
- MongoDB Atlas account with Vector Search enabled
- Access to HPE MLIS with:
  - Embedding model endpoint (e.g., NVIDIA embeddings)
  - LLM endpoint (e.g., Meta Llama)
  - Valid API tokens
- Jupyter Notebook or JupyterLab

### Quick Start

1. **Clone the repository:**
   ```bash
   git clone https://github.com/mkemeric/Mongo-Atlas-pm-demo.git
   cd Mongo-Atlas-pm-demo
   ```

2. **Install dependencies:**
   ```bash
   pip install python-dotenv pymongo voyageai openai httpx
   ```

3. **Configure environment:**
   Create a `.env` file with your HPE MLIS and MongoDB Atlas credentials:
   ```env
   # MongoDB Atlas
   MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/
   DATABASE_NAME=pmd
   
   # HPE MLIS Endpoints
   EMBEDDING_API_ENDPOINT=https://your-embedding-model.domain/v1
   EMBEDDING_API_KEY=your-mlis-token
   EMBEDDING_MODEL=nvidia/nv-embedqa-e5-v5
   
   LLM_API_ENDPOINT=https://your-llm-model.domain/v1
   LLM_API_KEY=your-mlis-token
   COMPLETION_MODEL=meta/llama-3.1-8b-instruct
   
   # Configuration
   USE_OPENAI_COMPATIBLE_EMBEDDINGS=true
   DISABLE_SSL_VERIFICATION=true
   ```

4. **Run the notebook:**
   ```bash
   jupyter notebook rag_mongodb_voyage_openai.ipynb
   ```

**For detailed setup instructions, see [SETUP_GUIDE.md](SETUP_GUIDE.md)**

## Data

The project includes sample datasets in the `data/` folder:
- `interviews.json` - Historical maintenance interview transcripts
- `manuals.json` - Equipment technical manuals and documentation
- `workorders.json` - Historical work order records
- `inventory.json` - Equipment inventory and specifications
- `maintenance_staff.json` - Staff expertise and availability data

## Tech Stack

- **Python 3.8+** - Core programming language
- **MongoDB Atlas** - Cloud database with Vector Search capability
- **HPE MLIS** - Machine Learning Inference Service (OpenAI-compatible)
  - Embedding models: NVIDIA, custom models
  - LLM models: Meta Llama, custom models
- **OpenAI Python Client** - API client for MLIS integration
- **Jupyter Notebooks** - Interactive development and execution
- **httpx** - HTTP client with SSL configuration support

## Features

✅ **Implemented:**
- Full RAG pipeline with HPE MLIS integration
- MongoDB Atlas Vector Search with cosine similarity
- Dual API support (OpenAI-compatible and Voyage AI native)
- SSL certificate handling for internal deployments
- Environment-based configuration management
- Batch embedding generation with configurable sizes
- Multi-collection vector search (manuals, interviews, work orders)
- Token-based authentication for MLIS
- Automatic dimension detection for vector indexes

📋 **Configuration Options:**
- Flexible model selection via environment variables
- Support for various embedding models (NVIDIA, Voyage, custom)
- Support for various LLMs (Llama, GPT, custom)
- Configurable batch sizes and search parameters
- SSL verification toggle for dev/prod environments

## Documentation

- **[SETUP_GUIDE.md](SETUP_GUIDE.md)** - Complete step-by-step setup instructions
- **[PROJECT_CONTEXT.md](PROJECT_CONTEXT.md)** - Technical documentation for LLMs and developers
- **[EMBEDDING_API_GUIDE.md](EMBEDDING_API_GUIDE.md)** - Embedding API configuration reference
- **[.env.example](.env.example)** - Environment variable template

## Use Cases

- **Maintenance Support:** Answer technician questions using historical documentation
- **Knowledge Base Search:** Semantic search across equipment manuals and procedures
- **Troubleshooting Assistant:** Retrieve relevant context for failure diagnosis
- **Training Support:** Help new technicians learn from historical incidents

## Extending the System

1. **Add More Data Sources:** Include additional JSON files in the `data/` directory
2. **Custom Models:** Update model names in `.env` to use different MLIS models
3. **Additional Collections:** Follow the same pattern for new document types
4. **Advanced Queries:** Modify search parameters and context retrieval logic
5. **Production Deployment:** Scale to production MongoDB cluster and MLIS infrastructure

## Support

- **Issues:** Open an issue on the GitHub repository
- **MongoDB Atlas:** [MongoDB Documentation](https://docs.atlas.mongodb.com/)
- **HPE MLIS:** Contact your HPE administrator

## License

This project is provided as-is for demonstration and educational purposes.
