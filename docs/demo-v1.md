# Demo v1

This is a five-minute portfolio demo script for the Personal AI Operations
Platform. It focuses on what is already implemented and keeps private runtime
details out of the public repository.

## Flow

1. Start with the problem: personal knowledge, job search, and market research
   become difficult to operate when they stay as one-off chats.
2. Show the platform boundary: OpenClaw Gateway coordinates agents, while the
   infrastructure repo owns shared Postgres, Redis, and the HTTPS edge.
3. Walk through the workspace pipeline model: project specs live under
   workspace projects, outputs are written as dated artifacts, and repeatable
   runs can be invoked manually or by cron.
4. Explain the current agents: `main` for daily questions, `deep` for research,
   `graph` for visual work, `task` for background jobs, plus Companion Nodes
   `m1pro` / `m2max` for local node execution.
5. Close with the roadmap: currently Phase 2a mid-late (Personal Knowledge OS
   with Vault + keyword/chunk ingest), then vector RAG, observability,
   Kubernetes, and GitOps—without claiming vectors/K8s as Done.

## Notes

- Do not show secrets, private domains, raw job descriptions, or personal data.
- Use sanitized screenshots or diagrams when presenting the control plane.
- Keep the live demo focused on read-only views and generated public artifacts.
