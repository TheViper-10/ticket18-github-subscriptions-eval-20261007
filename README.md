# mock/ticket22-fixture-signals (metadata-only branch)

This branch holds only one nonsecret registration latch file per environment for the
ticket-22 fixture pipeline. It is separate from the build branch `mock/ticket22-cci-fixture`,
so writing a latch here never touches the pinned build commit and never starts a pipeline
(every commit on this branch uses `[skip ci]`).

## What lives here

`latch/register-dev.txt` and `latch/register-qa.txt`, each a single plain-text line
containing only the current CircleCI job number (`$CIRCLE_BUILD_NUM`) for the publish job
that is allowed to proceed. No JSON signal registry, no delivery ids, no receipts, no
per-build file history -- just the one current job number per environment, overwritten on
each real attempt.

Each latch file is written ONLY by the local fixture controller, after it has confirmed the
real subscription registration for that exact running job number, and only then approves the
matching native `register-dev` / `register-qa` CircleCI approval hold. The CircleCI job polls
this exact path over unauthenticated raw HTTPS (no credential) and proceeds only once the
latch's text equals its own `$CIRCLE_BUILD_NUM` exactly; anything else (missing file, stale
job number) holds, and the bounded cap fails closed. A fresh replacement job has a different
job number, so it can never consume a stale latch written for an earlier attempt.

## Who writes here, and when

The local fixture controller only, from its own local CircleCI/DevRev credentials -- never
from inside CI, never a model-authored value. No hosted announcement bridge, no
`T22_ANNOUNCE_URL`, no delivery ids, no receipts, no receipt polling. The [MOCK RELEASE]
Slack announcement itself is posted by the local E2E controller after it observes genuine
CircleCI publish success for the real named job; nothing on this branch participates in that
post.

Not a manual agent resume: the real CircleCI terminal event remains the only thing that wakes
the downstream agent flow. This branch is read-only evidence for the CI job's own polling
loop.
