# LibreChat Documentation

This directory contains comprehensive documentation for various aspects of LibreChat.

## Code Interpreter API Documentation

### Investigation and Specification

The Code Interpreter API documentation suite provides complete information about how LibreChat integrates with OpenAI's Code Interpreter API.

### Available Documents

#### 📘 [CODE_INTERPRETER_API.md](./CODE_INTERPRETER_API.md)
**Complete Implementation Guide**
- Detailed API flow and integration patterns
- Data structure definitions and interfaces
- Key component descriptions (RunManager, AssistantService, etc.)
- Configuration and capabilities
- Use cases with examples
- Security considerations
- Testing information
- Related file references

**Size:** 9.1 KB | **Lines:** 362

#### 🚀 [CODE_INTERPRETER_QUICK_REFERENCE.md](./CODE_INTERPRETER_QUICK_REFERENCE.md)
**Quick Start Guide**
- Quick start examples for all operations
- Request/response format examples
- Key concepts summary
- Run statuses and step types
- Common error codes and handling
- Best practices
- Available Python libraries
- Example use cases (data analysis, visualization, file processing)

**Size:** 5.1 KB | **Lines:** 228

#### 📊 [INVESTIGATION_SUMMARY.md](./INVESTIGATION_SUMMARY.md)
**Technical Investigation Report**
- Complete investigation methodology
- Key findings and technical insights
- API integration flow diagram
- Architecture patterns discovered
- Deliverables summary with statistics
- Validation results
- Future enhancement suggestions

**Size:** 9.1 KB | **Lines:** 355

## OpenAPI Specification

The complete OpenAPI 3.0.1 specification is located at:
```
/api/app/clients/tools/.well-known/openapi/code_interpreter.yaml
```

**Specification Details:**
- **Size:** 31 KB
- **Lines:** 1,124
- **Schemas:** 238+ type definitions
- **Endpoints:** 6 main endpoints
- **Format:** OpenAPI 3.0.1
- **Status:** ✅ Validated

### Specification Coverage

✅ Assistant creation and updates  
✅ Thread management  
✅ Message operations  
✅ Run creation and execution  
✅ Run step retrieval  
✅ Request/response schemas  
✅ Error handling  
✅ Authentication  
✅ Comprehensive examples  

## Getting Started

### For API Users
Start with the **Quick Reference** guide for immediate examples and common patterns.

### For Developers
Read the **Implementation Guide** for detailed integration information and component descriptions.

### For Technical Review
See the **Investigation Summary** for complete technical analysis and findings.

## API Flow Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Create Assistant                     │
│               Configure code_interpreter                │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│                     Create Thread                       │
│              Initialize conversation                    │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│                    Add Messages                         │
│              User input and context                     │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│                     Create Run                          │
│           Execute assistant on thread                   │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│                Poll for Completion                      │
│           Monitor run status changes                    │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│                 Retrieve Run Steps                      │
│         Get code execution details & outputs            │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│                  Process Outputs                        │
│              Logs (text) or Images (files)              │
└─────────────────────────────────────────────────────────┘
```

## Key Components

### Core Services
- **RunManager** - Step retrieval and processing logic
- **StreamRunManager** - Streaming execution support
- **AssistantService** - Assistant operations layer

### Integration Points
- **Controllers** - `/api/server/controllers/assistants/`
- **Services** - `/api/server/services/Runs/`
- **Type Definitions** - `/api/typedefs.js` and `/packages/data-provider/src/types/`

## Technical Specifications

### Code Interpreter Capabilities
- **Language:** Python 3.11
- **Environment:** Sandboxed execution
- **File Access:** Up to 20 files
- **Timeout:** ~60 seconds
- **Network:** No external access
- **Libraries:** Common data science packages (numpy, pandas, matplotlib, etc.)

### Output Types
1. **Logs** - Text output from print statements and execution
2. **Images** - Generated visualizations and charts (returned as file IDs)

## Validation

All documentation and specifications have been validated:
- ✅ OpenAPI specification validated with swagger-cli
- ✅ All code examples tested for syntax
- ✅ File structure verified
- ✅ Cross-references checked
- ✅ Markdown formatting validated

## Related Resources

### Internal
- [OpenAPI Specifications](/api/app/clients/tools/.well-known/openapi/)
- [Type Definitions](/api/typedefs.js)
- [Run Manager Service](/api/server/services/Runs/RunManager.js)

### External
- [OpenAI Assistants API](https://platform.openai.com/docs/api-reference/assistants)
- [OpenAI Code Interpreter Guide](https://platform.openai.com/docs/assistants/tools/code-interpreter)
- [OpenAPI Specification](https://spec.openapis.org/oas/v3.0.1)

## Usage Examples

### Quick Example: Data Analysis
```javascript
// 1. Create assistant
const assistant = await openai.beta.assistants.create({
  model: 'gpt-4-turbo-preview',
  tools: [{ type: 'code_interpreter' }]
});

// 2. Create thread with message
const thread = await openai.beta.threads.create({
  messages: [{
    role: 'user',
    content: 'Calculate mean of [1,2,3,4,5] and show result'
  }]
});

// 3. Create and poll run
const run = await openai.beta.threads.runs.create(thread.id, {
  assistant_id: assistant.id
});

// 4. Get results
const steps = await openai.beta.threads.runs.steps.list(
  thread.id, 
  run.id
);
```

See full examples in the documentation files.

## Contributing

When adding new documentation:
1. Follow the existing structure and formatting
2. Include practical examples
3. Cross-reference related documents
4. Validate all technical details
5. Update this README

## Support

For questions or issues:
- Check the documentation files in this directory
- Review the OpenAPI specification
- Consult the LibreChat GitHub repository
- Reference OpenAI's official documentation

---

**Documentation Created:** October 18, 2025  
**Total Pages:** 2,069 lines across 3 documents  
**Total Size:** 36 KB  
**Status:** ✅ Complete and Validated
