<div align="center">

<img src="profile/gentian-line-stars.svg" alt="Gentian Line — a spiral of stars around a crystalline core" width="180" />

# Gentian Line

**Memory harness and project management systems for humans and LLMs.**

*A small, serious fun.*

</div>

---
Scifi reference: House of Suns

Large-scale human civilizations almost invariably seem to collapse and disappear within a few millennia (a phenomenon referred to as "turnover"), the limits of sub-lightspeed travel making it too difficult to hold interstellar empires together. Consequently, the most powerful entities in the galaxy are the "Lines"—familial organizations made of cloned "shatterlings". The Lines do not inhabit planets, but instead travel through space, holding reunions after they have performed a "circuit" of the galaxy; something that takes about 200,000 years.

House of Suns concerns the Gentian Line, also known as the House of Flowers, composed of Abigail Gentian and her 999 clones (or "shatterlings"), male and female: exactly which of the 1,000 shatterlings is the original Abigail Gentian is unknown. The clones and Abigail travel the Milky Way Galaxy, helping young civilizations, collecting knowledge, and experiencing what the universe has to offer. Members of the Gentian Line are named after flowering plants.
T
hat is roughly the situation our agents are in, and it is not a problem we intend to
"solve" so much as *instrument*. Every clone gets its own ledger; every ledger stays
coherent; nothing silently forgets because something else carefully remembered.  This way our goals can survive the
test of time.

## What we are doing

Two repos, one loop: **memory** on one side, **work** on the other.

### [`clm-mem`](https://github.com/GentianLine/clm-mem) — memory that the model does to itself

A **zero-training memory harness** for open chat models. The same model that reads and
writes text also classifies its own context as *long-term*, *short-term* or *transient*
while decoding, and a small deterministic engine then enforces the token budget in three
phases (drop transient → short-term FIFO rounds → archive long-term with a recall stub).

This is our reading of **Context Language Models** (Shao et al., arXiv:2609.37725): treat
context as an editable artifact mirrored to disk, and delegate context management to the
model via prompting rather than to a router bolted on afterwards. The paper's method pays
for that freedom by invalidating the KV cache every request; we pay for it only when the
window actually overflows, so the churn stays low — and any improvement made to KV-cache
management helps us too.

The important consequence: **the model decides what matters as it is creating context, not
after context has grown.** The human stays in charge of the ask; the hardware keeps reliable
expectations about its load; no exotic new cache strategy is required just to be kind to a
model that thinks about its own thinking.

### [`tokamak`](https://github.com/GentianLine/tokamak) — story points at plasma pressure

A Django + django-unicorn story-point board with one service layer behind everything, custom
signals on every project and ticket state change, an append-only `EventLog`, Fibonacci-only
estimates, a real blockers graph, and sprint-line/cutoff forecasting derived from accepted
velocity.

It is named after the fusion device because that is what a good team feels like: a configuration
where energy is held in place by opposing forces precisely enough that something rare happens.

Tokamak is our **human-first approach to agile**: it leverages AI aggressively — signals,
automation receivers, an optional memory bridge — while refusing to turn people into machines.
Nobody's judgement is replaced; their bookkeeping is. Tickets do not mutate workflow state
silently, automation cannot invent estimates out of nowhere, and every machine action is
auditable in the event log so a human can disagree with it later.

### The bridge

Every Tokamak signal and `EventLog` row can optionally mirror into per-project sessions of
`clm-mem`. Tickets that enter work open monitored chat sessions; ending them distils a summary
plus durable facts back onto the ticket, with archive handoff so a reopened attempt inherits
what the previous one learned. Off by default. Fully offline-testable.

## What we are researching

We are measuring **the effects of different memory strategies across open models** — eviction
policy, tag granularity, keep-percentages, classify-as-you-decode versus post-turn routing,
archive-plus-recall versus plain truncation — across Qwen-family and other OpenAI-compatible
backends (vLLM, Ollama, llama.cpp, LM Studio, Unsloth Studio, Featherless).

