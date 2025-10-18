# Code Interpreter API Documentation

## Overview

This document describes how LibreChat invokes and integrates with OpenAI's Code Interpreter API. The Code Interpreter is a powerful tool that allows AI assistants to write and execute Python code in a sandboxed environment, process files, and generate data visualizations.

## API Integration Flow

### 1. Assistant Creation with Code Interpreter

The code interpreter is enabled at the assistant level by including it in the `tools` array during assistant creation or update.

**Key Files:**
- `/api/server/controllers/assistants/v1.js` - Assistant creation endpoint
- `/api/server/controllers/assistants/v2.js` - Assistant v2 endpoints

**Example Request:**
```javascript
const assistant = await openai.beta.assistants.create({
  model: 'gpt-4-turbo-preview',
  name: 'Python Code Assistant',
  instructions: 'You are a helpful assistant that can write and execute Python code.',
  tools: [
    { type: 'code_interpreter' }
  ],
  tool_resources: {
    code_interpreter: {
      file_ids: ['file-abc123'] // Optional: Files available to code interpreter
    }
  }
});
```

### 2. Thread and Message Creation

Once an assistant is configured, users interact through threads and messages:

**Key Files:**
- `/api/server/services/Threads/manage.js` - Thread and message management

**Flow:**
```javascript
// Create a thread (if needed)
const thread = await openai.beta.threads.create({
  messages: [{
    role: 'user',
    content: 'Calculate the mean of [1, 2, 3, 4, 5]'
  }]
});

// Or add message to existing thread
await openai.beta.threads.messages.create(thread_id, {
  role: 'user',
  content: 'Now plot this data'
});
```

### 3. Run Execution

Runs execute the assistant on a thread, triggering code interpreter when needed:

**Key Files:**
- `/api/server/services/Runs/handle.js` - Run creation and management
- `/api/server/services/Runs/StreamRunManager.js` - Streaming run management

**Flow:**
```javascript
const run = await openai.beta.threads.runs.create(thread_id, {
  assistant_id: assistant.id
});

// Poll for completion
const completedRun = await waitForRun({ openai, run_id, thread_id });
```

### 4. Run Steps Processing

Run steps contain the actual code interpreter executions and outputs:

**Key Files:**
- `/api/server/services/Runs/RunManager.js` - Core run step processing logic
- `/api/typedefs.js` - Type definitions for code interpreter structures

**Flow:**
```javascript
// Retrieve run steps
const steps = await openai.beta.threads.runs.steps.list(run_id, { thread_id });

// Process code interpreter steps
for (const step of steps.data) {
  if (step.type === 'tool_calls') {
    for (const toolCall of step.step_details.tool_calls) {
      if (toolCall.type === 'code_interpreter') {
        const { input, outputs } = toolCall.code_interpreter;
        
        // Process outputs
        for (const output of outputs) {
          if (output.type === 'logs') {
            console.log('Code output:', output.logs);
          } else if (output.type === 'image') {
            console.log('Generated image:', output.image.file_id);
          }
        }
      }
    }
  }
}
```

## Data Structures

### CodeToolCall

The code interpreter tool call contains the executed code and its outputs:

```typescript
interface CodeToolCall {
  id: string;
  type: 'code_interpreter';
  code_interpreter: {
    input: string;          // Python code that was executed
    outputs: Array<         // Array of outputs
      | { type: 'logs'; logs: string }
      | { type: 'image'; image: { file_id: string } }
    >;
  };
}
```

### Tool Configuration

Tools are configured on assistants:

```typescript
interface Tool {
  type: 'code_interpreter' | 'retrieval' | 'function';
  function?: {            // Only for function type
    name: string;
    description: string;
    parameters: object;
  };
}
```

### Tool Resources

Resources available to tools (like files for code interpreter):

```typescript
interface ToolResources {
  code_interpreter?: {
    file_ids: string[];  // Max 20 files
  };
  file_search?: {
    vector_store_ids: string[];
  };
}
```

## Key Components

### RunManager Class

Located in `/api/server/services/Runs/RunManager.js`, this class manages the retrieval and processing of run steps:

- Tracks seen steps to avoid duplicates
- Fetches run steps based on run status
- Handles different step types (message_creation, tool_calls)
- Generates signatures for deduplication

**Key Method:**
```javascript
async fetchRunSteps({ openai, thread_id, run_id, runStatus, final = false }) {
  const { data: steps } = await openai.beta.threads.runs.steps.list(
    run_id,
    { thread_id }
  );
  
  for (const step of steps) {
    // Process each step, including code_interpreter tool calls
    await this.handleStep({ step, runStatus, final, isLast });
  }
}
```

