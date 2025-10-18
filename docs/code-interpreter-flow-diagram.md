# Code Interpreter API Flow Diagram

## High-Level Architecture

```
┌──────────────┐
│  LibreChat   │
│   Frontend   │
└──────┬───────┘
       │ User Request
       │ (with file optional)
       ▼
┌──────────────────────────────────────┐
│      LibreChat Backend               │
│  (api/server/controllers/)           │
│                                      │
│  ┌────────────────────────────┐    │
│  │  Assistant Controller      │    │
│  │  (chatV2.js)               │    │
│  └────────┬───────────────────┘    │
│           │                         │
│           ▼                         │
│  ┌────────────────────────────┐    │
│  │  OpenAI Client Init        │    │
│  │  (initalize.js)            │    │
│  └────────┬───────────────────┘    │
└───────────┼──────────────────────────┘
            │
            │ HTTPS/TLS
            ▼
┌────────────────────────────────────┐
│   OpenAI Assistants API (beta v2)  │
│   https://api.openai.com/v1        │
│                                    │
│   ┌──────────────────────────┐   │
│   │   Code Interpreter       │   │
│   │   (Sandboxed Python)     │   │
│   └──────────────────────────┘   │
└────────────────────────────────────┘
```

## Detailed API Call Flow

```
┌─────────────┐
│   Client    │
│  (Browser)  │
└──────┬──────┘
       │
       │ 1. POST /assistants/chat
       │    { text, assistant_id, file_ids }
       ▼
┌────────────────────────────────────────────┐
│            LibreChat Server                │
│                                            │
│  Step 1: Initialize Thread                │
│  ┌──────────────────────────────────────┐ │
│  │ if new conversation:                 │ │
│  │   POST /threads                      │ │
│  │   { messages: [{ role, content }] }  │ │
│  │ else:                                │ │
│  │   POST /threads/{id}/messages        │ │
│  │   { role, content, file_ids }        │ │
│  └──────────────────────────────────────┘ │
│           │                                │
│           │ thread_id                      │
│           ▼                                │
│  Step 2: Create Run with Code Interpreter │
│  ┌──────────────────────────────────────┐ │
│  │ POST /threads/{thread_id}/runs       │ │
│  │ {                                    │ │
│  │   assistant_id,                      │ │
│  │   tools: [{ type: "code_interpreter" }],│
│  │   model, instructions                │ │
│  │ }                                    │ │
│  └──────────────────────────────────────┘ │
│           │                                │
│           │ run_id, status: "queued"       │
│           ▼                                │
│  Step 3: Poll Run Status (Loop)           │
│  ┌──────────────────────────────────────┐ │
│  │ while (status in [queued, in_progress])│
│  │   GET /threads/{thread_id}/runs/{run_id}│
│  │   status → "in_progress"             │ │
│  │                                      │ │
│  │   Fetch Run Steps:                   │ │
│  │   GET /threads/{thread_id}/runs/     │ │
│  │       {run_id}/steps                 │ │
│  │                                      │ │
│  │   Process Steps (if new):            │ │
│  │   - code_interpreter input (code)    │ │
│  │   - code_interpreter outputs:        │ │
│  │     * logs (stdout)                  │ │
│  │     * images (file_id)               │ │
│  │                                      │ │
│  │   sleep(2000ms)                      │ │
│  └──────────────────────────────────────┘ │
│           │                                │
│           │ status: "completed"            │
│           ▼                                │
│  Step 4: Get Final Messages               │
│  ┌──────────────────────────────────────┐ │
│  │ GET /threads/{thread_id}/messages    │ │
│  │ filter by run_id                     │ │
│  └──────────────────────────────────────┘ │
│           │                                │
│           │ messages[]                     │
│           ▼                                │
│  Step 5: Download Files (if any)          │
│  ┌──────────────────────────────────────┐ │
│  │ for each image in content:           │ │
│  │   GET /files/{file_id}/content       │ │
│  │   save and serve to client           │ │
│  └──────────────────────────────────────┘ │
│           │                                │
└───────────┼────────────────────────────────┘
            │
            │ Stream response to client
            ▼
┌─────────────────────────────┐
│   Client receives:          │
│   - Text content            │
│   - Code execution details  │
│   - Generated images        │
│   - Tool call progress      │
└─────────────────────────────┘
```

## Code Interpreter Execution Flow

