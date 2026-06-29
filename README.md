# TextSummarizer

Async text summarization service built on FastAPI, Redis and WebSocket.

Tasks are submitted via an API Gateway, queued in Redis, processed asynchronously by a Worker (OpenAI or Hugging Face models), and results are streamed back to the client over WebSocket.

## Architecture

```
Client
  ├── POST /summarize → API Gateway → Redis queue
  └── WS  /ws/{task_id} ← Worker (async result)
```

The gateway and worker are decoupled services: gateway enqueues, worker polls and publishes, client subscribes.

## Stack

- **FastAPI** — API gateway and WebSocket endpoint
- **Redis** — task queue and pub/sub
- **OpenAI / Hugging Face** — summarization models (configurable)
- **WebSocket** — real-time result streaming
- **Docker Compose** — local deployment

## Quick start

```bash
cp .env.example .env           # add OPENAI_API_KEY or HF_TOKEN
docker compose up --build
```

Submit a task:
```bash
curl -X POST http://localhost:8000/summarize \
  -H "Content-Type: application/json" \
  -d '{"text": "Your long text here..."}'
# → {"task_id": "abc-123"}
```

Then connect to `ws://localhost:8000/ws/abc-123` to receive the result.

## Running tests

```bash
pip install -r requirements-test.txt
pytest tests/ -v
```

## Docs

- [Architecture](docs/ARCHITECTURE.md) — design decisions, Repository pattern, Pub/Sub rationale
- [API Examples](docs/API_EXAMPLES.md) — curl, Python and JS usage examples

## License

MIT
