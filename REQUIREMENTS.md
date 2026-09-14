# Requirements

## App idea

This project is a private web dashboard for one Ubuntu home server. It gives the server owner a quick view of system health, running software, network exposure, and firewall configuration.

It also includes a local AI assistant that can explain what is happening on the server. The assistant reads server information through predefined tools and uses a local Ollama model. It cannot change the server or run arbitrary commands.

The main problem it solves is having to remember and combine many Linux commands just to answer simple questions such as:

- What is using the most RAM?
- What is running on port 3000?
- Is SSH limited to Tailscale and trusted LAN devices?
- Why did a service fail?
- Which containers use the most resources?

## V1 requirements

1. Show CPU, RAM, disk, GPU, temperature, network, and uptime information.
2. Show listening TCP ports and bound UDP ports, including the owning process or service when available.
3. Show whether a listener appears to be local-only, LAN-only, Tailscale-only, potentially public, or unknown, with the evidence used for that result.
4. Show whether UFW is enabled and display its relevant defaults and rules.
5. Show running and failed systemd services, with bounded recent logs for a selected service.
6. Show Podman containers, their status, published ports, and current resource use.
7. Provide a simple React dashboard with clear loading, unavailable, stale, and error states.
8. Provide a simple chat interface backed by a local Ollama model.
9. Give the AI only predefined, validated, read-only server tools. Never expose a generic shell-command tool.
10. Run the backend without root privileges and keep the dashboard, API, and Ollama private.

## V1 scope

V1 includes one server, one owner, current metrics, service journal history already retained by Ubuntu, Podman, UFW, Tailscale-aware exposure checks, and non-streaming AI chat.

V1 deliberately excludes:

- Changing services, containers, firewall rules, packages, files, or any other server state
- Arbitrary shell access
- Historical metrics and charts
- Alerts and notifications
- Multiple servers or multiple users
- Docker and Kubernetes support
- A database, message queue, vector database, or RAG system
- Cloud AI fallback or public internet hosting
- Automatic fixes or autonomous administration

## Definition of success

V1 is complete when the dashboard works on the target Ubuntu server, all information endpoints require private authenticated access, parser and security tests pass, and the assistant can answer the example questions using tool evidence. When the available data cannot prove an answer, the UI and assistant must say that the result is unknown or incomplete.

## Feature tasks

Build one task at a time. A task includes its backend endpoint, frontend display, types, tests, and relevant documentation unless its description says otherwise.

### 1. Project foundation

Create the FastAPI and React applications, typed health endpoint, configuration, fixture-based local development, test setup, and dependency files.

Done when both applications start, the frontend can call the health endpoint, and tests run without Ubuntu, root access, a GPU, or Ollama.

### 2. Core system metrics

Add CPU, RAM, disk, uptime, GPU, and temperature collection and display.

Done when valid, missing, malformed, and unavailable fixture cases are tested and a failure in one collector does not hide the others.

### 3. Network ports and ownership

Add network interfaces, listening ports, and process or service ownership.

Done when TCP, UDP, IPv4, IPv6, wildcard bindings, missing owners, and port 3000 examples are tested.

### 4. Firewall and exposure

Add UFW information and an evidence-based exposure assessment for each listener.

Done when disabled or unreadable UFW, trusted LAN ranges, Tailscale, IPv6, and uncertain public reachability are represented correctly.

### 5. Services and logs

Add running and failed services, service details, and bounded recent journal logs.

Done when service names and time ranges are validated, logs are limited and redacted, and failure examples can be investigated without arbitrary journal arguments.

### 6. Containers

Add Podman inventory, published ports, and one-shot resource statistics.

Done when no-Podman, no-container, stopped-container, and partial-stat cases are handled and the inspected user/runtime scope is visible.

### 7. Local AI assistant

Add the fixed local Ollama client, explicit tool registry, bounded tool loop, and chat endpoint and interface.

Done when tests prove invented tools and invalid arguments never execute, tool loops are capped, Ollama failures are handled, and the example questions return evidence-based answers.

### 8. Private deployment

Add authenticated same-origin production serving, an unprivileged systemd service, and Tailscale access instructions.

Done when the target server passes the full test and build checks, Ollama and the backend are bound privately, access controls cover every data route, and no write-capable agent tool exists.

## Decisions to make during the relevant task

- Confirm the Ubuntu version and whether Podman containers run rootless, rootful, or both.
- Enter the actual trusted LAN CIDR ranges; do not guess them.
- Choose an installed, tool-capable Ollama model after measuring tool accuracy, response time, and VRAM use on the RTX 3080.
- If UFW, process, or journal data is unavailable to the service user, review the exact missing data before considering any narrowly scoped privileged helper.
