# MCP Access Point  

`MCP` Access Point is a lightweight protocol conversion gateway tool designed to establish a communication bridge between traditional `HTTP` services and `MCP` (Model Context Protocol) clients. It enables MCP clients to interact directly with existing HTTP services without requiring any server-side interface modifications.  
<p align="center">
  <a href="./README.md"><img alt="README in English" src="https://img.shields.io/badge/English-d9d9d9"></a>
  <a href="./README_CN.md"><img alt="简体中文版" src="https://img.shields.io/badge/简体中文-d9d9d9"></a>
</p>

## Introduction  
This project is built on `Pingora` - an ultra-high performance gateway proxy library capable of supporting massive-scale request proxy services. Pingora has been used to build services that handle core traffic for the Cloudflare platform, consistently serving over 40 million requests per second across the internet for years. It has become the technical cornerstone supporting a significant proportion of traffic on the Cloudflare platform.

## HTTP to MCP  
This mode allows clients like `Cursor Desktop` to communicate with remote HTTP servers through `SSE`, even when the servers themselves don't support the SSE protocol.

- Example setup includes two services:  
  - Service 1 runs locally at `127.0.0.1:8090`  
  - Service 2 runs remotely at `api.example.com`  
- Through the `MCP Access Point`, both services can be converted to MCP services without any code modifications.  
- Clients communicate with `Service 1` and `Service 2` via the MCP protocol. The MCP Access Point automatically distinguishes MCP requests and forwards them to the appropriate backend services.

```mermaid
graph LR
   A["Cursor Desktop"] <--> |sse| B["MCP Access Point"]
   B <--> |http 127.0.0.1:8090| C1["Existing API Server"]
   B <--> |http api.example.com| C2["Existing API Server"]

   style A fill:#ffe6f9,stroke:#333,color:black,stroke-width:2px
   style B fill:#e6e6af,stroke:#333,color:black,stroke-width:2px
   style C1 fill:#e6ffe6,stroke:#333,color:black,stroke-width:2px
   style C2 fill:#e6ffd6,stroke:#333,color:black,stroke-width:2px
```

### Transport Type (Specification)
Currently supports `SSE` and `Streamable HTTP` protocols.
- ✅ Streamable HTTP(stateless)
- ✅ SSE 2024-11-05

use `IP:PORT/sse` for `SSE` 
use `IP:PORT/mcp/` for `Streamable HTTP` 

## Quick Start  

### Installation  
```bash
# Install from source
git clone https://github.com/sxhxliang/mcp-access-point.git
cd mcp-access-point
cargo run -- -c config.yaml

# Use inspector for debugging (start service first)
npx @modelcontextprotocol/inspector node build/index.js
# Access http://127.0.0.1:6274/
# Select "see" and enter 0.0.0.0:8080/sse, then click connect
# or select "Streamable HTTP" and enter 0.0.0.0:8080/mcp/
```

### Parameter Details:  
1.  **`-c config.yaml`**  
   - `-c` (or `--config`) specifies the configuration file path (`config.yaml`).  
   - This file defines multiple MCP services and their configurations.  

#### config.yaml Example  
```yaml
# config.yaml example (supports multiple services)
mcps:
  - id: service-1  # Service identifier
    upstream_id: 1
    upstream_config: # Upstream service configuration (optional)
      headers:
        X-API-Key: "12345-abcdef"
        Authorization: "Bearer token123"
        User-Agent: "MyApp/1.0"
        Accept: "application/json"
      nodes:
        "127.0.0.1:8090": 1 # must be the same as upstream id in upstreams
    path: openapi_for_demo_patch1.json # Local OpenAPI file path

  - id: web-api-2 # Service identifier
    upstream_id: 2
    path: https://petstore.swagger.io/v2/swagger.json  # Supports network paths
    routes: # custom routes (additional routes)
      - id: 1
        operation_id: test_custom_route # Operation identifier
        uri: /api/v1/{id} # Path to match (e.g., /api/v1/*)
        method: GET
        meta:
          name: test_custom_route
          description: test by ID
          inputSchema: # Input schema validation (optional)
            type: object
            required:
              - id
            properties:
              id:
                type: integer
                minimum: 1

upstreams: # Upstream service configuration must be defined
  - id: 1
    nodes: #（e.g., a web server or API server)）
      "127.0.0.1:8090": 1 # address with weight

  - id: 2 # another upstream service 
    nodes:
      "127.0.0.1:8091": 1
```

To run the MCP Access Point using the configuration file, use the following command:
```bash
cargo run -- -c config.yaml
```

## Core Features  

- **Protocol Conversion**: Seamless conversion between HTTP and MCP protocols  
- **Zero-Intrusive Integration**: Full compatibility with existing HTTP services  
- **Client Empowerment**: Enables MCP clients to directly call standard HTTP services  
- **Lightweight Proxy**: Minimalist architecture with efficient protocol conversion  

## Running via Docker  

### Build Docker Image (Optional)  
```bash
# Clone repository
git clone https://github.com/sxhxliang/mcp-access-point.git
cd mcp-access-point

# Build image
docker build -t liangshihua/mcp-access-point:latest .
```

### Run Docker Container  
```bash
# Using environment variables (service running on host)
# Note: Replace /path/to/your/config.yaml with actual path

docker run -d --name mcp-access-point --rm \
  -p 8080:8080 \
  -e port=8080 \
  -v /path/to/your/config.yaml:/app/config/config.yaml \
  sxhxliang/mcp-access-point:latest
```

### Environment Variables  
- `port`: MCP Access Point listening port (default: 8080)

## Typical Use Cases  

- **Progressive Architecture Migration**: Facilitate gradual transition from HTTP to MCP  
- **Hybrid Architecture Support**: Reuse existing HTTP infrastructure within MCP ecosystem  
- **Protocol Compatibility**: Build hybrid systems supporting both protocols  

**Example Scenario**:  
When MCP-based AI clients need to interface with legacy HTTP microservices, the MCP Access Gateway acts as a middleware layer enabling seamless protocol conversion.