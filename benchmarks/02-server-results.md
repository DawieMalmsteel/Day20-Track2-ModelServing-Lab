# 02 — Server Load Test Results

## Configuration
- Model: Llama-3.2-3B-Instruct-Q4_K_M.gguf
- Threads: 8, GPU layers: 0 (CPU only)
- Context: 2048, Parallel: 4
- Server: llama.cpp python server (v0.3.22)

## Load Test Results

### 10 Users (1 minute)
- Requests: 5
- Avg response time: 31447ms (31.4s)
- Failures: 0 (0.00%)
- P50: 33000ms, P95: 42000ms, P99: 42000ms
- Requests/second: 0.11

### 50 Users (1 minute)
- Requests: 2
- Avg response time: 18098ms (18.1s)
- Failures: 0 (0.00%)
- P50: 20000ms, P95: 20000ms
- Requests/second: 0.09

## Smoke Test
- Status: ✅ Passed
- Endpoint: POST /v1/chat/completions
- Sample response: "Goodput at a Service Level Objective (SLO) refers to..."

## Observations
- Server xử lý chậm trên CPU-only (16-42s per request) do model 3B lớn
- 0 failures cho cả 2 load tests — server stable
- Với CPU-only, rất ít requests hoàn thành trong 1 phút
- Để cải thiện: dùng GPU offload (`--n_gpu_layers`) hoặc model nhỏ hơn (Qwen-0.5B)
