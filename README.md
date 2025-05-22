# SmolVLM-500M-Instruct + llama.cpp (Multimodal Inference)

This repository demonstrates how to run the [SmolVLM-500M-Instruct](https://huggingface.co/HuggingFaceTB/SmolVLM-500M-Instruct) vision-language model using [llama.cpp](https://github.com/ggerganov/llama.cpp) with image input.

---

## ✅ Prerequisites

- macOS (Apple Silicon) or Linux
- CMake ≥ 3.22
- `git`, `wget`

---

## 🔧 1. Install llama.cpp with Metal (Mac) or CPU/GPU (Linux)

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
mkdir build && cd build
cmake -DLLAMA_METAL=on ..         # Use -DLLAMA_CUBLAS=on for CUDA
cmake --build . --config Release
```

---

## 📦 2. Download SmolVLM-500M-Instruct (Q8\_0) and mmproj

```bash
mkdir -p ~/llama.cpp/models
cd ~/llama.cpp/models

# Model
wget -O SmolVLM-500M-Instruct-Q8_0.gguf \
https://huggingface.co/ggml-org/SmolVLM-500M-Instruct-GGUF/resolve/main/SmolVLM-500M-Instruct-Q8_0.gguf

# Multimodal projection
wget -O mmproj-SmolVLM-500M-Instruct-Q8_0.gguf \
https://huggingface.co/ggml-org/SmolVLM-500M-Instruct-GGUF/resolve/main/mmproj-SmolVLM-500M-Instruct-Q8_0.gguf
```

---

## 🚀 3. Launch the llama server

```bash
cd ~/llama.cpp/build/bin

./llama-server \
  -m ../../models/SmolVLM-500M-Instruct-Q8_0.gguf \
  --mmproj ../../models/mmproj-SmolVLM-500M-Instruct-Q8_0.gguf \
  -ngl 1     # Use -ngl 0 for CPU-only
```

Server will start on `http://localhost:8080`.

### 스크립트로 서버 시작하기

더 편리하게 서버를 시작하기 위해 다음과 같이 실행 스크립트를 만들어 사용할 수 있습니다:

```bash
# 스크립트 만들기
mkdir -p ~/bin
cat > ~/bin/start-smolVLM << 'EOL'
#!/bin/bash

# SmolVLM-500M 서버 시작 스크립트
cd ~/llama.cpp/build/bin

./llama-server \
  -m ../../models/SmolVLM-500M-Instruct-Q8_0.gguf \
  --mmproj ../../models/mmproj-SmolVLM-500M-Instruct-Q8_0.gguf \
  -ngl 1     # GPU 사용. CPU만 사용하려면 -ngl 0으로 변경

echo "SmolVLM-500M 서버가 시작되었습니다."
EOL

# 실행 권한 부여
chmod +x ~/bin/start-smolVLM

# PATH에 추가 (만약 아직 추가되지 않았다면)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

이제 다음 명령어로 서버를 간단히 시작할 수 있습니다:

```bash
start-smolVLM
```

---

## 🌐 4. Open the Web UI

```bash
open ../examples/server/public/index.html
```

Or open manually in browser:
`file:///Users/yourname/llama.cpp/examples/server/public/index.html`

---

## 📤 5. API Request Format (example)

```json
POST /v1/chat/completions
Content-Type: application/json

{
  "messages": [
    {
      "role": "user",
      "content": [
        { "type": "text", "text": "What do you see?" },
        { "type": "image_url", "image_url": { "url": "data:image/jpeg;base64,..." } }
      ]
    }
  ],
  "max_tokens": 100
}
```

---

## ✅ Example Result

```
Asian man in front of a mirror looking at himself.
```

---

## Notes

* Requires llama.cpp after [PR #13050](https://github.com/ggerganov/llama.cpp/pull/13050)
* For real-time video/image input, use camera + canvas → base64 → API

---

```
