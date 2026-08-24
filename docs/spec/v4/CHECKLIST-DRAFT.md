# EthTrust v4 — Candidate Requirements Checklist

## AI-generated Working Draft — HUMAN REVIEW REQUIRED

This checklist accompanies `WORKING-DRAFT.md`. It is an AI-generated first pass and is **not an approved EEA EthTrust checklist or certification baseline**. Requirement IDs and levels are provisional until reviewed and adopted by the EEA EthTrust Security Levels Working Group.

| Candidate | Level | Review question | Status |
|---|---:|---|---|
| Do Not Use Account Code as a Security Boundary | S-v4-01 | Does the Tested Code avoid using code-size/code-hash/EOA classification as the sole security boundary for trust, non-programmability, reentrancy, or authorization? | ☐ |
| Verify `tx.origin` Under Delegated Account Execution | Q-v4-02 | If `tx.origin` is used, has behavior been reviewed and tested when the origin has EIP-7702 delegated code, including multi-call/reentrancy/atomic-execution assumptions? | ☐ |
| Secure EIP-7702 Delegation Logic | M-v4-03 | If the Tested Code implements/verifies/relays delegated operations, are replay, nonce, chain, target, calldata, value, gas and authority bindings correctly protected as applicable? | ☐ |
| Protect Delegated Account Initialization and Storage | M-v4-04 | Is initialization authenticated and front-run safe, storage collision-safe, upgrade/delegation change controlled, and delegated privilege least-privileged? | ☐ |
| Verify Transient Storage Lifetime | M-v4-05 | For security-sensitive `TSTORE`/`TLOAD`, is same-transaction lifetime documented and are later calls protected from stale transient values? | ☐ |
| Verify Transient Storage Under `DELEGATECALL` | M-v4-06 | Where transient storage and delegated execution interact, are ownership and slot collisions explicitly reviewed and tested? | ☐ |
| Update `SELFDESTRUCT` Semantics | S/M update | Does any allowed use avoid relying on obsolete code/storage deletion, `CREATE2` redeployment, or old Ether-burn semantics? | ☐ |
| Bound Security-Critical Operations to Transaction Gas Cap | M-v4-07 | Can security-critical atomic operations complete under the intended network's transaction gas cap in worst-case documented state, or is safe chunking available? | ☐ |
| Verify Cryptographic Precompile Results | M-v4-08 | Are precompile input/output semantics checked, call success distinguished from verification success, and replay/domain/malleability assumptions handled? | ☐ |
| Check Current Compiler Security Advisories | S-v4-09 | Were current Solidity security advisories/known bugs checked at certification time and the compiler/configuration/source/date recorded? | ☐ |
| Reassess After Security-Relevant Network Upgrades | GP-v4-10 | Has a prior certification been reconsidered after a fork changed an opcode, precompile, account model, gas bound, or semantic on which the Tested Code depends? | ☐ |

## Required review before this checklist can become normative

- [ ] Validate requirement levels `[S]`, `[M]`, `[Q]`, `[GP]` against the existing EthTrust conformance model.
- [ ] Map each candidate to existing v3 requirements and remove duplication.
- [ ] Define objective/automatable tests for `[S]` requirements.
- [ ] Add concrete negative/positive test cases for EIP-7702, EIP-1153, EIP-6780, EIP-7825 and EIP-7951.
- [ ] Validate static-analysis/tool support for `TLOAD`, `TSTORE`, `CLZ`, delegation indicators and new precompiles.
- [ ] Validate the compiler-advisory source and archival evidence expected from certifiers.
- [ ] Resolve relevant items from `EntEthAlliance/EthTrust-public#7`.
- [ ] Complete public review in `EntEthAlliance/EthTrust-public`.
- [ ] Replace placeholder IDs with final v4 requirement anchors after Working Group approval.
