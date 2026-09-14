# Architecture

## Tech stack

| Area | Choice |
| --- | --- |
| Frontend | React, TypeScript, Vite |
| Backend | Python, FastAPI, Pydantic |
| System data | Python/psutil where practical; fixed read-only Linux commands where needed |
| Containers | Podman |
| AI | Ollama with a local quantized, tool-capable model |
| Database | None in V1 |
| Testing | Pytest for backend and parsers; Vitest and React Testing Library for frontend |
| Deployment | Unprivileged systemd service, same-origin frontend/API, private access through Tailscale |

Exact package versions belong in dependency files created during the foundation task.

## System shape

```text
Browser over private network
          |
          v
React dashboard and chat
          |
          v
FastAPI routes + authentication
          |
          v
Typed application services
       /         \
      v           v
Read-only       AI orchestrator
collectors          |
      |             v
      |       fixed tool registry
      |             |
      +------<------+ 
      |             |
      v             v
Ubuntu/Podman     Ollama on localhost
```

The dashboard and AI use the same application services. The AI has no special path to the operating system.

## Repository structure

Create folders only when their feature is implemented.

```text
README.md
REQUIREMENTS.md
ARCHITECTURE.md
AGENTS.md
backend/
  pyproject.toml
  requirements.txt
  requirements-dev.txt
  app/
    main.py
    config.py
    api/              # HTTP routes and authentication
    models/           # Pydantic request/response models
    services/         # application logic and exposure assessment
    system/
      runner.py       # private bounded process execution
      collectors/     # fixed read-only operations
      parsers/        # pure parsers for command output
    agent/
      client.py       # local Ollama client
      tools.py        # explicit tool definitions and registry
      orchestrator.py # bounded tool-call loop
  tests/
    fixtures/
    unit/
    integration/
frontend/
  package.json
  src/
    api/
    components/
    pages/
    types/
deploy/
  systemd/
```

## Main boundaries

### System inspection

Only code under `backend/app/system/` may invoke operating-system commands. Prefer Python APIs or structured command output. Commands must use fixed executable paths and fixed argument templates, pass arguments as an array with `shell=False`, and have time and output limits.

Parsers are pure functions. They accept captured text and return typed data, which makes Ubuntu output testable on another development machine.

### API and application services

API routes validate input and call typed services. Routes never run commands directly. Responses include a UTC collection time and a status such as `ok`, `partial`, `unavailable`, or `error`. Missing readings are `null`, never fake zeroes.

Metrics use short-lived in-memory caching and normal HTTP polling. V1 needs no database or WebSocket layer.

### AI assistant

The Ollama endpoint and model are operator configuration, not chat input. The model may request only registered tools such as `get_memory_usage`, `get_listening_ports`, or `get_service_status`. Pydantic validates every call before a tool dispatcher invokes the same services used by the API.

The orchestration loop limits tool calls, rounds, input size, result size, execution time, and concurrent model requests. Host output is untrusted data and cannot add tools or alter policy. The assistant separates observed evidence from possible explanations.

### Exposure assessment

Exposure is derived from listening addresses, network interfaces, UFW defaults and rules, trusted LAN ranges, Tailscale addresses, and container port publishing. A wildcard listener alone does not prove public exposure, and UFW alone cannot prove the absence of router forwarding. Results therefore include a classification, evidence, and confidence or uncertainty.

### Deployment

FastAPI runs directly on the Ubuntu host as a dedicated unprivileged user because host inspection from a container creates namespace and permission problems. The production frontend and API share one origin. FastAPI and Ollama bind to loopback, and Tailscale provides private remote access. Tailscale Funnel is outside V1.

Every server-information and chat route requires authentication. Use a server-validated single-user session, secure HTTP-only cookies, origin/CSRF checks for state-changing requests, strict allowed hosts, and no wildcard credentialed CORS.

## Security rules

- Never create a generic shell, command, script, code execution, file write, service control, container control, firewall modification, or package-management tool.
- Never run the backend as root or grant broad passwordless sudo access.
- Validate all tool inputs and reject extra fields. Service and container identifiers must come from discovered data and cannot be interpreted as command options.
- Do not expose process environments or full command lines by default.
- Bound and redact logs before they reach the browser or model. Treat process names, container labels, and logs as untrusted text.
- Never send prompts or server data to a cloud model and never fall back to one.
- Report permission gaps and incomplete visibility rather than silently omitting data.
- Do not claim a service is safe from the public internet without enough network, firewall, and routing evidence.

## Initial read-only tool set

```text
get_cpu_usage
get_memory_usage
get_disk_usage
get_gpu_status
get_temperatures
get_network_interfaces
get_listening_ports
get_processes
get_services
get_service_status
get_recent_service_logs
get_ufw_rules
get_containers
get_container_stats
get_system_uptime
```

Tools that accept input use small schemas with enumerated sort fields, bounded result counts, validated discovered identifiers, and bounded log time ranges. There is no tool that accepts an executable name, shell string, arbitrary argument list, path, URL, or raw journal expression.
