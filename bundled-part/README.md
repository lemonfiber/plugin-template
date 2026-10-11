> **This is the bundled-part variant of the lemonfiber plugin template.** How a
> part's repository is made from it is in the template's
> [docs/development.md](../docs/development.md#a-bundled-part). Delete this note.

# SABnzbd for lemonfiber

**Downloads from Usenet.**
SABnzbd is one of the parts the stack lemonfiber ships is built from.
Without it: No Usenet downloads.

## How lemonfiber carries it

The default bundle in
[lemonfiber-plugins](https://github.com/lemonfiber/lemonfiber-plugins) pins this
repository by a revision and the digests of its manifest and its tree, and
lemonfiber compiles that tree in.
Until its adapter ships, lemonfiber assembles SABnzbd into the stack it ships as a
stack service, and reaches it through the in-process adapter its `api` names.
Its image is SABnzbd's own, so the release train never tags this repository, and
its pin moves by a reviewed pull request on the catalogue.

## What is here

| Path | What it is |
| --- | --- |
| `plugin.toml` | The part's manifest |
| `config/sabnzbd/` | The files SABnzbd starts with, written into its configuration directory |
| `recordings/sabnzbd/` | The responses its claims are proved against |
| `targets.toml` | The lemonfiber its checks run against |
| `proofs.json` | What those checks said, committed |

## Checks

The plugin verdict is one job of spec's
[`first-party-plugin`](https://github.com/lemonfiber/spec/blob/main/.github/workflows/first-party-plugin.yml)
workflow: the manifest as the lemonfiber `targets.toml` names reads it, every
claim against its recording, and every pinned digest. This repository holds no
Python of its own. `just ci` runs the gates that need nothing but this tree.

## Licence

The data in this repository is under the licence in `LICENSE`.
SABnzbd itself is under GPL-2.0-only and is not distributed here: this
repository names an image, it does not contain one.
