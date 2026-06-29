# 📚 API Documentation & Examples

## Overview

Text Summarizer API provides RESTful endpoints for text summarization with real-time WebSocket updates.

**Base URL:** `http://localhost:8000`

---

## 🔗 REST API Endpoints

### 1. Create Summarization Task

Create a new text summarization task.

**Endpoint:** `POST /summarize`

**Request Body:**
```json
{
  "text": "Your long text here that needs to be summarized..."
}
```

**Response:**
```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "queued"
}
```

**Status Codes:**
- `200` - Task created successfully
- `422` - Validation error (empty text)

**cURL Example:**
```bash
curl -X POST http://localhost:8000/summarize \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Artificial intelligence is transforming the world..."
  }'
```

**Python Example:**
```python
import requests

response = requests.post(
    "http://localhost:8000/summarize",
    json={"text": "Your long text here..."}
)
task = response.json()
print(f"Task ID: {task['task_id']}, Status: {task['status']}")
```

**JavaScript Example:**
```javascript
fetch('http://localhost:8000/summarize', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    text: 'Your long text here...'
  })
})
.then(res => res.json())
.then(data => console.log('Task created:', data));
```

---

### 2. List All Tasks

Retrieve all summarization tasks (newest first).

**Endpoint:** `GET /tasks`

**Response:**
```json
[
  {
    "task_id": "550e8400-e29b-41d4-a716-446655440000",
    "status": "done",
    "text": "Original text...",
    "summary": "Generated summary..."
  },
  {
    "task_id": "660e8400-e29b-41d4-a716-446655440001",
    "status": "queued",
    "text": "Another text...",
    "summary": ""
  }
]
```

**Status Values:**
- `queued` - Task is waiting in queue
- `processing` - Task is being processed by worker
- `done` - Task completed successfully
- `failed` - Task failed with error

**cURL Example:**
```bash
curl http://localhost:8000/tasks
```

**Python Example:**
```python
import requests

response = requests.get("http://localhost:8000/tasks")
tasks = response.json()

for task in tasks:
    print(f"{task['task_id']}: {task['status']}")
```

---

### 3. Delete Task

Remove a specific task from history.

**Endpoint:** `DELETE /tasks/{task_id}`

**Response:**
```json
{
  "ok": true
}
```

**cURL Example:**
```bash
curl -X DELETE http://localhost:8000/tasks/550e8400-e29b-41d4-a716-446655440000
```

**Python Example:**
```python
import requests

task_id = "550e8400-e29b-41d4-a716-446655440000"
response = requests.delete(f"http://localhost:8000/tasks/{task_id}")
print(response.json())
```

---

## 🔌 WebSocket API

Real-time updates about task status changes.

**Endpoint:** `WS /ws`

### Connection

**JavaScript Example:**
```javascript
const ws = new WebSocket('ws://localhost:8000/ws');

ws.onopen = () => {
  console.log('Connected to WebSocket');
};

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);
  console.log('Update received:', update);
  
  // Handle different event types
  if (update.event === 'created') {
    console.log('New task created:', update.task_id);
  } else if (update.event === 'updated') {
    console.log('Task completed:', update.task_id);
    console.log('Summary:', update.summary);
  } else if (update.event === 'deleted') {
    console.log('Task deleted:', update.task_id);
  }
};

ws.onclose = () => {
  console.log('Disconnected from WebSocket');
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};
```

**Python Example (using websockets library):**
```python
import asyncio
import websockets
import json

async def listen_updates():
    uri = "ws://localhost:8000/ws"
    async with websockets.connect(uri) as websocket:
        print("Connected to WebSocket")
        
        async for message in websocket:
            update = json.loads(message)
            print(f"Update: {update}")
            
            if update['event'] == 'updated':
                print(f"Task {update['task_id']} completed!")
                print(f"Summary: {update['summary']}")

asyncio.run(listen_updates())
```

### Event Types

**1. Task Created:**
```json
{
  "event": "created",
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "queued",
  "text": "Original text..."
}
```

**2. Task Updated (Completed):**
```json
{
  "event": "updated",
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "done",
  "summary": "Generated summary...",
  "text": "Original text..."
}
```

**3. Task Deleted:**
```json
{
  "event": "deleted",
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

## 📊 Complete Workflow Example

### Python Script

```python
import requests
import time
import json
from websockets.sync.client import connect

