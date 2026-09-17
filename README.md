# lyrics-llm-worker

Obraz RunPod Serverless do pisania tekstów piosenek dla aplikacji Music AI: `runpod-workers/worker-vllm` v2.27.0 (vLLM 0.29)
z wypieczonym modelem. Trzeci poziom łańcucha podpowiedzi tekstu
(1. model Apple na urządzeniu → 2. Gemma 4 E4B pobrana na telefon → 3. ten endpoint przez Cloud Function `musicSuggestLyrics`).

| Tag | Model | Wagi | Pula GPU | Uwagi |
| --- | --- | --- | --- | --- |
| `v2` (aktualny) | `cyankiwi/gemma-4-26B-A4B-it-qat-AWQ-INT4` (Gemma 4 26B-A4B, int4 W4A16 g32 z oficjalnego checkpointu QAT Google, Apache-2.0) | 17,2 GB | 24 GB | MoE 3,8B aktywnych – jakość ~31B, szybkość ~4B; ta sama pula (i cena godziny) co v1 |
| `v1` | `google/gemma-4-E4B-it` bf16 | 16 GB | 24 GB | pierwsza wersja, template awaryjny |

Dlaczego nie 31B: oficjalny `google/gemma-4-31B-it-qat-w4a16-ct` ma 22 GB samych wag tekstowych – na 24 GB nie zostaje miejsce
na KV cache, a pula 48 GB kosztuje ~1,8× więcej za godzinę (optymalizujemy koszt, nie szybkość).

Build: Actions → „Build lyrics LLM worker image” (tag, `model_name`, wersje worker-vllm / vLLM). Obraz ~28 GB, ~20 min.

Env template RunPod (patrz `ringtones_repo/runpod/README.md`): `MODEL_NAME=cyankiwi/gemma-4-26B-A4B-it-qat-AWQ-INT4`, `BASE_PATH=/models`,
`HF_HUB_OFFLINE=1`, `MAX_MODEL_LEN=4096`, `GPU_MEMORY_UTILIZATION=0.9`, `ENFORCE_EAGER=true`, `MAX_NUM_SEQS=8`, `DTYPE=bfloat16`,
`LIMIT_MM_PER_PROMPT={"image":0}` (tekst-only: vLLM nie ładuje wieży wizyjnej), `ENABLE_LOG_REQUESTS=false`.
Szablon czatu Gemma 4 ma `enable_thinking` domyślnie wyłączone – brak tokenów myślenia.

Wywołanie produkcyjne (kolejka RunPod, bo trasa OpenAI-compatible jest synchroniczna i urywa się po ~60 s przy cold starcie):
`POST https://api.runpod.ai/v2/<ENDPOINT_ID>/run` z `Authorization: Bearer <RUNPOD_API_KEY>`, body
`{"input":{"messages":[...],"sampling_params":{"max_tokens":700,"temperature":0.9,"top_p":0.95}}}`, potem `GET /status/<id>`
(output: lista z obiektem OpenAI ChatCompletion, `choices[0].message.content`).
Szybki test (tylko ciepły worker): `POST …/openai/v1/chat/completions -d @test_chat.json`.
