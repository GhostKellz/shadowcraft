# Shadowcraft

<div align="center">
  <img src="assets/icons/shadowcraft-icon.png" alt="shadowcraft icon" width="128" height="128">

**A self‑hosted, Pure‑Rust developer platform — repos, issues/MRs, GPU‑aware CI/CD, and blazing‑fast code search.**

![Rust](https://img.shields.io/badge/Rust-stable-orange?logo=rust)
![CI/CD](https://img.shields.io/badge/CI%2FCD-Pipelines-0a84ff)
![GPU](https://img.shields.io/badge/Runners-GPU%20%28NVIDIA%2FCUDA%29-2aa745)
![Search](https://img.shields.io/badge/Code-Search%20%28Tantivy%29-brightgreen)
![Auth](https://img.shields.io/badge/Auth-OIDC%20%2F%20SSO-blue)
![License](https://img.shields.io/badge/License-AGPL--3.0-lightgrey)

</div>

---

## Overview

**Shadowcraft** is a **Pure‑Rust alternative to GitLab/Gitea**. It bundles repositories over HTTP(S)/SSH, issues & merge requests, a fast **CI/CD engine with GPU‑aware runners**, and built‑in **code search** — all designed for performance, safety, and a minimal ops footprint.

> Design goals: **single language (Rust)**, **predictable deployments (systemd or containers)**, **API‑first**, and **sane defaults** for security and observability.

---

## Core Features

### Repositories

* Git over HTTP(S) and SSH with smart protocol
* Projects/groups, protected branches, required reviews
* Web editor (diffs, blame, history), LFS (optional)

### Issues & Merge Requests

* Labels, milestones, assignees, templates
* Required approvers, code owners
* Status checks from CI (block/allow merge)

### CI/CD (Pure‑Rust)

* YAML → DAG pipelines (stages, needs, rules)
* **GPU/NVIDIA runners** (containerized, `--gpus=all` auto)
* Artifacts, caches, environments, secrets/vars
* S3/MinIO artifact storage; logs streamed live

### Runners

* Cross‑platform Rust runner (Linux x86\_64/aarch64)
* Container backends: Docker or containerd (NVIDIA toolkit supported)
* Sandboxing: seccomp/apparmor profiles; host‑mount allowlists

### Code Search

* **Tantivy** (Rust Lucene) indexer
* Repo‑scoped or org‑wide search; filters by lang/author/path

### Registry (optional)

* OCI registry proxy/cache with S3 backend
* SBOM + image signing (cosign) roadmap

### Auth & Access

* OIDC SSO (Azure Entra, Google, GitHub)
* PATs, deploy keys, SSH keys
* RBAC: Owner, Maintainer, Developer, Reporter, Guest

### Observability & Ops

* Prometheus metrics; health endpoints
* Structured logs (Loki‑friendly)
* Webhooks for pipeline/repo events

---

## Why Pure Rust?

* **Safety + speed** across API, runners, pipeline engine, and search
* **One toolchain** → simpler builds, fewer supply‑chain surprises
* Rich async ecosystem (axum, tokio, sqlx, tonic) and first‑class search (tantivy)

---

## Architecture (High Level)

* **shadowcraft‑api**: axum + sqlx(Postgres); REST/WS + OpenAPI
* **shadowcraft‑ci**: scheduler/worker; pipeline compiler & leases; S3 for artifacts
* **shadowcraft‑runner**: executes jobs; connects over WS/gRPC; NVIDIA support
* **shadowcraft‑search**: tantivy indexer & query service
* **shadowcraft‑auth**: OIDC, sessions, PATs, SSH CA (future)

> Deploy as systemd services or containers. Single‑node to start; scales horizontally by adding runners and API replicas.

---

## Quick Start (dev)

```bash
# 1) API
cd crates/shadowcraft-api
cp .env.example .env   # set DATABASE_URL, OIDC, S3 creds (optional)
cargo run

# 2) Runner (local host)
cd ../shadowcraft-runner
cargo run -- --url http://localhost:8080 --token $RUNNER_TOKEN

# 3) Pipelines
# push a repo with .shadowcraft.yml — pipeline auto-creates on push
```

### Minimal Pipeline

```yaml
stages: [build, test]

build:
  stage: build
  image: rust:1.80
  script:
    - cargo build --release
  artifacts:
    paths: [target/release/]

test:
  stage: test
  needs: [build]
  image: rust:1.80
  script:
    - cargo test --all --locked
```

### GPU Job Example

```yaml
cuda-build:
  stage: build
  image: ghcr.io/you/cuda-rust:12.5
  gpu: true
  script:
    - cargo build -p my-cuda-crate --release
```

---

## Roadmap

* [ ] Repos (HTTP/SSH, LFS) & Projects
* [ ] Issues/MRs, reviews, CODEOWNERS, templates
* [ ] CI compiler (YAML→DAG), artifacts & cache (S3/MinIO)
* [ ] Runner v1 (Docker/containerd; NVIDIA)
* [ ] Search v1 (tantivy) with background indexing
* [ ] Registry proxy/cache
* [ ] OIDC + RBAC + audit log
* [ ] Web UI (Leptos or React)

---

## Security

* Mandatory TLS (reverse proxy or rustls)
* Signed webhooks; scoped tokens; encrypted secrets
* Supply‑chain: `cargo deny`, `cargo audit`, SLSA provenance (future)

---

## License

**MIT** — keeps improvements upstream for hosted offerings

---

## Naming Cheatsheet

* `shadowcraft-api` — HTTP/WS API & web
* `shadowcraft-ci` — pipeline compiler & scheduler
* `shadowcraft-runner` — job executor (GPU‑aware)
* `shadowcraft-search` — code search service
* `shadowctl` — admin/dev CLI

