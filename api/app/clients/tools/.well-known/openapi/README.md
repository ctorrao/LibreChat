# OpenAPI Specifications

This directory contains OpenAPI 3.0 specifications for various APIs integrated with LibreChat.

## Available Specifications

### code_interpreter.yaml
Complete OpenAPI specification for OpenAI's Code Interpreter API integration.

**Features:**
- Assistant creation and management with code interpreter tool
- Thread and message operations
- Run execution and step retrieval
- Code interpreter tool call structures with outputs (logs and images)
- Complete request/response schemas

**Documentation:** See `/docs/CODE_INTERPRETER_API.md` for detailed usage guide.

**Endpoints Covered:**
- `POST /assistants` - Create assistant with code interpreter
- `POST /assistants/{assistant_id}` - Update assistant
- `POST /threads` - Create conversation thread
- `POST /threads/{thread_id}/messages` - Add messages to thread
- `POST /threads/{thread_id}/runs` - Execute assistant on thread
- `GET /threads/{thread_id}/runs/{run_id}/steps` - Retrieve run steps with code execution details

### scholarai.yaml
OpenAPI specification for ScholarAI integration, allowing search and retrieval of scientific articles.

### askyourpdf.yaml
OpenAPI specification for AskYourPDF integration, enabling PDF document analysis.

## Using These Specifications

### Validation
```bash
npx @redocly/cli lint <filename>.yaml
```

### Preview
```bash
npx @redocly/cli preview-docs <filename>.yaml
```

### Generate Documentation
```bash
npx @redocly/cli build-docs <filename>.yaml -o output.html
```

## Specification Format

All specifications follow the OpenAPI 3.0.1 standard with:
- Complete endpoint definitions
- Request/response schemas
- Authentication requirements
- Example requests and responses
- Detailed descriptions

## Contributing

When adding new API specifications:
1. Follow OpenAPI 3.0.1 standard
2. Include comprehensive schemas
3. Provide examples
4. Document authentication
5. Validate with OpenAPI tools
6. Add entry to this README

## Resources

- [OpenAPI Specification](https://spec.openapis.org/oas/v3.0.1)
- [OpenAPI Tools](https://openapi.tools/)
- [Redocly CLI](https://redocly.com/docs/cli/)
