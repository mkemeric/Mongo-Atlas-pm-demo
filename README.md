# Predictive Maintenance AI with RAG & LangGraph

A Python-based demonstration of retrieval-augmented generation (RAG) and agentic AI workflows for predictive maintenance scenarios. This project showcases how to build intelligent systems that analyze machine failures, retrieve relevant context from documentation and historical data, and generate comprehensive maintenance reports. 

This demo showcases:

- **Intelligent Retrieval:** RAG system that combines semantic search with LLMs to answer maintenance questions
- **Agentic Workflows:** LangGraph-based agents that autonomously diagnose failures and generate repair instructions
- **Multi-Modal Data Integration:** Unified handling of manuals, historical records, interviews, and real-time alerts
- **Vector Search:** MongoDB Atlas vector search for fast, context-rich semantic similarity matching

## Architecture

**How it works:**

1. **Detection:** Process receives machine failure alerts with error codes and diagnostic data.
2. **Retrieval:** The RAG system queries MongoDB vector indexes to find relevant manuals and historical records.
3. **Analysis:** The Failure Agent uses LangGraph to orchestrate multi-step analysis with specialized tools.
4. **Generation:** LLM synthesizes findings into comprehensive incident reports with repair instructions.

## Getting Started

### Prerequisites

- Python 3.8+
- MongoDB instance (local or MongoDB Atlas)
- API keys for:
  - Voyage AI (for embeddings)
  - OpenAI (for language completion)
- Jupyter environment or VS Code with Jupyter support

### Setup

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd demo-rag-pm
   ```

2. **Create a virtual environment (recommended):**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables:**
   Create a `.env` file in the project root with:
   ```env
   MONGODB_URI=<your-mongodb-connection-string>
   VOYAGE_API_KEY=<your-voyage-ai-api-key>
   VOYAGE_API_ENDPOINT=https://api.voyageai.com/v1
   LLM_API_KEY=<your-openai-api-key>
   LLM_API_ENDPOINT=https://api.openai.com/v1
   ```

5. **Run the notebooks:**
   - Start with `rag_mongodb_voyage_openai.ipynb` to set up the RAG system and ingest data
   - Then explore `failure_agent_langgraph.ipynb` to see the agent workflow in action

## Data

The project includes sample datasets in the `data/` folder:
- `interviews.json` - Historical maintenance interview transcripts
- `manuals.json` - Equipment technical manuals and documentation
- `workorders.json` - Historical work order records
- `inventory.json` - Equipment inventory and specifications
- `maintenance_staff.json` - Staff expertise and availability data

## Tech Stack

- **Python 3.x**
- **MongoDB Atlas** - Document database with vector search
- **LangGraph** - Agent workflow orchestration
- **Voyage AI** - Advanced embedding model (v3)
- **OpenAI API** - Language model completion
- **Jupyter Notebooks** - Interactive execution and documentation

## Current Status

✅ **Complete:**
- RAG pipeline implementation with Voyage AI embeddings
- MongoDB vector search integration
- OpenAI-based question answering
- LangGraph agent framework setup
- Multi-tool failure diagnosis workflow
- Data ingestion and embedding pipeline

🔧 **In Development:**
- Production deployment configuration
- Error handling and retry logic
- Persistent incident report storage
- Advanced checkpointing for long-running operations

## Next Steps

To extend this project:
1. Connect to real MongoDB production instances
2. Implement persistent storage for incident reports
3. Add more specialized tools (asset tracking, parts inventory, technician availability)
4. Create a web API wrapper for the agents
5. Set up monitoring and logging for production use
6. Add chat interface for interactive agent queries
