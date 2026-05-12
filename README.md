# quox-skills-index

Curated marketplace index for [QuoxSkills](https://quox.ai/docs/concepts) — the procedural-knowledge tier of the Quox three-tier knowledge stack (Skills + Tools + Crystals).

This repo is the **canonical index** that `quox-core@official` subscribers fetch from. QuoxFlow instances poll the raw-content URL of this repo on a sync cadence (default ~1h) to discover new and updated skills.

## Structure

```
quox-skills-index/
├── index.json                       # master catalog (consumed by subscribers)
├── trust_roots.json                 # curator public keys (consumed by signature verifiers)
├── skills/
│   └── <skill-name>/
│       └── <version>/
│           ├── manifest.json         # {name, version, body_md, tests, required_tools, metadata}
│           └── signature.bin         # Ed25519 sig over manifest.json bytes
└── README.md
```

## How publish works

1. An author with `edition='stable'` + all tests passing + safety-review pass invokes `POST /api/quox-skills/:id/publish` on their QuoxFlow instance.
2. QuoxFlow builds a manifest JSON, signs it with the org's Ed25519 private key, opens a PR against this repo.
3. Curator reviews the PR (or auto-merges per policy).
4. On merge, subscribers see the new entry within their next sync window.

## How subscribe works

1. Subscriber QuoxFlow instances fetch `https://raw.githubusercontent.com/quoxai/quox-skills-index/main/index.json` on a schedule.
2. Diff against locally-installed versions; surface "N new skills available" in `/skills`.
3. On install: fetch manifest + signature, verify signature against `trust_roots.json` public keys, install at `edition='experimental'`, run local tests + Crystal eval, auto-promote to `stable` on pass.

## Trust model

- Root keys are Ed25519 and live in `trust_roots.json`.
- Curators sign org keys with their root key (chain-of-trust) — extension for v2.
- Subscribers MUST verify the signature before installing. Tampered manifests are rejected.
- Revocation: append an entry with `active: false` to `trust_roots.json` — subscribers stop trusting that key on next sync.

## Federated telemetry

Optional. See sibling repo `quoxai/quox-skills-telemetry` for anonymised stabilisation metrics that subscribers can opt in to contributing.

## Related streams

- QSKL-8a (Quox roadmap) — publish + subscribe + install flow, owns this repo's wire protocol.
- QSKL-SAFETY — Ed25519 signature verification, firewall, LLM-judge safety review.
- QSKL-8b — federated telemetry relay.
