# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Phan Dương Định  
**Cohort:** E403
**MSSV:** 2A202600277
**Ngày submit:** 2026-05-06

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

- **OS:** Linux (Ubuntu/Debian)
- **CPU:** Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
- **Cores:** 8 physical / 8 logical (không có hyper-threading)
- **CPU extensions:** AVX2
- **RAM:** 23.4 GB
- **Accelerator:** CPU only (không có NVIDIA CUDA, AMD ROCm, Apple Metal, hay Vulkan)
- **llama.cpp backend đã chọn:** CPU (AVX/NEON tuning)
- **Recommended model tier:** Llama-3.2-3B-Instruct (Q4_K_M)

**Setup story** (≤ 80 chữ):

Phải cài thêm `uvicorn`, `fastapi`, `pydantic-settings`, `sse-starlette`, `starlette-context` do `llama_cpp.server` cần. Sửa `start-server.sh` để dùng `.venv/bin/python` thay vì system python. Tạo thủ công `models/active.json`. Do bartowski repo không có Q2_K cho Llama-3.2-3B nên tải Qwen2.5-0.5B Q2_K làm compare model.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| Llama-3.2-3B-Instruct-Q4_K_M.gguf | 2047 | 690 / 1865 | 149.3 / 288.1 | 10193 / 19577 / 21957 | 6.7 |
| Qwen2.5-0.5B-Instruct-Q2_K.gguf | 1199 | 167 / 797 | 54.5 / 92.0 | 3636 / 6378 / 7207 | 18.3 |

**Một quan sát** (≤ 50 chữ): Qwen2.5-0.5B nhanh gấp ~2.7 lần (18.3 vs 6.7 tok/s) do model nhỏ hơn 6 lần. Tuy nhiên so sánh không hoàn toàn fair vì khác kiến trúc model. Q4_K_M chậm hơn kỳ vọng do chạy CPU-only.

---

## 3. Track 02 — llama-server load test

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|---:|---:|---:|---:|---:|
| 10 | 0.11 | 33000 | 42000 | 42000 | 0 (0.00%) |
| 50 | 0.09 | 20000 | 20000 | 20000 | 0 (0.00%) |

**KV-cache observation** (từ `record-metrics.py`): Không lấy được `llamacpp:kv_cache_usage_ratio` do version llama-cpp-python (0.3.22) không hỗ trợ endpoint `/metrics`. Cần build llama.cpp từ source (Bonus track) để có Prometheus metrics.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only
- **N17 (Data pipeline):** stub: in-memory dict
- **N18 (Lakehouse):** stub: SQLite
- **N19 (Vector + Feature Store):** stub: TOY_DOCS

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: ~0 ms (không dùng embedding model)
- retrieve: 0.0 ms (in-memory dict lookup)
- llama-server: 5730.7 – 29845.3 ms (tùy độ dài câu trả lời)

**Reflection** (≤ 60 chữ): Bottleneck nằm ở llama-server (5-30s mỗi request) do chạy model 3B trên CPU-only. Điều này khớp với kỳ vọng — CPU decode rate chậm (~6.7 tok/s). Không có bottleneck ở retrieve vì dùng in-memory stub.

---

## 5. Bonus — The single change that mattered most

> **Most important section.** Tối ưu hóa số lượng threads cho llama-bench trên CPU-only.

**Change:** `make sweep-thread` — chạy llama-bench với các giá trị `-t` (threads) = [1, 2, 4, 8, 16]

**Before vs after:**

```
before: t=1  → 3.7 tok/s
after:  t=4  → 8.2 tok/s (best)
speedup: ~2.2×
```

**Tại sao nó work:**

CPU-only inference bị giới hạn bởi memory bandwidth, không phải compute. Với 8 cores vật lý, số threads tối ưu là 4 (không phải 8). Khi tăng quá 4 threads, các threads đấu tranh nhau trên cùng memory channels → chậm hơn. Đây là memory bandwidth ceiling.

---

## 6. (Optional) Điều ngạc nhiên nhất

Model Qwen2.5-0.5B (0.5B parameters) decode nhanh gấp ~2.7 lần Llama-3.2-3B dù nhỏ hơn ~6 lần. Điều này cho thấy memory bandwidth là bottleneck chính trên CPU, không phải compute.

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit (hoặc paste path snapshot vào section 1)
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [x] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [x] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [x] `make verify` exit 0 (chạy ngay trước khi push)
- [x] Repo trên GitHub ở chế độ **public**
- [x] Đã paste public repo URL vào VinUni LMS

---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.
