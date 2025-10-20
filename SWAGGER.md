# Swagger UI Documentation

## Overview
The Gmail MCP Server now includes an interactive Swagger UI for easy API testing and documentation.

## Accessing Swagger UI

Once you have the server running in HTTP mode with proper Gmail OAuth credentials:

1. Start the server:
   ```bash
   ./gmail-mcp-server --http --port 8080
   ```

2. Open your browser and navigate to:
   ```
   http://localhost:8080/swagger/index.html
   ```

## Available Endpoints

The Swagger UI documents the following endpoints:

### 1. **GET /** - Server Information
- Returns HTML page with server info and configuration
- Shows available tools and Cursor integration instructions

### 2. **GET /health** - Health Check
- Returns JSON with server health status
- Includes version, timestamp, and Gmail authentication status

### 3. **GET|POST|OPTIONS /mcp** - MCP JSON-RPC Endpoint
- Experimental endpoint for MCP tool execution
- Supports CORS for browser-based clients
- Note: Full MCP support requires stdio mode

## Regenerating Swagger Docs

If you modify the API handlers or add new endpoints, regenerate the Swagger documentation:

```bash
~/go/bin/swag init
```

This will update the following files:
- `docs/docs.go`
- `docs/swagger.json`
- `docs/swagger.yaml`

## Adding New Endpoints

To add Swagger documentation for new endpoints:

1. Add godoc comments above your handler function with Swagger annotations:
   ```go
   // handlerName godoc
   // @Summary Brief description
   // @Description Detailed description
   // @Tags category
   // @Accept json
   // @Produce json
   // @Param name path string true "Description"
   // @Success 200 {object} ResponseType
   // @Router /path [get]
   func handlerName(w http.ResponseWriter, r *http.Request) {
       // handler code
   }
   ```

2. Run `swag init` to regenerate docs

3. Rebuild the application

## Example API Calls

### Health Check
```bash
curl http://localhost:8080/health
```

### MCP Endpoint
```bash
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/list",
    "id": 1
  }'
```

## Notes

- The Swagger UI is only available when running in HTTP mode (`--http` flag)
- OAuth credentials must be configured before the server can start
- All endpoints support CORS with wildcard origin (`*`)
