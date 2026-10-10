# Guide spec: OpenClaw on OpenShift

Status: approved design (2026-10-08). Run: `2026-10-08-create`. Format: Antora with Agent Starter Kits branding. Rigor: standard. No internal part. Why card added and approved by the author on 2026-10-10 (cc-guide FR-043).

## Why

The why card for the start page and the section overview pages (cc-guide `overview-contract.md`). Written for a reader who knows neither OpenClaw nor OpenShell.

| Slot | Content | Sources |
|---|---|---|
| Problem | An AI agent that acts for you holds credentials, reads mail, and runs commands on real computers, so where it runs and what it may reach matters. Running one on your own platform usually means stitching the pieces together yourself. | `openclaw:docs/start/why-openclaw.md:11-13`, `starter-kits:README.md:11` |
| Outcome | An OpenClaw agent on OpenShift that you chat with in the browser, answering with a model you serve yourself through vLLM, deployed with namespace-scoped access and no cluster-admin rights. | `starter-kits:agents/openclaw/deployment/README.md:12,17-20` |
| Components: OpenClaw | An open-source AI agent that acts on your behalf. Its state, memory, and credentials stay on your own infrastructure, and the model behind it is a plugin you can swap. In this recipe you reach it through its web Control UI. | `openclaw:README.md:18,20`, `openclaw:docs/start/why-openclaw.md:11` |
| Components: OpenShell | A runtime that runs AI agents in isolated sandboxes. A declared policy decides which files, system calls, and network destinations an agent may use, and the agent never sees real credentials. | `openshell:README.md:19,25`, `openshell:docs/about/overview.mdx:11,21` |
| Components: vLLM | A library for running large language models and serving them through an OpenAI-compatible API; the recipe also accepts KServe or an external API as the endpoint. | `url:https://docs.vllm.ai/en/latest/` (unpinned, fetched 2026-10-10), `starter-kits:agents/openclaw/deployment/README.md:12,19` |
| Fit | The OpenClaw gateway runs as a single-replica Deployment, serves the Control UI behind a Route, and calls your model endpoint. Alternatively, the same OpenClaw runs inside an OpenShell sandbox on the cluster, built from the recipe's `Containerfile.openshell`. | `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml`, `starter-kits:agents/openclaw/deployment/manifests/02-configmap.yaml`, `starter-kits:agents/openclaw/deployment/Containerfile.openshell` |
| Why this combination | Your prompts go only to the model provider you configure, and here that is your own model server. OpenClaw's own sandboxing is off by default; OpenShell adds policy-enforced isolation around the agent. | `openclaw:README.md:20`, `openclaw:docs/start/why-openclaw.md:29,51`, `openshell:README.md:19,25` |
| Status and limits | The OpenClaw image is a community build, not built, scanned, or validated by Red Hat. The sandbox path is meant for experimentation. The gateway runs as one replica with no high availability. | `starter-kits:agents/openclaw/deployment/README.md:5-10`, `starter-kits:agents/openclaw/templates/openclaw_agent/README.md:21`, `starter-kits:agents/openclaw/deployment/docs/installer-deployment.md:438`, `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml:8-10` |

Upstream candidate: the recipe's agent description calls OpenClaw an "AI coding assistant" (`starter-kits:agents/openclaw/templates/openclaw_agent/README.md:5`), while OpenClaw's own README describes a general AI assistant (`openclaw:README.md:18`). The guide follows OpenClaw's own description.

## Purpose

The Agentic Starter Kits ship an OpenClaw recipe (`agents/openclaw/`) that runs the OpenClaw gateway on OpenShift, optionally with MLflow tracing, and an OpenShell sandbox image for the same agent (`Containerfile.openshell` on the shared `sandboxes/base` image). The recipe's own documentation is spread over a README and five Markdown files, several of which disagree with the manifests and with the upstream projects they depend on.

This guide gives each reader one path from zero to a working result, with every step verified against pinned sources and, where the sources leave a gap or contradict each other, against a live cluster. Where the recipe documentation is wrong, the guide states what the code does, and the difference becomes an upstream finding.

Success means:

- An administrator deploys OpenClaw with the raw manifests, reaches the Control UI through a port-forward or the Route, changes the configuration, enables MLflow tracing, upgrades, and removes the deployment, using only the guide.
- A user connects to a running deployment, pairs a browser, and holds a first conversation, knowing which features the recipe switches off.
- A developer builds the sandbox image, runs OpenClaw in an OpenShell 0.1.2 sandbox on OpenShift with a working model configuration, and knows where the recipe can be extended.
- Every factual statement traces to a pinned source location or to a recorded live check.

