# Code Interpreter API Integration Documentation

## Overview

This document describes how LibreChat integrates with the OpenAI Assistants API to enable Code Interpreter capabilities. The Code Interpreter is a powerful tool that allows assistants to write and run Python code in a sandboxed execution environment.

## Architecture

### Key Components

1. **OpenAI Client Initialization** (`api/server/services/Endpoints/assistants/initalize.js`)
   - Initializes the OpenAI SDK client with API key and base URL
   - Supports custom base URLs via `ASSISTANTS_BASE_URL` environment variable
   - Uses OpenAI Beta API version header: `OpenAI-Beta: assistants=v2`

2. **Thread Management** (`api/server/services/Threads/manage.js`)
   - Creates threads for conversation context
   - Adds messages to threads
   - Manages thread metadata

3. **Run Management** (`api/server/services/Runs/`)
   - Creates and monitors assistant runs
   - Polls run status with configurable intervals
   - Handles run cancellation and timeouts

4. **Assistant Service** (`api/server/services/AssistantService.js`)
   - Processes run steps including code interpreter tool calls
   - Handles code interpreter outputs (logs and images)
   - Manages file retrieval and processing

## API Flow

### 1. Thread Creation

```javascript
// Create a new thread or add message to existing thread
const { thread_id } = await openai.beta.threads.create({
  messages: [{
    role: "user",
    content: "Analyze this data...",
    file_ids: ["file-abc123"]  // Optional files for code interpreter
  }],
  metadata: { conversationId: "..." }
});
```

### 2. Run Creation with Code Interpreter

```javascript
// Create a run with code_interpreter tool enabled
const run = await openai.beta.threads.runs.create(thread_id, {
  assistant_id: "asst_abc123",
  tools: [{ type: "code_interpreter" }],  // Enable code interpreter
  instructions: "Use code to analyze the data...",
  model: "gpt-4"
});
```

### 3. Run Polling and Step Monitoring

```javascript
// Poll run status until completion
while (run.status === "in_progress" || run.status === "queued") {
  // Fetch run steps to monitor code execution
  const steps = await openai.beta.threads.runs.steps.list(run_id, { thread_id });
  
  // Process code interpreter steps
  for (const step of steps.data) {
    if (step.type === "tool_calls") {
      for (const toolCall of step.step_details.tool_calls) {
        if (toolCall.type === "code_interpreter") {
          // Access executed code
          const code = toolCall.code_interpreter.input;
          
          // Process outputs
          for (const output of toolCall.code_interpreter.outputs) {
            if (output.type === "logs") {
              // Handle text/log output
              console.log(output.logs);
            } else if (output.type === "image") {
              // Download generated image
              const file = await retrieveFile(output.image.file_id);
            }
          }
        }
      }
    }
  }
  
  await sleep(pollIntervalMs);
  run = await openai.beta.threads.runs.retrieve(run_id, { thread_id });
}
```

### 4. Message Retrieval

```javascript
// Get assistant's response after run completes
const messages = await openai.beta.threads.messages.list(thread_id);
const assistantMessages = messages.data.filter(msg => msg.run_id === run_id);
```

## Code Interpreter Tool Details

### Tool Type Specification

```javascript
{
  type: "code_interpreter"
}
```

### Input/Output Structure

#### Code Execution Input
```javascript
{
  id: "call_abc123",
  type: "code_interpreter",
  code_interpreter: {
    input: "import pandas as pd\ndf = pd.read_csv('data.csv')\nprint(df.describe())",
    outputs: [...]
  }
}
```

#### Log Output
```javascript
{
  type: "logs",
  logs: "       column1  column2\ncount     100      100\nmean      50.5     ..."
}
```

#### Image Output
```javascript
{
  type: "image",
  image: {
    file_id: "file-xyz789"  // Can be downloaded via Files API
  }
}
```

## File Handling

### Upload Files for Code Interpreter

```javascript
// Upload a file
const file = await openai.files.create({
  file: fs.createReadStream('data.csv'),
  purpose: 'assistants'
});

// Attach to message
await openai.beta.threads.messages.create(thread_id, {
  role: "user",
  content: "Analyze this CSV",
  file_ids: [file.id]
});
```

### Download Generated Files

