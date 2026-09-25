# EthTrust v4 candidate review checklist

The EthTrust Working Group is currently inactive. This is an EEA-hosted exploratory technical review. It does not reactivate the Working Group, create an EEA specification, establish a certification baseline, or represent EEA approval. Feedback may inform a future Working Group process if one is established.

This AI-assisted checklist accompanies the [working draft](./WORKING-DRAFT.md), [README](./README.md) and [HTML review page](./index.html). All candidate IDs and levels are provisional. `MUST`, `[S]`, `[M]`, `[Q]` and `[GP]` test possible future language; this is not an approved certification checklist. Independent human security review and a future EEA adoption process would be required.

Review each row independently. Record the evidence, result and reviewer; justify any inapplicability. Absence of `tx.origin` does not make M-v4-02a inapplicable. Certification references in M-v4-09 describe the proposed future process only.

| Candidate | Level | Review question and evidence | Result |
|---|---|---|---|
| S-v4-01: No Account-Code Introspection | S | Does Tested Code avoid `EXTCODESIZE`, `EXTCODEHASH`, `address.code.length` and `address.codehash`, unless the M-v4-01a override is met? Record tool scope and findings. | Pending |
| M-v4-01a: Verify Account-Code Introspection | M | Is each purpose documented with independent security controls, without using introspection alone to prove EOA status, non-programmability or trust, or caching account classification across transactions? Is any same-transaction permission limited to delegation-indicator stability? Test constructor calls, inspection before/after deployment in one transaction, and delegation changes between transactions. | Pending |
| Q-v4-02: Verify tx.origin Under Delegated Account Execution | Q | For each use of `tx.origin`, do the existing v3 obligations and delegated-origin tests hold without treating equality with `msg.sender` as proof of topmost execution, non-programmability or protection from multi-call/reentrancy? | Pending |
| M-v4-02a: Verify Delegated Account Balance and Nonce Assumptions | M | Independently of `tx.origin`, identify native balance/EOA nonce dependencies. Test a called delegated account spending its balance and creating contracts to change its nonce during execution. If there is no relevant dependency, record why; do not skip based on absence of `tx.origin`. | Pending |
| M-v4-03: Secure EIP-7702 Delegation Logic | M | Are operation authorizations protected against replay and bound to chain/policy, target, calldata, value, gas/griefing limits and authority as applicable? Record reasons for omissions and distinguish operation signatures from protocol delegation authorization. | Pending |
| M-v4-04: Protect Delegated Account Initialization and Storage | M | Is privileged initialization authenticated and front-run safe? Are storage collisions/migrations and delegation changes reviewed, and are delegated privileges limited? | Pending |
| GP-v4-11: Test Counterparty Behavior for Delegated Accounts | GP | Do code-gated receiver paths handle delegated accounts, including missing ERC-721/1155 or applicable ERC-777/custom callbacks? Review differences between internal code observations and the externally visible indicator. | Pending |
| M-v4-05: Verify Transient Storage Lifetime | M | Are lifetime and isolation explicit, repeated/reentrant calls safe, and call-scoped values restored/cleared? Test normal return, caught failure and propagated revert, including inner-call rollback and surviving pre-frame values. Do temporary mappings avoid assumptions of frame-local memory or cross-transaction persistence? | Pending |
| M-v4-06: Verify Transient Storage Under DELEGATECALL | M | Do `DELEGATECALL`/`CALLCODE` modules account for the caller's shared namespace, collision-resistant slot derivation and documented intentional sharing? Test collisions and rollback. | Pending |
| Update: No selfdestruct() / Protect Self-destruction | Existing S/M paths | Are the v3 prohibition, authorization and documentation duties retained? Does allowed-use review account for value transfer, self-beneficiary behavior and the creation-transaction exception without relying on later code/storage deletion or later CREATE2 replacement? | Pending |
| M-v4-07: Bound Security-Critical Operations to the Transaction Gas Cap | M | Can each critical atomic operation complete under the target network/fork cap in documented worst-case state? Test safe chunking or justify that a documented limitation cannot cause unacceptable DoS/asset locking. | Pending |
| M-v4-08: Verify Cryptographic Precompile Results | M | Verify target-fork support, input encoding/bounds, call failure handling, exact output and domain/replay protections. P-256 requires exactly 32-byte integer 1, rejects empty/unexpected output and handles signature malleability if signatures/hashes are uniqueness keys. For BLS addition, establish required subgroup membership of every external point; test malformed/empty inputs, points and infinity as appropriate. Do not infer canonical MSM scalars or uniqueness from raw unreduced bytes. | Pending |
| M-v4-09: Check Compiler Security Advisories at Certification Time | M | Would a final advisory refresh immediately before issuance establish one UTC certification cutoff, repeated if issuance is delayed? Assess advisories published on/before that cutoff against all compiler versions/configurations; block unresolved applicability or unavailable sources. Record cutoff/issuance/consultation times, exact compiler build, configuration, Tested Code source/artifacts, advisory URLs, immutable revisions or hashed snapshots, and applicability/mitigation evidence. Later advisories should trigger reassessment when relevant without rewriting the historical determination. | Pending |
| GP-v4-10: Reassess After Security-Relevant Network Upgrades | GP | Has a prior certification been reconsidered when a fork changes a depended-on opcode, precompile, account model, gas bound or execution semantic? Record target network/fork and the affected assumptions. | Pending |

The advisory evidence in M-v4-09 uses the same issuance-stage cutoff as the working draft, not an earlier review date. Existing per-bug `[S]` checks remain; extending their list is an open review question. The working draft records the disposition of [EthTrust-public #15-#20](./WORKING-DRAFT.md#4-reconcile-with-v3-and-public-feedback).

## Review process still required

- Establish or reactivate the appropriate EEA process and obtain independent human security review.
- Reconcile candidates with v3, validate levels and define objective tests for `[S]` predicates.
- Validate the through-Fusaka inventory and target-fork tooling; add positive and negative cases for the candidates.
- Validate reproducible compiler-advisory evidence and record public comment dispositions.
- Obtain applicable EEA approval and produce a complete specification and checklist before any future adoption.

## GBBC RMF context

The [RMF section](./WORKING-DRAFT.md#5-gbbc-capital-markets-risk-mitigation-framework) is entirely non-normative and adds no certification check. Any formal mapping needs an identified RMF edition, exact control IDs and review with GBBC/RMF maintainers through the appropriate future process. No endorsement by GBBC or its participants is implied.