## Audience and personas

| Persona | Who | Entry page | Section voice |
|---|---|---|---|
| Administrator | Platform or namespace admin who deploys and operates OpenClaw on OpenShift; comfortable with `oc` and Kustomize | `admin/index` | technical |
| User | Person who chats with OpenClaw through the Control UI; no cluster access needed | `user/index` | tutorial |
| Developer | Engineer who builds the sandbox image, runs OpenClaw in OpenShell, or extends the recipe | `developer/index` | technical |

Overview pages use the conversational voice; Reference pages use the reference voice (cc-prose profiles).

## Sources and baseline

| Id | Repository | Pin | Role |
|---|---|---|---|
| `starter-kits` | red-hat-data-services/agentic-starter-kits | d25f4c7 (head of PR #373) | Primary: manifests, overlays, docs, sandbox image |
| `openclaw` | OpenClaw, fetched from the fork `rhuss/openclaw` | fc23bc8 (2026.9.8) | Gateway behavior: entrypoint, doctor, config patch, origins, pairing, health, OpenTelemetry, Control UI |
| `openshell` | NVIDIA/OpenShell | v0.1.2 | Sandbox CLI, policy schema, Helm install on OpenShift |
| `claw-installer` | sallyom/claw-installer | fdcd761 (main) | The installer path |

PR #373 updates the recipe to OpenClaw 2026.9.8 and OpenShell 0.1.2 and fixes the Route. The guide is pinned to its head and gets re-pinned to `main` with `/guide:evolve` after the merge; no content is labeled experimental, because the whole guide describes the PR state.

The `openclaw` source is fetched from a fork because git access to github.com/openclaw/openclaw is blocked on the authoring machine. The commit is identical to the upstream tag, but the `{openclaw-blob}` and `{openclaw-tree}` attributes point to the fork. Pages therefore link to OpenClaw through `https://docs.openclaw.ai/` or plain `https://github.com/openclaw/openclaw` URLs, never through those two attributes (ruling R-FORK).

## Tested environment

- Gateway, MLflow, and OpenShell: OpenShift 4.22.15 (cluster `cntbh`, registered in `guide.yaml` as disposable), RHOAI 3.3.1 with the MLflow operator, OpenShell Helm chart 0.1.2 with Agent Sandbox v1.0.5.
- Model: vLLM (Red Hat AI Inference Server, CUDA image) serving gpt-oss-20b on a ROSA HCP 4.22.16 cluster with one `g5.xlarge` GPU node, reached through a Route with an API key. This replaces the CPU test model (Qwen2.5-0.5B-Instruct), which only finishes OpenClaw turns with test-only settings. Pages that show model output describe gpt-oss-20b behavior; the reference page keeps the recipe's own tested line.

## Live verification results (2026-10-08)

These results override the recipe documentation where they differ. Details and evidence are in `.guide/runs/2026-10-08-create/research.md` (local).

| Item | Result | Consequence for pages |
|---|---|---|
| D04 Route | Before PR #373's fix the Route answered 403 `proxy_attribution_required`. With `set-forwarded-headers: never` (now in the manifests) plus `gateway.controlUi.allowedOrigins`, the Control UI works and stayed connected. Putting the cluster network into `gateway.trustedProxies` breaks the probes. | `admin/access` documents both ways in; troubleshooting covers the 403 |
| D05 API key | `VLLM_API_KEY` in the environment overrides any `apiKey` in the config. | `admin/configure-model` puts the key in the Secret |
| D10 parser | The tool-call parser depends on the model family (`hermes` for Qwen2.5, `openai` for gpt-oss). | `admin/prerequisites`, `reference/model-compatibility` |
| D28 tracing after restart | Since 2026.9.6 restarts use SIGUSR2; tracing continues after an in-process restart. SIGUSR1 opens the Node inspector. | `admin/tracing` drops the old gap |
| D29 metrics | The overlay enables metrics, but the collector has no metrics pipeline (404, silent). | `admin/tracing` notes it |
| D31 default command | The image default command starts the gateway; `args` is kept so doctor runs. | none (fixed in PR #373) |
| D32 overlay plugins | The tracing overlay's `plugins.allow` limits loaded plugins (node onboarding join codes unavailable). | `admin/tracing`, `reference/configuration` |
| D33 approval | `devices approve --latest` only previews; approval needs the exact request ID. | `admin/access`, `user/connect` |
| Origin removal | Removing `allowedOrigins` with `config patch` takes effect only after a restart. | `admin/change-config` note |

## Persona journeys

**Administrator.** Reads `overview/choose-a-path`, checks `admin/prerequisites`, prepares the model settings and Secret (`admin/configure-model`), deploys (`admin/deploy`), opens the Control UI (`admin/access`), learns how configuration changes reach a running gateway (`admin/change-config`), optionally enables tracing (`admin/tracing`), and keeps the deployment healthy over time (`admin/maintain`). Two optional branches: `admin/install-openshell` for clusters that will run sandboxes, and `admin/deploy-installer` for teams that prefer claw-installer. Troubleshooting and the manifest reference support every step.

**User.** Starts at `user/index`, gets the address and token from the administrator, connects and pairs (`user/connect`), holds a first conversation (`user/first-chat`), and learns what the recipe switches off and how to ask for changes (`user/whats-enabled`).

**Developer.** Starts at `developer/index`, understands the image layers (`developer/sandbox-image`), builds and pushes them (`developer/build-image`), runs OpenClaw in a sandbox (`developer/run-in-sandbox`), gives it a model configuration (`developer/sandbox-model-config`), and extends the recipe (`developer/extend`). Needs an OpenShell gateway, which `admin/install-openshell` sets up.

## Page list

Paths are relative to `content/modules/ROOT/pages/`.

### Overview (conversational)

**`index`: OpenClaw on OpenShift.** Follows the start page shape of `overview-contract.md` and carries the `## Why` card (revised 2026-10-10): the outcome and the problem first; one definition each for OpenClaw, OpenShell, and vLLM from their own sources; how they fit and why together, with diagram 4; the three ways to run it (raw manifests, claw-installer, OpenShell sandbox) in one or two sentences each with a link to `overview/choose-a-path`; status and limits (community image, sandbox for experimentation, single replica); the persona entry links; the baseline note. Repository layout, manifest lists, image pins, and overlays stay on `overview/architecture` and `overview/choose-a-path`.

**`overview/architecture`: How the deployment fits together.** What `oc apply -k manifests/` creates and how a request flows: Route (edge TLS) to Service to the gateway container; the init container that seeds `openclaw.json` once and moves old state; the PVC mounted at `/home/node` with state in `~/.openclaw`; the Secret and ConfigMap; the model endpoint; the optional OpenTelemetry collector sidecar sending traces to RHOAI MLflow. Explains why the PVC sits at `/home/node` and why the gateway starts through `args`. Shows diagram 1.

**`overview/choose-a-path`: Choose how to run OpenClaw.** Compares raw manifests, claw-installer, and the OpenShell sandbox on access model (token and device pairing vs OpenShift OAuth proxy vs `forward start`), image pinning (digest vs `:latest`), upgrade behavior (doctor runs vs skipped), who needs cluster-admin (MLflow ClusterRole on RHOAI 3.3.1, OpenShell CRDs and Helm chart), and maturity. Lists known limits: single replica (SQLite), no hardening beyond the restricted SCC (no NetworkPolicy, no read-only root filesystem, manual token rotation).

### Administrator guide (technical)

**`admin/index`: Administrator guide.** The administrator journey as an ordered list with one sentence per page, the decisions to make up front (namespace, storage class, model endpoint, Route or port-forward, tracing), and where the optional branches start.

**`admin/prerequisites`: Check the prerequisites.** OpenShift 4.17 or later with namespace access; a block storage class (not NFS, SQLite needs POSIX locking); an OpenAI-compatible endpoint with tool calling, with the vLLM flags `--enable-auto-tool-choice` and a `--tool-call-parser` that matches the model family; which optional parts need cluster-admin. Verification commands and their expected output.

**`admin/configure-model`: Configure the model endpoint and secrets.** Find the model ID with `/v1/models`; the naming rule (`models[].id` is the endpoint's ID, `agents.defaults.model` is `<provider>/<id>`, which gives `vllm/vllm/<model>` behind OGX); edit the ConfigMap or an overlay; set `OPENCLAW_GATEWAY_TOKEN` and `VLLM_API_KEY` in the Secret, and why the Secret's key wins over the ConfigMap's `apiKey`; the storage class patch.

**`admin/deploy`: Deploy OpenClaw.** Create the namespace, apply the manifests or an overlay, watch the init container and the gateway start, and check `[gateway] ready` and the active model in the log. Expected pod state and the startup budget of the startup probe.

**`admin/access`: Open the Control UI.** Port-forward on local port 18789 (local pairing is approved automatically). The Route: what the `set-forwarded-headers: never` annotation does, adding the Route to `gateway.controlUi.allowedOrigins` with `config patch`, approving the browser by request ID (`devices list`, `devices approve <id>`; `--latest` only previews), and the warning not to use `gateway.trustedProxies` for the router. Security note: the Route makes the gateway reachable for anyone who can reach the router; the token and device approval protect it.

**`admin/change-config`: Change the configuration of a running gateway.** Why the ConfigMap only seeds the first start; `config patch --stdin` with merge rules (objects merge, arrays and scalars replace, `null` deletes); the restart hints OpenClaw prints; keeping the ConfigMap in sync for new PVCs; the gotcha that removals take effect only after a restart; the write-safety behavior (rejected writes, last-known-good). Shows diagram 2.

**`admin/tracing`: Send traces to MLflow.** Prerequisites (RHOAI MLflow with workspaces; the `mlflow-integration` ClusterRole, which RHOAI 3.3.1 lacks and the admin creates), the tracing ServiceAccount and RoleBinding, creating the experiment through the MLflow API, filling the overlay placeholders, applying it, and applying the tracing config to an existing deployment with `config patch`. Verification in the MLflow UI. Known issues: experiment 404, the harmless "error parsing protobuf response", HTTP 400 with gzip, metrics going nowhere, and the overlay's narrow `plugins.allow`. The span schema in short.

**`admin/maintain`: Back up, upgrade, and remove.** Backup with `tar` through `oc exec` (and the old layout variant); upgrade by applying new manifests: the init container's one-time state move, `doctor --fix` with its database backup, the 5-minute startup budget, and the log lines to check; removal with `oc delete -k` and what deleting the PVC destroys. Restore is not covered by any source and is listed as a known gap.

**`admin/install-openshell`: Install OpenShell for sandboxes.** Optional branch. Install the Agent Sandbox controller and CRDs, the OpenShell 0.1.2 Helm chart on OpenShift (no privileged SCC), wait for the gateway StatefulSet, register it with the CLI (port-forward, `openshell gateway add`, TLS client bundle), check with `openshell status`. Caveats: the RHCOS 5.14 kernel runs sandboxes in legacy read-only mode; 0.0.x sandboxes must be recreated. Verified live on cntbh.

**`admin/deploy-installer`: Deploy with claw-installer.** Optional branch, written from the installer code: run `./run.sh` (podman) or the Node.js dev flow, the OpenShift provider plugin after `oc login`, the form fields that matter (image, model endpoint, "Use image entrypoint"), what gets created (OAuth proxy sidecar, Route to `oauth-ui`, PVC `openclaw-home-pvc`), the Instances tab, and pairing after SSO. A caution box lists the differences from the raw manifests: `:latest` images, the config copied on every start, the replaced gateway command that skips doctor unless "Use image entrypoint" is set, the gateway token stored in the ConfigMap, and probes on `/`.

### User guide (tutorial)

**`user/index`: User guide.** What a user can do with this deployment (chat with an agent that reads and writes files in its workspace and runs the tools the recipe allows), what to ask the administrator for (the address, the gateway token, approval of the browser), and the order of the next pages.

**`user/connect`: Connect and pair your browser.** Open the address, enter the gateway token in "Gateway secret", wait for the administrator to approve the device (or automatic approval on a local port-forward), and what each state on the login screen means. Pairing again after clearing site data or using a new browser profile; the rate limiter after repeated stale tokens.

**`user/first-chat`: Hold your first conversation.** Send a message, read the answer, start a new session, find earlier sessions, and see which model answers. What to expect from the tested model (gpt-oss-20b) with tools, and how a reply with raw JSON or a stalled turn looks when the model does not handle tool calls.

**`user/whats-enabled`: What this deployment can and cannot do.** The recipe's configuration in user terms: no web search or browser tools, no messaging channels, sandbox mode off, heartbeat as configured; why each is off; and what to ask the administrator for when a feature is needed.

### Developer guide (technical)

**`developer/index`: Developer guide.** The developer journey: from the image layers to a running sandbox and on to extending the recipe; what you need (podman, a registry, the `openshell` CLI, an OpenShell gateway).

**`developer/sandbox-image`: How the sandbox image is built.** The shared base (UBI 10 minimal, the `sandbox` user, `/sandbox` and `/workspace`, the entrypoint that picks the agent, the image policy in schema version 1 and why an invalid policy blocks start), and the OpenClaw layer (`Containerfile.openshell`: official Node.js 24 build with checksum verification, because OpenClaw refuses the UBI Node.js packages that link SQLite 3.46.1; the verify step).

**`developer/build-image`: Build and push the images.** Build with podman (`--platform linux/amd64` from Apple Silicon), push to a registry the cluster can pull from, and build the base image yourself instead of using the development image in a personal registry.

**`developer/run-in-sandbox`: Run OpenClaw in an OpenShell sandbox.** `openshell sandbox create` with the gateway as the main process, `openshell forward start` to reach it, the egress rule for the model endpoint bound to `/usr/local/bin/node`, and the OpenShift specifics (UID from the namespace range, `/sandbox` as working directory, `/workspace` not writable). Shows diagram 3.

**`developer/sandbox-model-config`: Configure the model inside the sandbox.** How to give OpenClaw in the sandbox a provider configuration and API key, and how to keep it across sandbox restarts. No pinned source covers this; the page task works it out live on cntbh with the OpenShell 0.1.2 CLI and OpenClaw's config commands, records the commands and results, and states only what the live run shows. If no workable method exists, the page shrinks to what the sources support and the gap is recorded.

**`developer/extend`: Extend the recipe.** Kustomize overlays (namespace, storage class, resources), the agent template `templates/openclaw_agent/agent.yaml` as far as the sources describe it, and OpenClaw extension points (workspace files, skills, plugins) that work with the recipe's configuration.

### Reference (reference)

**`reference/manifests`: Manifest reference.** Every resource of `manifests/` and the tracing overlay with its key fields: images by digest, env, probes and budgets, resources, security context, volumes, labels, Route annotations.

**`reference/configuration`: Configuration reference.** The `openclaw.json` keys the recipe sets in the base ConfigMap and the tracing overlay, each with meaning, effect, and whether a change hot-applies or restarts the gateway.

**`reference/model-compatibility`: Model compatibility.** The recipe's tested-model table with its date, the criterion (structured tool calls), the tool-call parser per model family, provider prefixes and API types, and the models of the tested lines (recipe and guide).

**`reference/troubleshooting`: Troubleshooting.** The recipe's entries corrected against the code (configuration changes through `config patch` instead of ConfigMap edits, the CrashLoopBackOff entry without the stale `/data/.openclaw` path, `devices clear --yes`, `agents.entries`), the Route 403, `EPERM fchmod`, pairing and rate limiting, and the diagnostic commands.

## Diagrams

| # | Name | Shows | Used on |
|---|---|---|---|
| 1 | `deployment-architecture` | Route, Service, Pod with init container, gateway, and optional collector; PVC, ConfigMap, Secret; model endpoint; RHOAI MLflow | `overview/architecture` |
| 2 | `config-state-lifecycle` | ConfigMap seed on first start, the PVC copy as source of truth, `config patch` through `oc exec`, the upgrade path through the init move and doctor | `admin/change-config` |
| 3 | `sandbox-flow` | Workstation build and push, registry, OpenShell gateway, sandbox pod (supervisor, gateway on loopback, image policy), egress rule, `forward start` | `developer/run-in-sandbox` |
| 4 | `big-picture` | The reader's browser, the OpenShift cluster with the OpenClaw gateway (Deployment path) and, alternatively, OpenClaw inside an OpenShell sandbox whose requests the OpenShell supervisor (own pod) checks against policy, and the model endpoint both call (vLLM, KServe, or an external API; the recipe does not deploy it); no Kubernetes object detail | `index` |

## Experimental content

None. No change request is tracked separately; the baseline itself is the PR head.

## Internal part

None (`internal.enabled: false`).

## Upstream findings to collect

The research notes list 31 discrepancies (D01 to D31), the new items D32 and D33, eight stale references, and the origin removal behavior. Page tasks report the ones they touch as Upstream candidates; the wrap-up files them with `runs.py findings-add` for a later `/guide:upstream`.

## Open questions and known gaps

- Restore from a backup and rollback after a doctor migration: no source covers it; `admin/maintain` states only the backup.
- Getting a vLLM endpoint on RHOAI: out of scope; `admin/prerequisites` states what the endpoint must provide.
- Hardening (NetworkPolicy, token rotation, read-only root filesystem): out of scope; listed as a limit on `overview/choose-a-path`.
- Upgrading installer-based deployments: only the differences are documented on `admin/deploy-installer`.
- The sandbox model configuration (`developer/sandbox-model-config`) depends on a live investigation; its scope is decided by that task.
