# Home Server Dashboard

A private dashboard for monitoring one Ubuntu home server, with a local read-only AI assistant powered by Ollama.

The dashboard will show system health, listening ports and their owners, firewall information, services, logs, and Podman containers. The assistant will answer questions about that information through an explicit set of safe inspection tools. It will not administer the server.

## Project status

Planning is complete. Application implementation has not started.

Read these files before building:

- [REQUIREMENTS.md](REQUIREMENTS.md) defines the product, V1 scope, feature order, and completion criteria.
- [ARCHITECTURE.md](ARCHITECTURE.md) defines the stack, component boundaries, repository shape, and security model.
- [AGENTS.md](AGENTS.md) tells Codex and other coding agents how to work in this repository.

## Target server

- Ubuntu Server, headless
- Intel i9-11900
- 32 GB RAM
- NVIDIA RTX 3080 with 10 GB VRAM
- 1 TB SSD
- Tailscale installed
- UFW enabled
- SSH intended for Tailscale and explicitly trusted LAN sources only

## Planned stack

React and TypeScript in the browser call a Python FastAPI backend. The backend gathers server data through predefined read-only collectors and gives the same safe tools to a local Ollama agent. V1 has no database.

## Building the project

Choose exactly one feature task from `REQUIREMENTS.md`. Ask the coding agent to implement that task and its definition of done, then review and test it before moving to the next task.

Recommended first prompt:

> Read `README.md`, `REQUIREMENTS.md`, `ARCHITECTURE.md`, and `AGENTS.md`. Implement Feature 1: Project foundation. Follow its definition of done and do not implement later features.

Run instructions will be added when the project foundation is implemented.
