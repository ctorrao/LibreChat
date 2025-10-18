# Code Interpreter API Contract - Quick Reference

## Base URL
```
https://api.openai.com/v1
```

## Authentication
```
Authorization: Bearer YOUR_API_KEY
OpenAI-Beta: assistants=v2
```

## Core API Endpoints

### 1. Thread Management

#### Create Thread
```http
POST /threads
Content-Type: application/json

{
  "messages": [{
    "role": "user",
    "content": "string",
    "file_ids": ["string"]
  }],
  "metadata": {}
}
```

**Response**: Thread object with `id`

### 2. Message Management

#### Create Message
```http
POST /threads/{thread_id}/messages
Content-Type: application/json

{
  "role": "user",
  "content": "string",
  "file_ids": ["string"]
}
```

#### List Messages
```http
GET /threads/{thread_id}/messages?limit=20&order=desc
```

### 3. Run Management

#### Create Run with Code Interpreter
```http
POST /threads/{thread_id}/runs
Content-Type: application/json

{
  "assistant_id": "asst_xxx",
  "tools": [{"type": "code_interpreter"}],
  "model": "gpt-4",
  "instructions": "string"
}
```

**Response**: Run object with `status` field

#### Retrieve Run Status
```http
GET /threads/{thread_id}/runs/{run_id}
```

**Run Status Values**:
- `queued` - Waiting to start
- `in_progress` - Currently executing
- `completed` - Finished successfully
- `requires_action` - Needs function tool outputs
- `failed` - Execution failed
- `cancelled` - User cancelled
- `expired` - Run timed out

#### Cancel Run
```http
POST /threads/{thread_id}/runs/{run_id}/cancel
```

### 4. Run Steps (Code Execution Details)

#### List Run Steps
```http
GET /threads/{thread_id}/runs/{run_id}/steps?limit=20&order=asc
```

**Response**: Array of step objects showing:
- Python code executed (`code_interpreter.input`)
- Execution outputs (`code_interpreter.outputs[]`)
  - Type: `logs` - Text output from print/stdout
  - Type: `image` - Generated image file reference

### 5. File Operations

#### Upload File
```http
POST /files
Content-Type: multipart/form-data

file: <binary>
purpose: assistants
```

**Supported File Types**: CSV, Excel, JSON, PDF, images (PNG, JPG), text files

**Limits**:
- Max file size: 512 MB
- Max files per message: 10

#### Retrieve File Info
```http
GET /files/{file_id}
```

#### Download File Content
```http
GET /files/{file_id}/content
```

## Key Data Structures

### Code Interpreter Tool Call
```json
{
  "id": "call_xxx",
  "type": "code_interpreter",
  "code_interpreter": {
    "input": "import pandas as pd\ndf = pd.read_csv('data.csv')\nprint(df.head())",
    "outputs": [
      {
        "type": "logs",
        "logs": "   col1  col2\n0     1     2\n..."
      },
      {
        "type": "image",
        "image": {
          "file_id": "file-xxx"
        }
      }
    ]
  }
}
```

### Message Content
```json
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": {
        "value": "Here's the analysis...",
        "annotations": []
      }
    },
    {
      "type": "image_file",
      "image_file": {
        "file_id": "file-xxx"
      }
    }
  ]
}
```

## Typical Workflow

```
1. Create Thread
   POST /threads
   → thread_id

2. Add Message with File (optional)
   POST /threads/{thread_id}/messages
   {
     "role": "user",
     "content": "Analyze this data",
     "file_ids": ["file-xxx"]
   }

3. Create Run with Code Interpreter
   POST /threads/{thread_id}/runs
   {
     "assistant_id": "asst_xxx",
     "tools": [{"type": "code_interpreter"}]
   }
   → run_id, status: "queued"

4. Poll Run Status
   GET /threads/{thread_id}/runs/{run_id}
   → status: "in_progress" → "completed"

5. Get Run Steps (see code execution)
   GET /threads/{thread_id}/runs/{run_id}/steps
   → Array of steps with code_interpreter details

6. Get Messages (final response)
   GET /threads/{thread_id}/messages
   → Filter by run_id
```

## Code Interpreter Capabilities

### Python Environment
- **Version**: Python 3.x
- **Libraries**: pandas, numpy, matplotlib, scipy, scikit-learn, requests
- **Timeout**: ~120 seconds per execution
- **Storage**: Temporary, not persistent between runs

### Operations
✅ Data analysis and transformation  
✅ File reading (CSV, Excel, JSON, text)  
✅ Mathematical computations  
✅ Plot/chart generation (matplotlib)  
✅ File generation (CSV, images)  
✅ Text processing  

❌ Network requests (no internet access)  
❌ Persistent storage  
❌ External API calls  
❌ System commands  

## Error Handling

### Common HTTP Status Codes
- `200` - Success
- `400` - Bad request (validation error)
- `401` - Authentication failed
- `404` - Resource not found
- `429` - Rate limit exceeded
- `500` - Server error

### Run Errors
Check `run.last_error` object:
```json
{
  "code": "server_error",
  "message": "Error description"
}
```

## Rate Limits

Refer to OpenAI's current rate limits for your tier:
- Requests per minute
- Tokens per minute
- Files per day

## LibreChat Implementation Files

- **Client Init**: `api/server/services/Endpoints/assistants/initalize.js`
- **Run Management**: `api/server/services/Runs/handle.js`
- **Step Processing**: `api/server/services/AssistantService.js`
- **Thread Management**: `api/server/services/Threads/manage.js`

## Environment Variables

```bash
ASSISTANTS_API_KEY=sk-xxx
ASSISTANTS_BASE_URL=https://api.openai.com/v1  # Optional
OPENAI_ORGANIZATION=org-xxx                     # Optional
PROXY=http://proxy:8080                         # Optional
```

## References

- Full OpenAPI Spec: `api/app/clients/tools/.well-known/openapi/code-interpreter.yaml`
- Integration Guide: `docs/code-interpreter-api-integration.md`
- OpenAI Docs: https://platform.openai.com/docs/assistants/tools/code-interpreter
