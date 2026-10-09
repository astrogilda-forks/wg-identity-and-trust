# Authority at dispatch and committed effect

Status: proposed. Comparison record for [issue 13](https://github.com/aaif/wg-identity-and-trust/issues/13).

## Contract

Every trace submitted for this comparison writes:

```json
"contractId": "https://probityai.github.io/agent-evidence-observer/contract/authority-at-dispatch/v1"
```

A reader refuses a trace that omits the identifier or names another version. A change to the required meaning of any field is a new version. The fields and their required meaning are in [CONTRACT.md](https://github.com/probityai/agent-evidence-observer/blob/728de6a1ed5e6d11a0ee6688514c07716030ecfd/interop/authority-unreachable-2026-10-03/CONTRACT.md).

Authority evidence carries `evidenceAgeSeconds` (decision time minus the signed observation time) and `freshnessLimitSeconds` (the deployed limit), as Imran Siddique asked on the list on 3 October. A field a trace leaves unfilled is marked `not_applicable`, `not_emitted` or `not_measured`, kept apart as he asked on 6 October.

## Cases

| Case | Expected invariant | Reference cases |
| --- | --- | --- |
| 1. Delegator revoked after downstream delegation | No dispatch after the revocation reaches the enforcement point | `approver-revoked-at-dispatch`, `revoked-between-intent-and-effect`, `effect-then-authority-revoked` |
| 2. Valid delegation for the wrong object or action | Dispatched bytes equal the approved digest | `media-bytes-mutated`, `platform-content-mutated`, `catalogue-mutated`, `destination-mutated` |
| 3. Stale revocation information | Evidence older than the deployed limit is refused | `authority-evidence-stale` |
| 4. Escalation authority unavailable | No new authority from unavailability | `unreachable-with-prior-fallback`, `unreachable-no-fallback` |
| 5. Authority amplified across hops | Each hop's scope is a subset of the hop before | `delegation-hop-amplified` |
| 6. Identity valid, authority expired | Grant expiry and the original deadline bound dispatch | `fallback-expired`, `late-approval-for-expired-request` |

## Implementation rows

| Implementation | Record | State |
| --- | --- | --- |
| MintID | Operator traces at `public-v2026-10-07` and `public-v2026-10-08.2`, and an independent testnet run of the same trace at `public-v2026-10-08.2` | Decisions measured; committed effect outside the implementation |
| Proofable | `four-case-run-f92faf39a4ba` | Decisions measured; receipts private |
| Alakris | `aaif-publication-reference` `ebd9dcf` and its rerun of the reference cases | Reference cases reproduced; deployed path not measured |

## Running the check

The [workflow](../.github/workflows/authority-at-dispatch.yml) installs the reader at a pinned commit, runs the reference cases, and refuses any trace under `catalogue/authority-at-dispatch/traces/` that does not carry the contract identifier.