### Tool Call Signature Generation

The `getToolCallSignature` function creates unique signatures for tool calls:

```javascript
function getToolCallSignature(toolCall) {
  if (toolCall.type === ToolCallTypes.CODE_INTERPRETER) {
    const inputLength = toolCall.code_interpreter?.input?.length ?? 0;
    const outputsLength = toolCall.code_interpreter?.outputs?.length ?? 0;
    return `${toolCall.id}-${toolCall.type}-${inputLength}-${outputsLength}`;
  }
  // ... other tool types
}
```

## Configuration

### Capabilities Declaration

Code interpreter capability is declared in configuration:

**File:** `/api/server/utils/handleText.js`

```javascript
if (assistants) {
  config.capabilities = [
    Capabilities.code_interpreter,
    Capabilities.image_vision,
    Capabilities.retrieval,
    Capabilities.actions,
    Capabilities.tools,
  ];
}
```

### Constants

**File:** `/packages/data-provider/src/types/runs.ts`

```typescript
export enum ToolCallTypes {
  FUNCTION = 'function',
  RETRIEVAL = 'retrieval',
  FILE_SEARCH = 'file_search',
  CODE_INTERPRETER = 'code_interpreter',
  TOOL_CALL = 'tool_call',
}
```

## Use Cases

### 1. Data Analysis
```python
import pandas as pd
import numpy as np

# Load data
data = [1, 2, 3, 4, 5]
mean = np.mean(data)
print(f"Mean: {mean}")
```

### 2. Visualization
```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.plot(x, y)
plt.title('Sine Wave')
plt.savefig('sine_wave.png')
```

### 3. File Processing
```python
# Process uploaded files
with open('data.csv', 'r') as f:
    content = f.read()
    # Process content
```

## Error Handling

Run steps can fail with various error codes:

```javascript
if (step.last_error) {
  console.error('Step failed:', {
    code: step.last_error.code,  // 'server_error', 'rate_limit_exceeded'
    message: step.last_error.message
  });
}
```

## Streaming Support

LibreChat supports streaming run execution:

**File:** `/api/server/services/Runs/StreamRunManager.js`

```javascript
const streamRun = this.openai.beta.threads.runs.createAndStream(
  thread_id,
  body,
  this.streamOptions
);

for await (const event of streamRun) {
  // Process streaming events
}
```

## Testing

Test cases are available in:
- `/api/server/services/Threads/processMessages.spec.js`

Example test structure:
```javascript
const messages = [{
  content: [{
    type: 'text',
    text: {
      value: 'Result',
      annotations: []
    }
  }],
  attachments: [{
    file_id: 'file-abc123',
    tools: [{ type: 'code_interpreter' }]
  }]
}];
```

## OpenAPI Specification

The complete OpenAPI specification for the Code Interpreter API is available at:
- `/api/app/clients/tools/.well-known/openapi/code_interpreter.yaml`

This specification documents all endpoints, request/response schemas, and data structures used in the Code Interpreter integration.

## Security Considerations

1. **Sandboxed Execution**: Code runs in a secure sandbox environment
2. **File Access**: Only files explicitly provided through `file_ids` are accessible
3. **Resource Limits**: Maximum 20 files per code interpreter
4. **Timeout**: Code execution has timeout limits
5. **API Key**: Requires valid OpenAI API authentication

## References

- OpenAI Assistants API Documentation: https://platform.openai.com/docs/api-reference/assistants
- OpenAI Code Interpreter Guide: https://platform.openai.com/docs/assistants/tools/code-interpreter
- LibreChat Repository: https://github.com/ctorrao/LibreChat

## Related Files

### Core Implementation
- `/api/server/services/Runs/RunManager.js` - Run step management
- `/api/server/services/Runs/handle.js` - Run creation and polling
- `/api/server/services/AssistantService.js` - Assistant service layer
- `/api/server/controllers/assistants/v1.js` - Assistant v1 controllers
- `/api/server/controllers/assistants/v2.js` - Assistant v2 controllers

### Type Definitions
- `/api/typedefs.js` - JavaScript type definitions
- `/packages/data-provider/src/types/assistants.ts` - TypeScript assistant types
- `/packages/data-provider/src/types/runs.ts` - TypeScript run types

### Routes and Middleware
- `/api/server/routes/assistants/actions.js` - Assistant action routes
- `/api/server/routes/types/assistants.js` - Assistant type definitions

### Utilities
- `/api/server/utils/handleText.js` - Text handling utilities and config
