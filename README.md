# Agentic Pay - conceptual Argent draft (Axiom mapping)

Status: **compiled successfully against the real `argentc` compiler**
(`michaelsutton/argent` @ master, built locally with rustc 1.91.1,
`cargo run -- build app.ag --out build/`). Verified by first compiling the
real `tickets.ag` and `examples/stones/app.ag` to confirm the toolchain
itself works, then compiling this draft the same way. All four `.ag` files
parse and produce structurally valid Silverscript output
(`BuyerAgent.sil`, `Escrow.sil`, `ProviderAgent.sil`).

**Important caveat on what "compiles" means here:** per the repo's own
README, Argent currently has "no full Argent typechecker, minimal
diagnostics." It validates routing structure (`become`/`consumes`/`emits`,
output-template checks) but does **not** verify that function calls inside
`require()` bodies are real Silverscript builtins. Concretely: the
`timeout_elapsed(timeout_height)` call in `escrow.ag` (see caveat below)
compiled and was passed through unchanged into the generated `.sil` file -
that does not mean it's a valid Silverscript function, only that Argent
didn't check. A real Silverscript-level compile (which I don't have access
to here) would be the next real validation step.

## What changed vs. the earlier (Grok-assisted) draft

The earlier version invented syntax that doesn't exist in the real compiler:

- `delegate ... consumes oracle: Oracle` as an inline parameter - wrong.
  Real grammar: `consumes` is its own block *after* the parameter list,
  before `emits`: `entry foo(...) consumes { name: Actor; } emits {...} { ... }`.
  Verified directly in `examples/stones/player.ag` and `settle.ag`, and
  confirmed by successful compilation of this draft.
- `require(observes oracle with delivery_proof_template)` - this pattern
  does not exist in the implemented compiler. The one file that sketches
  cross-covenant observation (`examples/icc/minter_proxy_observer.ag`)
  is explicitly commented by Sutton: *"Syntax sketch only: observed foreign
  covenant lanes are not compiled yet."* So oracle-verified delivery proof
  inside the contract is not currently possible - this draft uses a
  provider-signature release instead (see escrow.ag for the tradeoff).
- `multi_sig_verify(...)`, `oracle_validate_price(...)`, `verify_permission(...)`,
  `current_block()` - none of these appear anywhere in the real source or
  examples. Removed. Replaced with the patterns real examples actually use:
  `blake2b(pk) == owner`, `checkSig(sig, pk)`, plain `require()` comparisons
  on state fields and `.value`.
- A separate `logs.igra` actor - doesn't exist and isn't needed; the prunable
  UTXO/become history is the audit trail. Removed as its own file, noted as
  a comment in escrow.ag instead.
- A separate `intent.igra`/`auth.igra` actor pair - collapsed into the
  `BuyerAgent.purchase` entry's own `require()` checks, since Argent has no
  primitive for "intent" or "auth" as distinct covenants; those are Axiom-layer
  vocabulary, not Argent-layer constructs.

## What is still honestly unverified

- `timeout_elapsed(timeout_height)` in `escrow.ag` - compiled (Argent doesn't
  typecheck function calls), but is **not confirmed to be a real Silverscript
  builtin**. No locktime/height function appears in any real Argent example
  or in `src/`. Needs checking against an actual Silverscript spec/compiler,
  which wasn't available in this environment.
- `ProviderAgent.received_total` aggregation - `escrow.ag` overwrites rather
  than accumulates. A correct version would `consumes { prev: ProviderAgent; }`
  and write `prev.received_total + amount`. Flagged, not fixed, since no real
  example shows this exact accumulate-into-existing-actor pattern to copy from.
- Whether `release()` being a single-provider-signature claim is an
  acceptable security model for real funds, or whether it needs a 2-of-2
  buyer+provider delegate check, is a design decision, not a syntax question.

## Honest mapping vs. the original 7-step Agentic Pay diagram

| Axiom step  | Where it actually lives in this draft                          |
|-------------|------------------------------------------------------------------|
| intent.igra | `BuyerAgent.purchase` params (provider, amount, timeout_height)  |
| auth.igra   | `require(checkSig(...))` + spend-policy checks in same entry     |
| data.igra   | **not modeled** - no oracle/observes available yet (see above)   |
| escrow.igra | `Escrow` actor's state, created by `purchase`'s `become`          |
| pay.igra    | the value transfer inside `Escrow.release`'s `become`             |
| settle.igra | same `become` - BlockDAG confirmation is what makes it final      |
| logs.igra   | implicit - the UTXO/become chain itself, not a separate actor     |

Five of seven steps map cleanly and now compile against the real tool.
`data.igra` (price/oracle lookup) has no real implementation path yet given
`observes` is unshipped. That's a real gap in the Axiom-to-Argent mapping
worth being upfront about, not papering over with invented syntax.
