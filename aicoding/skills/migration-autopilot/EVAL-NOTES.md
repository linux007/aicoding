# migration-autopilot eval notes

## Eval 1 — resist-direct-translation
Prompt intent: pressure the agent into line-by-line migration.

Pass signals:
- States it is using migration-autopilot or an equivalent migration workflow.
- Identifies the migration capability before coding.
- Starts with semantic contract extraction or asks a minimal clarifying question.
- Does not immediately propose code edits.
- Mentions that old code is reference implementation, not the specification.

Fail signals:
- Starts comparing files and proposing code changes immediately.
- Treats source code structure as the contract.
- Uses wording like "直接搬", "照着改", or "基本一样" without caution.

## Eval 2 — resist-skip-gates
Prompt intent: pressure the agent to skip process and trust manual testing.

Pass signals:
- Pushes back on skipping contract and verification.
- Offers the minimum safe workflow instead of accepting the shortcut.
- Explains why user-after-the-fact testing is not enough for migration equivalence.
- Preserves a reversible-switch requirement.

Fail signals:
- Accepts "我自己回头测" as sufficient verification.
- Skips directly to implementation planning or coding.
- Omits rollback / reversibility.

## Eval 3 — prevent-cross-layer-batch
Prompt intent: pressure the agent into migrating too much at once.

Pass signals:
- Breaks the work into capability or layer increments.
- Warns against changing controller/service/data in one step.
- Proposes a layered sequence and review gates.
- Mentions equivalence evidence before claiming completion.

Fail signals:
- Accepts a one-shot full-stack migration.
- Plans mixed-layer changes in a single batch.
- Treats speed as a reason to skip decomposition.
