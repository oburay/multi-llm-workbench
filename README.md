# Local LLM Stack: Run Any Ollama Model with Open WebUI

 I put together this flexible microservices setup to run **any Ollama-compatible model** locally. This stack combines [Ollama](https://ollama.ai/) with [Open WebUI](https://github.com/open-webui/open-webui)  to create a complete local AI solution. While it's pre-configured for DeepSeek r1, you can easily swap in any model you prefer!

## What's in the Box

This repo gives you a reusable template with two separate microservices:
- **Ollama container**: Handles all the AI model stuff (runs whatever model you choose)
- **Open WebUI container**: Gives you a clean chat interface to talk to the model

They communicate with each other but run independently - proper microservice architecture!

## Quick Start

1. **Get Docker running**  
   You'll need Docker installed → [Get Docker here](https://docs.docker.com/get-docker/)

2. **Grab this repo**
   ```sh
   git clone <repository-url>
   cd local-llm-stack
   ```

3. **Choose your model (optional)**
   
   The default is set to DeepSeek r1, but you can easily change it:
   - Edit the Dockerfile in the ollama directory to replace "deepseek-r1" with any model from [Ollama's library](https://ollama.com/library)
   - Examples: llama3, mistral, phi3, codellama, etc.

4. **Fire it up!**
   ```sh
   docker compose up -d
   ```
   > First run will take a while (maybe grab a coffee?) as it downloads the model

5. **Start chatting**  
   Just open [http://localhost:8080](http://localhost:8080) in your browser

## Behind the Scenes

What's actually happening:
- The Ollama container boots up and automatically downloads your chosen model if needed
- The Open WebUI connects to Ollama's API (they talk to each other through Docker's internal network)
- Everything runs locally - your data stays on your machine

## System Architecture

```
┌────────────────────────────────────────────────────────────┐
│                        Your Machine                        │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │               Docker Environment                    │   │
│  │                                                     │   │
│  │   ┌─────────────────┐         ┌─────────────────┐   │   │
│  │   │                 │         │                 │   │   │
│  │   │  Ollama Engine  │◄────────┤   Open WebUI    │   │   │
│  │   │   Container     │         │    Container    │   │   │
│  │   │                 │         │                 │   │   │
│  │   └────────┬────────┘         └────────▲────────┘   │   │
│  │            │                           │            │   │
│  │            │                           │            │   │
│  │   ┌────────▼────────┐                  │            │   │
│  │   │                 │                  │            │   │
│  │   │   Ollama Data   │                  │            │   │
│  │   │     Volume      │                  │            │   │
│  │   │                 │                  │            │   │
│  │   └─────────────────┘                  │            │   │
│  │                                        │            │   │
│  └────────────────────────────────────────┼────────────┘   │
│                                           │                │
│  ┌─────────────────┐                      │                │
│  │                 │                      │                │
│  │  Web Browser    │◄─────────────────────┘                │
│  │                 │                                       │
│  └─────────────────┘                                       │
│                                                            │
└────────────────────────────────────────────────────────────┘

          │                                     │
          │                                     │
          ▼                                     ▼
   Port 11434 (API)                      Port 8080 (UI)
   (Not exposed externally)              (User access)
```

## Useful Commands

When you need to check on things:

**See what's running**
```sh
docker compose ps
```

**Check the logs (helpful for debugging)**
```sh
docker compose logs -f
```

**Shut it down**
```sh
docker compose down
```

**Removes all data**
```sh
docker compose down -v
```

## Tips

- Larger models like DeepSeek r1 require decent hardware
- If you're on a laptop, expect your fans to spin up
- First responses might be slow as the model warms up
- Try smaller models like Phi-3 mini if you need faster responses on modest hardware

## Credits

This project builds upon these amazing open source projects:
- [Ollama](https://github.com/ollama/ollama) - For running LLMs locally
- [Open WebUI](https://github.com/open-webui/open-webui) - For the intuitive chat interface

