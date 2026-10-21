# local-ai-cheatsheet-es-pt 🇪🇸 🇧🇷

**IA 100% local, sin suscripciones.** Todo en esta hoja se ejecuta en tu propio hardware: sin cuotas mensuales, sin que tus datos salgan de tu máquina.

> 🧾 **Autor:** Ezra-Zhao · Última revisión: octubre de 2026
>
> Nota de calidad: cada comando de esta hoja fue verificado contra la documentación pública de cada proyecto en octubre de 2026. Las cifras de hardware son aproximadas ("~") y sirven como orientación rápida, no como garantía.

---
---

# 🇪🇸 ESPAÑOL


Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 1 · Inferencia LLM local — modelos corriendo en tu máquina

| Herramienta | Para qué sirve (1 frase) | Instalar / arrancar | Ejemplo mínimo |
|---|---|---|---|
| **Ollama** | La forma más sencilla de correr LLMs open source: descarga modelos y los sirve por API con un solo comando. | `curl -fsSL https://ollama.com/install.sh \| sh` | `ollama pull llama3`  ·  `ollama run llama3 "Explica qué es la IA local en una frase"` |
| **llama.cpp** | Motor de inferencia en C/C++ (GGUF), rapidísimo y sin dependencias pesadas; base de Ollama y LocalAI. | `git clone https://github.com/ggml-org/llama.cpp`<br>`cd llama.cpp`<br>`cmake -B build && cmake --build build --config Release` | `llama-cli -hf bartowski/Llama-3.2-3B-Instruct-GGUF:Q8_0`<br>(descarga el modelo desde Hugging Face y abre el chat) |
| **llama.cpp (Python)** | Enlace Python de llama.cpp para usar GGUF dentro de tus scripts. | `pip install llama-cpp-python` | `python -c "from llama_cpp import Llama; llm=Llama(model_path='modelo.gguf'); print(llm('Hola', max_tokens=32)['choices'][0]['text'])"` |
| **LocalAI** | Sustituto local de la API de OpenAI: hablas a `localhost` con el formato `/v1` y todo corre en tu PC. | `docker run -ti -p 8080:8080 --name local-ai localai/localai:latest-aio-cpu` | `curl http://localhost:8080/v1/models`  (lista los modelos instalados) |

💡 Ollama también expone una API compatible con OpenAI en `http://localhost:11434/v1` una vez corre `ollama serve`.

## 2 · Interfaz web local — tu "ChatGPT" en el navegador

| Herramienta | Para qué sirve (1 frase) | Instalar / arrancar | Ejemplo mínimo |
|---|---|---|---|
| **Open WebUI** | Interfaz web tipo ChatGPT que se conecta a tu Ollama local (o a otras APIs) y guarda todo en tu máquina. | `docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main` | Abre `http://localhost:3000` → crea tu cuenta local → elige el modelo Ollama y chatea. |

## 3 · Voz local — transcribe y habla sin la nube

| Herramienta | Para qué sirve (1 frase) | Instalar / arrancar | Ejemplo mínimo |
|---|---|---|---|
| **whisper.cpp** | Convierte voz en texto localmente (C/C++, rápido); ideal cuando quieres transcripción 100% offline. | `pip install openai-whisper` (ruta fácil)<br>o compila: `git clone https://github.com/ggml-org/whisper.cpp && cmake -B build && cmake --build build -j --config Release` | `whisper audio.mp3 --language Spanish`  ·  C++: `./build/bin/whisper-cli -m models/ggml-base.bin audio.wav` |
| **faster-whisper** | Versión Python de Whisper hasta ~4× más rápida y con menos memoria, basada en CTranslate2. | `pip install faster-whisper` | `python - <<'EOF'`<br>`from faster_whisper import WhisperModel`<br>`model = WhisperModel("base", device="cpu", compute_type="int8")`<br>`segments, info = model.transcribe("audio.mp3", language="es")`<br>`for s in segments: print(s.text)`<br>`EOF` |
| **Piper TTS** | Texto-a-voz neuronal local, rapidísima y con voces en español y portugués; sin internet tras la descarga. | `pip install piper-tts` | `echo "Hola, esto es IA local y gratis." \| piper --model es_ES-sharvard-medium --output_file saludo.wav`<br>(más voces en `huggingface.co/rhasspy/piper-voices`: formato `idioma_REGIÓN-nombre-calidad`) |

## 4 · Agentes y flujos de trabajo — automatiza con IA local

