# Delegation lifecycle test cases

Status: proposed. Test cases for the two use cases in
[issue 18](https://github.com/aaif/wg-identity-and-trust/issues/18): disputed
revocation across a delegation tree, and re-registration under a new identifier.

Each case asks one question: does this presenter hold this delegated authority at
the time the relying party observed the presentation? A case is a directory of
signed records (identifier registrations, grants, revocations, disputes,
resolutions, identifier links, retirements) and one presentation signed by the
presenter and countersigned by the relying party, with the policy that judges
them in `case.json`. Every record is a DSSE envelope signed with Ed25519 test
keys anyone can re-derive. [INDEX.md](INDEX.md) gives every case's situation,
expected outcome and rule; `MANIFEST.json` holds the same outcomes for a
program to read.

The cases are copied byte for byte from
[agent-evidence-vectors](https://github.com/probityai/agent-evidence-vectors/tree/v0.19.0/vectors-delegation-lifecycle)
release `v0.19.0`. The [workflow](../../.github/workflows/delegation-lifecycle.yml)
installs the verifier `aee-verify` pinned to that release and checks that every
case gets its expected outcome. To run it yourself:

```sh
go install github.com/probityai/agent-evidence-vectors/cmd/aee-verify@v0.19.0
aee-verify test-cases/delegation-lifecycle
```

To test another verifier, run it over each `cases/<id>/` directory and compare
its answer with `expected` in `MANIFEST.json`.

## Revocation across a delegation tree

An organisation delegates to a planner, which delegates to a worker and a
sibling; the worker delegates to a sub-worker.

| Case | Outcome | What it tests |
| --- | --- | --- |
| `tree-chain-valid` | accept | the baseline chain, nothing revoked |
| `tree-parent-revoked-child-presented` | reject | the planner's grant is revoked and the worker presents its child grant afterwards |
| `tree-child-presented-before-revocation` | accept | the time rule: a revocation applies from its `effective_at`, against the relying party's `observed_at` |
| `tree-sibling-of-revoked-child` | accept | revoking the worker leaves its sibling's grant standing |
| `tree-grandchild-under-revoked-parent` | reject | the cascade reaches two levels down |
| `tree-child-revocation-keeps-parent` | accept | revoking a child does not revoke its parent |
| `tree-revocation-backdated` | reject | a revocation issued late applies from its stated effective time |
| `tree-revocation-without-standing` | accept | a revocation signed by someone outside the chain above the grant is ignored |
| `tree-revocation-forged` | accept | a revocation whose signature does not verify stops nothing |
| `tree-revocation-incomplete` | reject | a revocation without stated grounds still stops authority, and the refusal says so |
| `tree-conflicting-revocations` | reject | of two revocations with standing, the earliest effective time applies |
| `tree-dispute-open` | reject | a worker cut off by the cascade disputes; the dispute does not suspend the revocation, and the refusal says it is open |
| `tree-dispute-confirmed` | reject | the dispute is resolved by confirming the revocation |
| `tree-resolution-without-standing` | reject | the disputing worker cannot resolve its own dispute |
| `tree-dispute-restored-by-new-grant` | accept | a dispute resolved in the planner's favour restores it through a replacement grant |
| `tree-revoked-grant-after-restoration` | reject | the revoked grant itself stays revoked |
| `tree-cascade-victim-after-restoration` | reject | the worker's old grant still names the revoked parent |
| `tree-cascade-victim-reissued` | accept | the worker's grant re-issued under the replacement |
| `tree-grant-tampered` | reject | a grant edited after signing |
| `tree-presentation-unattested` | reject | a presentation the relying party did not countersign |
| `tree-chain-skips-a-grant` | reject | a chain with a grant missing |
| `tree-root-not-trusted` | reject | a top-level grant from a principal the policy does not trust |
| `tree-record-not-json` | no verdict | a record that is not JSON |

## Re-registration under a new identifier

An organisation grants authority to `agent-v1`, which is re-registered as
`agent-v2`.

| Case | Outcome | What it tests |
| --- | --- | --- |
| `rereg-linked-successor` | accept | a link signed by the old key, the new key and the registry carries the grant to the new identifier |
| `rereg-no-link` | reject | a new identifier with no link holds nothing |
| `rereg-link-signed-by-wrong-key` | reject | the new identifier signs in the old identifier's place |
| `rereg-link-not-registry-attested` | reject | a link without the registry's signature |
| `rereg-two-old-one-new` | reject | two old identifiers linked to one new one |
| `rereg-one-old-two-new` | reject | one old identifier linked to two new ones |
| `rereg-link-after-old-revoked` | reject | a link made after the registry revoked the old identifier for key compromise |
| `rereg-chain-old-middle-new` | accept | links followed hop by hop, old to middle to new |
| `rereg-chain-middle-revoked` | reject | the middle identifier was revoked before its onward link |
| `rereg-old-presents-after-retirement` | reject | the old identifier presents after the registry retired it |
| `rereg-old-presents-before-retirement` | accept | the old identifier presents after the link and before its retirement |
| `rereg-revocation-follows-link` | reject | a revocation of the grant reaches the successor |

## Where the rules come from

Standing, cascade and irreversibility follow the
[UCAN revocation specification](https://github.com/ucan-wg/revocation): the issuer
of a grant, or any issuer above it in the chain, may revoke it; a revocation
breaks every chain through the revoked grant and no other; and a revocation is
never undone, so a restoration is a new grant. For the disputed revocation, these
cases state the choice the use case asks a design to state: containment over
recoverability.

Rules with no precedent are marked `Proposed:` in `INDEX.md`: that a dispute does
not suspend a revocation, that a resolution needs standing, that a restoration
does not cascade, that a revocation applies from its stated effective time, that
a revocation without grounds still stops authority, that links are one to one,
and that an old identifier stays usable until it is retired. These are the rules
the group can change; each change moves the outcome of the cases that name it.
