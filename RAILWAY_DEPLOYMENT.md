# Deploying Letta to Railway

This guide will help you deploy Letta to Railway using their GitHub integration.

## Prerequisites

- A [Railway](https://railway.app) account
- This repository pushed to GitHub
- API keys for your preferred LLM providers (OpenAI, Anthropic, etc.)

## Quick Deploy

### Step 1: Create a New Project on Railway

1. Go to [Railway](https://railway.app) and log in
2. Click **"New Project"**
3. Select **"Deploy from GitHub repo"**
4. Choose your `letta-letta-was-a-bullfrog` repository

### Step 2: Add PostgreSQL Database

1. In your Railway project, click **"New"** → **"Database"** → **"Add PostgreSQL"**
2. Railway will automatically create a PostgreSQL database and set up connection variables
3. The database variables will be automatically linked to your Letta service

### Step 3: Configure Environment Variables

In your Railway project dashboard, go to your Letta service and add these environment variables:

#### Required Variables

```bash
# LLM Provider API Key (choose at least one)
OPENAI_API_KEY=sk-...
# or
ANTHROPIC_API_KEY=sk-ant-...
# or
GROQ_API_KEY=gsk_...
# or
GEMINI_API_KEY=...
```

#### Optional Variables

```bash
# Server Configuration (already set in railway.toml)
PORT=8283
HOST=0.0.0.0
LETTA_ENVIRONMENT=PRODUCTION

# Additional LLM Providers
AZURE_API_KEY=...
AZURE_BASE_URL=...
AZURE_API_VERSION=2024-02-15-preview

# Observability
LETTA_OTEL_EXPORTER_OTLP_ENDPOINT=...
CLICKHOUSE_ENDPOINT=...
CLICKHOUSE_PASSWORD=...

# Sentry Error Tracking
SENTRY_DSN=...
```

### Step 4: Deploy

1. Railway will automatically detect the `Dockerfile` and `railway.toml`
2. The build process will start automatically
3. Wait for the deployment to complete (this may take 5-10 minutes on first deploy)
4. Once deployed, Railway will provide you with a public URL

### Step 5: Verify Deployment

Visit your Railway-provided URL at the `/v1/health` endpoint:
```
https://your-app.railway.app/v1/health
```

You should see a health check response indicating the server is running.

## Architecture

The Railway deployment includes:

- **Letta Server**: Running on port 8283 (mapped to Railway's public port)
- **PostgreSQL Database**: Managed by Railway with automatic backups
- **OpenTelemetry Collector**: For observability (configured in the Docker image)

## Configuration Files

This repository includes Railway-specific configuration:

- **`railway.toml`**: Railway service configuration
- **`.railwayignore`**: Files to exclude from deployment
- **`.env.railway.example`**: Template for environment variables
- **`Dockerfile`**: Multi-stage build optimized for production

## Database Migrations

Database migrations run automatically on startup via the `startup.sh` script using Alembic.

## Connecting to Your Letta Instance

### Python SDK

```python
from letta_client import Letta

client = Letta(base_url="https://your-app.railway.app")

# Create an agent
agent_state = client.agents.create(
    model="openai/gpt-4",
    embedding="openai/text-embedding-3-small",
    memory_blocks=[
        {"label": "human", "value": "The user's name is..."},
        {"label": "persona", "value": "I am a helpful assistant..."}
    ]
)
```

### TypeScript SDK

```typescript
import { LettaClient } from '@letta-ai/letta-client'

const client = new LettaClient({ 
  baseUrl: "https://your-app.railway.app" 
});

const agentState = await client.agents.create({
    model: "openai/gpt-4",
    embedding: "openai/text-embedding-3-small",
    memoryBlocks: [
        { label: "human", value: "The user's name is..." },
        { label: "persona", value: "I am a helpful assistant..." }
    ]
});
```

## Monitoring and Logs

- **Logs**: View real-time logs in the Railway dashboard under your service's "Logs" tab
- **Metrics**: Railway provides built-in CPU, memory, and network metrics
- **Health Checks**: Railway automatically monitors the `/v1/health` endpoint

## Scaling

Railway automatically handles:
- Horizontal scaling (add more instances in project settings)
- Vertical scaling (upgrade your plan for more resources)
- Zero-downtime deployments

## Troubleshooting

### Build Failures

If the build fails:
1. Check the build logs in Railway dashboard
2. Ensure all required files are committed to GitHub
3. Verify the Dockerfile builds locally: `docker build -t letta .`

### Database Connection Issues

If you see database connection errors:
1. Verify the PostgreSQL service is running in Railway
2. Check that `LETTA_PG_URI` is correctly referencing `${{Postgres.DATABASE_URL}}`
3. View the startup logs to see the connection string being used

### API Key Issues

If you get authentication errors:
1. Verify your API keys are correctly set in Railway environment variables
2. Check that there are no extra spaces or quotes around the keys
3. Ensure you're using the correct key format for your provider

### Memory Issues

If the service crashes due to memory:
1. Upgrade your Railway plan for more memory
2. Consider using a lighter embedding model
3. Monitor memory usage in Railway metrics

## Cost Optimization

- **Database**: Use Railway's PostgreSQL (included in most plans)
- **Compute**: Start with the Hobby plan ($5/month) and scale as needed
- **Storage**: Railway includes persistent storage in all plans

## Security Best Practices

1. **Never commit API keys** to your repository
2. **Use Railway's environment variables** for all secrets
3. **Enable HTTPS** (Railway provides this automatically)
4. **Regularly update dependencies** by rebuilding from the latest code
5. **Monitor logs** for suspicious activity

## Support

- **Railway Docs**: https://docs.railway.app
- **Letta Docs**: https://docs.letta.com
- **Letta Discord**: https://discord.gg/letta
- **Railway Discord**: https://discord.gg/railway

## Next Steps

After deployment:
1. Set up custom domain (optional)
2. Configure observability with OpenTelemetry
3. Set up automated backups for your database
4. Implement CI/CD for automatic deployments on push

---

**Note**: This deployment uses the official Letta Dockerfile which includes PostgreSQL with pgvector extension, ensuring full compatibility with Letta's memory and embedding features.
