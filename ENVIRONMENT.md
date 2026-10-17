# Environment variables

Every variable below with a default is baked into its service's own
Dockerfile as the container-internal value — you should not need to
override most of these. The only one a self-deployer actually has to
supply is `HF_TOKEN`.

| Variable | Service | Secret? | Required? | How to supply it | Default (in-container) |
|---|---|---|---|---|---|
| `HF_TOKEN` | injection-guard | **Yes — real credential** | Only to use injection-guard | `~/.config/gramatr-guards/injection-guard.env` (gitignored, 0600, never committed) via Compose `env_file` | none — service fails to load the model without it |
| `SECRET_GUARD_HOST` / `SECRET_GUARD_PORT` | secret-guard | No | No | Dockerfile `ENV` | `0.0.0.0` / `8732` inside the container (compose publishes only to `127.0.0.1`) |
| `PII_GUARD_HOST` / `PII_GUARD_PORT` | pii-guard | No | No | Dockerfile `ENV` | `0.0.0.0` / `8731` inside the container |
| `INJECTION_GUARD_HOST` / `INJECTION_GUARD_PORT` | injection-guard | No | No | Dockerfile `ENV` | `0.0.0.0` / `8730` inside the container |
| `INJECTION_GUARD_DEVICE` | injection-guard | No | No | Dockerfile `ENV` (locked to `cpu` on the `:cpu` image this stack uses) | `cpu` |
| `INJECTION_GUARD_MODEL_ID` | injection-guard | No | No | not set by this stack | the eval-verified 86M model — overriding this logs a loud startup warning, see injection-guard's own README |
| `INJECTION_GUARD_AUTH_TOKEN` | injection-guard | Yes, if set | No | not set by this stack (loopback-only deployment has no need for it) | unset — only relevant for a hosted-tier deployment, see `SERVER-SIDE.md` |
| `HF_HOME` | injection-guard | No | No | Dockerfile `ENV` | `/app/.cache/huggingface` — model weight cache inside the container, not persisted across `docker compose down` unless you add a volume |

## Why `HF_TOKEN` can't leak to anyone self-deploying this

- It never appears in any Dockerfile, in `docker-compose.yml`, or in
  this repo at all — the compose file only references the *path* to a
  local file (`${HOME}/.config/gramatr-guards/injection-guard.env`)
  that each deployer creates themselves, outside this repo.
- That path is `.gitignore`d here (`*.env`) and enforced to `0600` by
  the reference self-heal implementation
  ([claude-plugin-guards](https://github.com/gramatr/claude-plugin-guards)'
  `guards-selfheal.sh`).
- Confirmed by testing (not assumed): `docker compose logs` on all
  three services after real requests, including malformed ones, shows
  only request method/path/status — no request bodies, no header
  values, no token strings, anywhere in any service's log output.

## Resilience, confirmed by testing

- Missing/invalid JSON on any of the three `/analyze`, `/classify`,
  `/scan` endpoints returns a proper 4xx (`422` for pii-guard/
  injection-guard via FastAPI/pydantic validation, `400`/`200`-fail-open
  for secret-guard) — none of the three crash or hang on malformed
  input.
- A 5MB single-request payload to secret-guard completed in ~1.6s with
  no crash and the service still healthy afterward.
- Every service has `restart: unless-stopped` + a `mem_limit`/`cpus`
  ceiling in `docker-compose.yml` — an unbounded request that actually
  exhausts a container's memory gets OOM-killed by that ceiling and
  restarted automatically, rather than taking down the host.
