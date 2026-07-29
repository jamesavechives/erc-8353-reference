# ERC-8353 Reference Implementation

Reference implementation of **ERC-8353: Staked Weighted Verification Gate** —
a minimal interface for claims that become trusted only through weighted
third-party verification, with optional stake and revocation.

- Draft: [ethereum/ERCs#1918](https://github.com/ethereum/ERCs/pull/1918)
- Discussion: [Ethereum Magicians](https://ethereum-magicians.org/t/erc-8353-staked-weighted-verification-gate/29194)

## The primitive

A claim carries no trusted status when made. It becomes trusted only when
**third parties** verify it — never through self-endorsement — and the force
of a verification is **weighted by the verifier's own verified depth**, not
counted. Trusted status gates consumption (verify before settle) and is
revocable with an audit trail.

```
              offer()               verify()              settle()
  (none) ─────────────▶ Offered ─────────────▶ Verified ─────────────▶ Settled
                           │                      │                    (terminal)
                           │ revoke()             │ revoke()
                           ▼                      ▼
                                  Revoked  (terminal)
```

## Layout

| Path | What |
|---|---|
| `src/IVerificationGate.sol` | The interface as specified: four transitions, two views, four events |
| `src/VerificationGate.sol` | Minimal abstract base implementing every normative requirement |
| `examples/MarketGate.sol` | Adapter sketch: staked marketplace shape — mandatory stake, arbiter revocation burns it, verifier depth recursive over the gate's own settled claims |
| `examples/IdentityGate.sol` | Adapter sketch: credential-registry shape — zero stake, depth read from an external oracle, issuer-revocable |
| `test/VerificationGate.t.sol` | 15 tests covering both shapes and every red line |

`VerificationGate` fixes the lifecycle and the invariants; policy is exposed
as virtual hooks — `weightOf`, `_requiredStake`, `_promotionThreshold`,
`_canSettle` / `_canRevoke`, `_onSettled`. The two adapters differ only in
those hooks, which is the point: one interface, two very different systems.

## What the base contract guarantees

1. **No self-verification** — a `verify` from the claim's subject or offerer
   reverts; a verifier counts at most once per claim.
2. **Weight, not count** — promotion is a function of `weightOf(verifier)`.
   A swarm of fresh addresses accumulates zero weight (see
   `test_zero_weight_verifiers_never_promote`).
3. **Verify before settle** — `settle` reverts unless the claim is `Verified`.
4. **`Settled` is terminal** — not revocable, by anyone, ever.
5. **Revocation leaves a trace** — the record stays queryable; slashed stake
   is burned and never credited to whoever triggered the revocation.

## Build and test

```bash
forge install foundry-rs/forge-std
forge test -vv
```

Foundry, solc 0.8.20. All 15 tests pass.

## Status

The ERC is a Draft under editor review; this implementation tracks it and may
change with the specification. Not audited — it is a reference for
implementers, not production code.

## License

CC0-1.0. See [LICENSE](LICENSE).
