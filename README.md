# Run DeepSeek r1 Locally with Ollama & Open WebUI

Hey there! I put together this microservices setup to run DeepSeek r1 (the latest version) locally using Ollama as the backend engine and Open WebUI as the frontend interface. No cloud services needed - everything runs right on your machine!

## What's in the Box

This repo gives you two separate microservices:
- **Ollama container**: Handles all the AI model stuff (running DeepSeek r1)
- **Open WebUI container**: Gives you a clean chat interface to talk to the model

They communicate with each other but run independently - proper microservice architecture!

## Quick Start

1. **Get Docker running**  
   You'll need Docker installed → [Get Docker here](https://docs.docker.com/get-docker/)

2. **Grab this repo**
   ```sh
   git clone <repository-url>
   cd ollama-deepseek-docker
   ```

3. **Fire it up!**
   ```sh
   docker compose up -d
   ```
   > First run will take a while (maybe grab a coffee?) as it downloads the DeepSeek r1 model (~7GB)

4. **Start chatting**  
   Just open [http://localhost:8080](http://localhost:8080) in your browser

## Behind the Scenes

What's actually happening:
- The Ollama container boots up and automatically downloads DeepSeek r1 if needed
- The WebUI connects to Ollama's API (they talk to each other through Docker's internal network)
- Everything runs locally - your data stays on your machine

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

**Nuclear option (removes all data)**
```sh
docker compose down -v
```

## Tips

- DeepSeek r1 is a beast of a model, so performance depends on your hardware
- If you're on a laptop, expect your fans to spin up
- First responses might be slow as the model warms up

Enjoy your local AI setup! Let me know if you run into any issues.


