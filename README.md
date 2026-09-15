# plugin-template

**The plugin you copy.** One manifest, its recorded responses, and a workflow
that runs the same commands the catalogue's CI runs — so the first thing you see
is the bar you will be held to, and it is already green.

```sh
gh repo create my-plugin --template lemonfiber/plugin-template
```

Then replace the service, re-record the fixtures, and push.

## Why it describes something real

[`F10-R7`](https://github.com/lemonfiber/spec/blob/main/10-functional/features/f-extensibility/f10-authoring.md)
asks for a template that **validates and proves unmodified**. A template whose
proofs are invented would satisfy that sentence and teach an author to invent
proofs, so this one describes Kavita — a comics, manga and ebook reader — and
every recording under `fixtures/` came off that image at the digest
`plugin.toml` names.

Nothing here is a recommendation to install Kavita. It is here so that the
proofs are proofs.

## What is in it

| File | What it is |
|------|------------|
| `plugin.toml` | The whole of what lemonfiber will act on. Identity, the service, the capability it claims, the proofs, the checks it contributes. |
| `fixtures/*.json` | Recorded responses, so everything provable is provable with no live instance anywhere |
| `targets.toml` | Which lemonfiber release this is validated against. A CI fact, not a manifest one. |
| `proofs.json` | The record the release train re-reads. Generated; CI fails if it is stale. |
| `.github/interim/` | The stand-in harness, until `lemonfiber plugin validate` exists. Not part of what an operator installs. |

## The shape, in the order the manifest carries it

**The service.** Which image runs, at which digest, on which port, and whether
that port is an admin surface or a household one. Not how the container is
assembled — lemonfiber writes that, which is what makes *what can this plugin
reach* answerable from the format rather than from the instance.

**What it can do.** `provides` carries two kinds of name and no third. A **core**
name comes from lemonfiber's published capability vocabulary and means the
contracted thing that vocabulary defines; something asking for it can be answered
by this plugin, which is what makes a plugin useful rather than merely readable.
A **namespaced** name — `kavita:opds` — is this plugin's own and is inert until
something asks for it.

**The claim.** A core capability is *demonstrated, not asserted*. The vocabulary
says what must be shown; the `[[claim]]` says where to ask it on this service and
what the answer must be, with a recording of that answer. Leave a probe unbound,
or bind one with a weaker expectation than the probe permits, and the manifest is
refused naming the probe.

**The proofs.** What must hold before this is installed. Every one asserts a
body, never only a status — Docker publishes a port by putting a proxy in front
of it, and that proxy accepts a connection before knowing whether anything inside
is listening. One of the two proofs here exists only to record that: asked for a
path it does not implement, this service answers `200` and its single-page app.

**The contributions.** A row in a register lemonfiber already runs. The doctor
already runs checks independently, bounds each one, keeps `unverified` distinct
from `pass`, and carries a remedy on anything that does not pass; a contributed
check is another row in it, attributed to this plugin wherever it appears. No
code is contributed and none can be — there is nothing for contributed code to
*be*.

The check here and the `guarded` probe ask almost the same question, and the
difference is the point: a probe is asked once, at install; a check is asked
every time the doctor runs. An upgrade, a reconfiguration or a reverse proxy in
front of the service are all ways for it to stop refusing what it used to refuse.

## Running the checks

```sh
python3 .github/interim/validate.py --self-test          # the gate refuses what it should
python3 .github/interim/validate.py                      # the manifest, offline
python3 .github/interim/validate.py --published <dir>    # and the rules lemonfiber decides
python3 .github/interim/vocabulary_gate.py               # fetches those artefacts and does both
python3 .github/interim/prove.py                         # everything declared, against the recordings
python3 .github/interim/prove.py --against http://127.0.0.1:5000
python3 .github/interim/image_gate.py                    # the digest, its tag, its signature state
python3 .github/interim/schema_gate.py                   # fails the day the real schema lands
```

`validate.py` on its own decides nothing that depends on knowing what lemonfiber
publishes, and **says which rules it did not decide** rather than passing them.
`vocabulary_gate.py` is the half that asks.

## Recording a fixture

A recording is a moment, and it is reviewed like the manifest: a fixture nobody
can read is a place for something to hide. Record with the service running at the
digest `plugin.toml` names, and write down what state it was in:

```json
{
  "recorded_from": "docker.io/jvmilazz0/kavita@sha256:…",
  "note": "What this instance was, and why this response is worth keeping",
  "request":  { "method": "GET", "path": "/api/health" },
  "response": { "status": 200, "headers": { "content-type": "text/plain" },
                "json": null, "body_starts_with": "Ok" }
}
```

Moving the image pin means re-recording every fixture against the new image in
the same change. A recording that describes a different image is worse than no
recording.

## The recipe this does not declare, and what one looks like

`[[recipe]]` is in the format and is checked. It is **not** declared here, and the
reason is worth copying rather than the block: a manifest declaring one asks for
`recipe.run` by name, so a lemonfiber that cannot run recipes refuses it — and a
template an author copies should not be one that installs nowhere.

What one looks like, when the capability is offered:

```toml
[requires]
capabilities = ["service.add", "service.health.http", "doctor.contribute", "recipe.run"]

[[recipe]]
id    = "adopt-the-library-the-stack-already-fills"
title = "Point it at the comics on disk instead of asking the operator to"
why   = "A reader with no library configured is a reader nobody can read anything in."

[[recipe.step]]
id      = "sign-in"
call    = { method = "POST", to = "kavita", path = "/api/account/login" }
expect  = { status = 200 }
capture = [{ name = "token", from = "json.token", origin = "stack-service" }]

[[recipe.step]]
id     = "create"
call   = { method = "POST", to = "kavita", path = "/api/library/create",
           headers = { Authorization = "Bearer {{token}}" } }
expect = { status = 200 }

[[recipe.pair]]
value = "token"
to    = "kavita"
```

Three rules do the work, and the validator holds all three without running
anything. Every destination is a **name** — a service in this stack or a DNS name
outside it — never an address, a range or a bare host port, because a declared
address is a declared address wherever it points and this machine sits beside a
router's administration page. Every substitution refers to something an earlier
step captured. And every value that could reach a destination has a
`[[recipe.pair]]` behind it: a captured token going anywhere no pair permits is a
validation failure found before a call is made, not after two have landed.

The verbosity is the point. Three captures and two destinations are six possible
flows, most of which a plugin will never want — and the ones it does not declare
are the ones nobody has to wonder about afterwards.

## What is deliberately absent

`[[secret]]` and `[[override]]`, because capturing a value and changing a bundled
setting are recipe verbs and recipes arrive with `F8`.

A dashboard widget, because a widget reads a service's API with a credential, and
a credential needs a recipe. A dashboard panel or a command of its own, because
there is no point to declare either at. A minimum lemonfiber version, because a
manifest carries none — `[requires].capabilities` says what this needs, and an
unmet requirement is refused by naming the capability rather than a number that
cannot say which one.

## Licence

MIT. See [LICENSE](LICENSE).