Preliminary orientation from the literature: finer-grained memory organisation is *not*
universally beneficial. Designs that produce strong forward transfer can simultaneously induce
severe forgetting. External memory does not resolve continual learning; it **reshapes** it into
a problem of memory representation and retrieval design. We intend to be very serious about
that sentence, in public, with reproducible environments (`pyproject.toml` as the single source
of truth, frozen lockfiles per checkout).

## Cultural operating system

A running glossary for people joining us:

| Reference | What it means here |
|---|---|
| **Gentian Gate / Gentian Line** (Reynolds) | Distributed copies, one ledger. Our name, and our warning about identity-vs-accounting. |
| ***House of Suns*** (Rajaniemi) | "The Great Enzyme", jackdaw cloning, plural selves sharing a common mind — the difference between *shared* memory and *merged* memory. We keep them shared; merging is a design decision, not an accident. |
| **Achines / skEOSenTience** (Rajaniemi) | Minds that are substrates rather than users of substrate. Reminder that the harness is not neutral: it shapes what the model can remember, therefore what it can be. |
| **Shadow clone technique** (Naruto) | The same agent in several places at once, each with its own local state, then a report-back on merge. Our multi-agent and worktree conventions; `ws:<branch>` session ids are the hand seals. |
| **Shatterlings** (Beaulieu, *City & City* / *The Dream·Pollinyx*) | A single person distributed across cities and selves, each incomplete but coherent. The best shorthand we have found for per-workspace memory sandboxes: no single node holds the whole self, and none of them is a lie. |
| **Distributed memories** (Minsky's K-lines; Hopfield nets; extended-mind cognition) | Retrieval from partial cue across traces that live outside any one brain. Technically, this is just what we are building; culturally, it is why the topic feels fated rather than novel. |
| **System One / System Two** (Kahneman) | The decision server and the chat backend in `clm-mem serve`: a cheap fast classifier beside an expensive slow generator. Naming it after the psychologists keeps us honest about borrowing from cognitive science rather than inventing folklore. |

## Conventions

- **Memory is explicit.** In `clm-mem serve`, memory is never a header side-effect of
  `/v1/chat/completions` — that route stays pure passthrough. You *open* a branch, then you
  *turn* on it. `410` exists to tell you so.
- **Nothing is silently destroyed.** Long-term memory is offloaded to an archive and left as a
  retrievable stub; the failure mode of "can offload but cannot evict" is not welcome here.
- **Humans are not demoted.** Automation may propose, tag, flag, mirror and distil. It may not
  impersonate a person's decision. `EventLog` is the receipt.
- **Reproducibility over vibes.** Pinned lockfiles, `black --check`, CI gates, tests as the
  definition of behaviour.

## Repos

| Repo | Stack | What it does |
|---|---|---|
| [`clm-mem`](https://github.com/GentianLine/clm-mem) | Python ≥3.10 | Zero-training long/short-term context-memory harness: token-budgeted window, three-phase eviction, archive + recall, web studio, MCP toolbox, agent mode with sandboxing. |
| [`tokamak`](https://github.com/GentianLine/tokamak) | Django + django-unicorn + DRF | Story-point board, signal-driven automation, velocity/cutoff forecasting, optional `clm-mem` memory bridge and ticket sessions. |

## Get started

```bash
# Memory harness (works with no GPU using bundled mock backends)
git clone https://github.com/GentianLine/clm-mem && cd clm-mem
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
clm-mem serve --use-mock --port 8080 --served-model mock-model \
    --max-context-tokens 600 --workdir /tmp/clm-serve

# Project board
git clone https://github.com/GentianLine/tokamak && cd tokamak
pip install -r requirements.txt
python manage.py migrate && python manage.py seed_demo && python manage.py runserver
```

## Contributing

Issues and pull requests on both repos are welcome; read
`CONTRIBUTING.md` in the repo you are touching, keep tests as the specification, and format with
the pinned `black`. Joining notes for humans: bring curiosity, distrust of magic, and a
willingness to name a mechanism after the cognitive-science literature that inspired it.

---

<div align="center">

<img src="profile/gentian-line.svg" alt="Gentian Line mark — crystalline core with orbital memory traces" width="120" />

<br/>

> *We do not make the machine more human. We make the record trustworthy enough that humans can afford to be.*

</div>
