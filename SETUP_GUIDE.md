# Setup Guide: MongoDB Atlas RAG with HPE PC AI Jupyter Server

This guide walks you through setting up and running the MongoDB Atlas RAG (Retrieval-Augmented Generation) system on HPE PC AI using a Jupyter Server and notebook.

## Prerequisites

Before you begin, ensure you have:

- Access to **HPE PC AI** with a Jupyter Server environment
- Access to **MongoDB Atlas** with Vector Search enabled
- Access to **HPE MLIS** with deployed models for:
  - Embeddings (e.g., `nvidia/nv-embedqa-e5-v5`)
  - Text completion (e.g., `meta/llama-3.1-8b-instruct`)
- **MLIS API tokens** for authentication

## Step 1: Access HPE PC AI Jupyter Server

1. Log in to your **HPE PC AI** environment
2. Navigate to the Jupyter Server interface
3. Clone or upload the repository to your Jupyter workspace

```bash
git clone https://github.com/mkemeric/Mongo-Atlas-pm-demo.git
cd Mongo-Atlas-pm-demo
```

## Step 2: Setup Environment Configuration

**Note:** You will **NOT** need to create a virtual environment or manually install dependencies. All dependencies are installed directly in the notebook cells.

Setup will be accomplished using a `.env` file that will be copied into the environment.

## Step 3: Set Up MongoDB Atlas

