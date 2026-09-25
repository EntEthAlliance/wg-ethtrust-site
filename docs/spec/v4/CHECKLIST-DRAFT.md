# EthTrust v4 — Candidate Requirements Checklist

## AI-generated Working Draft — HUMAN REVIEW REQUIRED

This checklist accompanies `WORKING-DRAFT.md`. It is an AI-generated first pass and is **not an approved EEA EthTrust checklist or certification baseline**. Requirement IDs and levels are provisional until reviewed and adopted by the EEA EthTrust Security Levels Working Group.

**Revised 25 September 2026** to match `WORKING-DRAFT.md`'s incorporation of `EntEthAlliance/EthTrust-public#15`-`#20` (Ignacio Freire, Olympix): `S-v4-01` split into `S-v4-01`/`M-v4-01a`, new `GP-v4-11` row added, `S-v4-09` renamed `M-v4-09`, and the `Q-v4-02`/`M-v4-05`/`M-v4-06`/`M-v4-08` questions updated.

| Candidate | Level | Review question | Status |
|---|---:|---|---|
| No Account-Code Introspection | S-v4-01 | Does the Tested Code avoid `EXTCODESIZE`/`EXTCODEHASH`/`address.code.length`/`address.codehash` outside the `[M-v4-01a]` Overriding Requirement? | ☐ |
| Verify Account-Code Introspection | M-v4-01a | Where introspection is used, is the result treated as valid only for the duration of the current transaction, never cached or persisted across transactions? | ☐ |
| Verify `tx.origin` Under Delegated Account Execution | Q-v4-02 | If `tx.origin` is used, has behavior been reviewed and tested when the origin has EIP-7702 delegated code, including the balance-only-decreases and nonce-stable-within-tx invariants EIP-7702 breaks? | ☐ |
| Secure EIP-7702 Delegation Logic | M-v4-03 | If the Tested Code implements/verifies/relays delegated operations, are replay, nonce, chain, target, calldata, value, gas and authority bindings correctly protected as applicable? | ☐ |
| Protect Delegated Account Initialization and Storage | M-v4-04 | Is initialization authenticated and front-run safe, storage collision-safe, upgrade/delegation change controlled, and delegated privilege least-privileged? | ☐ |
| Test Counterparty Behavior for Delegated Accounts | GP-v4-11 | Has behavior been reviewed for counterparties whose account has delegated code, especially `onERC721Received`/`onERC1155Received`/ERC-777 receiver hooks gated on code presence? | ☐ |
| Verify Transient Storage Lifetime | M-v4-05 | For security-sensitive `TSTORE`/`TLOAD`, is same-transaction lifetime documented, are later calls protected from stale transient values, is revert rollback accounted for, and is transient storage never used as an in-memory-mapping substitute? | ☐ |
| Verify Transient Storage Under `DELEGATECALL` | M-v4-06 | Where transient storage and delegated execution interact, is the shared-namespace rule under `DELEGATECALL`/`CALLCODE` reflected in slot derivation, and are collisions explicitly reviewed and tested? | ☐ |
| Update `SELFDESTRUCT` Semantics | S/M update | Does any allowed use avoid relying on obsolete code/storage deletion, `CREATE2` redeployment, or old Ether-burn semantics? | ☐ |
| Bound Security-Critical Operations to Transaction Gas Cap | M-v4-07 | Can security-critical atomic operations complete under the intended network's transaction gas cap in worst-case documented state, or is safe chunking available? | ☐ |
| Verify Cryptographic Precompile Results | M-v4-08 | Are the exact EIP-7951 return values checked, is P-256 signature malleability handled where used as a uniqueness/replay key, and is EIP-2537 G1ADD/G2ADD subgroup membership checked where required? | ☐ |
| Check Compiler Security Advisories at Certification Time | M-v4-09 | Were Solidity security advisories/known bugs published on or before the certification date checked, and the compiler/configuration/source/date recorded? | ☐ |
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