```
┌────────────────────────────────────────┐
│  Assistant Run Created                 │
│  tools: [{ type: "code_interpreter" }] │
└────────────┬───────────────────────────┘
             │
             ▼
┌────────────────────────────────────────┐
│  OpenAI Backend                        │
│                                        │
│  1. LLM decides to use code            │
│     ┌─────────────────────────┐       │
│     │ Generate Python code    │       │
│     │ based on user request   │       │
│     └──────────┬──────────────┘       │
│                ▼                       │
│  2. Code Interpreter Tool Call         │
│     ┌─────────────────────────┐       │
│     │ type: "code_interpreter"│       │
│     │ input: "import pandas..."│       │
│     └──────────┬──────────────┘       │
│                ▼                       │
│  3. Execute in Sandbox                 │
│     ┌─────────────────────────┐       │
│     │  Python 3.x Runtime     │       │
│     │  - Libraries: pandas,   │       │
│     │    numpy, matplotlib    │       │
│     │  - File access (if any) │       │
│     │  - No internet          │       │
│     │  - 120s timeout         │       │
│     └──────────┬──────────────┘       │
│                ▼                       │
│  4. Collect Outputs                    │
│     ┌─────────────────────────┐       │
│     │ outputs: [              │       │
│     │   { type: "logs",       │       │
│     │     logs: "..." },      │       │
│     │   { type: "image",      │       │
│     │     image: { file_id }} │       │
│     │ ]                       │       │
│     └──────────┬──────────────┘       │
│                ▼                       │
│  5. Continue or Complete               │
│     ┌─────────────────────────┐       │
│     │ If more code needed:    │       │
│     │   → Loop back to step 1 │       │
│     │ Else:                   │       │
│     │   → Generate response   │       │
│     │   → Mark run complete   │       │
│     └─────────────────────────┘       │
└────────────────────────────────────────┘
```

## Run Step States

```
┌─────────────────────────────────────────────────┐
│              Run Step Lifecycle                 │
└─────────────────────────────────────────────────┘

Step Created
    │
    ▼
┌───────────────┐
│  in_progress  │ ◄──── Code is being executed
└───────┬───────┘
        │
        ├─── (if successful) ───►┌────────────┐
        │                        │ completed  │
        │                        └────────────┘
        │
        ├─── (if error) ───────►┌────────────┐
        │                       │  failed    │
        │                       └────────────┘
        │
        └─── (if cancelled) ───►┌────────────┐
                                │ cancelled  │
                                └────────────┘
```

## File Processing Flow

```
User uploads file
       │
       ▼
┌──────────────────────┐
│ POST /files          │
│ purpose: "assistants"│
└──────┬───────────────┘
       │ file_id
       ▼
┌────────────────────────────┐
│ Attach to message          │
│ file_ids: ["file-abc123"]  │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│ Run with code_interpreter  │
└────────┬───────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Python code can access file:    │
│ df = pd.read_csv('uploaded.csv')│
└────────┬────────────────────────┘
         │
         ▼
┌────────────────────────────────┐
│ Generate output (optional)     │
│ plt.savefig('plot.png')        │
└────────┬───────────────────────┘
         │
         ▼
┌────────────────────────────────┐
│ Output file available:         │
│ { type: "image",               │
│   image: { file_id: "..." } }  │
└────────┬───────────────────────┘
         │
         ▼
┌────────────────────────────────┐
│ Download output:               │
│ GET /files/{file_id}/content   │
└────────────────────────────────┘
```

## Error Handling Flow

```
                  ┌─────────────────┐
                  │   API Request   │
                  └────────┬────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
         ▼                 ▼                 ▼
    ┌────────┐      ┌──────────┐      ┌──────────┐
    │ 401    │      │ 429      │      │ 500      │
    │ Auth   │      │ Rate     │      │ Server   │
    │ Failed │      │ Limit    │      │ Error    │
    └────────┘      └──────────┘      └──────────┘
         │                 │                 │
         │                 │                 │
         ▼                 ▼                 ▼
    Check API Key    Exponential      Retry with
                     Backoff          Exponential
                                     Backoff

Run Errors:
    
    run.status === "failed"
           │
           ▼
    Check run.last_error
           │
           ├─► code: "server_error"
           ├─► code: "rate_limit_exceeded"  
           └─► message: "..."
```

## LibreChat Implementation Mapping

```
┌─────────────────────────────────────────────────┐
│          LibreChat Code Structure               │
└─────────────────────────────────────────────────┘

Controller Layer:
  api/server/controllers/assistants/
    ├─ chatV2.js ────────────► Main chat handler
    ├─ helpers.js ───────────► Client initialization
    └─ errors.js ────────────► Error handling

Service Layer:
  api/server/services/
    ├─ Endpoints/assistants/
    │   └─ initalize.js ─────► OpenAI client setup
    │
    ├─ Threads/
    │   ├─ manage.js ────────► Thread operations
    │   └─ processMessages.js► Message processing
    │
    ├─ Runs/
    │   ├─ RunManager.js ────► Step management
    │   ├─ handle.js ────────► Run creation/polling
    │   └─ methods.js ───────► Helper methods
    │
    ├─ AssistantService.js ──► Main service logic
    │                           • Process code_interpreter
    │                           • Handle outputs
    │                           • Manage files
    │
    └─ Files/
        └─ process.js ───────► File download/process

Type Definitions:
  api/typedefs.js ───────────► JSDoc type definitions
```

## Summary

This flow diagram shows:
1. **Request Flow**: From client through LibreChat to OpenAI API
2. **Code Execution**: How Python code is executed in sandbox
3. **Step Processing**: Monitoring and handling run steps
4. **File Handling**: Upload and download flows
5. **Error Management**: Error detection and handling
6. **Code Mapping**: Where to find implementation in LibreChat

For detailed API specifications, see:
- OpenAPI Spec: `api/app/clients/tools/.well-known/openapi/code-interpreter.yaml`
- Integration Guide: `docs/code-interpreter-api-integration.md`
- Quick Reference: `docs/code-interpreter-api-contract.md`
