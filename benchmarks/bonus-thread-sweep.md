# Bonus — Thread sweep

Model: `Llama-3.2-3B-Instruct-Q4_K_M.gguf`  ·  GPU layers: `0`

| threads | tg64 (tok/s) |
|---:|---:|
| 1 | 3.7 |
| 2 | 6.4 |
| 4 | 8.2 |
| 8 | 6.7 |
| 16 | 5.8 |

**Best**: `-t 4` at 8.2 tok/s.

Look at the curve. If it peaks around your **physical** core count and drops as you go higher, that's the memory-bandwidth ceiling: extra threads fight over the same memory channels and slow each other down.
