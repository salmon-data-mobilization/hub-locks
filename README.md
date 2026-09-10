# hub-locks

Claim refs for the salmon data ecosystem hub queue. **There is no code here and
there never should be.**

Every branch under `claim/` is one held work item. Its tip commit contains a
single file, `claim.yaml`, naming the queue item, the agent holding it, and the
lease expiry. The queue itself, the protocol, and the rules live in
[metasalmon](https://github.com/salmon-data-mobilization/metasalmon): see
`HUB.md` and `queue/`.

## Why a push is the lock

An orphan commit has no parent, so it can never be a fast-forward of an
existing branch. The first agent to push a claim creates the branch and wins;
every later push for that item is rejected by the server. That rejection is the
lock. Every subsequent record for a held item (heartbeat, release, hand-back,
reclaim) is a child of the tip the agent just read, so it lands only if nobody
appended in between.

This is git's own compare-and-swap. There is no API call, no database, and no
coordination service anywhere in it.

## Why this branch exists

`main` holds nothing but this file, and it exists so that it, rather than a
claim ref, is the default branch. GitHub makes the first branch pushed to an
empty repository the default, and a default branch cannot be deleted, so
without this the first claim ever made would have been undeletable. Found by
running the proof on 2026-09-10 rather than by reading.

## Housekeeping

Claim refs are not deleted when work finishes. A hand-back appends a `handoff`
record and the ref stays until the work is merged, so that finished work never
looks claimable again.
