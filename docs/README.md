# Code Interpreter API Documentation Index

This directory contains comprehensive documentation for the Code Interpreter API integration in LibreChat.

## 📚 Documentation Overview

### 1. OpenAPI Specification (Formal API Contract)
**Location**: `../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml`

The formal, machine-readable API specification in OpenAPI 3.0.1 format.

**Use this for:**
- Generating API clients/SDKs
- API validation
- Integration with API tools (Swagger, Postman, etc.)
- Automated testing
- Contract-first development

**Size**: 31 KB | **Schemas**: 30+ | **Endpoints**: 10

---

### 2. Integration Guide (For Implementers)
**Location**: `code-interpreter-api-integration.md`

Deep dive into how LibreChat integrates with the Code Interpreter API.

**Use this for:**
- Understanding the implementation architecture
- Learning the complete workflow with code examples
- Environment configuration
- Error handling strategies
- Best practices
- Finding implementation files in the codebase

**Audience**: Backend developers, system architects

---

### 3. API Contract Quick Reference (For Developers)
**Location**: `code-interpreter-api-contract.md`

Concise, at-a-glance reference for the API endpoints and data structures.

**Use this for:**
- Quick lookup of endpoints and parameters
- Copy-paste ready HTTP request examples
- Understanding request/response formats
- Typical workflow reference
- Troubleshooting with error codes

**Audience**: All developers, QA engineers, support staff

---

### 4. Flow Diagrams (For Visual Learners)
**Location**: `code-interpreter-flow-diagram.md`

Visual ASCII diagrams showing system architecture and data flow.

**Use this for:**
- Understanding the big picture
- Tracing requests through the system
- Learning the run lifecycle
- Seeing how components interact
- Code structure navigation

**Audience**: All team members, new developers, stakeholders

---

## 🎯 Quick Navigation by Use Case

### "I want to understand how it works"
1. Start with: [Flow Diagrams](code-interpreter-flow-diagram.md) - See the big picture
2. Then read: [Integration Guide](code-interpreter-api-integration.md) - Learn the details
3. Reference: [OpenAPI Spec](../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml) - See the contract

### "I need to make an API call"
1. Check: [Quick Reference](code-interpreter-api-contract.md) - Find the endpoint
2. Refer to: [OpenAPI Spec](../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml) - See complete schemas

### "I'm debugging an issue"
1. Use: [Quick Reference](code-interpreter-api-contract.md) - Check error codes
2. Review: [Integration Guide](code-interpreter-api-integration.md) - Error handling section
3. Trace: [Flow Diagrams](code-interpreter-flow-diagram.md) - Follow the error flow

### "I'm implementing a new feature"
1. Study: [Integration Guide](code-interpreter-api-integration.md) - Implementation patterns
2. Reference: [OpenAPI Spec](../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml) - Data structures
3. Map: [Flow Diagrams](code-interpreter-flow-diagram.md) - Code location mapping

### "I need to integrate with external tools"
1. Use: [OpenAPI Spec](../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml) - Import into your tool
2. Supplement with: [Quick Reference](code-interpreter-api-contract.md) - Examples

---

## 📊 Documentation Statistics

| Document | Size | Purpose | Detail Level |
|----------|------|---------|--------------|
| OpenAPI Spec | 31 KB | Formal contract | Complete |
| Integration Guide | 8.6 KB | Implementation | High |
| Quick Reference | 5.6 KB | Lookup | Medium |
| Flow Diagrams | 11.5 KB | Visual overview | Conceptual |
| **Total** | **~57 KB** | **Complete coverage** | **All levels** |

---

## 🔑 Key Concepts

### OpenAI Assistants API
The Code Interpreter is part of OpenAI's Assistants API (beta v2), which provides:
- **Threads**: Persistent conversation contexts
- **Messages**: User and assistant messages within threads
- **Runs**: Assistant execution instances
- **Run Steps**: Detailed execution logs including code execution
- **Tools**: Extensions like code_interpreter, retrieval, functions