### 3.1 Create MongoDB Atlas Account
1. Go to [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
2. Sign up for a free account or log in
3. Create a new cluster (M0 Free Tier is sufficient for testing)

### 3.2 Configure Database Access
1. In Atlas, go to **Database Access**
2. Click **Add New Database User**
3. Create a username and password (save these!)
4. Grant **Read and write to any database** privileges

### 3.3 Configure Network Access
1. Go to **Network Access**
2. Click **Add IP Address**
3. For testing: Click **Allow Access from Anywhere** (0.0.0.0/0)
   - **Production:** Restrict to your specific IP addresses

### 3.4 Get Connection String
1. Click **Connect** on your cluster
2. Choose **Connect your application**
3. Copy the connection string (looks like: `mongodb+srv://username:password@cluster.mongodb.net/`)
4. Replace `<password>` with your actual password

## Step 4: Get HPE MLIS Endpoints and API Keys

### 4.1 Identify Your Model Endpoints

Your MLIS endpoints will follow this pattern:
```
https://<model-name>.<namespace>.<cluster-domain>/v1
```

Example endpoints:
- Embedding: `https://embedqa-e5-v5.luann-hpe-com-9df0372c.serving.prod.lg-pcai.hou/v1`
- LLM: `https://llama-3-1-8b-instruct-1.luann-hpe-com-9df0372c.serving.prod.lg-pcai.hou/v1`

### 4.2 Get API Keys/Tokens

HPE MLIS uses Kubernetes service account tokens for authentication. To get your token:

```bash
# If you have kubectl access
kubectl get secret <service-account-secret> -n <namespace> -o jsonpath='{.data.token}' | base64 --decode
```

Or obtain from your MLIS administrator.

### 4.3 Verify Model Names

Confirm the exact model names your MLIS deployment uses:
- Embedding model (e.g., `nvidia/nv-embedqa-e5-v5`)
- Completion model (e.g., `meta/llama-3.1-8b-instruct`)

## Step 5: Configure Environment Variables

Create a `.env` file in your Jupyter workspace directory. This file will be copied into the notebook environment.

You can create it directly in Jupyter using the text editor, or upload a pre-configured `.env` file.

Add the following configuration:

```bash
# ============================================================================
# MongoDB Configuration
# ============================================================================
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/?appName=myApp
DATABASE_NAME=pmd

# ============================================================================
# LLM Configuration (for answer generation)
# ============================================================================
LLM_API_ENDPOINT=https://your-llm-endpoint.example.com/v1
LLM_API_KEY=your-mlis-api-token
COMPLETION_MODEL=meta/llama-3.1-8b-instruct

# ============================================================================
# Embedding API Configuration
# ============================================================================

# Set to 'true' to use OpenAI-compatible API (HPE MLIS)
USE_OPENAI_COMPATIBLE_EMBEDDINGS=true

# HPE MLIS Embedding Settings
EMBEDDING_API_ENDPOINT=https://your-embedding-endpoint.example.com/v1
EMBEDDING_API_KEY=your-mlis-api-token
EMBEDDING_MODEL=nvidia/nv-embedqa-e5-v5

# ============================================================================
# SSL Configuration
# ============================================================================

# Disable SSL verification for internal/dev environments with self-signed certs
# Set to 'true' only for development/internal deployments
DISABLE_SSL_VERIFICATION=true
```

### Environment Variable Reference

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `MONGODB_URI` | Yes | MongoDB Atlas connection string | `mongodb+srv://user:pass@cluster.mongodb.net/` |
| `DATABASE_NAME` | Yes | Database name to use | `pmd` |
| `LLM_API_ENDPOINT` | Yes | HPE MLIS LLM endpoint URL | `https://llama-...hou/v1` |
| `LLM_API_KEY` | Yes | MLIS API token for LLM | `eyJhbGciOiJS...` |
| `COMPLETION_MODEL` | Yes | Model name for text completion | `meta/llama-3.1-8b-instruct` |
| `USE_OPENAI_COMPATIBLE_EMBEDDINGS` | Yes | Set to `true` for MLIS | `true` |
| `EMBEDDING_API_ENDPOINT` | Yes | HPE MLIS embedding endpoint URL | `https://embedqa-...hou/v1` |
| `EMBEDDING_API_KEY` | Yes | MLIS API token for embeddings | `eyJhbGciOiJS...` |
| `EMBEDDING_MODEL` | Yes | Model name for embeddings | `nvidia/nv-embedqa-e5-v5` |
| `DISABLE_SSL_VERIFICATION` | No | Disable SSL for self-signed certs | `true` (dev only) |

## Step 6: Verify Configuration (Optional)

### 6.1 Test Embedding Endpoint

```bash
curl -k -X POST "https://your-embedding-endpoint/v1/embeddings" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "input": ["test text"],
    "model": "nvidia/nv-embedqa-e5-v5",
    "extra_body": {"input_type": "passage"}
  }'
```

### 6.2 Test LLM Endpoint

```bash
curl -k -X POST "https://your-llm-endpoint/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "meta/llama-3.1-8b-instruct",
    "messages": [{"role": "user", "content": "Hello"}],
    "max_tokens": 10
  }'
```

Both should return valid JSON responses (not 404 or authentication errors).

## Step 7: Prepare Data

The notebook expects data files in a `data/` directory:

```bash
# Ensure data directory exists
mkdir -p data
```

The repository should include:
- `data/manuals.json` - Equipment manuals and documentation
- `data/interviews.json` - Historical maintenance interviews
- `data/workorders.json` - Work order records

If these files are missing, the notebook will alert you during execution.

## Step 8: Run the Notebook

### 8.1 Open the Notebook in Jupyter Server

In your HPE PC AI Jupyter Server interface, navigate to and open: `rag_mongodb_voyage_openai.ipynb`

### 8.2 Execute Cells Sequentially

**Important:** Run cells in order from top to bottom. All dependencies will be installed automatically within the notebook.

1. **Cell 1** - Install dependencies (runs `pip install` commands in the notebook)
2. **Cell 2** - Import libraries and load configuration
   - Verify you see: `✓ Embedding API type: OpenAI-compatible`
   - Verify you see your model names printed
3. **Cell 3** - Connect to MongoDB Atlas
4. **Cell 4** - Load data from `data/` folder
5. **Cell 5** - Ingest data into MongoDB
6. **Cell 6** - Generate embeddings (this may take several minutes)
7. **Cell 7** - Create vector search indexes
8. **Cell 8** - Initialize RAG system
9. **Cell 9** - Run example queries

### 8.3 Monitor for Errors

Watch for these common issues:

**Connection Errors:**
- Verify MongoDB URI is correct
- Check network access settings in Atlas
- Ensure IP is whitelisted

**Authentication Errors (401/403):**
- Verify API keys are correct and not expired
- Check MLIS token expiration date

**404 Errors:**
- Verify endpoint URLs are correct (include `/v1`)
- Confirm model names exactly match MLIS deployment

**SSL Errors:**
- Set `DISABLE_SSL_VERIFICATION=true` for internal deployments
- For production, obtain proper SSL certificates

## Step 9: Verify Vector Search

After running all cells, verify vector search indexes were created:

1. Go to MongoDB Atlas web interface
2. Navigate to your cluster
3. Click on **Atlas Search**
4. You should see indexes named `vector_index` for each collection

## Troubleshooting

### Issue: Kernel Doesn't Pick Up .env Changes

**Solution:**
1. In Jupyter: **Kernel → Restart**
2. Re-run all cells from the beginning

### Issue: SSL Certificate Verification Failed

**Solution:**
Add to `.env`:
```bash
DISABLE_SSL_VERIFICATION=true
```

### Issue: Model Not Found (404)

**Solution:**
1. Verify exact model name with `curl` test
2. Check for extra quotes or spaces in `.env`
3. Ensure no quotes around values: `EMBEDDING_MODEL=nvidia/nv-embedqa-e5-v5` (not `"nvidia/nv-embedqa-e5-v5"`)

### Issue: Token Expired

**Solution:**
MLIS tokens have expiration dates. Generate a new token and update `.env`.

### Issue: Embedding Dimension Mismatch

**Solution:**
Different embedding models produce different dimensions. If you switch models:
1. Drop existing vector indexes in MongoDB Atlas
2. Re-run embedding generation cells
3. Re-create vector indexes with correct dimensions

## Next Steps

Once the notebook runs successfully:

1. **Experiment with queries** - Modify the example queries in Cell 9
2. **Add your own data** - Place JSON files in `data/` directory
3. **Adjust parameters** - Tune batch sizes, temperature, max tokens
4. **Monitor performance** - Check embedding generation times and query latency
5. **Scale up** - Move to production MongoDB cluster and MLIS deployment

## Additional Resources

- [MongoDB Atlas Documentation](https://docs.atlas.mongodb.com/)
- [MongoDB Vector Search Guide](https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/)
- [HPE Machine Learning Inference Service Documentation](https://www.hpe.com/)
- [OpenAI API Compatibility](https://platform.openai.com/docs/api-reference)

## Support

For issues specific to:
- **MongoDB Atlas**: Check Atlas documentation or contact MongoDB support
- **HPE MLIS**: Contact your HPE administrator or support team
- **This project**: Open an issue on the GitHub repository
