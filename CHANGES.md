# CHANGES

## Base URL: everything moved under `/qwen-3tts/`

### Why

The API previously served every route at the site root:

- Web UI at `/`
- OpenAI API at `/v1/...`
- Swagger/ReDoc at `/docs` and `/redoc`
- Health at `/health`
- Static assets at `/static`
- Voice Studio at `/voice-studio`

This collided with the wider deployment where the host is shared with other
services, so the whole app needed to live on a dedicated path.

### How

All routes are now physically prefixed with `/qwen-3tts`:

| Old path             | New path                                |
| -------------------- | --------------------------------------- |
| `/`                  | `/qwen-3tts/`                           |
| `/v1/...`            | `/qwen-3tts/v1/...`                     |
| `/docs`              | `/qwen-3tts/docs`                       |
| `/redoc`             | `/qwen-3tts/redoc`                      |
| `/health`            | `/qwen-3tts/health`                     |
| `/static`            | `/qwen-3tts/static`                     |
| `/voice-studio`      | `/qwen-3tts/voice-studio`               |
| `/openapi.json`      | `/qwen-3tts/openapi.json`               |

#### Code changes (`api/main.py`)

- Router include prefix changed from `/v1` to `/qwen-3tts/v1`
- Root, health, static-mount, and Voice Studio mount paths updated
- `openapi_url` set to `/qwen-3tts/openapi.json`
- Voice Studio app is passed a base URL including `/qwen-3tts` so its
  requests hit `/qwen-3tts/v1/audio/speech` and `/qwen-3tts/v1/voices`

#### Frontend (`api/static/index.html` and the generated root page)

- Link cards, `base_url` example, and all `fetch()` calls updated to the
  prefixed paths (`/qwen-3tts/v1/...`, `/qwen-3tts/health`, etc.)

#### Gradio Voice Studio (`gradio_voice_studio.py`)

- Default `TTS_BASE_URL` now includes `/qwen-3tts` so the standalone launch
  stays consistent with the mounted version

#### Tests

- `tests/test_api.py`, `tests/test_custom_voices.py`,
  `tests/test_frontend_parity.py` updated to the `/qwen-3tts/v1/...` and
  `/qwen-3tts/health` paths

#### Helper scripts

- `bench_tts.py`, `benchmark_official.py`, `extended_warmup.py`,
  `test_opts_simple.py`, `verify_optimizations.py` updated to use
  `http://localhost:8880/qwen-3tts/...`

#### Docs

- `README.md`, `CPU_BACKEND_GUIDE.md`, `VLLM_BACKEND_STATUS.md`,
  `docs/custom-voice-persistent.md`, `docs/vllm-backend.md`,
  `docs/voice-library.md` updated so examples use the prefixed URLs

#### Resulting URL map (default `localhost:8880`)

- Web UI: `http://localhost:8880/qwen-3tts/`
- OpenAI API base: `http://localhost:8880/qwen-3tts/v1`
- Docs: `http://localhost:8880/qwen-3tts/docs`
- Health: `http://localhost:8880/qwen-3tts/health`
- Voice Studio: `http://localhost:8880/qwen-3tts/voice-studio`

### Configuration notes

- `ROOT_PATH` is no longer required for direct access because the routes are
  hard-coded in the app. Removing it avoids the scheme being applied twice.
  Example `docker-compose` environment (note the corrected spelling):

  ```yaml
  environment:
        HOST: "0.0.0.0"
        PORT: "8880"
        WORKERS: "1"
        TTS_BACKEND: "official"
        TTS_MODEL_NAME: "Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice"
        ENABLE_VOICE_STUDIO: "true"
        NVIDIA_DRIVER_CAPABILITIES: "compute,utility"
  ```

- If the app is ever placed behind a reverse proxy that *strips* the
  `/qwen-3tts` prefix, set `ROOT_PATH` to exactly `/qwen-3tts` at that point.