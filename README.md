# gRPC vs. REST Performance Comparison

## Overview
This project compares gRPC and REST performance in terms of latency and payload handling. It implements both protocols in Python to evaluate their throughput and overhead under different load scenarios.

## Implementation Details

### 1. gRPC Service
Implemented a gRPC server using Python and Protocol Buffers that supports the four standard communication modes:
- **Unary**: Basic request-response.
- **Server Streaming**: Sending multiple items from the server.
- **Client Streaming**: Receiving multiple items from the client.
- **Bidirectional Streaming**: Simultaneous two-way communication.

**Features included:**
- **Interceptors**: Basic logging interceptor for tracking calls.
- **Reflection**: Enabled for service discovery.
- **Error Handling**: Uses gRPC status codes (e.g., `NOT_FOUND`).

### 2. REST Service (FastAPI)
The REST API is built with **FastAPI** to provide a modern, asynchronous comparison to gRPC. It uses Pydantic for data validation and serialization.

### 3. Docker & Infrastructure
- **Containerization**: Both services are containerized with Docker for easy deployment and testing.
- **Protobuf Generation**: Code generation using `grpc_tools.protoc` to maintain shared contracts between server and client.

## Benchmark Results
Testing was conducted across various payload sizes and call volumes to compare latency and throughput.

### Key Insights
- **Performance**: On average, gRPC performed approximately **2x faster** than REST for small to medium payloads.
- **Resource Utilization**: gRPC utilized more CPU resources (~75%) compared to REST (~60%), reflecting its efficiency in processing high-frequency calls.
- **Payload Sensitivity**: While gRPC excels at high call volumes, its performance advantage diminishes with very large messages (over 256 KB). For messages exceeding 4 MB, custom gRPC configurations are required to bypass default limits.
- **Optimization Strategy**: For large data transfers, the best performance was achieved by balancing message size and call frequency. For example, transferring 4 MB via 512 calls of 8 KB messages was significantly faster than making hundreds of thousands of calls with 16-byte payloads.

| Scenario (Total Data) | gRPC (Total Time) | REST (Total Time) |
| :--- | :--- | :--- |
| 16,384 Calls (256 KB) | ~8.87 seconds | ~20.07 seconds |
| 65,536 Calls (1 MB) | ~35.39 seconds | ~80.30 seconds |
| 262,144 Calls (4 MB) | ~141.86 seconds | ~319.30 seconds |
| 512 Calls (4 MB Total) | **~0.31 seconds** | ~0.64 seconds |


## Technology Stack
- **Language**: Python
- **Frameworks**: FastAPI, gRPC
- **Data**: Protocol Buffers, JSON
- **Tools**: Docker, Uvicorn, Requests

## Project Structure
- `/server_grpc`: gRPC server implementation.
- `/server_rest`: FastAPI implementation.
- `client_grpc_performance_test.py`: Benchmark runner.
- `client_grpc_features.py`: Functional test for gRPC modes.
- `myitems.proto`: Service definition and message types.

## How to Run

### 1. Requirements
- Docker
- Python 3.x (to run client scripts)

### 2. Start Servers with Docker
**gRPC Server:**
```
docker build -t server_grpc ./server_grpc
docker run -p 50051:50051 server_grpc
```

**REST Server:**
```
docker build -t server_rest ./server_rest
docker run -p 8000:8000 server_rest
```

### 3. Run Benchmark
```bash
pip install -r venv_packages.txt
python client_grpc_performance_test.py
```