```javascript
// Download image or other output file
const fileContent = await openai.files.retrieveContent(file_id);
```

## Environment Variables

```bash
# OpenAI API Configuration
ASSISTANTS_API_KEY=sk-...                    # OpenAI API key
ASSISTANTS_BASE_URL=https://api.openai.com/v1  # Optional custom base URL

# Proxy Configuration (optional)
PROXY=http://proxy:8080

# Organization (optional)
OPENAI_ORGANIZATION=org-...
```

## Run Status Flow

```
queued → in_progress → completed
                    → requires_action  # For function tools (not code_interpreter)
                    → cancelling → cancelled
                    → failed
                    → expired
```

## Code Interpreter Capabilities

### Supported Operations
- **Python Code Execution**: Full Python 3 environment in sandbox
- **Data Analysis**: pandas, numpy, scipy, scikit-learn available
- **File Processing**: Read CSV, Excel, JSON, text files
- **Visualization**: Generate plots with matplotlib
- **File Generation**: Create output files (CSV, images, etc.)

### File Type Support
- Text: txt, csv, json, xml
- Spreadsheets: xlsx, csv
- Documents: pdf (text extraction)
- Images: png, jpg, gif (for vision models)
- Code: py, js, java, etc.

### Limitations
- Execution timeout: ~120 seconds per code execution
- File size limit: 512 MB per file
- Maximum 10 files per message
- No network/internet access from code
- No persistent storage between runs

## Implementation References

### Key Files in LibreChat

1. **Client Initialization**
   - `api/server/services/Endpoints/assistants/initalize.js`
   - Creates OpenAI client with beta assistants support

2. **Run Management**
   - `api/server/services/Runs/RunManager.js`: Manages run steps
   - `api/server/services/Runs/handle.js`: Creates and waits for runs
   - `api/server/services/Runs/methods.js`: Helper methods

3. **Assistant Service**
   - `api/server/services/AssistantService.js`: Main service logic
   - Handles code interpreter outputs
   - Processes images and files

4. **Thread Management**
   - `api/server/services/Threads/manage.js`: Thread operations
   - `api/server/services/Threads/processMessages.js`: Message processing

5. **Type Definitions**
   - `api/typedefs.js`: TypeScript/JSDoc definitions for all types

### Tool Call Processing

The code interpreter tool calls are processed in `AssistantService.js`:

```javascript
if (toolCall.type === ToolCallTypes.CODE_INTERPRETER && 
    step.status === StepStatus.COMPLETED) {
  const { outputs } = toolCall[toolCall.type];
  
  for (const output of outputs) {
    if (output.type === 'image') {
      const { file_id } = output.image;
      const file = await retrieveAndProcessFile({
        openai,
        client: openai,
        file_id,
        basename: `${file_id}.png`,
      });
      // Process and display image
    }
  }
}
```

## Error Handling

### Common Errors

1. **Run Timeout**: Run exceeds maximum execution time
   - Default timeout: 180 seconds
   - Configurable via `timeout` parameter

2. **Code Execution Error**: Python code fails during execution
   - Check `step.last_error` for details
   - Run status will be `failed`

3. **File Access Error**: Code cannot access specified file
   - Ensure file is properly uploaded with `purpose: 'assistants'`
   - Verify file_id is correct

4. **Rate Limiting**: Too many requests to OpenAI API
   - Implement exponential backoff
   - Use polling interval wisely

## Best Practices

1. **Polling Interval**: Use 2-3 second intervals to balance responsiveness and API usage
2. **Timeout Configuration**: Set appropriate timeouts based on expected code complexity
3. **File Management**: Clean up unused files periodically
4. **Error Recovery**: Implement retry logic for transient failures
5. **Progress Tracking**: Monitor run steps to provide user feedback
6. **Cancellation**: Implement cancellation to allow users to stop long-running operations

## API Version

LibreChat uses OpenAI Assistants API v2 (beta). The API is subject to changes as it's in beta.

**Header**: `OpenAI-Beta: assistants=v2`

## Additional Resources

- [OpenAI Assistants API Documentation](https://platform.openai.com/docs/assistants/overview)
- [Code Interpreter Tool Documentation](https://platform.openai.com/docs/assistants/tools/code-interpreter)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference/assistants)
