# Data Format vs Network Format
- Data formats (JSON, Protobuf) define how data is packed, while network frameworks (REST, gRPC) define how data is routed and delivered.
- You put your plain-text JSON letter into an envelope and send it via the REST postal system to an endpoint like [www.example.com/orders](https://www.example.com/orders)
- While REST + JSON and gRPC + Protobuf are the standard pairings, the transport layer (REST/gRPC) and the serialization format (JSON/Protobuf) are decoupled and can be mixed.

|Pairing| Payload |Transport|Primary Advantage
|-|-|-|-|
REST + JSON (Standard)|JSON|HTTP/1.1 or HTTP/2|"High human readability, universal web support"
gRPC + Protobuf (Standard)|Protobuf Binary|HTTP/2|"Maximum performance, low latency, multiplexed streaming"
REST + Protobuf (Mixed)|Protobuf Binary|HTTP/1.1 or HTTP/2|Smaller payloads on legacy REST infrastructure
gRPC + JSON (Mixed)|JSON|HTTP/2|gRPC streaming/connection features with readable data


# Why do we need REST, gRPC??
REST and gRPC are the management frameworks built on top of HTTP. They provide the routing, client code, error rules, and function-mapping so your application knows what to do with those bytes.

```
CLIENT SIDE                                                       SERVER SIDE
┌─────────────────────────┐                                 ┌─────────────────────────┐
│  Client Application     │                                 │   Server Application    │
│  e.g. getUser(id: 123)  │                                 │   def getUser(id): ...  │
└────────────┬────────────┘                                 └────────────▲────────────┘
             │                                                           │
             ▼                                                           │
┌─────────────────────────┐                                 ┌────────────┴────────────┘
│  gRPC / REST Layer      │                                 │  gRPC / REST Layer      │
│  • Maps call to endpoint│                                 │  • Routes incoming call │
│  • Handles deadlines    │                                 │    to getUser() function│
│  • Generates client SDK │                                 │  • Converts error codes │
└────────────┬────────────┘                                 └────────────▲────────────┘
             │                                                           │
             ▼                                                           │
┌─────────────────────────┐                                 ┌────────────┴────────────┘
│  Protobuf / JSON Layer  │                                 │  Protobuf / JSON Layer  │
│  • Encodes data to      │                                 │  • Decodes bytes/text   │
│    Binary or JSON text  │                                 │    back into objects    │
└────────────┬────────────┘                                 └────────────▲────────────┘
             │                                                           │
             ▼                                                           │
┌─────────────────────────┐     HTTP/1.1 or HTTP/2 Transport │─────────────────────────┐
│  HTTP Transport         ├────────────────────────────────►│  HTTP Transport         │
│  • Raw socket bytes     │        (Across Network)         │  • Receives TCP packets │
└─────────────────────────┘                                 └─────────────────────────┘

```
## What Extra Value Do gRPC and REST Provide over HTTP?

Without gRPC or REST, you would have to manually open raw HTTP connections, parse byte headers, write custom routing trees, and write your own networking code in every programming language you use.
- Routing & Function Execution
  - The Problem: The server receives 50 bytes of binary data over HTTP. Which piece of backend code should process it?
  - REST Solution: Uses URIs and HTTP verbs (GET /users/123, POST /orders).
- Standardized Error Handling
  - Without a framework: How do you indicate a database timeout? Do you send HTTP 500? A JSON body {"error": "timeout"}?
  - REST: Uses standard HTTP status codes (404 Not Found, 401 Unauthorized).
 
# gRPC or REST?
- REST (with JSON)
  - Pros: Simple, human-readable, works everywhere (browsers, mobile, etc.).
  - Cons: Slower, more bandwidth, no built-in schema.
- gRPC (with Protobuf)
  - Pros: Faster, smaller payloads, streaming, code generation, strong typing.
  - Cons: Not browser-friendly, requires schema, more complex setup.
