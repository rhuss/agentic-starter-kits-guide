# OpenClaw on OpenShift Guide Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Write the 27 pages and 3 diagrams of the Antora guide "OpenClaw on OpenShift", each verified against the pinned sources.

**Architecture:** Task 1 creates every page as a stub and the full navigation, so cross-references resolve from the start. Each later task replaces one stub (or draws one diagram) following `page-procedure.md` or the `guide:diagrams` skill; diagram tasks precede the pages that show them.

**Tech Stack:** Antora 3.2 (AsciiDoc), cc-guide toolkit (`make check`, `tools/guide/*.py`), Excalidraw MCP for diagrams, `oc` against cluster cntbh for live checks.

**Spec:** `docs/guide/spec.md` (research and live results: `.guide/runs/2026-10-08-create/research.md`, local only).

## Global Constraints

- Guide: "OpenClaw on OpenShift", format `antora`, rigor `standard`, cc-prose present (present: write with the section voice profiles and run the prose check; absent: follow `style-rules.md`).
- Baseline (captured 2026-10-08); every statement is verified at these commits, never at a branch head:

  | Source id | Repository | Commit | Ref |
  |---|---|---|---|
  | `starter-kits` | red-hat-data-services/agentic-starter-kits | d25f4c7ed617519db99ee1c5b3c01e564931cf03 | head of PR #373 |
  | `openclaw` | git@github.com:rhuss/openclaw.git (fork of openclaw/openclaw) | fc23bc864e4553c2d215e479eeec47b67a0bf943 | v2026.9.8 |
  | `openshell` | NVIDIA/OpenShell | 6648bd0c290efbc41ba131ee9831ee45cd431f94 | v0.1.2 |
  | `claw-installer` | sallyom/claw-installer | fdcd761faee6403d22db66bb77ddaa05e9c30d2e | main |

