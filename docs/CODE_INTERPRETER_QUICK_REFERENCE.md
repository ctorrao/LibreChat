# Code Interpreter API Quick Reference

## Quick Start

### 1. Create Assistant with Code Interpreter
```javascript
POST https://api.openai.com/v1/assistants
{
  "model": "gpt-4-turbo-preview",
  "name": "Python Assistant",
  "instructions": "You are a helpful coding assistant.",
  "tools": [
    { "type": "code_interpreter" }
  ],
  "tool_resources": {
    "code_interpreter": {
      "file_ids": ["file-abc123"]  // Optional
    }
  }
}
```

### 2. Create Thread
```javascript
POST https://api.openai.com/v1/threads
{
  "messages": [{
    "role": "user",
    "content": "Calculate the mean of [1, 2, 3, 4, 5]"
  }]
}
```

### 3. Create Run
```javascript
POST https://api.openai.com/v1/threads/{thread_id}/runs
{
  "assistant_id": "asst_abc123"
}
```

### 4. Retrieve Run Steps
```javascript
GET https://api.openai.com/v1/threads/{thread_id}/runs/{run_id}/steps
```

## Response Examples

### Code Interpreter Output - Logs
```json
{
  "id": "step_abc123",
  "type": "tool_calls",
  "status": "completed",
  "step_details": {
    "type": "tool_calls",
    "tool_calls": [{
      "id": "call_abc123",
      "type": "code_interpreter",
      "code_interpreter": {
        "input": "import numpy as np\nresult = np.mean([1, 2, 3, 4, 5])\nprint(f'Mean: {result}')",
        "outputs": [{
          "type": "logs",
          "logs": "Mean: 3.0"
        }]
      }
    }]
  }
}
```

### Code Interpreter Output - Image
```json
{
  "id": "step_xyz789",
  "type": "tool_calls",
  "status": "completed",
  "step_details": {
    "type": "tool_calls",
    "tool_calls": [{
      "id": "call_xyz789",
      "type": "code_interpreter",
      "code_interpreter": {
        "input": "import matplotlib.pyplot as plt\nplt.plot([1,2,3], [1,4,9])\nplt.savefig('plot.png')",
        "outputs": [{
          "type": "image",
          "image": {
            "file_id": "file-generated123"
          }
        }]
      }
    }]
  }
}
```

## Key Concepts

### Tool Types
- `code_interpreter` - Execute Python code
- `retrieval` - Search knowledge base (deprecated, use file_search)
- `function` - Call custom functions

### Run Statuses
- `queued` - Waiting to execute
- `in_progress` - Currently executing
- `completed` - Finished successfully
- `requires_action` - Needs function call response
- `failed` - Execution failed
- `cancelled` - User cancelled
- `expired` - Timeout exceeded

### Step Types
- `message_creation` - Assistant created a message
- `tool_calls` - Tool (code interpreter) was executed

### Output Types
- `logs` - Text output from print statements
- `image` - Generated visualization or chart

## Limits

- **Files per code interpreter**: 20 maximum
- **Code execution timeout**: ~60 seconds
- **File size**: Per OpenAI file upload limits
- **Sandboxed environment**: Python 3.11 with common data science libraries

## Available Python Libraries

Common pre-installed libraries include:
- numpy
- pandas
- matplotlib
- scipy
- scikit-learn
- pillow
- And many more data science libraries

## Error Handling

### Common Error Codes
- `server_error` - OpenAI service issue
- `rate_limit_exceeded` - Too many requests
- `invalid_request_error` - Malformed request
- `authentication_error` - Invalid API key

### Example Error Response
```json
{
  "error": {
    "message": "Rate limit exceeded",
    "type": "rate_limit_error",
    "code": "rate_limit_exceeded"
  }
}
```

## Best Practices

1. **File Management**: Clean up unused files after processing
2. **Error Handling**: Always check run status and handle failures
3. **Timeouts**: Implement polling with exponential backoff
4. **Code Structure**: Write clear, documented code for the interpreter
5. **Output Handling**: Check output type before processing
6. **Resource Limits**: Stay within file and execution limits

## Security

- Code runs in isolated sandbox
- No network access from code interpreter
- Only uploaded files are accessible
- All executions are logged
- Authentication required via API key

## Related Documentation

- Full API Reference: `/api/app/clients/tools/.well-known/openapi/code_interpreter.yaml`
- Implementation Guide: `/docs/CODE_INTERPRETER_API.md`
- LibreChat Integration: See source files in `/api/server/services/Runs/`

## Example Use Cases

### Data Analysis
```python
import pandas as pd
import numpy as np

data = pd.DataFrame({
    'values': [1, 2, 3, 4, 5]
})
print(f"Mean: {data['values'].mean()}")
print(f"Std: {data['values'].std()}")
```

### Visualization
```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.figure(figsize=(10, 6))
plt.plot(x, y)
plt.title('Sine Wave')
plt.xlabel('x')
plt.ylabel('sin(x)')
plt.grid(True)
plt.savefig('sine_wave.png', dpi=300, bbox_inches='tight')
```

### File Processing
```python
# Assumes file was uploaded with file_ids
import csv

with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)
```

## Support

For issues or questions:
- Check OpenAI documentation: https://platform.openai.com/docs
- LibreChat GitHub: https://github.com/ctorrao/LibreChat
- OpenAI Community: https://community.openai.com
