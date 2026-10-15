# Bosch_cob_15thOct2026

### ollama  cli 

### 

# Bosch_cob_15thOct2026

### ollama  cli 

### starting ollama server on 0.0.0.0  (any mac or linux vm)

```
export  OLLAMA_HOST="0.0.0.0:11223"
 humanfirmware@darwin  ~  
 humanfirmware@darwin  ~  ollama serve 

time=2026-10-15T14:50:57.646+05:30 level=INFO source=routes.go:2117 msg="server config" env="map[HTTPS_PROXY: HTTP_PROXY: LLAMA_ARG_FIT: LLAMA_ARG_FIT_TARGET: NO_PROXY: OLLAMA_CONTEXT_LENGTH:0 OLLAMA_CREATE_REMOTE:false OLLAMA_DEBUG:INFO OLLAMA_DEBUG_LOG_REQUESTS:false OLL

```

### client side -- ollama cli / python (any lang code)

```
from openai import OpenAI

endpoint = "http://10.207.71.83:11223/v1"
deployment_name = "llama3.2:1b"
api_key = "ollama"

client = OpenAI(
    base_url=endpoint,
    api_key=api_key
)

response = client.responses.create(
    model=deployment_name,
    input="What is the capital of India?",
)

#print(f"answer: {response.output[0]}")
print(f"answer: {response.output_text}")

```