# Third-party notices

This repository's own code (the Docker Compose definition and
documentation) is MIT-licensed — see `LICENSE`. It composes three
independent guards, each its own project with its own license. This
file summarizes what that means for anyone running `docker compose up`
here; it is not a substitute for reading each project's own license.

## secret-guard

[gramatr/secret-guard](https://github.com/gramatr/secret-guard) — MIT.
Wraps [gitleaks](https://github.com/gitleaks/gitleaks), itself MIT. No
model weights, no gated license, nothing to accept.

## pii-guard

[gramatr/pii-guard](https://github.com/gramatr/pii-guard) — MIT. Built
on [Microsoft Presidio](https://github.com/microsoft/presidio) (MIT)
and its default NER backend, spaCy's `en_core_web_lg` (MIT, Explosion).
No gated license, nothing to accept.

## injection-guard — the real carve-out

[gramatr/injection-guard](https://github.com/gramatr/injection-guard)
is MIT for its own code, but it loads
[meta-llama/Llama-Prompt-Guard-2-86M](https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M)
at runtime, which is separately licensed under **Meta's Llama 4
Community License** — not MIT, and not something this repository (or
any MIT license) can grant on your behalf. Running the
`injection-guard` service from this stack means:

- **You need your own Hugging Face account** with that model's gated
  license accepted, and your own token supplied via
  `~/.config/gramatr-guards/injection-guard.env` (see
  `docker-compose.yml`). This cannot be automated — accepting someone
  else's license is inherently a per-user, manual step.
- **"Built with Llama" must be displayed** prominently wherever you
  ship something built on this (injection-guard's own `/about` endpoint
  carries the required notice; if you build a product around this
  stack, that obligation follows you too).
- **If Meta's attribution/MAU terms ever apply to your deployment**
  (see the Llama 4 Community License's own 700M MAU clause), that's
  between you and Meta — this stack doesn't change or absorb that.

The authoritative license text and attribution files live in
`gramatr/injection-guard` itself — `NOTICE` and
`THIRD_PARTY_LICENSES/LLAMA4_COMMUNITY_LICENSE.txt` there, fetched
verbatim from Meta's gated Hugging Face repo. This file summarizes;
that repo is the source of truth.

## What this means for contributors

This stack — the compose definition, health checks, the whole idea of
running these three guards as one shared service — is meant to be
genuinely open: MIT, PRs welcome, no gate to clear beyond the normal
review. The one thing nobody can relicense is the Llama model
`injection-guard` optionally loads. If you only want the MIT-clean
path, run `secret-guard` and `pii-guard` and skip `injection-guard` —
both the Compose file and `guards-selfheal.sh` in downstream adapters
already treat each service as independently optional.