| Herramienta | Para qué sirve (1 frase) | Instalar / arrancar | Ejemplo mínimo |
|---|---|---|---|
| **n8n** | Automatización visual de flujos: conecta apps, webhooks y tus modelos locales con nodos de arrastrar y soltar. | `docker volume create n8n_data`<br>`docker run -d --name n8n -p 5678:5678 --restart unless-stopped -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n` | Abre `http://localhost:5678` → crea un flujo con el nodo "AI Agent" apuntando a tu Ollama (`http://host.docker.internal:11434`). |
| **Dify** | Plataforma para construir chatbots, RAG y agentes de IA con interfaz no-code, todo autoalojado. | `git clone https://github.com/langgenius/dify.git`<br>`cd dify/docker && cp .env.example .env`<br>`docker compose up -d` | Abre `http://localhost/install` → crea el usuario admin → en **Ajustes → Proveedor de modelos** añade tu Ollama local y crea tu primer asistente. |

## 5 · Hardware — requisitos aproximados

| Carga | GPU (VRAM) | Solo CPU (RAM) |
|---|---|---|
| LLM 3B–8B (cuantizado Q4, p. ej. Llama 3 8B) | ~8 GB | ~8 GB RAM |
| LLM 13B cuantizado | ~10–12 GB | ~16 GB RAM |
| LLM 70B cuantizado | ~40 GB+ (varias GPUs) | ~64 GB RAM |
| Whisper `base`/`small` | — | ~2–4 GB RAM |
| Whisper `large-v3` (faster-whisper) | ~4–10 GB | ~8 GB RAM |
| Piper TTS | — | funciona en casi cualquier PC (~60 MB por voz) |
| Open WebUI / n8n / Dify (sin contar el modelo) | — | ~2–4 GB RAM; Dify ~20 GB de disco para sus servicios |

> Regla de oro: si tu PC corre un navegador moderno con soltura, ya puede correr Piper, n8n, Open WebUI y modelos LLM pequeños. La GPU solo acelera (mucho).

---
---

# 🇧🇷 PORTUGUÊS

## 1 · Inferência LLM local — modelos rodando na sua máquina

| Ferramenta | Para que serve (1 frase) | Instalar / iniciar | Exemplo mínimo |
|---|---|---|---|
| **Ollama** | O jeito mais fácil de rodar LLMs open source: baixa modelos e serve por API com um único comando. | `curl -fsSL https://ollama.com/install.sh \| sh` | `ollama pull llama3`  ·  `ollama run llama3 "Explique o que é IA local em uma frase"` |
| **llama.cpp** | Motor de inferência em C/C++ (GGUF), rapidíssimo e sem dependências pesadas; base do Ollama e do LocalAI. | `git clone https://github.com/ggml-org/llama.cpp`<br>`cd llama.cpp`<br>`cmake -B build && cmake --build build --config Release` | `llama-cli -hf bartowski/Llama-3.2-3B-Instruct-GGUF:Q8_0`<br>(baixa o modelo do Hugging Face e abre o chat) |
| **llama.cpp (Python)** | Binding Python do llama.cpp para usar GGUF dentro dos seus scripts. | `pip install llama-cpp-python` | `python -c "from llama_cpp import Llama; llm=Llama(model_path='modelo.gguf'); print(llm('Olá', max_tokens=32)['choices'][0]['text'])"` |
| **LocalAI** | Substituto local da API da OpenAI: você fala com `localhost` no formato `/v1` e tudo roda no seu PC. | `docker run -ti -p 8080:8080 --name local-ai localai/localai:latest-aio-cpu` | `curl http://localhost:8080/v1/models`  (lista os modelos instalados) |

💡 O Ollama também expõe uma API compatível com a OpenAI em `http://localhost:11434/v1` quando `ollama serve` está rodando.

## 2 · Interface web local — seu "ChatGPT" no navegador

| Ferramenta | Para que serve (1 frase) | Instalar / iniciar | Exemplo mínimo |
|---|---|---|---|
| **Open WebUI** | Interface web estilo ChatGPT que se conecta ao seu Ollama local (ou a outras APIs) e guarda tudo na sua máquina. | `docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main` | Abra `http://localhost:3000` → crie sua conta local → escolha o modelo do Ollama e converse. |

## 3 · Voz local — transcreva e fale sem a nuvem

| Ferramenta | Para que serve (1 frase) | Instalar / iniciar | Exemplo mínimo |
|---|---|---|---|
| **whisper.cpp** | Converte voz em texto localmente (C/C++, rápido); ideal para transcrição 100% offline. | `pip install openai-whisper` (caminho fácil)<br>ou compile: `git clone https://github.com/ggml-org/whisper.cpp && cmake -B build && cmake --build build -j --config Release` | `whisper audio.mp3 --language Portuguese`  ·  C++: `./build/bin/whisper-cli -m models/ggml-base.bin audio.wav` |
| **faster-whisper** | Versão Python do Whisper até ~4× mais rápida e com menos memória, baseada em CTranslate2. | `pip install faster-whisper` | `python - <<'EOF'`<br>`from faster_whisper import WhisperModel`<br>`model = WhisperModel("base", device="cpu", compute_type="int8")`<br>`segments, info = model.transcribe("audio.mp3", language="pt")`<br>`for s in segments: print(s.text)`<br>`EOF` |
| **Piper TTS** | Texto-para-fala neural local, rapidíssimo e com vozes em português e espanhol; sem internet após o download. | `pip install piper-tts` | `echo "Olá, isto é IA local e gratuita." \| piper --model VOZ_PT --output_file ola.wav`<br>(baixe uma voz `pt_*` em `huggingface.co/rhasspy/piper-voices`, formato `idioma_REGIÃO-nome-qualidade`, e use o nome do arquivo `.onnx`) |

