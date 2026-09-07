# Execution model and lifetime

## Execution model

- One isolated runtime pair per plugin. No cross references.
- Budgets: infinite loops die at 100ms. Chains get 8ms (previous output
  plus warning past it). Memory cap 32MB, stack 512KB.
- Non-`0` `GUMICORD_SAFE_MODE` starts without loading any plugin.

## Patch chain in seven rules

- Bottom-up traversal (children before parents).
- Matching uses the pre-apply stable ID.
- No recursing into output. Output is final.
- Same-node patches chain in registration order.
- Exceptions revert their node. Nothing else is touched.
- Plugins chain in load order. Each sees the previous output.
- Patches are pure functions. No side effects.

Without the no-recursion rule, wrapping recurses forever. A wrapped child
matches the same ID again, wraps again, and the stack runs out. Finishing
children first leaves nothing left to visit after patching self.

Matching on pre-apply IDs matters because matching on output IDs would
make the patch set depend on run order. Matching on pre-apply IDs fixes
the set by tree shape alone.

Across plugin boundaries the whole subtree passes on, so later plugins
scan earlier output. Later patches reacting to earlier insertions is
intended.

## Lifetime

1. Dropped into the folder, found at startup, loaded.
2. Capabilities ask approval on first sight. Until granted, that
   permission counts as absent.
3. Afterwards patches run on every draw. Heavy work cannot live here
   (killed at 100ms).
4. Repeated breakage disables automatically with notice. Re-enable from
   settings.
5. Denied ones never load. Re-approve from settings on second thought.

## Failure counting

- Failures revert their branch and count under the plugin ID. 100 per 60
  seconds disables the plugin with notice. Re-enable from settings.
- The first failure reverts only its node and lands in the log.
- Unacceptable output counts as one too. Unknown stable IDs or broken
  shapes discard that frame's output of that plugin, reuse the pre-apply
  subtree, and record a failure. Partial repair would leave unpredictable
  survivors.
- Broken-patch failures happen per frame, so 100 per 60 seconds stops
  breakage within 2 seconds while sparing sporadic failures.
