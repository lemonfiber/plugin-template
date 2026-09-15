# AGENTS.md — plugin-template

Guidance for any AI agent working in this repo.

> **Common rules for every lemonfiber repo are canonical in the spec:**
> [50-governance/ai-contributors.md](https://github.com/lemonfiber/spec/blob/main/50-governance/ai-contributors.md).
> Read them. This file is the `plugin-template`-specific header only.

## What this repo is

The plugin an author copies, and the **canonical home of the interim CI
harness**. `plugin-komga` and `plugin-uptime-kuma` carry byte-identical copies of
`.github/interim/` and each has a job that fails when its copy drifts from this
one. Change the harness here, then copy it to both; changing it there fails.

The plugin itself describes Kavita, and it describes something real on purpose:
`F10-R7` asks for a template that validates and proves *unmodified*, and a
template whose proofs are invented teaches an author to invent proofs.

## The rules you cannot break

- **Nothing here executes.** A plugin is declarative data (`F3-R1`), and
  contributed code is never run, under any opt-in (`F3-R6`). The Python under
  `.github/interim/` is CI harness, is not part of what an operator installs,
  and is deleted when lemonfiber's own verbs replace it.
- **The plugin is `plugin.toml` and `fixtures/`.** Proofs, claims and
  contributions live in the manifest, not beside it: an installer reads one file,
  and something the installer never reads cannot be what `F3-R4` refuses an
  install over.
- **The image is named by digest** (`F3-R8`). Moving the pin means re-recording
  every fixture against the new image in the same change.
- **A proof asserts a body, never only a status.** Docker's port proxy accepts
  before anything inside is listening. The one exception is a refusal: a `401`
  is not something a port proxy can produce.
- **A proof that could not be run is unproven** (`F3-R5`), never a pass. The
  three verdicts stay three.
- **No field beyond the contract's set.** A manifest carrying one is refused by
  name rather than ignored (`ARCH-R84`).
- **Which names exist is lemonfiber's to say.** `validate.py` must never carry a
  list of capability names or extension points — `F10-R2` forbids a second,
  hand-maintained description of the format, and a second list would disagree
  with the parser the day one was edited. Rules that need them are skipped and
  **named as skipped**; `vocabulary_gate.py` fetches the published artefacts and
  decides them.

## Checks

```
python3 .github/interim/validate.py --self-test
python3 .github/interim/validate.py
python3 .github/interim/vocabulary_gate.py
python3 .github/interim/prove.py --against fixtures --report proofs.json
python3 .github/interim/image_gate.py
python3 .github/interim/schema_gate.py
```

`proofs.json` is generated and committed; CI fails when the committed one is not
what the run would write.

## Before you open a PR

- Cite a spec identifier in a commit `Spec:` trailer and the PR body.
- No AI attribution in commits.