### Code Interpreter Tool
A sandboxed Python execution environment that can:
- ✅ Run Python 3.x code
- ✅ Access uploaded files (CSV, Excel, PDF, etc.)
- ✅ Generate visualizations (matplotlib)
- ✅ Create output files (images, CSV)
- ✅ Use data science libraries (pandas, numpy, scipy)
- ❌ Access the internet
- ❌ Persist data between runs

### LibreChat Integration
LibreChat acts as:
1. **Client**: Makes API calls to OpenAI
2. **Coordinator**: Manages threads, runs, and messages
3. **Processor**: Handles code execution results
4. **Server**: Serves results to frontend

---

## 🔗 Related Resources

### Internal
- Type definitions: `../api/typedefs.js`
- Client initialization: `../api/server/services/Endpoints/assistants/initalize.js`
- Run management: `../api/server/services/Runs/`
- Assistant service: `../api/server/services/AssistantService.js`

### External
- [OpenAI Assistants API Docs](https://platform.openai.com/docs/assistants/overview)
- [Code Interpreter Tool Guide](https://platform.openai.com/docs/assistants/tools/code-interpreter)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference/assistants)
- [OpenAPI Specification](https://spec.openapis.org/oas/v3.0.1.html)

---

## 🚀 Getting Started

### For New Developers
1. Read the [Flow Diagrams](code-interpreter-flow-diagram.md) to understand the system
2. Review the [Quick Reference](code-interpreter-api-contract.md) for API basics
3. Study the [Integration Guide](code-interpreter-api-integration.md) for implementation details

### For API Users
1. Import the [OpenAPI Spec](../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml) into your tool
2. Use the [Quick Reference](code-interpreter-api-contract.md) for examples
3. Check the [Integration Guide](code-interpreter-api-integration.md) for best practices

### For Maintainers
1. Keep the [OpenAPI Spec](../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml) in sync with API changes
2. Update the [Integration Guide](code-interpreter-api-integration.md) when implementation changes
3. Refresh the [Flow Diagrams](code-interpreter-flow-diagram.md) if architecture changes

---

## 📝 Documentation Standards

All documentation in this collection follows these principles:
- **Accuracy**: Reflects actual implementation
- **Completeness**: Covers all aspects
- **Clarity**: Written for multiple skill levels
- **Maintainability**: Easy to update
- **Accessibility**: Multiple formats (spec, guide, reference, diagrams)

---

## 🛠️ Tools & Validation

### Validating the OpenAPI Spec
```bash
# Using Python (already validated)
python3 -c "import yaml; yaml.safe_load(open('code-interpreter.yaml'))"

# Using Swagger CLI (if installed)
swagger-cli validate code-interpreter.yaml

# Using OpenAPI Generator (if installed)
openapi-generator-cli validate -i code-interpreter.yaml
```

### Using the Specification
```bash
# Generate client SDK
openapi-generator-cli generate -i code-interpreter.yaml -g python -o ./client

# Import into Postman
# File → Import → Select code-interpreter.yaml

# View in Swagger Editor
# https://editor.swagger.io/ → Import code-interpreter.yaml
```

---

## 📞 Support & Questions

For questions about:
- **API Usage**: Refer to [Quick Reference](code-interpreter-api-contract.md)
- **Implementation**: Check [Integration Guide](code-interpreter-api-integration.md)
- **Architecture**: See [Flow Diagrams](code-interpreter-flow-diagram.md)
- **API Contract**: Review [OpenAPI Spec](../api/app/clients/tools/.well-known/openapi/code-interpreter.yaml)

---

**Last Updated**: October 2025  
**API Version**: OpenAI Assistants API v2 (beta)  
**LibreChat Version**: v0.8.0+

---

## 📄 License & Attribution

This documentation describes the integration with OpenAI's Assistants API. The API itself is owned and operated by OpenAI. LibreChat is an open-source project. Please refer to the respective licenses for usage terms.
