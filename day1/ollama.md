# Ollama Serve / Server Configuration Cheat Sheet

## 1. Start the server

```bash
ollama serve
```

Check help:

```bash
ollama serve --help
```

Current `serve` options are essentially:

```bash
-h, --help
```

The important configuration is done through environment variables.

---

## 2. Network / host / port

Default bind address:

```text
127.0.0.1:11434
```

Meaning: your machine only.

```text
Your machine only
      │
      └── localhost:11434
```

Check the current value:

```powershell
$env:OLLAMA_HOST
```

Bind only to localhost:

```powershell
$env:OLLAMA_HOST="127.0.0.1:11434"
ollama serve
```

This is the safe default for a laptop.

Listen on all interfaces:

```powershell
$env:OLLAMA_HOST="0.0.0.0:11434"
ollama serve
```

This exposes the service to the local network:

```text
Windows machine
       │
       ├── localhost:11434
       │
       ├── LAN-IP:11434
       │
       └── other machines can potentially access it
```

This is useful when demonstrating from another PC:

```text
Other PC
   ↓
http://<OLLAMA-SERVER-IP>:11434
   ↓
Ollama
```

> Warning: do not expose `0.0.0.0:11434` directly to the internet. Put authentication, network controls, a reverse proxy, or a firewall in front of it.

`OLLAMA_HOST` controls the bind address and defaults to `127.0.0.1:11434`.

---

## 3. Change port

```powershell
$env:OLLAMA_HOST="127.0.0.1:8080"
ollama serve
```

Now you can access:

```text
http://localhost:8080
```

instead of:

```text
http://localhost:11434
```

---

## 4. Model storage

Default on Windows:

```text
%USERPROFILE%\.ollama\models
```

Set a custom model directory:

```powershell
$env:OLLAMA_MODELS="D:\Ollama\Models"
ollama serve
```

Useful if your `C:` drive is getting full and you have a larger SSD available.

---

## 5. Model keep-alive / timeout

Default:

```text
5 minutes
```

```powershell
$env:OLLAMA_KEEP_ALIVE="5m"
```

After the model has been idle for this period, it can be unloaded from memory.

Keep a model loaded for 30 minutes:

```powershell
$env:OLLAMA_KEEP_ALIVE="30m"
ollama serve
```

Keep a model loaded for 24 hours:

```powershell
$env:OLLAMA_KEEP_ALIVE="24h"
ollama serve
```

Never automatically unload:

```powershell
$env:OLLAMA_KEEP_ALIVE="-1"
ollama serve
```

Negative values mean effectively infinite keep-alive.

Unload immediately after inference:

```powershell
$env:OLLAMA_KEEP_ALIVE="0"
ollama serve
```

You can also set it at the request level:

```json
{
  "model": "qwen3:0.6b",
  "keep_alive": 0
}
```

> Request-level `keep_alive` overrides the server-level `OLLAMA_KEEP_ALIVE`.

---

## 6. Model loading timeout

This is different from keep-alive.

Default:

```text
5 minutes
```

It controls how long Ollama waits while loading a model before giving up.

```powershell
$env:OLLAMA_LOAD_TIMEOUT="10m"
ollama serve
```

Or:

```powershell
$env:OLLAMA_LOAD_TIMEOUT="30m"
ollama serve
```

Zero or negative values mean effectively unlimited.

```text
OLLAMA_KEEP_ALIVE
        ↓
How long the model stays loaded after becoming idle

OLLAMA_LOAD_TIMEOUT
        ↓
How long Ollama waits while loading a model
```

---

## 7. Parallel requests

Default:

```text
OLLAMA_NUM_PARALLEL=1
```

Meaning: one request at a time per model.

Set a higher value:

```powershell
$env:OLLAMA_NUM_PARALLEL="4"
ollama serve
```

```text
             Ollama
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
      Req 1   Req 2   Req 3
```

This increases memory usage because each parallel request adds context and runtime memory cost.

For a 16 GB RAM machine, `OLLAMA_NUM_PARALLEL=1` is a sensible starting point.

---

## 8. Maximum loaded models

```powershell
$env:OLLAMA_MAX_LOADED_MODELS="2"
ollama serve
```

This controls how many different models can remain loaded simultaneously, subject to available memory.

```text
RAM
 │
 ├── qwen3:0.6b
 │
 └── llama3.x
```

If memory is insufficient, Ollama may unload models to make room.

The current default is generally `3 × number of GPUs`, or `3` for CPU inference, provided memory permits.

---

## 9. Request queue

```text
OLLAMA_MAX_QUEUE
```

Default:

```text
512
```

It controls how many requests can wait in the queue while Ollama is busy before additional requests are rejected.

