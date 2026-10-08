# mock/ticket22-fixture-signals (metadata-only branch)

This branch holds only nonsecret, per-attempt receipt files for the ticket-22 fixture
pipeline's bridge-contract announcement gate. It is separate from the build branch
`mock/ticket22-cci-fixture`, so writing a receipt here never touches the pinned build commit
and never starts a pipeline (every commit on this branch uses `[skip ci]`).

## What lives here

`receipts/<delivery_id>.json`, one file per announcement attempt, where
`delivery_id = <pipeline_id>.<workflow_id>.<job_number>.<environment>` (all values already
visible in the CircleCI UI; no DON, token, person, or private data).

Each receipt is written ONLY by the fixture controller (the parent orchestrator), after it
has confirmed the real Slack post for that delivery, with the exact shape:

```json
{
  "repo": "TheViper-10/ticket18-github-subscriptions-eval-20261007",
  "pipeline_id": "...",
  "pipeline_number": "...",
  "workflow_id": "...",
  "job_number": "...",
  "commit": "...",
  "branch": "...",
  "environment": "dev | qa | prod-alpha",
  "version": "...",
  "delivery_id": "...",
  "posted": true,
  "channel": "C0C4K4EGZT6",
  "slack_ts": "<real Slack message ts>"
}
```

The CircleCI job that posted the announcement polls this exact path over unauthenticated raw
HTTPS and only treats the job as successful once every binding field above matches its own
values exactly, `posted` is `true`, `channel` is `C0C4K4EGZT6`, and `slack_ts` is a real,
non-empty value.

## Who writes here, and when

Not this prep worker. Receipts are written by whichever controller owns the live announce
endpoint (`T22_ANNOUNCE_URL`) once that endpoint is provisioned and a real pipeline run has
actually posted to Slack. This branch is created now with only this README so the branch
exists for the pipeline to resolve against; no receipt file is added until the parent confirms
the bridge is active and the endpoint is safely provisioned.

Not a manual agent resume: the CircleCI terminal event remains the only thing that wakes the
downstream agent flow. This branch is read-only evidence for the CI job's own polling loop.