- Baseline values available to pages (never type versions or commits): `{starter-kits-repo}`, `{starter-kits-commit}`, `{starter-kits-commit-short}`, `{starter-kits-blob}`, `{starter-kits-tree}`, `{openclaw-repo}`, `{openclaw-commit}`, `{openclaw-commit-short}`, `{openclaw-version}` (2026.9.8), `{openshell-repo}`, `{openshell-commit}`, `{openshell-commit-short}`, `{openshell-blob}`, `{openshell-tree}`, `{openshell-version}` (0.1.2), `{claw-installer-repo}`, `{claw-installer-commit}`, `{claw-installer-commit-short}`, `{claw-installer-blob}`, `{claw-installer-tree}`, `{baseline-date}`, `{baseline-sources}`. Never use `{openclaw-blob}` or `{openclaw-tree}` (they point to a fork); link OpenClaw through `https://docs.openclaw.ai/` or plain `https://github.com/openclaw/openclaw` URLs.
- Change requests documented as experimental (label only through the adapter snippet):
  none (the baseline itself is the PR #373 head; nothing is labeled experimental).
- Personas and their entry pages: `admin` (Administrator) `admin/index`, `user` (User) `user/index`, `developer` (Developer) `developer/index`.
- Sections and voices: Overview `conversational`, Administrator guide `technical`, User guide `tutorial`, Developer guide `technical`, Reference `reference`.
- Every page starts with a source block after the title; entries are `<source-id>:<path>[:<lines>]`, `url:<url>`, or (internal pages only) `input:<input-id>`. Facts without a pinned source stay out of the page.
- Inputs are hints, never proof. Registered inputs: none. Content derived from a confidential input (none) appears only on internal pages.
- Shareable pages never contain: (no confidential terms), text matching (no confidential patterns), or links into the internal part (`internal/`, not enabled).
- No em dash or en dash characters in any file.
- AsciiDoc, one sentence per line; source block as `// Sources:` lines; expectations as `// expect:`; baseline values as `{<name>}` (listings need `subs="attributes+"` and no `${VAR}`); experimental labels through `ROOT:partial$experimental-<tier>.adoc` with the anchor on the section heading; images `image::<name>.svg["<alt>",role=expand]` with quoted alt text; pages under `content/modules/ROOT/pages/`, internal pages under `internal/modules/internal/pages/`.
- Admonitions: only NOTE, TIP, IMPORTANT, WARNING, CAUTION, with their fixed meanings (see `page-procedure.md`).
- Task pages: a why-paragraph, numbered steps with copy-paste commands, `Check:` lines with expectations.
- Diagrams follow the diagrams skill; palette: platform gray (`#e9ecef`/`#495057`), governance orange (`#ffe8cc`/`#e8590c`), admin red (`#ffe3e3`/`#c92a2a`), user blue (`#d0ebff`/`#1971c2`); text and legends `#495057`.
- Run every command from the guide project root with path arguments; never `cd` into `.guide/cache/`.
- Containers: `podman`, never `docker`. Python packages: `uv`, never pip.
- Commit messages end with the trailer `Assisted-By: 🤖 Claude Code`. Only the controller pushes, and only when the author opted in (local commits only).
- Navigation (`content/modules/ROOT/nav.adoc`) has five titled lists in this order: `.Overview`, `.Administrator guide`, `.User guide`, `.Developer guide`, `.Reference`. Task 1 creates all entries; page tasks only fix an entry's link text when the page title changes.
- Commands that run OpenClaw's CLI inside the pod use the recipe's form `oc exec [-i] deployment/openclaw -c gateway -n my-openclaw -- node /app/dist/index.js <command>`. Examples use the namespace `my-openclaw`, as the recipe docs do. Never put a real token, password, API key, or cluster host name on a page; use placeholders such as `<gateway-token>` and `<route-host>`.
- Live checks: cluster `cntbh` is the current `oc` context and registered as disposable. Implementers may run read-only commands (`oc get`, `oc describe`, `oc logs`, `oc exec ... config get`, `devices list`, `helm list`, `openshell status`) in the namespaces `openclaw-fresh`, `openclaw-test`, `openclaw-model`, `openshell-test`, `openshell-agents`, and `redhat-ods-applications`; mutating commands only where a task says so. The OpenShell CLI always targets the cluster gateway with `-g openshell-test`; never change the CLI's active gateway. A live result can support a page statement only together with a pinned source for the command or setting it uses; record live runs in the task report.
- The live verification table in `docs/guide/spec.md` ("Live verification results") is binding: where it contradicts a recipe document, pages follow the table and cite the code location; the task report lists the contradicted document as an Upstream candidate.
- Tested environment statements: the recipe's own tested lines come from `starter-kits` docs; the guide's model line (gpt-oss-20b on vLLM with a GPU) is stated only on `reference/model-compatibility` and `user/first-chat`.

## Review Focus

1. A reader whose cluster has no `gp3-csi` storage class (Azure, vSphere, bare metal) expects to be told how to pick and set another block storage class before deploying; Task 8 pins it.
2. A reader upgrading a deployment made with manifests older than PR #373 (state at the root of the PVC) expects the backup command for the old layout and the `EPERM ... fchmod` symptom; Task 14 pins it.
3. A reader whose model endpoint needs an API key or sits behind OGX expects the `vllm/vllm/<model>` naming and the Secret key; Task 8 pins it.
4. A reader building the sandbox image on Apple Silicon expects `--platform linux/amd64`; Task 19 pins it.
5. A reader whose RHOAI lacks the `mlflow-integration` ClusterRole, or runs MLflow outside `redhat-ods-applications`, expects how to create the role and where to change the endpoint; Task 13 pins it.

---

### Task 1: Navigation and page skeleton

**Files:**
- Create: the 26 stub pages listed below under `content/modules/ROOT/pages/`
- Modify: `content/modules/ROOT/nav.adoc`, `guide-rules.yaml`

**Interfaces:**
- Produces: every page path of the guide, so later tasks can use `xref:<path>.adoc[]` to any page.

Persona: none. Internal: no.

Create each stub with exactly this content (title from the table):

```asciidoc
= <Title>
:guide-no-sources:

// STUB
This page is not written yet.
```

| Path | Title |
|---|---|
| `overview/architecture.adoc` | How the deployment fits together |
| `overview/choose-a-path.adoc` | Choose how to run OpenClaw |
| `admin/index.adoc` | Administrator guide |
| `admin/prerequisites.adoc` | Check the prerequisites |
| `admin/configure-model.adoc` | Configure the model endpoint and secrets |
| `admin/deploy.adoc` | Deploy OpenClaw |
| `admin/access.adoc` | Open the Control UI |
| `admin/change-config.adoc` | Change the configuration of a running gateway |
| `admin/tracing.adoc` | Send traces to MLflow |
| `admin/maintain.adoc` | Back up, upgrade, and remove |
| `admin/install-openshell.adoc` | Install OpenShell for sandboxes |
| `admin/deploy-installer.adoc` | Deploy with claw-installer |
| `user/index.adoc` | User guide |
| `user/connect.adoc` | Connect and pair your browser |
| `user/first-chat.adoc` | Hold your first conversation |
| `user/whats-enabled.adoc` | What this deployment can and cannot do |
| `developer/index.adoc` | Developer guide |
| `developer/sandbox-image.adoc` | How the sandbox image is built |
| `developer/build-image.adoc` | Build and push the images |
| `developer/run-in-sandbox.adoc` | Run OpenClaw in an OpenShell sandbox |
| `developer/sandbox-model-config.adoc` | Configure the model inside the sandbox |
| `developer/extend.adoc` | Extend the recipe |
| `reference/manifests.adoc` | Manifest reference |
| `reference/configuration.adoc` | Configuration reference |
| `reference/model-compatibility.adoc` | Model compatibility |
| `reference/troubleshooting.adoc` | Troubleshooting |

Replace `nav.adoc` with the five titled lists in the Global Constraints order. `.Overview` lists `index.adoc` ("Start here"), then the two overview pages; every other list holds its section's pages in the table order, entry page first.

Rules to add to `guide-rules.yaml` (global, they stay for the whole run):

```yaml
  - glob: "content/modules/ROOT/pages/**/*.adoc"
    forbid:
      - '\{openclaw-(blob|tree)\}'
      - '\bdocker (run|build|push|pull|login)\b'
      - '\bpip3? install\b'
  - glob: "content/modules/ROOT/pages/user/*.adoc"
    forbid:
      - '\boc (login|get|exec|rsh|port-forward|apply|annotate|delete)\b'
```

Check: `make check` passes with all stubs in place, and `build/site/guide/` contains one HTML file per page.

---

### Task 2: Diagram `deployment-architecture`

**Files:**
- Create: `diagrams/deployment-architecture.excalidraw`, `content/modules/ROOT/images/deployment-architecture.svg`

Shown on: `overview/architecture` (Task 4). Follow the `guide:diagrams` skill.

What it shows: what `oc apply -k manifests/` creates in one namespace and how traffic, configuration, state, and traces flow.

Entities and palette:
- User blue: Browser (Control UI).
- Platform gray: Route `openclaw` (edge TLS, forwarded headers off), Service `openclaw` (ClusterIP 18789), Pod `openclaw` with containers `init-config` (init), `gateway` (port 18789), `otel-collector` (optional, tracing overlay; draw dashed); PVC `openclaw-state` mounted at `/home/node`; ConfigMap `openclaw-config`; model endpoint (vLLM or OGX, outside the namespace).
- Governance orange: Secret `openclaw-secrets` (gateway token, `VLLM_API_KEY`); ServiceAccount `openclaw-tracing` with RoleBinding (optional, dashed).
- Admin red: RHOAI MLflow in `redhat-ods-applications` (port 8443, optional, dashed).

Arrows: Browser to Route to Service to gateway; ConfigMap to init-config ("seeds openclaw.json once"); init-config and gateway to PVC; Secret to gateway (env); gateway to model endpoint (OpenAI-compatible API); gateway to otel-collector (OTLP http/protobuf, localhost:4318); otel-collector to MLflow (`/v1/traces`, SA token).

Source hints: `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml`, `starter-kits:agents/openclaw/deployment/manifests/06-route.yaml`, `starter-kits:agents/openclaw/deployment/overlays/mlflow-tracing/deployment-patch.yaml`, `starter-kits:agents/openclaw/deployment/overlays/mlflow-tracing/otel-collector-config.yaml`.

---

### Task 3: `index` (Start here)

Page: `content/modules/ROOT/pages/index.adoc`. Section: Overview, voice `conversational`. Persona: all. Internal: no.

Outline: What the recipe is: the OpenClaw gateway as a single-replica Deployment on OpenShift with a model served by vLLM, plus an OpenShell sandbox image. The three ways to run it (raw manifests, claw-installer, OpenShell sandbox) in one paragraph each, the support statement (upstream community image, not built or validated by Red Hat), the persona entry links, and the baseline note. Remove the scaffold's placeholder text and `:guide-no-sources:`.

Facts to verify (source hints):
- Recipe scope and file layout: `starter-kits:agents/openclaw/deployment/README.md`, `starter-kits:agents/openclaw/templates/openclaw_agent/README.md`.
- Support statement: `starter-kits:agents/openclaw/deployment/README.md` (lines 5-10), `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml` (image warning).
- Raw manifests recommended over the installer: `starter-kits:agents/openclaw/deployment/docs/installer-deployment.md` (line 5).
- Sandbox variant: `starter-kits:agents/openclaw/deployment/Containerfile.openshell`, `starter-kits:sandboxes/base/README.md`.
- Versions only through `{openclaw-version}` and `{openshell-version}`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/index.adoc"
    require:
      - 'include::(ROOT:)?partial\$baseline-note\.adoc\[\]'
      - 'xref:admin/index\.adoc'
      - 'xref:user/index\.adoc'
      - 'xref:developer/index\.adoc'
      - 'xref:overview/choose-a-path\.adoc'
      - '\{openclaw-version\}'
    forbid:
      - 'guide-no-sources'
```

---

### Task 4: `overview/architecture`

Page: `content/modules/ROOT/pages/overview/architecture.adoc`. Section: Overview, voice `conversational`. Persona: all. Internal: no.

Outline: What `oc apply -k manifests/` creates and how a request flows: Route (edge TLS, forwarded headers off) to Service to the gateway container; the init container that seeds `openclaw.json` once and moves old state; the PVC mounted at `/home/node` with state in `~/.openclaw`; the Secret and ConfigMap; the model endpoint; the optional OpenTelemetry collector sidecar sending traces to RHOAI MLflow. Why the PVC sits at `/home/node` (OpenClaw tightens its state directory, which must not be a root-owned mount point) and why the gateway starts through `args` (the image entrypoint runs `doctor --fix` first). Shows diagram `deployment-architecture`.

Facts to verify (source hints):
- Deployment shape, Recreate strategy, security context, init script, args, env, resources, probes: `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml`.
- Service and Route: `starter-kits:agents/openclaw/deployment/manifests/05-service.yaml`, `starter-kits:agents/openclaw/deployment/manifests/06-route.yaml`.
- PVC: `starter-kits:agents/openclaw/deployment/manifests/03-pvc.yaml`.
- Entrypoint runs doctor before a foreground gateway: `openclaw:docker-entrypoint.mjs`, `openclaw:gateway-run-argv.mjs`, `openclaw:docs/install/docker.md`.
- Tracing sidecar: `starter-kits:agents/openclaw/deployment/overlays/mlflow-tracing/deployment-patch.yaml`, `starter-kits:agents/openclaw/deployment/docs/mlflow-tracing.md`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/overview/architecture.adoc"
    require:
      - 'image::deployment-architecture\.svg\['
      - '/home/node'
      - 'doctor --fix'
      - 'xref:admin/deploy\.adoc'
```

---

### Task 5: `overview/choose-a-path`

Page: `content/modules/ROOT/pages/overview/choose-a-path.adoc`. Section: Overview, voice `conversational`. Persona: all. Internal: no.

Outline: Compares raw manifests, claw-installer, and the OpenShell sandbox on access model (gateway token and device pairing vs OpenShift OAuth proxy vs `openshell forward start`), image pinning (digest vs `:latest`), upgrade behavior (doctor runs vs skipped unless "Use image entrypoint"), who needs cluster-admin (the `mlflow-integration` ClusterRole on RHOAI 3.3.1, OpenShell CRDs and Helm chart), and maturity. Lists known limits: single replica (SQLite), no NetworkPolicy, no read-only root filesystem, manual token rotation. Ends with links to the three starting pages.

Facts to verify (source hints):
- Raw manifests: `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml`, `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md`.
- Installer: `claw-installer:provider-plugins/openshift/src/openshift-deployer.ts`, `claw-installer:provider-plugins/openshift/templates/oauth-proxy-container.yaml`, `claw-installer:src/server/deployers/k8s-helpers.ts` (image defaults), `claw-installer:src/server/deployers/k8s-manifests.ts` (init copy, probes).
- Sandbox: `starter-kits:agents/openclaw/deployment/README.md` (OpenShell section), `openshell:docs/kubernetes/openshift.mdx`.
- Cluster-admin needs: `starter-kits:agents/openclaw/deployment/docs/mlflow-tracing.md` (prerequisites), `openshell:docs/kubernetes/setup.mdx`.
- Single replica: `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml`, `starter-kits:agents/openclaw/deployment/docs/installer-deployment.md` (lines 434-442).
- `readOnlyRootFilesystem` upstream vs recipe: `openclaw:scripts/k8s/manifests/deployment.yaml`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/overview/choose-a-path.adoc"
    require:
      - 'xref:admin/deploy\.adoc'
      - 'xref:admin/deploy-installer\.adoc'
      - 'xref:developer/run-in-sandbox\.adoc'
      - '(?i)single replica|one replica'
```

---

### Task 6: `admin/index`

Page: `content/modules/ROOT/pages/admin/index.adoc`. Section: Administrator guide, voice `technical`. Persona: admin (entry page). Internal: no.

Outline: The administrator journey as an ordered list with one sentence per page (prerequisites, configure-model, deploy, access, change-config, tracing, maintain), the decisions to make up front (namespace, storage class, model endpoint, Route or port-forward, tracing), and the two optional branches (install-openshell, deploy-installer). Links to troubleshooting and the manifest reference.

Facts to verify (source hints): `starter-kits:agents/openclaw/deployment/README.md` (customization table), `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/index.adoc"
    require_order:
      - 'xref:admin/prerequisites\.adoc'
      - 'xref:admin/configure-model\.adoc'
      - 'xref:admin/deploy\.adoc'
      - 'xref:admin/access\.adoc'
      - 'xref:admin/change-config\.adoc'
      - 'xref:admin/tracing\.adoc'
      - 'xref:admin/maintain\.adoc'
    require:
      - 'xref:admin/install-openshell\.adoc'
      - 'xref:admin/deploy-installer\.adoc'
      - 'xref:reference/troubleshooting\.adoc'
```

---

### Task 7: `admin/prerequisites`

Page: `content/modules/ROOT/pages/admin/prerequisites.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: OpenShift 4.17 or later with namespace access; a block storage class (not NFS, because SQLite needs POSIX locking); an OpenAI-compatible endpoint with tool calling: for vLLM `--enable-auto-tool-choice` and a `--tool-call-parser` that matches the model family (`openai` for gpt-oss, `hermes` for Qwen2.5); which optional parts need cluster-admin. Verification commands (`oc version`, `oc whoami`, `oc get storageclass`) with `Check:` lines.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/README.md` (prerequisites), `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` (lines 9-23), `starter-kits:agents/openclaw/deployment/manifests/03-pvc.yaml` (NFS comment).
- Tool-call parser: `starter-kits:agents/openclaw/deployment/docs/model-compatibility.md` (gpt-oss flags). For Qwen2.5 and `hermes` there is no pinned source; state it only as the guide's tested setup if the live deployment `vllm` in `openclaw-model` (read its args with `oc get deploy vllm -n openclaw-model -o jsonpath='{.spec.template.spec.containers[0].args}'`) shows it, and phrase it as "the tested Qwen2.5 deployment used".
- Cluster-admin needs: spec "Live verification results", `starter-kits:agents/openclaw/deployment/docs/mlflow-tracing.md` (lines 126-131), `openshell:docs/kubernetes/setup.mdx`.
- Node.js floor is irrelevant here (image-internal); leave it out.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/prerequisites.adoc"
    require:
      - '^Check:'
      - 'oc get storageclass'
      - '--tool-call-parser'
      - '--enable-auto-tool-choice'
      - '(?i)NFS'
```

---

### Task 8: `admin/configure-model`

Page: `content/modules/ROOT/pages/admin/configure-model.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Find the model ID with `/v1/models`; the naming rule (`models[].id` is the endpoint's ID, `agents.defaults.model` is `<provider>/<id>`, which gives `vllm/vllm/<model>` behind OGX); edit `02-configmap.yaml` or copy `overlays/example`; set `OPENCLAW_GATEWAY_TOKEN` and `VLLM_API_KEY` in `01-secret.yaml`, and why the Secret's `VLLM_API_KEY` wins over the ConfigMap's `apiKey`; choose and set the storage class (default `gp3-csi`; others such as `managed-csi` or `thin-csi` through the overlay patch), checked with `oc get storageclass`.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` (lines 25-134), `starter-kits:agents/openclaw/deployment/manifests/02-configmap.yaml`, `starter-kits:agents/openclaw/deployment/manifests/01-secret.yaml`, `starter-kits:agents/openclaw/deployment/overlays/example/kustomization.yaml`, `starter-kits:agents/openclaw/deployment/overlays/example/configmap-patch.yaml`.
- Provider keys and env interpolation: `openclaw:docs/providers/vllm.md`.
- Env precedence (`VLLM_API_KEY` overrides the config `apiKey`): live result D05; find the code location in `openclaw:src` (search for `VLLM_API_KEY` and provider API key resolution) and cite it.
- Overlay replaces the whole `openclaw.json`: `starter-kits:agents/openclaw/deployment/overlays/example/configmap-patch.yaml`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/configure-model.adoc"
    require:
      - '/v1/models'
      - 'vllm/vllm/'
      - 'VLLM_API_KEY'
      - 'OPENCLAW_GATEWAY_TOKEN'
      - 'storageClassName|storage class'
      - 'oc get storageclass'
      - '^Check:'
```

---

### Task 9: `admin/deploy`

Page: `content/modules/ROOT/pages/admin/deploy.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Create the namespace, apply `manifests/` or an overlay with `oc apply -k`, watch the init container and the gateway start (`oc get pods`, `oc logs ... -c init-config`, `oc logs ... -c gateway`), check `[gateway] ready` and the `agent model` line; expected pod state `1/1 Running`; the startup probe's 5-minute budget. Ends with the link to `admin/access`.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` (deployment and verify startup), `starter-kits:agents/openclaw/deployment/README.md` (quick start), `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml` (init script messages "Config initialized", probes).
- Live: the log of `openclaw-fresh` may be read to confirm the log lines (`oc logs deployment/openclaw -c gateway -n openclaw-fresh | grep -E "agent model|ready"`).

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/deploy.adoc"
    require_order:
      - 'oc new-project|oc create (namespace|ns)'
      - 'oc apply -k'
      - '\[gateway\] ready'
    require:
      - '^Check:'
      - 'xref:admin/access\.adoc'
```

---

### Task 10: `admin/access`

Page: `content/modules/ROOT/pages/admin/access.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Two ways in. Port-forward on local port 18789 (the only origins allowed by default are `http://localhost:18789` and `http://127.0.0.1:18789`; local pairing is approved automatically). The Route: what the `haproxy.router.openshift.io/set-forwarded-headers: never` annotation does and why (HTTP 403 `proxy_attribution_required` otherwise); adding the Route to `gateway.controlUi.allowedOrigins` with `config patch`; approving the browser by request ID (`devices list`, `devices approve <request-id>`; `--latest` only previews); the warning not to put the cluster network into `gateway.trustedProxies` (kubelet probes then get 403). Security note on Route exposure (token plus device approval). Device management: list and revoke devices.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` ("Access the Control UI" with "Through a port-forward" and "Through the Route"), `starter-kits:agents/openclaw/deployment/manifests/06-route.yaml`.
- Origins: `openclaw:src/gateway/origin-check.ts`, `openclaw:src/gateway/startup-control-ui-origins.ts`, `openclaw:docs/gateway/troubleshooting/agent-replies-and-control-ui.md`.
- Proxy attribution: `openclaw:docs/gateway/trusted-proxy-auth.md` (line 93).
- Pairing: `openclaw:docs/web/control-ui/connect-and-pair.md`, `openclaw:docs/cli/devices.md` (line 42 on `--latest`).

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/access.adoc"
    require:
      - 'oc port-forward'
      - 'set-forwarded-headers'
      - 'allowedOrigins'
      - 'proxy_attribution_required'
      - 'devices approve <request-id>'
      - 'trustedProxies'
      - '^Check:'
```

---

### Task 11: Diagram `config-state-lifecycle`

**Files:**
- Create: `diagrams/config-state-lifecycle.excalidraw`, `content/modules/ROOT/images/config-state-lifecycle.svg`

Shown on: `admin/change-config` (Task 12). Follow the `guide:diagrams` skill.

What it shows: where the configuration lives over the deployment's life, as three lanes left to right: first start, running, upgrade.

Entities and palette:
- Platform gray: ConfigMap `openclaw-config`, init container, PVC `~/.openclaw/openclaw.json`, gateway.
- Admin red: administrator actions `oc apply -k` (first start and upgrade), `oc exec ... config patch --stdin` (running), backup `tar` (upgrade).
- Governance orange: `doctor --fix` (state migration with database backup), write-safety (`openclaw.json.rejected.*`).

Arrows: first start: ConfigMap to init container to PVC ("only if missing"); running: admin to gateway via config patch, gateway writes the PVC copy, ConfigMap edits do not reach the running gateway (crossed arrow); upgrade: backup, apply, init container moves old root-level state into `.openclaw` once, entrypoint runs doctor, gateway ready within the startup budget.

Source hints: `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml`, `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md`, `openclaw:docker-entrypoint.mjs`, `openclaw:docs/gateway/configuration.md`.

---

### Task 12: `admin/change-config`

Page: `content/modules/ROOT/pages/admin/change-config.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Why the ConfigMap only seeds the first start; `config patch --stdin` with merge rules (objects merge, arrays and scalars replace, `null` deletes) and a worked example; the restart hints OpenClaw prints and how to restart (`oc rollout restart`); `--dry-run`; keeping the ConfigMap in sync for new PVCs; write safety (invalid writes rejected and saved as `openclaw.json.rejected.*`); reading the effective configuration with `config get`. Shows diagram `config-state-lifecycle`.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` ("Update the configuration"), `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml` (seed if missing).
- `openclaw:docs/cli/config.md` (patch, dry-run, restart hints, stdin limit), `openclaw:docs/gateway/configuration.md` (write safety), `openclaw:docs/gateway/configuration/hot-reload.md` (which keys restart).
- Live: `oc exec deployment/openclaw -c gateway -n openclaw-fresh -- node /app/dist/index.js config get gateway` to confirm the `config get` form.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/change-config.adoc"
    require:
      - 'image::config-state-lifecycle\.svg\['
      - 'config patch --stdin'
      - '--dry-run'
      - 'oc rollout restart'
      - 'config get'
      - '^Check:'
```

---

### Task 13: `admin/tracing`

Page: `content/modules/ROOT/pages/admin/tracing.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Prerequisites (RHOAI MLflow with workspaces; the `mlflow-integration` ClusterRole, which RHOAI 3.3.1 lacks and the admin creates from the shared definition; where to change the endpoint when MLflow runs outside `redhat-ods-applications`); the tracing ServiceAccount and RoleBinding; creating the experiment through the MLflow API; filling the overlay placeholders; applying the overlay; applying the tracing config to an existing deployment with `config patch`; verification in the MLflow UI. Known issues: experiment 404, the harmless "error parsing protobuf response" (the collector retries, MLflow stores each trace once), HTTP 400 with gzip (`compression: none`), metric exports going nowhere (the overlay enables metrics but the collector has only a traces pipeline), and the overlay's `plugins.allow` limiting loaded plugins. The span schema in short.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/mlflow-tracing.md`, `starter-kits:agents/openclaw/deployment/overlays/mlflow-tracing/` (all files), `starter-kits:docs/mlflow-openshift-auth-and-tls.md` (ClusterRole definition).
- `openclaw:docs/gateway/opentelemetry/setup.md` (enablement, protocol, hot reload of the exporter).
- Restart behavior: `openclaw:docs/cli/gateway/restart-and-supervision.md` (SIGUSR2); do not repeat the recipe's SIGUSR1 gap.
- Live: `oc get pods -n openclaw-test`, `oc logs deployment/openclaw -c otel-collector -n openclaw-test --tail=20`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/tracing.adoc"
    require:
      - 'mlflow-integration'
      - 'redhat-ods-applications'
      - 'compression: none'
      - 'X-MLFLOW-WORKSPACE'
      - 'oc apply -k'
      - '^Check:'
    forbid:
      - 'SIGUSR1'
```

---

### Task 14: `admin/maintain`

Page: `content/modules/ROOT/pages/admin/maintain.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Backup with `tar` through `oc exec` while no agent run is active, plus the variant for manifests older than PR #373 (`-C /home/node/.openclaw .`); upgrade by applying new manifests: the init container's one-time state move, `doctor --fix` with its database backup, the 5-minute startup budget, the log lines to check (`Auto-migrated legacy state`, `[gateway] ready`); the `EPERM: operation not permitted, fchmod` symptom of the old layout and its fix; removal with `oc delete -k` and what deleting the PVC destroys (sessions, state, logs). Restore is not covered by any source: say so in one sentence.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` ("Upgrading an existing deployment"), `starter-kits:agents/openclaw/deployment/manifests/04-deployment.yaml` (init move), `starter-kits:agents/openclaw/deployment/docs/troubleshooting.md` (EPERM entry).
- `openclaw:docs/install/updating.md`, `openclaw:docs/install/docker.md` (doctor on image replacement), `openclaw:src/infra/state-migrations.messages.ts`.
- PVC deletion consequences: `starter-kits:agents/openclaw/deployment/docs/installer-deployment.md` (lines 434-442).

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/maintain.adoc"
    require_order:
      - 'tar czf'
      - 'oc apply -k'
      - '\[gateway\] ready'
    require:
      - '-C /home/node/\.openclaw \.'
      - 'EPERM'
      - 'oc delete -k'
      - '^Check:'
```

---

### Task 15: `admin/install-openshell`

Page: `content/modules/ROOT/pages/admin/install-openshell.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Optional branch for clusters that run sandboxes. Install the Agent Sandbox controller and CRDs, the OpenShell Helm chart in version `{openshell-version}` on OpenShift (no privileged SCC; a CNI that enforces NetworkPolicy), wait for the gateway StatefulSet, register the gateway with the CLI (port-forward to `svc/openshell`, `openshell gateway add`, the TLS client bundle), and check with `openshell status`. Caveats: the RHCOS 5.14 kernel runs sandboxes in legacy read-only mode; sandboxes created with 0.0.x must be recreated.

Facts to verify (source hints):
- `openshell:docs/kubernetes/openshift.mdx`, `openshell:docs/kubernetes/setup.mdx`, `openshell:docs/upgrade/0-1-0.mdx`.
- Live (read-only): `helm list -n openshell-test`, `oc get statefulset,pods -n openshell-test`, `openshell -g openshell-test status`. The cluster install already exists; do not reinstall or upgrade it.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/install-openshell.adoc"
    require_order:
      - '(?i)agent sandbox'
      - 'helm install'
      - 'openshell gateway add'
      - 'openshell status'
    require:
      - '\{openshell-version\}'
      - '^Check:'
```

---

### Task 16: `admin/deploy-installer`

Page: `content/modules/ROOT/pages/admin/deploy-installer.adoc`. Section: Administrator guide, voice `technical`. Persona: admin. Internal: no.

Outline: Optional branch, written from the installer code at `{claw-installer-commit-short}`: run `./run.sh` (podman) or the Node.js dev flow, the OpenShift provider plugin after `oc login`, the form fields that matter (image, model endpoint, "Use image entrypoint"), what gets created (OAuth proxy sidecar, Route to `oauth-ui`, PVC `openclaw-home-pvc`, ConfigMaps, Secrets), the Instances tab, and device pairing after SSO. A CAUTION block lists the differences from the raw manifests: `:latest` images, the config copied on every start, the replaced gateway command that skips doctor unless "Use image entrypoint" is set, the gateway token also stored in a ConfigMap, and probes on `/`.

Facts to verify (source hints):
- `claw-installer:README.md`, `claw-installer:run.sh`, `claw-installer:package.json`, `claw-installer:provider-plugins/openshift/docs/deploy-openshift.md`, `claw-installer:provider-plugins/openshift/src/openshift-deployer.ts`, `claw-installer:provider-plugins/openshift/src/route.ts`, `claw-installer:provider-plugins/openshift/templates/oauth-proxy-container.yaml`, `claw-installer:src/server/deployers/k8s-manifests.ts`, `claw-installer:src/server/deployers/k8s-helpers.ts`.
- The recipe's installer doc (`starter-kits:agents/openclaw/deployment/docs/installer-deployment.md`) is stale in several places (research D13 to D21); follow the code and list the doc as an Upstream candidate.
- No live check: the installer is not run in this task.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/admin/deploy-installer.adoc"
    require:
      - '\./run\.sh'
      - 'oauth-proxy|OAuth proxy'
      - 'openclaw-home-pvc'
      - '^\[CAUTION\]'
      - '(?i)use image entrypoint'
```

---

### Task 17: `developer/index`

Page: `content/modules/ROOT/pages/developer/index.adoc`. Section: Developer guide, voice `technical`. Persona: developer (entry page). Internal: no.

Outline: The developer journey from the image layers to a running sandbox and on to extending the recipe (sandbox-image, build-image, run-in-sandbox, sandbox-model-config, extend); what you need (podman, a registry the cluster can pull from, the `openshell` CLI in version `{openshell-version}`, an OpenShell gateway set up per `admin/install-openshell`).

Facts to verify (source hints): `starter-kits:agents/openclaw/deployment/README.md` (OpenShell section), `starter-kits:sandboxes/base/README.md`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/developer/index.adoc"
    require_order:
      - 'xref:developer/sandbox-image\.adoc'
      - 'xref:developer/build-image\.adoc'
      - 'xref:developer/run-in-sandbox\.adoc'
      - 'xref:developer/sandbox-model-config\.adoc'
      - 'xref:developer/extend\.adoc'
    require:
      - 'xref:admin/install-openshell\.adoc'
```

---

### Task 18: `developer/sandbox-image`

Page: `content/modules/ROOT/pages/developer/sandbox-image.adoc`. Section: Developer guide, voice `technical`. Persona: developer. Internal: no.

Outline: The shared base (UBI 10 minimal, packages, the `sandbox` user with UID 1001 and GID 0, `/sandbox` and `/workspace`, the entrypoint that picks the agent from `$1` or `AGENT_NAME`, the image policy in schema version 1 and why an invalid policy blocks sandbox start since OpenShell 0.1) and the OpenClaw layer (`Containerfile.openshell`: official Node.js build with SHASUMS256 verification, because OpenClaw refuses the UBI Node.js packages that link SQLite 3.46.1; `npm install -g openclaw@{openclaw-version}`; the verify step that fails on an "unsupported" runtime; `AGENT_NAME=openclaw`). The base image is shared with the claude-code, codex, and opencode flavors.

Facts to verify (source hints):
- `starter-kits:sandboxes/base/Containerfile`, `starter-kits:sandboxes/base/entrypoint.sh`, `starter-kits:sandboxes/base/policy.yaml`, `starter-kits:sandboxes/base/README.md`, `starter-kits:agents/openclaw/deployment/Containerfile.openshell`.
- `openshell:docs/how-it-works/policies/schema.mdx`, `openshell:docs/how-it-works/policies/manage-policies.mdx`.
- `openclaw:docs/install/node-compatibility.md` (Node.js and SQLite floor).

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/developer/sandbox-image.adoc"
    require:
      - 'version: 1'
      - 'filesystem_policy'
      - 'SHASUMS256'
      - '(?i)sqlite'
      - 'AGENT_NAME'
```

---

### Task 19: `developer/build-image`

Page: `content/modules/ROOT/pages/developer/build-image.adoc`. Section: Developer guide, voice `technical`. Persona: developer. Internal: no.

Outline: Build the OpenClaw flavor with podman, with `--platform linux/amd64` when building on Apple Silicon for an x86_64 cluster; push to a registry the cluster can pull from; build and push the base image yourself (from `sandboxes/base/`) and point the flavor's `FROM` at it instead of the development image `quay.io/hmoghani/openshell-base:latest`. `Check:` lines with `podman run --rm <image> openclaw --version` and `node --version`.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/README.md` (build and push commands), `starter-kits:sandboxes/base/README.md` (build commands, published image, production target), `starter-kits:agents/openclaw/deployment/Containerfile.openshell` (`FROM`, `ARG`s).
- Live (optional): a local `podman build` of the flavor may be run to confirm the commands; do not push anywhere.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/developer/build-image.adoc"
    require:
      - 'podman build'
      - '--platform linux/amd64'
      - 'podman push'
      - 'Containerfile\.openshell'
      - '^Check:'
```

---

### Task 20: Diagram `sandbox-flow`

**Files:**
- Create: `diagrams/sandbox-flow.excalidraw`, `content/modules/ROOT/images/sandbox-flow.svg`

Shown on: `developer/run-in-sandbox` (Task 21). Follow the `guide:diagrams` skill.

What it shows: the path from a developer workstation to OpenClaw running in an OpenShell sandbox on OpenShift, and how traffic leaves and enters the sandbox.

Entities and palette:
- User blue: developer workstation (podman, `openshell` CLI, browser on `localhost:18789`).
- Platform gray: container registry; OpenShift namespace with the OpenShell gateway (Helm release, StatefulSet `openshell`); sandbox pod; model endpoint.
- Governance orange: image policy (filesystem, landlock, process), network policy rule `model_endpoint` for `/usr/local/bin/node`, default-deny egress.
- Admin red: supervisor (PID 1) inside the sandbox starting `openclaw gateway --bind loopback` as the main process.

Arrows: build and push image to registry; `openshell sandbox create --from <image>` to the OpenShell gateway, which creates the sandbox pod pulling the image; `openshell policy update --add-endpoint` to the rule; sandbox to model endpoint only through the rule; `openshell forward start 18789` from the workstation browser to the gateway on loopback.

Source hints: `starter-kits:agents/openclaw/deployment/README.md` (OpenShell section), `starter-kits:sandboxes/base/policy.yaml`, `openshell:docs/how-it-works/sandboxes/overview.mdx`, `openshell:docs/how-it-works/policies/network-rules.mdx`, `openshell:docs/kubernetes/openshift.mdx`.

---

### Task 21: `developer/run-in-sandbox`

Page: `content/modules/ROOT/pages/developer/run-in-sandbox.adoc`. Section: Developer guide, voice `technical`. Persona: developer. Internal: no.

Outline: `openshell sandbox create --name openclaw --detach --from <image> -- openclaw gateway --bind loopback --auth none --port 18789 --allow-unconfigured` (the trailing command is the sandbox's main process; the supervisor is PID 1 and ignores the image entrypoint); `openshell forward start 18789 openclaw --background`; the egress rule with `openshell policy update ... --add-endpoint <host>:<port> --binary /usr/local/bin/node --rule-name model_endpoint --wait` (real binary path, rule covers child processes, private addresses need an exact host name); OpenShift specifics (UID from the namespace range, `/sandbox` as working directory, `/workspace` not writable, recreate 0.0.x sandboxes). Shows diagram `sandbox-flow`.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/README.md` (OpenShell section).
- `openshell:crates/openshell-cli/src/main.rs` (flags of `sandbox create`, `forward start`, `policy update`), `openshell:docs/how-it-works/sandboxes/overview.mdx`, `openshell:docs/how-it-works/sandboxes/runtimes.mdx`, `openshell:docs/how-it-works/policies/network-rules.mdx`, `openshell:docs/kubernetes/openshift.mdx`.
- Live: an existing sandbox may be inspected with `openshell -g openshell-test sandbox list` and `openshell -g openshell-test sandbox get <name>`; creating a new sandbox named `guide-check` in that gateway is allowed and must be deleted at the end of the task.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/developer/run-in-sandbox.adoc"
    require_order:
      - 'openshell sandbox create'
      - 'openshell forward start'
      - 'openshell policy update'
    require:
      - 'image::sandbox-flow\.svg\['
      - '/usr/local/bin/node'
      - '/workspace'
      - '^Check:'
```

---

### Task 22: `developer/sandbox-model-config`

Page: `content/modules/ROOT/pages/developer/sandbox-model-config.adoc`. Section: Developer guide, voice `technical`. Persona: developer. Internal: no.

Outline: How to give OpenClaw in the sandbox a model provider configuration and API key, and how to keep it across sandbox restarts. No pinned source describes this combination; work it out live:

1. Read the OpenShell CLI for ways to run commands in or copy files into a sandbox, and for provider or credential features (`openshell:crates/openshell-cli/src/main.rs`, `openshell:docs/`), and OpenClaw's configuration commands (`openclaw:docs/cli/config.md`, `openclaw:docs/providers/vllm.md`).
2. In gateway `openshell-test`, create a sandbox named `guide-model` from the image the cluster already uses for tests (find it with `openshell -g openshell-test sandbox list` or the OpenShell test notes in `.guide/runs/2026-10-08-create/research.md`), configure the vLLM provider for the model endpoint `http://vllm.openclaw-model.svc.cluster.local:8000/v1` (model `qwen2.5-0.5b-instruct`, no key) with an egress rule, run one `openclaw agent --local` turn or a gateway turn, then restart the sandbox's main process and check whether the configuration survives.
3. Delete the sandbox and any policy you added at the end of the task.

Write only what the live run shows, each command cited to the CLI source that defines it. If no workable method exists, the page states what the sources support (the gateway starts unconfigured with `--allow-unconfigured`) and says that persistent configuration is not documented; report the gap as an Upstream candidate (`missing-doc`, source `starter-kits`).

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/developer/sandbox-model-config.adoc"
    require:
      - 'openshell '
      - 'openclaw '
      - '(?i)restart'
```

---

### Task 23: `developer/extend`

Page: `content/modules/ROOT/pages/developer/extend.adoc`. Section: Developer guide, voice `technical`. Persona: developer. Internal: no.

Outline: Kustomize overlays (namespace, storage class, resources, ConfigMap replacement) with a minimal overlay example; the agent template `templates/openclaw_agent/agent.yaml` and README as far as the sources describe them (metadata card; how the repository's tooling reads `agent.yaml`); OpenClaw extension points that work with the recipe's configuration (workspace files under `~/.openclaw/workspace`, skills, plugins) with links to the upstream docs; what the recipe's `tools.deny` and empty `channels` mean for extensions.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/overlays/example/`, `starter-kits:agents/openclaw/deployment/README.md` (customization table), `starter-kits:agents/openclaw/templates/openclaw_agent/agent.yaml`, `starter-kits:agents/openclaw/templates/openclaw_agent/README.md`, `starter-kits:AGENTS.md`.
- `openclaw:docs/` for workspace files, skills, and plugins (search `docs/` for "workspace", "skills", "plugins").

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/developer/extend.adoc"
    require:
      - 'kustomization\.yaml'
      - 'agent\.yaml'
      - 'tools\.deny|tools\.profile|"deny"'
```

---

### Task 24: `user/index`

Page: `content/modules/ROOT/pages/user/index.adoc`. Section: User guide, voice `tutorial`. Persona: user (entry page). Internal: no.

Outline: What a user can do with this deployment (chat with an OpenClaw agent through the Control UI in the browser), what to ask the administrator for (the address, the gateway token, approval of the browser), and the order of the next pages (connect, first-chat, whats-enabled). No `oc` commands on user pages.

Facts to verify (source hints): `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` (access), `openclaw:docs/web/control-ui.md`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/user/index.adoc"
    require_order:
      - 'xref:user/connect\.adoc'
      - 'xref:user/first-chat\.adoc'
      - 'xref:user/whats-enabled\.adoc'
```

---

### Task 25: `user/connect`

Page: `content/modules/ROOT/pages/user/connect.adoc`. Section: User guide, voice `tutorial`. Persona: user. Internal: no.

Outline: Open the address, enter the gateway token in "Gateway secret", wait for the administrator to approve the device (automatic on a local port-forward), and what each state on the login screen means ("This Gateway expects its token", pairing required, limited access); the token stays in the tab's session storage and a per-device token is used after pairing; pairing again after clearing site data or using a new browser profile; the rate limiter after repeated stale tokens and what to ask the administrator to do.

Facts to verify (source hints):
- `openclaw:docs/web/control-ui.md`, `openclaw:docs/web/control-ui/connect-and-pair.md`, `openclaw:docs/cli/devices.md`.
- `starter-kits:agents/openclaw/deployment/docs/troubleshooting.md` (pairing, rate limiter).
- Live: the login screen texts may be confirmed with a headless browser against the `openclaw-fresh` Route; never put the token or host on the page.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/user/connect.adoc"
    require:
      - 'Gateway secret'
      - '(?i)pair'
      - '(?i)site data'
```

---

### Task 26: `user/first-chat`

Page: `content/modules/ROOT/pages/user/first-chat.adoc`. Section: User guide, voice `tutorial`. Persona: user. Internal: no.

Outline: Send a message, read the answer, start a new session, find earlier sessions, and see which model answers. What to expect from the tested model (gpt-oss-20b served by vLLM on a GPU) with tools, and how a reply with raw JSON or a stalled turn looks when a model does not handle tool calls, and what to tell the administrator then.

Facts to verify (source hints):
- `openclaw:docs/web/control-ui.md` (chat, sessions, sidebar, model display), `starter-kits:agents/openclaw/deployment/docs/model-compatibility.md` (failure modes).
- Live: the controller points `openclaw-fresh` at the GPU model before this task (ledger note). Run one turn with `oc exec deployment/openclaw -c gateway -n openclaw-fresh -- node /app/dist/index.js agent --agent main --message "<prompt>"` to observe the behavior you describe; the page itself has no `oc` commands.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/user/first-chat.adoc"
    require:
      - '(?i)session'
      - '(?i)model'
      - 'gpt-oss-20b'
```

---

### Task 27: `user/whats-enabled`

Page: `content/modules/ROOT/pages/user/whats-enabled.adoc`. Section: User guide, voice `tutorial`. Persona: user. Internal: no.

Outline: The recipe's configuration in user terms: no web search, web fetch, or browser tools (`tools.deny: [group:web, browser]`), no messaging channels (`channels: {}`), sandbox mode off, heartbeat behavior, and what the administrator can change for you (with a link to `admin/change-config` for administrators). Explain each in one or two sentences of why.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/manifests/02-configmap.yaml`, `starter-kits:agents/openclaw/deployment/overlays/mlflow-tracing/configmap-patch.yaml`.
- `openclaw:docs/gateway/sandbox-vs-tool-policy-vs-elevated.md` (`group:web`), `openclaw:docs/gateway/heartbeat.md`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/user/whats-enabled.adoc"
    require:
      - '(?i)web search|web_search'
      - '(?i)browser'
      - '(?i)channel'
      - 'xref:admin/change-config\.adoc'
```

---

### Task 28: `reference/manifests`

Page: `content/modules/ROOT/pages/reference/manifests.adoc`. Section: Reference, voice `reference`. Persona: admin, developer. Internal: no.

Outline: Every resource of `manifests/` (Secret, ConfigMap, PVC, Deployment, Service, Route, kustomization) and the tracing overlay (ServiceAccount, RoleBinding, collector ConfigMap, Deployment patch) as tables of key fields: images by digest, env, args, probes with budgets, resources, security context, volumes and mount paths, labels, Route annotations. Each table cites its file.

Facts to verify (source hints): all files under `starter-kits:agents/openclaw/deployment/manifests/` and `starter-kits:agents/openclaw/deployment/overlays/mlflow-tracing/`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/reference/manifests.adoc"
    require:
      - 'openclaw-state'
      - '/startupz'
      - '/healthz'
      - 'set-forwarded-headers'
      - 'otel-collector'
```

---

### Task 29: `reference/configuration`

Page: `content/modules/ROOT/pages/reference/configuration.adoc`. Section: Reference, voice `reference`. Persona: admin, developer. Internal: no.

Outline: The `openclaw.json` keys the recipe sets in the base ConfigMap and the tracing overlay, each with meaning, value, and whether a change hot-applies or restarts the gateway; plus the keys the guide adds (`gateway.controlUi.allowedOrigins`) and the ones it warns about (`gateway.trustedProxies`).

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/manifests/02-configmap.yaml`, `starter-kits:agents/openclaw/deployment/overlays/mlflow-tracing/configmap-patch.yaml`, `starter-kits:agents/openclaw/deployment/overlays/example/configmap-patch.yaml`.
- `openclaw:docs/gateway/configuration/hot-reload.md`, `openclaw:docs/gateway/config-gateway.md`, `openclaw:docs/providers/vllm.md`, `openclaw:docs/gateway/opentelemetry/setup.md`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/reference/configuration.adoc"
    require:
      - 'agents\.defaults\.model'
      - 'models\.providers'
      - 'gateway\.bind'
      - 'allowedOrigins'
      - 'diagnostics\.otel'
      - 'plugins\.allow'
```

---

### Task 30: `reference/model-compatibility`

Page: `content/modules/ROOT/pages/reference/model-compatibility.adoc`. Section: Reference, voice `reference`. Persona: admin. Internal: no.

Outline: The recipe's tested-model table with its date, the criterion (structured tool calls) and failure modes, the tool-call parser per model family, provider prefixes and API types, and the tested lines: the recipe's (from its docs) and the guide's (gpt-oss-20b on vLLM, Red Hat AI Inference Server CUDA image, one NVIDIA A10G GPU). The controller adds the guide's tested line details to the ledger before this task.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/model-compatibility.md`, `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md` (tested lines), `starter-kits:agents/openclaw/deployment/docs/mlflow-tracing.md` (tested line).
- Live: read the GPU vLLM deployment's args from the ledger note.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/reference/model-compatibility.adoc"
    require:
      - '--tool-call-parser'
      - 'gpt-oss-20b'
      - '(?i)tested'
```

---

### Task 31: `reference/troubleshooting`

Page: `content/modules/ROOT/pages/reference/troubleshooting.adoc`. Section: Reference, voice `reference`. Persona: admin. Internal: no.

Outline: The recipe's entries corrected against the code: Route 403 `proxy_attribution_required`; Route 503 with the installer's OAuth proxy; wrong model or "No API key" (fix with `config patch`, not ConfigMap edits); heartbeat messages (`agents.defaults.heartbeat`, per-agent entries in `agents.entries`); pairing after SSO; CrashLoopBackOff (without the stale `/data/.openclaw` path); `EPERM ... fchmod`; config clobbered; pairing rate limiter (`devices clear --yes`); diagnostic commands. Each entry: symptom, cause, fix, check.

Facts to verify (source hints):
- `starter-kits:agents/openclaw/deployment/docs/troubleshooting.md` (all entries), `starter-kits:agents/openclaw/deployment/docs/raw-deployment.md`.
- `openclaw:docs/gateway/heartbeat.md`, `openclaw:docs/gateway/doctor/config-migrations.md` (`agents.list` to `agents.entries`), `openclaw:docs/cli/devices.md` (`clear --yes`), `openclaw:docs/gateway/troubleshooting/config-validation-and-probes.md`.

Rules:

```yaml
  - glob: "content/modules/ROOT/pages/reference/troubleshooting.adoc"
    require:
      - 'proxy_attribution_required'
      - 'EPERM'
      - 'config patch'
      - 'devices clear --yes'
    forbid:
      - '/data/\.openclaw'
```