```powershell
$env:OLLAMA_MAX_QUEUE="100"
ollama serve
```

```text
Clients
  │
  ├── Request
  ├── Request
  ├── Request
  │
  ▼
Ollama Queue
  │
  ▼
Model
```

---

## 10. CORS / origins

```powershell
$env:OLLAMA_ORIGINS="http://localhost:3000"
ollama serve
```

Multiple origins:

```powershell
$env:OLLAMA_ORIGINS="http://localhost:3000,http://localhost:5173"
ollama serve
```

This controls which browser origins are allowed to access the API.

```text
React / Web UI
      ↓
Ollama API
```

Ollama already includes localhost-related allowed origins; custom origins can be supplied through `OLLAMA_ORIGINS`.

---

## 11. Debug logging

```powershell
$env:OLLAMA_DEBUG="1"
ollama serve
```

Useful for troubleshooting:

- model loading
- GPU detection
- memory
- API requests
- runtime problems

Debug levels:

```text
0 / false = INFO
1 / true  = DEBUG
2         = TRACE
```

---

## 12. Disable cloud features

```powershell
$env:OLLAMA_NO_CLOUD="1"
ollama serve
```

This disables cloud-related features such as remote inference and web search.

This is especially useful for on-premise or air-gapped setups.

---

## 13. GPU-related settings

Additional environment variables for GPU behavior include:

- `OLLAMA_GPU_OVERHEAD`
- `OLLAMA_SCHED_SPREAD`
- `OLLAMA_FLASH_ATTENTION`
- `OLLAMA_KV_CACHE_TYPE`
- `OLLAMA_LLM_LIBRARY`

Example:

- `OLLAMA_GPU_OVERHEAD` reserves part of GPU memory.
- `OLLAMA_SCHED_SPREAD` can force scheduling across available GPUs.

For a Windows CPU-only lab, avoid changing these until you are ready for Linux GPU + vLLM work.

---

## 14. Server-level vs request-level configuration

This is important for deployment behavior.

### Server level

```powershell
$env:OLLAMA_KEEP_ALIVE="30m"
ollama serve
```

This applies generally.

### Request level

```json
{
  "model": "qwen3:0.6b",
  "prompt": "Hello",
  "keep_alive": "2h"
}
```

A request-level setting can override the server default.

```text
                Ollama Server
                     │
            Server configuration
                     │
          OLLAMA_KEEP_ALIVE=30m
                     │
                     ▼
                Request
                     │
              keep_alive=2h
                     │
                     ▼
             Request wins
```

---

## 15. Recommended settings for a Windows 11 lab

For a machine with:

- Windows 11
- 4 vCPU
- 16 GB RAM
- 256 GB SSD
- CPU inference

A sensible starting configuration is:

```powershell
$env:OLLAMA_HOST="127.0.0.1:11434"
$env:OLLAMA_KEEP_ALIVE="5m"
$env:OLLAMA_NUM_PARALLEL="1"
$env:OLLAMA_MAX_QUEUE="512"

ollama serve
```

Do not change everything at once.

For training, the first five settings to demonstrate are:

1. localhost vs `0.0.0.0`
2. default port vs custom port
3. `keep_alive`
4. `num_parallel`
5. `max_queue`

These map directly to common LLM serving and production deployment concepts.

---

## Quick reference

| Variable | Default | Purpose |
| --- | --- | --- |
| `OLLAMA_HOST` | `127.0.0.1:11434` | Bind IP + port |
| `OLLAMA_MODELS` | `~/.ollama/models` | Model storage directory |
| `OLLAMA_KEEP_ALIVE` | `5m` | Model idle timeout; `0` unloads immediately, `-1` never unloads |
| `OLLAMA_LOAD_TIMEOUT` | `5m` | Timeout while loading a model; `<=0` means unlimited |
| `OLLAMA_NUM_PARALLEL` | `1` | Concurrent requests per model |
| `OLLAMA_MAX_LOADED_MODELS` | Auto | Maximum loaded models in memory |
| `OLLAMA_MAX_QUEUE` | `512` | Number of queued requests |
| `OLLAMA_ORIGINS` | none / localhost defaults | CORS / browser origins |
| `OLLAMA_DEBUG` | INFO | Debugging verbosity |
| `OLLAMA_NO_CLOUD` | unset | Disables cloud features |
| `OLLAMA_GPU_OVERHEAD` | none | Reserve GPU memory |
| `OLLAMA_SCHED_SPREAD` | none | Multi-GPU scheduling |
| `OLLAMA_FLASH_ATTENTION` | none | Flash attention support |
| `OLLAMA_KV_CACHE_TYPE` | `f16` | KV-cache type |

These are the current server-side configuration variables exposed by Ollama's configuration implementation.