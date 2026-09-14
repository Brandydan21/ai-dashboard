# Repository Instructions for Coding Agents

## Before making changes

Read `README.md`, `REQUIREMENTS.md`, and `ARCHITECTURE.md` completely. State which feature task you are implementing and briefly explain any important architectural choice before writing substantial code.

Implement only the feature the user selected and the minimum foundation it requires. Do not begin later features, add speculative abstractions, or expand V1 scope. If a requirement and architecture rule conflict, stop and explain the conflict before changing either document.

## Working style

- Keep modules small and responsibilities clear.
- Use Python type hints and Pydantic models at backend boundaries.
- Keep HTTP routes thin; place application logic in services and system access under `backend/app/system/`.
- Add dependencies only when the selected feature needs them. Pin and document them in the appropriate dependency file.
- Prefer direct code over large agent, monitoring, or orchestration frameworks.
- Preserve unavailable and partial states. Never substitute zero or an empty list for a collection failure.
- Do not create directories or placeholder code for future features.

## Non-negotiable security rules

- Never give the LLM a generic shell-command, code-execution, file-access, or system-modification tool.
- Never execute a command string produced by the LLM or an API caller.
- Only `backend/app/system/` may invoke operating-system commands.
- Every command must be a predefined read-only operation with fixed executable and argument structure, `shell=False`, strict input validation, timeout, and output limit.
- Never add service restart/control, container control, firewall modification, package installation, file modification, or sudo operations to the agent tool registry.
- The backend must run without root privileges. If data needs extra permissions, return a clear unavailable state and document the gap before proposing a narrow helper.
- Treat logs, process names, service metadata, container data, and model output as untrusted text. Escape output and redact sensitive values before sending it to the model or browser.
- Keep Ollama local. Do not add cloud inference, telemetry containing server data, or automatic model downloads.
- Do not describe a listener as publicly reachable or private without supporting network and firewall evidence. Return uncertainty when evidence is incomplete.

## Testing expectations

Test behavior that can fail or create a security boundary. In particular:

- Keep command-output parsing in pure functions and test it with sanitized Ubuntu fixtures.
- Cover valid, empty, malformed, partial, unavailable, permission-denied, timeout, IPv4, and IPv6 cases when relevant to the feature.
- Verify invalid inputs and unregistered agent tools are rejected before system execution.
- Use fake collectors and a fake Ollama client in normal tests. Tests must not require root, Ubuntu, NVIDIA hardware, Podman, or a running model.
- Run the relevant backend tests and frontend type/build tests before declaring a feature complete.
- Perform and document a read-only smoke check on the Ubuntu server when the selected feature reaches that stage.

Do not add tests that merely repeat trivial implementation details. Focus on parsers, public behavior, failure handling, and security controls.

## Completing a feature

A feature is complete only when its definition of done in `REQUIREMENTS.md` is met. Report:

- What changed and why
- How it was tested
- Any unavailable data, security limitation, or target-server check still needed
- The next feature in the requirements list, without starting it

Do not deploy, change the server, or proceed to another feature unless the user asks.

## Git and review workflow

- Start each feature from an up-to-date `main` branch and create a dedicated `feature/<short-name>` branch.
- Keep each merge request limited to one feature from `REQUIREMENTS.md`.
- Commit the completed feature only after its relevant checks pass.
- Push the feature branch and create a merge request for user review. Do not merge it.
- Write a complete merge request description covering the problem, implemented behavior, important architecture or security decisions, files or areas changed, tests run, manual verification, and known limitations.
- Wait for the user to review and merge the request before starting the next feature unless the user explicitly directs otherwise.
