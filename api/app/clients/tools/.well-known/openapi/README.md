# OpenAPI Specifications

This directory contains OpenAPI specifications for various external APIs and tools that LibreChat integrates with.

## Available Specifications

### code-interpreter.yaml
OpenAPI 3.0.1 specification for the OpenAI Assistants API with Code Interpreter functionality. This documents the API contract that LibreChat uses to enable code execution capabilities within assistant conversations.

**Key Features:**
- Thread and message management
- Assistant run execution with code interpreter
- Run step monitoring (including Python code execution details)
- File upload/download for code interpreter usage
- Tool call outputs (logs and generated files)

**Base URL:** `https://api.openai.com/v1`

**Authentication:** Bearer token (OpenAI API Key)

**API Version:** v2 (beta)

**Related Documentation:** See `/docs/code-interpreter-api-integration.md` for implementation details in LibreChat.

### scholarai.yaml
OpenAPI specification for ScholarAI API - allows searching and retrieving academic papers and research.

**Base URL:** `https://scholar-ai.net`

### askyourpdf.yaml
OpenAPI specification for AskYourPDF API - enables PDF document analysis and querying.

## Usage

These OpenAPI specifications serve multiple purposes:

1. **Documentation**: Provide clear API contract definitions for developers
2. **Validation**: Can be used to validate API requests and responses
3. **Code Generation**: Enable automatic client/SDK generation
4. **Testing**: Support automated API testing frameworks
5. **Integration**: Help third-party tools understand the API structure

## OpenAPI Format

All specifications follow the [OpenAPI 3.0.1 standard](https://spec.openapis.org/oas/v3.0.1.html) which provides:
- Standard schema definitions
- Request/response structures
- Authentication methods
- Path parameters and query strings
- Detailed type information
- Example values

## Adding New Specifications

When adding new API specifications to this directory:

1. Follow the OpenAPI 3.0+ standard
2. Use descriptive operation IDs
3. Include detailed descriptions for all endpoints
4. Document all required and optional parameters
5. Provide example values where helpful
6. Include authentication/security schemes
7. Tag endpoints for organization
8. Document error responses

## Tools for Working with OpenAPI

- **Swagger Editor**: [https://editor.swagger.io/](https://editor.swagger.io/)
- **OpenAPI Generator**: Generate client SDKs and server stubs
- **Postman**: Import and test APIs directly
- **ReDoc**: Generate beautiful API documentation

## Validation

You can validate these specifications using tools like:

```bash
# Using Swagger CLI
swagger-cli validate code-interpreter.yaml

# Using OpenAPI Generator
openapi-generator-cli validate -i code-interpreter.yaml
```

## Related Files

- Implementation code: `api/server/services/Endpoints/assistants/`
- Type definitions: `api/typedefs.js`
- Integration documentation: `docs/code-interpreter-api-integration.md`
