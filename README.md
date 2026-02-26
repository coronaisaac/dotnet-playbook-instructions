# 🏗️ dotnet-playbook-instructions

> Architectural blueprints and AI prompts for scaffolding production-ready .NET projects using **Clean Architecture**, **Hexagonal design**, **SOLID principles**, and **Minimal API**.

---

## What is this?

This repo is a collection of opinionated, **AI-ready templates** that let you spin up new .NET backend projects in minutes. Each template contains:

| File | Purpose |
|---|---|
| `instructions.md` | Architecture rules + code patterns consumed by the AI as context |
| `prompt.md` | Starter prompt you paste into Copilot / Cursor / Claude / ChatGPT |
| `README.md` | Human-readable guide: when to use it, requirements, examples |

---

## Available templates

| Template | Stack | When to use |
|---|---|---|
| [`minimal-api-hexagonal`](./templates/minimal-api-hexagonal/) | .NET 8/10 · Minimal API · EF Core · SQL Server | REST APIs following Clean + Hexagonal architecture |
| `worker-service` *(coming soon)* | .NET · BackgroundService | Message consumers, scheduled jobs |
| `grpc-service` *(coming soon)* | .NET · gRPC | Internal service-to-service communication |

---

## Quick start

1. **Pick a template** from the `templates/` folder.
2. **Open `prompt.md`** and paste its contents into your AI assistant (GitHub Copilot Chat, Cursor, Claude, etc.).
3. **Attach or reference `instructions.md`** as context for the AI.
4. **Answer the questions** the AI asks (project name, shared prefix, .NET version, etc.).
5. The AI will scaffold the full solution following the blueprint.

---

## Configurable tokens

Every template uses placeholder tokens that the AI replaces with your values:

| Token | Description | Example |
|---|---|---|
| `{{ProjectName}}` | Domain project prefix | `Whatsapp.Historial` |
| `{{SharedPrefix}}` | Shared libraries prefix | `MyCompany` |
| `{{TargetFramework}}` | .NET target moniker | `net8.0` |
| `{{DbContextName}}` | EF Core DbContext name | `HistorialDataContext` |
| `{{ConnectionStringKey}}` | appsettings section key | `ConnectionStringHistorialKey` |

---

## Architecture principles enforced

- ✅ **SOLID** — enforced at every layer
- ✅ **Clean Architecture** — strict ring dependency rules
- ✅ **Hexagonal / Ports & Adapters** — interfaces as ports, implementations as adapters
- ✅ **Modern .NET (≥ 8)** — Minimal API, global usings, async/await, CancellationToken, Options pattern
- ❌ No MVC controllers, no `.Result`/`.Wait()`, no hardcoded secrets

---

## Contributing

1. Fork the repo.
2. Create a new folder under `templates/your-template-name/`.
3. Add `instructions.md`, `prompt.md`, and `README.md`.
4. Open a PR with a description of the architecture pattern the template covers.

---

## License

MIT — see [LICENSE](./LICENSE).
