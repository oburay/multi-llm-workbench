# Multi-LLM Workbench: Run Multiple AI Models Locally

I built this flexible microservices setup to run **multiple Ollama-compatible models simultaneously** on your local machine. This advanced stack combines [Ollama](https://ollama.ai/) with [Open WebUI](https://github.com/open-webui/open-webui) to create a complete AI workbench where you can leverage different specialized models for various tasks!

## What's in the Box

This repo gives you a powerful multi-model template with isolated microservices:
- **Multiple Ollama containers**: Each running a different AI model in its own environment
  - DeepSeek r1: Great for complex reasoning and analysis
  - Llama3: Excellent for general conversation and creative tasks
  - Phi3: Optimized for code and technical documentation
  - Gemma3: Google's efficient and capable model
- **Single Open WebUI container**: Provides a unified interface to access all models

Each model runs independently in its own container with dedicated resources, as microservices.

## Quick Start

1. **Get Docker running**  
   You'll need Docker installed → [Get Docker here](https://docs.docker.com/get-docker/)

2. **Clone this repo**
   ```sh
   git clone https://github.com/oburay/multi-llm-workbench.git
   cd multi-llm-workbench
   ```

3. **Customize your model selection (optional)**
   
   The default setup includes DeepSeek r1, Llama3, Phi3, and Gemma3, but you can:
   - Edit any of the Dockerfiles in the ollama directory to use different models
   - Add new models by creating additional Dockerfiles following the pattern
   - Remove models by commenting out their sections in docker-compose.yml

4. **Launch your AI workbench!**
   ```sh
   docker compose up -d
   ```
   > First run will take a while as it downloads multiple models (a good time to make coffee!)

5. **Start interacting with your models**  
   Just open [http://localhost:8080](http://localhost:8080) in your browser

6. **Switch between models**  
   Use the model selector dropdown in the chat interface to choose which model handles each conversation

## Behind the Scenes

What's actually happening:
- Each Ollama container boots up and downloads its specific model if needed
- All models run in parallel, each in its own isolated environment
- The containers share Docker networks but have separate data volumes
- Open WebUI provides a unified interface to all models
- Everything runs locally - your data stays on your machine

## System Architecture

```
┌───────────────────────────────────────────────────────────────────────────┐
│                                Your Machine                               │
│                                                                           │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │                         Docker Environment                         │   │
│  │                                                                    │   │
│  │   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌────────┐ │   │
│  │   │             │   │             │   │             │   │        │ │   │
│  │   │ DeepSeek r1 │   │   Llama3    │   │    Phi3     │   │ Gemma3 │ │   │
│  │   │  Container  │   │  Container  │   │  Container  │   │Container││   │
│  │   │             │   │             │   │             │   │        │ │   │
│  │   └──────┬──────┘   └──────┬──────┘   └──────┬──────┘   └────┬───┘ │   │
│  │          │                 │                 │               │     │   │
│  │          │                 │                 │               │     │   │
│  │   ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐   ┌────▼───┐ │   │
│  │   │  DeepSeek   │   │   Llama3    │   │    Phi3     │   │ Gemma3 │ │   │
│  │   │    Data     │   │    Data     │   │    Data     │   │  Data  │ │   │
│  │   │   Volume    │   │   Volume    │   │   Volume    │   │ Volume │ │   │
│  │   └─────────────┘   └─────────────┘   └─────────────┘   └────────┘ │   │
│  │                                                                    │   │
│  │                            ┌─────────────┐                         │   │
│  │                            │             │                         │   │
│  │                            │ Open WebUI  │                         │   │
│  │                            │  Container  │◄────────────────────────┘   │
│  │                            │             │                             │
│  │                            └──────┬──────┘                             │
│  └───────────────────────────────────┼────────────────────────────────────┘
│                                      │                                    │
│  ┌─────────────┐                     │                                    │
│  │             │                     │                                    │
│  │Web Browser  │◄────────────────────┘                                    │
│  │             │                                                          │
│  └─────────────┘                                                          │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘

      Port 11434-11437 (APIs)                               Port 8080 (UI)
      (Each model on separate port)                         (User access)
```

## Useful Commands

These come in handy when you need to check on things:

**See what's running**
```sh
docker compose ps
```

**Check the logs for a specific model**
```sh
# For a specific model
docker logs ollama-llama3

# For the UI
docker logs ollama-webui

# For all containers
docker compose logs -f
```

**Shut everything down**
```sh
docker compose down
```

**Remove all model data**
```sh
docker compose down -v
```

**Check which models are available in a container**
```sh
docker exec -it ollama-llama3 ollama list
```

## Tips for Multi-Model Workbench

- **Resource management is important**:
  - Running multiple models simultaneously requires more RAM and CPU
  - Consider stopping models you're not actively using: `docker stop ollama-phi3`
  - Start them again when needed: `docker start ollama-phi3`

- **Choose the right model for each task**:
  - Use larger models (DeepSeek r1) for complex reasoning
  - Use code-specialized models (Phi3) for programming
  - Use smaller models for quick interactions

- **Expect slower initial responses** as each model warms up

- **Network usage**: Each model runs on a separate port internally:
  - DeepSeek r1: 11434
  - Llama3: 11435
  - Phi3: 11436
  - Gemma3: 11437

## Adding New Models

To add another model to your workbench:

1. Create a new Dockerfile in the ollama directory:
```dockerfile
# filepath: ./ollama/Dockerfile.yourmodel
FROM ollama/ollama:latest
EXPOSE 11434
RUN echo '#!/bin/sh\n\
MODEL_PATH="/root/.ollama/models/manifests/registry.ollama.ai/library/yourmodel/latest"\n\
if [ ! -f "$MODEL_PATH" ]; then\n\
  echo "Model not found, downloading..."\n\
  ollama serve & SERVER_PID=$!\n\
  sleep 5\n\
  ollama pull yourmodel\n\
  kill $SERVER_PID\n\
  wait $SERVER_PID\n\
  echo "Model downloaded successfully"\n\
fi\n\
echo "Starting Ollama server..."\n\
exec ollama serve\n' > /startup.sh && chmod +x /startup.sh
ENTRYPOINT ["/startup.sh"]
```

2. Add the new service to docker-compose.yml:
```yaml
  ollama-yourmodel:
    build:
      context: ./ollama
      dockerfile: Dockerfile.yourmodel
    container_name: ollama-yourmodel
    volumes:
      - ollama_yourmodel_data:/root/.ollama
    ports:
      - "11438:11434"  # Use next available port
    restart: unless-stopped
    networks:
      - ollama_network
```

3. Add the volume entry:
```yaml
volumes:
  # ... existing volumes
  ollama_yourmodel_data:
```

4. Update the webui environment to include the new model:
```yaml
  webui:
    # ... existing config
    environment:
      - OLLAMA_BASE_URL=http://ollama-deepseek:11434
      - OLLAMA_BASE_URLS=http://ollama-deepseek:11434;http://ollama-llama3:11434;http://ollama-phi3:11434;http://ollama-gemma3:11434;http://ollama-yourmodel:11434
```

## Credits

This project builds upon these amazing open source projects:
- [Ollama](https://github.com/ollama/ollama) - For running LLMs locally
- [Open WebUI](https://github.com/open-webui/open-webui) - For the intuitive chat interface

This project extends the single-model approach from [local-llm-stack](https://github.com/oburay/local-llm-stack) to support multiple specialized LLMs running simultaneously.
