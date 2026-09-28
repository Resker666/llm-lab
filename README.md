# LLM Lab

A personal lab for learning, deploying, testing, and debugging Large Language Models.

This repository records hands-on experience with local LLMs, inference frameworks, AI tools, Agents, MCP, RAG, and related engineering practices.

## Philosophy

This is not an AI encyclopedia.

The goal is to document things that I have actually:

- tried
- deployed
- benchmarked
- debugged
- understood
- found useful

> Learn by building. Record by doing.

## Topics

- Local LLM deployment
- Ollama / llama.cpp / vLLM
- Qwen / DeepSeek / GLM and other models
- GGUF and quantization
- Inference performance and hardware
- AI Agents
- MCP (Model Context Protocol)
- RAG
- AI development tools
- Troubleshooting and debugging

## Repository Structure

```text
llm-lab/
├── README.md
├── local-deployment/   # Local model deployment and runtime setup
├── models/             # Notes and experiments for specific model families
├── inference/          # Quantization, performance, hardware and benchmarks
├── agents/             # Agent experiments and workflows
├── mcp/                # Model Context Protocol notes and experiments
├── rag/                # Retrieval-Augmented Generation
├── tools/              # AI development tools
└── troubleshooting/    # Problems, root causes and fixes
```

The repository will grow gradually. Directories are added when there is real content to record rather than creating a large empty skeleton up front.

## Current Starting Points

- [Local deployment](./local-deployment/README.md)
- [Qwen notes](./models/qwen.md)
- [AI tools](./tools/README.md)
- [Troubleshooting](./troubleshooting/README.md)

## Related Repository

General Linux, Git, server, deployment, embedded Linux and debugging notes belong in my `tech-playbook` repository.

This repository focuses specifically on AI / LLM engineering and experiments.
