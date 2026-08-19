# InputLayer Ontology Registry

Ready-to-go ontologies for the [InputLayer](https://github.com/inputlayer/inputlayer) reasoning engine. Each entry packages everything one use case needs — rules, extraction contract, chat→KG mapping, demo script, and golden fixtures — installable into a running engine with one command.

The model is deliberately Helm-shaped: this repository is to an InputLayer engine what a chart repository is to a Kubernetes cluster. The `il` CLI is the only installer; deployment rides the engine's existing WebSocket protocol as a single atomic program.

```console
$ il search
ONTOLOGY           LATEST   ENGINE    TITLE
consistency-core   1.0.0    >=0.1.0   Verified Completions

$ il install consistency-core --kg _conv_demo --create
fetching consistency-core-1.0.0.tar.gz … sha256 verified
deploying over ws://localhost:8080/ws … 107 statements, 1 atomic round trip
✓ consistency-core@1.0.0 → _conv_demo (pinned in pack_meta)

$ il demo consistency-core
```

> **Status:** the registry format is stable enough to author against; the `il` CLI that consumes it is under active development in the engine repository. Until it ships, entries can be loaded by sending the rules file as a single program over the WebSocket API.

## Ontologies

| Ontology | Version | What it catches |
|---|---|---|
| [`consistency-core`](ontologies/consistency-core/) | 1.0.0 | Logical-consistency verification for AI conversations (Verified Completions): contradictions, timeline cycles, identity mix-ups, and policy violations — every finding backed by quoted spans and a proof tree. Validated against a 1,628-scenario adversarial corpus. |

## Anatomy of an entry

```
ontologies/<name>/
├── ontology.toml            # manifest: identity, extraction binding, chat→KG
│                            #   mapping, validation gates, report surface
├── rules/*.iql              # the rule pack — human-written, frozen at load
├── extraction/
│   ├── schema.json          # closed JSON schema for the extraction model
│   └── prompt.md            # the extraction system prompt
├── fixtures/
│   ├── golden.iql           # one scripted scenario with expected findings
│   └── golden.iql.out       # engine-verified snapshot of its output
└── demo/conversation.json   # scripted demo: turns + pre-extracted facts,
                             #   runs offline with no extraction model
```

Two principles every entry must honor:

1. **The trust boundary.** The LLM only ever writes *data* — claims, orderings, constraints, extensions to open seed lists. Rules are human-written, reviewed here, and frozen at load. Nothing in a conversation can reach them.
2. **No open-domain extraction.** Text is only ever translated to facts against a specific entry's schema, prompt, and mapping. The manifest's `[map]` section is an allowlist; the `[validate]` section drops anything that fails the entry's semantic gates (e.g. quotes that are not verbatim).

## Distribution

Releases are Helm-style: `index.json` at the repository root is the registry index; each `name@version` is a tarball on GitHub Releases with its sha256 digest recorded in the index. The CLI verifies the digest before anything is deployed — a mismatch is a hard refusal.

Tagging `<name>-v<version>` packages, checksums, uploads, and re-indexes automatically (`.github/workflows/release.yml`).

## Validation

Every entry is gated by CI (`.github/workflows/validate.yml`):

- `tools/lint_iql.py` on every `.iql` file — arity consistency, variable safety, one statement per line (the atomic-load contract).
- The entry's golden fixture executed against a live engine, diffed against the recorded snapshot. An entry that stops reproducing its expected findings cannot merge.

## Contributing an ontology

Author the entry layout above, make its golden fixture pass, and open a PR. Entries are reviewed as what they are: laws that a verifier will present as ground truth, with your name on the review.

## License

To be finalized. Until a LICENSE file lands, all rights reserved.
