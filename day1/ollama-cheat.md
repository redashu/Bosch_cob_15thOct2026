# Ollama CLI Cheat Sheet

## 1. Server

```bash
# Start Ollama server manually
ollama serve

# Check version
ollama --version

# Help
ollama help
ollama serve --help
```

> On Windows Desktop, Ollama normally runs in the background, so you usually do not need to manually run `ollama serve`.

---

## 2. Download / manage models

```bash
# Download model
ollama pull qwen3:0.6b

# List models stored locally
ollama list
# or
ollama ls

# Show model information
ollama show qwen3:0.6b

# Delete model from disk
ollama rm qwen3:0.6b

# Copy/rename a model
ollama cp qwen3:0.6b my-qwen
```

Important:

```text
ollama list
    ↓
Models available on DISK

ollama ps
    ↓
Models currently LOADED IN MEMORY
```

These are different things.

---

## 3. Run a model

```bash
ollama run qwen3:0.6b
```

This gives you an interactive session:

```text
>>> Hello
>>> Explain Docker
>>> What is Kubernetes?
```

One-shot prompt:

```bash
ollama run qwen3:0.6b "Explain Docker in one sentence"
```

```text
ollama run
     ↓
load model if necessary
     ↓
interactive inference
```

---

## 4. Check loaded models

```bash
ollama ps
```

Example:

```text
NAME          ID       SIZE      PROCESSOR    UNTIL
qwen3:0.6b    abc123   523 MB    100% CPU     4 minutes
```

This tells you what is currently loaded in memory.

---

## 5. Unload a model from memory

```bash
ollama stop qwen3:0.6b
```

This does not delete the model.

```text
ollama stop
     ↓
RAM/VRAM → freed
     ↓
Model remains on disk
```

You can verify:

```bash
ollama ps
```

Then later:

```bash
ollama run qwen3:0.6b
```

Ollama loads it again.

---

## 6. Delete a model completely

```bash
ollama rm qwen3:0.6b
```

### Difference

| Command | RAM/VRAM | Disk |
| --- | --- | --- |
| `ollama stop model` | ❌ unload | ✅ keep |
| `ollama rm model` | ❌ unload | ❌ delete |

---

## 7. Interactive commands

When you are inside:

```bash
ollama run qwen3:0.6b
```

you can use:

```text
/?
```

to see the available interactive commands.

Common ones include:

```text
/bye
/clear
/set
/show
/load
/save
```

For example:

```text
/clear
```

clears the current conversation context.

```text
/show info
```

shows model information.

```text
/show parameters
```

shows parameters.

```text
/bye
```

exits the interactive session.

---

## 8. Create a custom model

Create a `Modelfile`:

```text
FROM qwen3:0.6b

SYSTEM """
You are a Kubernetes expert.
Give concise technical answers.
"""

PARAMETER temperature 0.2
```

Then:

```bash
ollama create k8s-expert -f Modelfile
```

Run it:

```bash
ollama run k8s-expert
```

This is useful for demonstrating model customization without fine-tuning.

---

## 9. Server / API

Default local server:

```text
http://localhost:11434
```

Native API examples:

```text
POST /api/generate
POST /api/chat
```

```text
Python
   │
   ▼
HTTP
   │
   ▼
Ollama :11434
   │
   ▼
Model
```

Ollama also exposes an OpenAI-compatible API at:

```text
http://localhost:11434/v1
```

This is what you use with the OpenAI Python SDK.

---

## 10. Embeddings

Ollama can also run embedding models:

```bash
ollama run embeddinggemma
```

For a later RAG lab, the flow is:

```text
Documents
    ↓
Embedding model
    ↓
Vectors
    ↓
Vector DB
    ↓
Retriever
    ↓
Ollama LLM
```

---

## 11. Commands to remember

For Day 1 deployment training, memorize these:

```bash
# Server
ollama serve

# Download
ollama pull <model>

# Run
ollama run <model>

# Local models
ollama ls

# Loaded models
ollama ps

# Model details
ollama show <model>

# Unload from memory
ollama stop <model>

# Delete from disk
ollama rm <model>

# Copy model
ollama cp <source> <destination>

# Build custom model
ollama create <name> -f Modelfile

# CLI help
ollama help
```

### Most important mental model

```text
                 OLLAMA SERVER
                  :11434
                     │
        ┌────────────┴────────────┐
        │                         │
   Models on disk          Models in memory
   `ollama ls`             `ollama ps`
        │                         │
        │                    `ollama stop`
        │                         │
        └──────── `ollama run` ───┘
```

`ollama rm` is fundamentally different from `ollama stop`: `rm` removes the model's local data, while `stop` only stops the currently running model.