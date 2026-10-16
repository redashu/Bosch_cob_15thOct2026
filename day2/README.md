# Bosch_cob_15thOct2026

## Deploy LLM model on Ubuntu using vLLM

## Setting up Docker on Ubuntu

```bash
azureuser@ashuvm1:~$ sudo apt update
```

Install networking utilities:

```bash
sudo apt install net-tools -y
```

### Add Docker repository

```bash
# Add Docker's official GPG key
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to APT sources
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

### Install Docker

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Check Docker version

```bash
docker version
```

### setup vllm using docker 

```

docker  images
                                                                                                           i Info →   U  In Use
IMAGE   ID             DISK USAGE   CONTENT SIZE   EXTRA
root@ashuvm1:~# 


===> pulling vllm image 

docker pull vllm/vllm-openai-cpu



===verify 

root@ashuvm1:~# docker pull vllm/vllm-openai-cpu
Using default tag: latest
latest: Pulling from vllm/vllm-openai-cpu
Digest: sha256:495c84fe30f3572b83a78100a1e8ba423dbb77fc532ecefebe114e6b77327bd9
Status: Image is up to date for vllm/vllm-openai-cpu:latest
docker.io/vllm/vllm-openai-cpu:latest
root@ashuvm1:~# 
root@ashuvm1:~# 
root@ashuvm1:~# docker  images
                                                                                                           i Info →   U  In Use
IMAGE                         ID             DISK USAGE   CONTENT SIZE   EXTRA
vllm/vllm-openai-cpu:latest   495c84fe30f3       6.73GB         1.57GB        
root@ashuvm1:~# 



===> using vllm launching model


docker run -d  --name vllm-node1  --network host --shm-size=2g  -e VLLM_CPU_KVCACHE_SPACE=2 -v /root/models-cache:/root/.cache/huggingface vllm/vllm-openai-cpu:latest-x86_64  --model HuggingFaceTB/SmolLM2-360M-Instruct --dtype bfloat16 --max-model-len 512  --host 0.0.0.0 --port 8000 
ec8657100937e4691c5c72597f6ba8d94ffcf4df9141f1e010bcdc33f0f4b2ea
root@node11:~# 
root@node11:~# docker  ps
CONTAINER ID   IMAGE                                COMMAND                  CREATED         STATUS         PORTS     NAMES
ec8657100937   vllm/vllm-openai-cpu:latest-x86_64   "vllm serve --model …"   7 seconds ago   Up 6 seconds             vllm-node1
root@node11:~# 


```

### connecting with model 

```
curl -X POST http://20.161.26.245:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "HuggingFaceTB/SmolLM2-360M-Instruct",
    "messages": [
      {"role": "user", "content": "Explain Docker in two sentences."}
    ],
    "max_tokens": 60
  }'

  ```