## 4 · Agentes e fluxos de trabalho — automatize com IA local

| Ferramenta | Para que serve (1 frase) | Instalar / iniciar | Exemplo mínimo |
|---|---|---|---|
| **n8n** | Automação visual de fluxos: conecta apps, webhooks e seus modelos locais com nós de arrastar e soltar. | `docker volume create n8n_data`<br>`docker run -d --name n8n -p 5678:5678 --restart unless-stopped -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n` | Abra `http://localhost:5678` → crie um fluxo com o nó "AI Agent" apontando para o seu Ollama (`http://host.docker.internal:11434`). |
| **Dify** | Plataforma para criar chatbots, RAG e agentes de IA com interface no-code, tudo self-hosted. | `git clone https://github.com/langgenius/dify.git`<br>`cd dify/docker && cp .env.example .env`<br>`docker compose up -d` | Abra `http://localhost/install` → crie o usuário admin → em **Configurações → Provedor de modelos** adicione seu Ollama local e crie seu primeiro assistente. |

## 5 · Hardware — requisitos aproximados

| Carga | GPU (VRAM) | Só CPU (RAM) |
|---|---|---|
| LLM 3B–8B (quantizado Q4, ex. Llama 3 8B) | ~8 GB | ~8 GB de RAM |
| LLM 13B quantizado | ~10–12 GB | ~16 GB de RAM |
| LLM 70B quantizado | ~40 GB+ (várias GPUs) | ~64 GB de RAM |
| Whisper `base`/`small` | — | ~2–4 GB de RAM |
| Whisper `large-v3` (faster-whisper) | ~4–10 GB | ~8 GB de RAM |
| Piper TTS | — | roda em quase qualquer PC (~60 MB por voz) |
| Open WebUI / n8n / Dify (sem contar o modelo) | — | ~2–4 GB de RAM; Dify ~20 GB de disco para seus serviços |

> Regra de ouro: se o seu PC roda um navegador moderno com folga, já consegue rodar Piper, n8n, Open WebUI e modelos LLM pequenos. A GPU só acelera (muito).

---
---

# 🇬🇧 ENGLISH (summary)

A one-page cheatsheet (Spanish + Portuguese) for running AI fully locally with zero subscriptions. Covered tools, all with verified real commands:

| Tool | One-liner | Start command |
|---|---|---|
| Ollama | Easiest way to run open-source LLMs locally | `curl -fsSL https://ollama.com/install.sh \| sh` · `ollama pull llama3` · `ollama run llama3 "prompt"` |
| llama.cpp | Fast C/C++ GGUF inference engine | `git clone https://github.com/ggml-org/llama.cpp` + cmake build · `pip install llama-cpp-python` |
| LocalAI | Local OpenAI-compatible API (`localhost:8080/v1`) | `docker run -ti -p 8080:8080 --name local-ai localai/localai:latest-aio-cpu` |
| Open WebUI | ChatGPT-like web UI over your local models | `docker run -d -p 3000:8080 ... ghcr.io/open-webui/open-webui:main` → `http://localhost:3000` |
| whisper.cpp | Offline speech-to-text | `pip install openai-whisper` · `whisper audio.mp3 --language Spanish` |
| faster-whisper | ~4× faster Whisper (CTranslate2) | `pip install faster-whisper` · `WhisperModel("base", device="cpu", compute_type="int8")` |
| Piper TTS | Local neural text-to-speech (ES/PT voices) | `pip install piper-tts` · `echo "text" \| piper --model es_ES-sharvard-medium --output_file out.wav` |
| n8n | Visual workflow automation, self-hosted | `docker run -d -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n` |
| Dify | No-code AI apps/agents/RAG platform | `git clone https://github.com/langgenius/dify.git && cd dify/docker && cp .env.example .env && docker compose up -d` |

Hardware rule of thumb: an 8B quantized LLM needs ~8 GB RAM/VRAM; Whisper base runs on ~2–4 GB; Piper/n8n/Open WebUI run on almost any PC. All figures in the tables are marked approximate.

---

## Licencia / Licença / License

MIT — ver / veja / see [LICENSE](LICENSE). © 2026 Guangyi Zhao (Ezra-Zhao).
