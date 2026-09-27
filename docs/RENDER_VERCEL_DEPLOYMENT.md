# CIPHER — Render Free Deployment

## Architecture

```text
Render Free Web Service
  FastAPI + existing frontend
        |
        +-- Supabase PostgreSQL  (cases, users, evidence metadata, RAG, review)
        +-- Supabase Storage     (evidence and face-profile media)
        +-- Neo4j Aura           (graph projection)
        +-- Gemini/OpenRouter    (optional cloud AI)
```

The deployment uses **one Render Web Service**. It does not require a Render Worker or Render Persistent Disk.

## Supabase

Apply `supabase/migrations/001_init_schema.sql` and create a private Storage bucket named `cipher-evidence`.

Required variables:

```text
DATABASE_URL
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
SUPABASE_STORAGE_BUCKET=cipher-evidence
```

The service-role key must never be exposed to browser code.

## Neo4j Aura

```text
NEO4J_URI=neo4j+s://YOUR_INSTANCE.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=...
NEO4J_DATABASE=neo4j
NEO4J_REQUIRED_FOR_GRAPH=true
```

## Render

The included `render.yaml` defines the complete Free deployment.

Build command:

```bash
pip install --upgrade pip && pip install -r requirements.txt
```

Start command:

```bash
uvicorn app:app --host 0.0.0.0 --port $PORT
```

The existing frontend is served by FastAPI, so a separate frontend service is not required.

## Important environment variables

```text
SECRET_KEY=<long-random-secret>
DATABASE_URL=<Supabase PostgreSQL connection string>
SUPABASE_URL=<Supabase project URL>
SUPABASE_SERVICE_ROLE_KEY=<service role key>
SUPABASE_STORAGE_BUCKET=cipher-evidence
NEO4J_URI=<Aura URI>
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=<password>
NEO4J_DATABASE=neo4j
NEO4J_REQUIRED_FOR_GRAPH=true
GEMINI_API_KEY=<optional>
CIPHER_CORS_ORIGINS=<your browser origin(s)>
CIPHER_UPLOADS_DIR=/tmp/cipher-uploads
CIPHER_RAG_DIR=/tmp/cipher-rag-cache
CIPHER_USE_BACKGROUND_TASKS=false
SEED_DEMO_DATA=false
```

`/tmp` is only a cache. Evidence and face images are stored in Supabase Storage, and RAG chunks/embeddings are stored in PostgreSQL. Therefore a Render restart does not become the source of truth for application data.

## RAG

RAG remains part of CIPHER. PostgreSQL stores `rag_documents` and `rag_chunks`; FAISS is an optional local acceleration cache. If the Render cache disappears after a restart, retrieval can fall back to the database-stored embeddings/lexical path.

## Face intelligence

InsightFace/ArcFace remains enabled. Face reference images are stored in Supabase Storage when configured, while embeddings and profile metadata remain in PostgreSQL. The browser never receives the private Storage service-role key.

## Security

- `SEED_DEMO_DATA=false` for production.
- Never commit `.env`, database files, uploads, model caches, or generated runtime state.
- Use a strong `SECRET_KEY` in Render.
- Keep the Supabase Storage bucket private.
- Restrict `CIPHER_CORS_ORIGINS` to the actual browser origin(s).
