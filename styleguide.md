# OpenClaw on OpenShift style guide

Project terms and rules that every page follows, on top of the cc-prose voice profiles. Page implementers and reviewers read this file. Keep one rule per bullet.

## Terms

- "Agentic Starter Kits" is the repository (red-hat-data-services/agentic-starter-kits); "Agent Starter Kits" is the guide's brand name on the site header.
- "Recipe" means the OpenClaw deployment in the starter kits (`agents/openclaw/`). It is a domain term; prose-check hits for this sense are false positives.
- "Sandbox" means an OpenShell sandbox. "Gateway" means the OpenClaw gateway process, unless a page says "OpenShell gateway".
- Product names are spelled exactly: OpenClaw, OpenShell, OpenShift, Red Hat OpenShift AI (RHOAI after first use on a page), MLflow, vLLM, Red Hat AI Inference Server, OpenTelemetry, Node.js, Kustomize, claw-installer.
- Persona names: "Administrator", "User", and "Developer".
- Capitalize Kubernetes kinds: Pod, ConfigMap, Secret, Service, Namespace, Deployment, PersistentVolumeClaim, Route.

## Rules

- No em dash or en dash characters.
- AsciiDoc pages: one sentence per line.
- Versions and commits come from the baseline attributes, never typed.
- Task pages: a short paragraph on why, then numbered steps, then a `Check:` line with the expected result.
- Use `oc` for cluster commands and `podman` for container commands.
