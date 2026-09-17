# Company-wide demo runbook — Substrate Scope live

Validated end-to-end 2026-09-17 on kind-kagent-substrate
(kagent 0.10.1 + substrate 0.0.9, Ollama qwen3:4b, 9 SandboxAgents Ready).

## Preflight (5 min, morning of)

```bash
# 1. No stray clusters starving the Docker VM (the k3d-mcp-federation lesson)
docker stats --no-stream --format '{{.Name}} cpu={{.CPUPerc}} mem={{.MemUsage}}'

# 2. Demo cluster healthy (context matters — default context is k3d-ai-demo!)
kubectl --context kind-kagent-substrate get pods -n ate-system --no-headers | grep -cv -E 'Running|Completed'   # want 0
kubectl --context kind-kagent-substrate get sandboxagents -n kagent   # want 9 Ready

# 3. Ollama up
curl -s localhost:11434/api/version
```

## Start the rig (already running as of 2026-09-17 evening)

```bash
cd ~/substrate-scope
KUBE_CONTEXT=kind-kagent-substrate node server.mjs --live
```

- Board: http://localhost:8123
- kagent chat UI (auto-forwarded): http://localhost:8001
- KUBE_CONTEXT is REQUIRED on this machine: default kubectl context is the
  k3d-ai-demo enterprise cluster; Scope pins every kubectl call to the value.

Traffic (second terminal, start ~2 min before showtime so bays are warm):

```bash
cd ~/substrate-scope
node stimulate.mjs --load 0.1 --oversub 2
```

Ollama pacing note: qwen3:4b turns run 30–60s, which makes bays visibly
occupied — good for an audience. `--load 0.1` keeps most prompts short.
Ollama is free; no budget needed. Ctrl-C or the board's STOP DEMO halts it.

## The 5 beats (~6 min)

1. **The lie of kubectl** — terminal: `kubectl --context kind-kagent-substrate
   get pods -n kagent | grep kagent-default`. "Three quiet pods." Flip to the
   board: nine agents, sessions climbing. State lives in object storage, not pods.
2. **Watch one think** — click any RUNNING chip: drawer shows the prompt,
   restore → pod, the reply with latency, checkpoint. Then TALK TO IT: type in
   the drawer box, watch your own message become a restore.
3. **Break it on purpose** — SURGE twice. Queue lane fills (substrate rejects
   when full; retry queue is client-side). Fairness tab: 📢 noisy neighbor vs
   🍽️ starved — "usage is popularity, contention is noise."
4. **Let it heal** — AUTOSCALE is on: one jump to demand, then a one-jump
   walk-down on the 30s window. Point at the autoscale lines in the feed.
5. **The money chart** — Telemetry → CPU: dotted line = these agents as
   always-on pods, forever. Amber = what the pool reserves, breathing.

## If something wedges

- Workers pinned by ghost actors ("no free workers", bays look idle):
  **RESET POOL** button. Snapshots survive; ~15s.
- Board frozen: check the server terminal; restart the `node server.mjs --live`
  command. Port-forwards self-heal.
- Anything billable runs away: **STOP DEMO** (also stops stimulate within 2s).
- Nuclear, zero-cluster fallback: `node server.mjs` (sim mode) or the artifact —
  the full experience, synthetic data.

## Do not touch

- The k3d-ai-demo cluster (enterprise; not part of this demo).
- kagent 0.10 quirk, already handled in Scope: generated ActorTemplate names
  carry a 16-hex suffix; the server strips it. spec.platform no longer exists
  on SandboxAgent (0.10 rejects it; fleet manifest already clean).
