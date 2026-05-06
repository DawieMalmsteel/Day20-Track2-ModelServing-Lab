# Bonus — Thread sweep

Model: `Llama-3.2-3B-Instruct-Q4_K_M.gguf`  ·  GPU layers: `0`

| threads | tg64 (tok/s) |
|---:|---:|
| 1 | 3.5 |
| 2 | 7.4 |
| 4 | 9.0 |
| 8 | 7.2 |
| 16 | 6.6 |

**Best**: `-t 4` at 9.0 tok/s.

Look at the curve. If it peaks around your **physical** core count and drops as you go higher, that's the memory-bandwidth ceiling: extra threads fight over the same memory channels and slow each other down.
