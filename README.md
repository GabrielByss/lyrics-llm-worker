# lyrics-llm-worker

Obraz RunPod Serverless do pisania tekstów piosenek dla aplikacji Music AI: `runpod-workers/worker-vllm` (vLLM OpenAI server)
z wypieczonym modelem **google/gemma-4-E4B-it** (Apache-2.0). Trzeci poziom łańcucha podpowiedzi tekstu
(1. model Apple na urządzeniu → 2. Gemma 4 E2B pobrana na telefon → 3. ten endpoint przez Cloud Function `musicSuggestLyrics`).

Build: Actions → „Build lyrics LLM worker image” (tag `v1`, model, wersje). Obraz ~28 GB.

Env template RunPod (patrz `ringtones_repo/runpod/README.md`): `MODEL_NAME=google/gemma-4-E4B-it`, `BASE_PATH=/models`,
`HF_HUB_OFFLINE=1`, `MAX_MODEL_LEN=4096`, `GPU_MEMORY_UTILIZATION=0.9`, `ENFORCE_EAGER=true`, `MAX_NUM_SEQS=8`.

Wywołanie (OpenAI-compatible): `POST https://api.runpod.ai/v2/<ENDPOINT_ID>/openai/v1/chat/completions`
z `Authorization: Bearer <RUNPOD_API_KEY>`, body `{"model":"google/gemma-4-E4B-it","messages":[...],"max_tokens":600}`.