BASE_URL = "http://localhost:8000"

def create_task(text):
    """Create a new summarization task"""
    response = requests.post(
        f"{BASE_URL}/summarize",
        json={"text": text}
    )
    return response.json()

def get_task_status(task_id):
    """Get all tasks and find specific one"""
    response = requests.get(f"{BASE_URL}/tasks")
    tasks = response.json()
    
    for task in tasks:
        if task['task_id'] == task_id:
            return task
    return None

def main():
    # 1. Create a task
    text = """
    Artificial intelligence (AI) is intelligence demonstrated by machines,
    in contrast to the natural intelligence displayed by humans and animals.
    Leading AI textbooks define the field as the study of "intelligent agents"
    """
    
    print("Creating summarization task...")
    task = create_task(text)
    task_id = task['task_id']
    print(f"Task created: {task_id}")
    
    # 2. Poll for completion (or use WebSocket)
    print("Waiting for completion...")
    while True:
        task_status = get_task_status(task_id)
        
        if task_status and task_status['status'] == 'done':
            print(f"\n✅ Summary: {task_status['summary']}")
            break
        
        time.sleep(2)
        print(".", end="", flush=True)

if __name__ == "__main__":
    main()
```

### Node.js Script

```javascript
const axios = require('axios');
const WebSocket = require('ws');

const BASE_URL = 'http://localhost:8000';

async function createTask(text) {
  const response = await axios.post(`${BASE_URL}/summarize`, { text });
  return response.data;
}

function listenForUpdates(taskId) {
  return new Promise((resolve) => {
    const ws = new WebSocket(`ws://localhost:8000/ws`);
    
    ws.on('message', (data) => {
      const update = JSON.parse(data);
      
      if (update.task_id === taskId && update.event === 'updated') {
        console.log('\n✅ Summary:', update.summary);
        ws.close();
        resolve(update);
      }
    });
  });
}

async function main() {
  const text = `
    Artificial intelligence (AI) is intelligence demonstrated by machines,
    in contrast to the natural intelligence displayed by humans and animals.
  `;
  
  console.log('Creating summarization task...');
  const task = await createTask(text);
  console.log('Task created:', task.task_id);
  
  console.log('Waiting for completion...');
  await listenForUpdates(task.task_id);
}

main();
```

---

## 🔒 Error Handling

### Common Errors

**Empty Text:**
```json
{
  "detail": [
    {
      "loc": ["body", "text"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

**Task Not Found:**
```json
{
  "ok": false,
  "error": "not_found"
}
```

### Best Practices

1. **Always validate text before sending**
   - Check for non-empty strings
   - Consider max length limits

2. **Handle WebSocket reconnections**
   - Implement automatic reconnection logic
   - Keep track of connection state

3. **Use proper error handling**
   ```python
   try:
       response = requests.post(url, json=data, timeout=10)
       response.raise_for_status()
   except requests.exceptions.RequestException as e:
       print(f"Error: {e}")
   ```

---

## 🧪 Testing with Postman

1. **Import Collection:**
   - Create new collection in Postman
   - Add requests from examples above

2. **Set Environment Variables:**
   ```
   BASE_URL = http://localhost:8000
   TASK_ID = {{task_id}}  # Auto-populated from responses
   ```

3. **Test Flow:**
   - POST /summarize → Save task_id
   - GET /tasks → Verify task exists
   - Wait for completion
   - DELETE /tasks/{{task_id}} → Clean up

---

## 📈 Rate Limits & Performance

- **No built-in rate limiting** (add for production)
- **Typical response time:** 
  - Task creation: < 100ms
  - Summarization: 5-30s (depends on ML model)
- **Concurrent tasks:** Limited by worker instances

---

## 🔮 Future API Additions

- [ ] `GET /tasks/{task_id}` - Get single task
- [ ] `POST /tasks/batch` - Batch processing
- [ ] `GET /health` - Health check endpoint
- [ ] Authentication tokens
- [ ] Rate limiting headers
- [ ] Webhook callbacks

---

## 📞 Support

For issues or questions:
- GitHub Issues: [loguntsov-ae/TextSummarizer](https://github.com/loguntsov-ae/TextSummarizer/issues)
- Documentation: See [README.md](README.md) and [ARCHITECTURE.md](ARCHITECTURE.md)
