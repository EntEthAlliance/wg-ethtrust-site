# EEA EthTrust Security Levels Specification Version 4

## AI-generated Working Draft, NOT AN APPROVED EEA SPECIFICATION

**Draft date:** 24 August 2026 (revised 25 September 2026)  
**Status:** Initial technical working draft for human and public review  
**Baseline:** EEA EthTrust Security Levels Specification v3 (March 2025)  
**Tracking issue:** https://github.com/EntEthAlliance/wg-ethtrust-site/issues/6

**25 September 2026 revision:** incorporates the first substantive public review received, Ignacio Freire (Olympix), `EntEthAlliance/EthTrust-public#15` through `#20`. Affects `[S-v4-01]`/new `[M-v4-01a]`, `[Q-v4-02]`, new `[GP-v4-11]`, `[M-v4-05]`, `[M-v4-06]`, `[M-v4-08]`, and `[S-v4-09]` (renamed `[M-v4-09]`). Each change is marked inline with a rationale note; this is still an unreviewed AI-assisted draft, not a Working Group determination.

> **IMPORTANT REVIEW NOTICE**
>
> This initial Version 4 working draft was generated with AI assistance (OpenAI ChatGPT) from the published EthTrust v3 specification, Ethereum Improvement Proposals and Ethereum network-upgrade documentation, and public materials for the GBBC Capital Markets Risk Mitigation Framework (RMF).
>
> **It has not yet been technically reviewed, approved, or adopted by the EEA EthTrust Security Levels Working Group or by the Enterprise Ethereum Alliance. It MUST NOT be represented as an EEA standard, an approved certification baseline, or a statement endorsed by GBBC or any RMF participant.**
>
> Every proposed normative requirement in this document requires human security review. The Working Group should validate the EIP analysis, requirement level, wording, testability, interactions with existing v3 requirements, and references before publication. Public review should occur through the `EntEthAlliance/EthTrust-public` repository before a final v4 release.

---

## Abstract

This working draft proposes updates to EEA EthTrust Security Levels following changes to Ethereum mainnet since the technical baseline used for Version 3.

Version 3 was published in March 2025, but states that its compiler-security review was finalized in late 2023 and explicitly identifies `SOL-2023-3` as the latest Solidity compiler bug incorporated into its normative requirements. Since that baseline Ethereum has activated Dencun (13 March 2024), Pectra (7 May 2025), and Fusaka (3 December 2025).

The most significant smart-contract security changes for EthTrust are:

1. **EIP-7702, Set Code for EOAs**, which changes long-standing assumptions about externally owned accounts and delegated execution.
2. **EIP-1153, transient storage**, which introduces transaction-scoped state with security implications for reentrancy, state lifetime, and `DELEGATECALL`.
3. **EIP-6780, changed `SELFDESTRUCT` semantics**, which invalidates explanations and patterns based on deletion of code and storage after deployment.
4. **EIP-7825, per-transaction gas cap**, which creates a hard execution bound relevant to liveness, administrative functions, recovery operations, settlement batches, and denial-of-service analysis.
5. **New cryptographic precompiles**, particularly EIP-2537 (BLS12-381) and EIP-7951 (secp256r1/P-256), which require correct result handling and signature-domain assumptions.
6. **Compiler/security advisories after the v3 cutoff**, which should be incorporated through an evergreen review requirement rather than waiting for the next static specification release.

This draft also proposes a non-normative mapping between EthTrust and the **GBBC Capital Markets Risk Mitigation Framework (RMF)**. EthTrust should remain a focused smart-contract security standard; it should not become a general institutional risk-management framework. The intended relationship is for EthTrust controls to serve as a concrete technical reference within broader RMF application-security and smart-contract risk categories where appropriate.

---

# 1. Scope and relationship to Version 3

This document is initially drafted as a **delta specification** over Version 3 to make review tractable. Unless explicitly changed below, the definitions, conformance model, Security Levels `[S]`, `[M]`, `[Q]`, Recommended Good Practices `[GP]`, and normative requirements of EthTrust v3 remain unchanged.

Before final publication, the Working Group SHOULD merge approved changes into a complete, self-contained Version 4 specification and checklist.

Version 4 is intended to review Ethereum mainnet through **Fusaka**, including the BPO1 and BPO2 blob-parameter-only forks. Draft or future-fork proposals, including Glamsterdam and later upgrades, are out of the normative baseline until sufficiently stable and activated or otherwise explicitly adopted by the Working Group.

## 1.1 Scope principle

A protocol change belongs in EthTrust only when it changes the security properties, assumptions, review methods, or operational safety of the Tested Code.

Consensus-layer, networking, scaling, or client changes with no direct smart-contract security consequence SHOULD be recorded as reviewed but SHOULD NOT generate new EthTrust requirements merely because they are part of an Ethereum upgrade.

---

# 2. Proposed Version 4 normative changes

**All requirement levels and wording in this section are provisional pending human review.**

## 2.1 Account model and EIP-7702

EIP-7702 allows an EOA to set a delegation indicator causing calls to execute code from another address in the EOA's context. It breaks or weakens assumptions that historically treated EOAs as non-programmable accounts.

### [S-v4-01] No Account-Code Introspection

Tested Code **MUST NOT** contain `EXTCODESIZE`, `EXTCODEHASH`, or the Solidity expressions `address.code.length` or `address.codehash`, unless it meets the Overriding Requirement **[M-v4-01a] Verify Account-Code Introspection**.

### [M-v4-01a] Verify Account-Code Introspection

Tested Code **MUST NOT** treat the result of account-code introspection as a durable property of an address. A result observed in one transaction **MUST NOT** be persisted, cached, or relied on in a later transaction as evidence that an address is non-programmable, is programmable, or is eligible for elevated authorization, because an EIP-7702 delegation may be installed or revoked permissionlessly between transactions.

Within a single transaction an account's delegation status does not change, and account-code introspection MAY be relied on for the duration of that transaction.

Account-code introspection MAY also be used for other documented purposes where its result is not treated as a security boundary.

This is an Overriding Requirement for **[S-v4-01] No Account-Code Introspection**.

**Rationale:** a delegated EOA's code is visible to introspection, not hidden from it. EIP-7702 writes the delegation indicator `0xef0100 || address` into the account as code, so `EXTCODESIZE` on a delegated account returns 23 and `address.code.length == 0` is false. What EIP-7702 breaks is durability, not visibility: delegation can be installed or revoked permissionlessly between transactions, so caching or persisting an introspection result from an earlier transaction is unsound. Delegation cannot change mid-transaction, so introspection MAY still be relied on for the duration of a single transaction. An earlier draft of this requirement put a trust judgment (sole basis for asserting non-programmability/trust) inside an `[S]`-level predicate; that judgment is not evaluable by an unguided static tool, so it moved to the `[M-v4-01a]` Overriding Requirement.

Reference: https://eips.ethereum.org/EIPS/eip-7702

### [GP-v4-11] Test Counterparty Behavior for Delegated Accounts

Tested Code that varies its behavior according to the presence of account code SHOULD be reviewed for correct behavior when the counterparty is an EIP-7702 delegated account.

This applies in particular to recipient-callback paths gated on account code, including `onERC721Received`, `onERC1155Received`, `onERC1155BatchReceived`, and ERC-777 hooks, where a delegated account that does not implement the expected callback interface causes an otherwise valid transfer to revert. During delegated execution, `CODESIZE`/`CODECOPY` observe the delegate's code while `EXTCODESIZE`/`EXTCODECOPY` on the authority return the 23-byte delegation indicator, so a check comparing self-observed to externally observed code is inconsistent by design.

**Rationale:** this is a different failure class from `[S-v4-01]`/`[M-v4-01a]`. The introspection result is correct and the calling code behaves as written; what changed is the population of addresses that now hit the code-present branch, which can silently and reversibly break asset transfers to a delegated EOA.

### [Q-v4-02] Verify `tx.origin` Under Delegated Account Execution

This requirement **updates** v3 `[Q] Verify tx.origin Usage`.

For Tested Code that uses `tx.origin`, each use **MUST** be reviewed under EIP-7702 semantics. The review **MUST NOT** assume that `tx.origin == msg.sender` can only occur in the topmost execution frame or that such equality prevents programmable multi-call execution.

Where `tx.origin` participates in authorization, reentrancy protection, flash-loan or atomic-execution restrictions, anti-bot logic, or other security-sensitive behavior, the reviewer **MUST** document and test the behavior when the originating account has EIP-7702 delegated code.

The existing `[S] No tx.origin` requirement remains the preferred baseline.

Tested Code **MUST NOT** rely on either of the following assumptions, both invalidated by EIP-7702: that an account's balance can only decrease as a result of a transaction originating from that account, or that an externally owned account's nonce cannot increase after transaction execution has begun. Where Tested Code snapshots a counterparty balance, or derives an address or ordering guarantee from an externally owned account's nonce, the review **MUST** test behavior when that account has delegated code.

**Rationale:** EIP-7702 explicitly changes the historical invariant around `tx.origin == msg.sender` and identifies consequences for atomic-sandwich and reentrancy assumptions. Its Backwards Compatibility section also invalidates two further invariants (balance can only decrease from an own-transaction call; an EOA nonce cannot increase mid-execution) that an earlier draft of this requirement did not cover.

### [M-v4-03] Secure EIP-7702 Delegation Logic

Tested Code that acts as an EIP-7702 delegate implementation, constructs or verifies delegated-operation authorizations, or relays delegated operations **MUST** ensure that security-sensitive authorizations bind the information necessary to prevent unintended execution, including as applicable:

- replay protection / nonce,
- intended chain or explicit cross-chain behavior,
- execution target,
- calldata or operation payload,
- transferred value,
- gas or other griefing-sensitive execution bounds, and
- the authority or signer whose privilege is exercised.

Any intentional omission **MUST** be documented and shown not to violate the security objectives of the Tested Code.

### [M-v4-04] Protect Delegated Account Initialization and Storage

Where Tested Code is intended to execute as EIP-7702 delegated code:

- initialization that establishes privileged or security-sensitive state **MUST** be authenticated against the account authority and **MUST** be protected against front-running;
- storage layout **MUST** be designed or namespaced to prevent unintended collisions with prior or future delegate implementations, or an equivalent migration-safety mechanism **MUST** be demonstrated;
- changes to delegation or delegate implementation **MUST** be treated as security-sensitive upgrade operations; and
- privilege granted to modules, session keys, relayers, or sub-keys **MUST** follow the principle of least privilege.

ERC-7201-style namespaced storage is one possible mitigation but is not mandated if an equivalent reviewed design is used.

---

## 2.2 Transient storage, EIP-1153

EIP-1153 adds `TLOAD` and `TSTORE`. Transient state is discarded at the end of the transaction, shared across frames belonging to the owning contract, follows persistent-storage ownership rules under `DELEGATECALL`, and is reverted with frame reverts.

### [M-v4-05] Verify Transient Storage Lifetime

For each transient storage value that affects authorization, reentrancy protection, accounting, settlement, pricing, or another security invariant, Tested Code **MUST**:

- document the intended lifetime and meaning of the transient value;
- **MUST NOT** rely on the value persisting across transactions;
- account for multiple calls to the contract within the same transaction; and
- ensure that a non-zero value left after a call cannot cause unintended behavior in a later call within the same transaction.

Where a transient value is intended to be scoped to a call rather than to the transaction, Tested Code **MUST** explicitly restore or clear it before the relevant call completes.

Tested Code **MUST** account for the revert semantics of transient storage: writes made within a frame that subsequently reverts are rolled back, including writes made in inner calls, in the same way as persistent storage. Where Tested Code handles a reverting call without propagating the revert, the review **MUST** establish which transient values survive that path.

Tested Code **MUST NOT** use transient storage as a substitute for in-memory data structures. Transient storage is not discarded when a call returns, so a value written for the duration of one call remains readable by a later call in the same transaction, including a re-entrant one.

**Rationale (added):** EIP-1153 states explicitly that transient-storage writes are reverted on frame revert "in the same way persistent storage is," and separately warns that developers may be tempted to use transient storage as an in-memory-mapping substitute without realizing it survives a returning call, creating an unintended same-transaction reentrancy channel. Neither case was covered by the original lifetime wording above.

Reference: https://eips.ethereum.org/EIPS/eip-1153

### [M-v4-06] Verify Transient Storage Under `DELEGATECALL`

`TSTORE` and `TLOAD` address the transient storage of the executing account. Under `DELEGATECALL` or `CALLCODE` that is the calling account, so all delegated modules executing in a given account share one transient-storage namespace.

Where Tested Code combines transient storage with `DELEGATECALL` or `CALLCODE`, delegated modules **MUST NOT** share transient-storage slots unless the sharing is intended and documented, and slot derivation **MUST** be collision-resistant across modules.

This requirement is related to v3 `[S] No delegatecall()`, `[M] Protect External Calls`, and `[Q] Verify External Calls`.

**Rationale (revised):** the requirement previously stated the reviewer's obligation without first stating the EVM rule that produces it; the rule (shared namespace under `DELEGATECALL`/`CALLCODE`) is what makes the obligation a checkable fact rather than a judgment call.

---

## 2.3 `SELFDESTRUCT`, EIP-6780

Version 3 correctly discourages `selfdestruct()`, but parts of its explanatory text still describe the pre-Dencun behavior in which a deployed contract is destroyed and its code/storage removed.

### Update: [S] No `selfdestruct()`

The existing v3 prohibition remains appropriate and SHOULD remain normative.

The explanatory text MUST be updated to state that, after EIP-6780, `SELFDESTRUCT` normally transfers the account balance and halts the frame **without deleting code or storage**, except when invoked in the same transaction in which the contract was created.

### Update: [M] Protect Self-destruction

Where the v3 overriding path permits reviewed use of `SELFDESTRUCT`, the review **MUST NOT** rely on:

- deletion of code or storage for contracts created in an earlier transaction;
- redeployment of different code at the same address through `CREATE2` after later `SELFDESTRUCT`; or
- burning Ether by self-destructing to the executing contract where EIP-6780 no longer provides that behavior.

The value-transfer effect and any lifecycle assumptions **MUST** be documented and compatible with the security objectives of the Tested Code.

Reference: https://eips.ethereum.org/EIPS/eip-6780

---

## 2.4 Gas bounds and Fusaka, EIP-7825

Fusaka caps a single Ethereum transaction at `16,777,216` (`2^24`) gas.

### [M-v4-07] Bound Security-Critical Operations to the Transaction Gas Cap

Where the correct or safe operation of Tested Code depends on an administrative, recovery, upgrade, settlement, liquidation, migration, emergency, or other security-critical transaction completing atomically, the reviewer **MUST** establish that the operation can execute within the transaction gas cap of the intended deployment network under the documented worst-case state.

If the operation may exceed that bound, Tested Code **MUST** provide a safe bounded/chunked alternative or the limitation **MUST** be explicitly documented and shown not to create an unacceptable denial-of-service or asset-locking condition.

Reviewers SHOULD test close-to-worst-case state rather than relying only on deployment-time gas estimates.

Reference: https://eips.ethereum.org/EIPS/eip-7825

---

## 2.5 Cryptographic precompiles, EIP-2537 and EIP-7951

Pectra added BLS12-381 precompiles and Fusaka added the secp256r1/P-256 verification precompile.

### [M-v4-08] Verify Cryptographic Precompile Results

Where Tested Code depends on a cryptographic precompile for authentication, authorization, consensus proofs, bridge verification, or asset-control decisions, it **MUST**:

- validate the precompile-specific input encoding and bounds required by the protocol;
- distinguish successful EVM call execution from successful cryptographic verification;
- validate the exact expected return length and value;
- apply appropriate replay protection and domain separation to the signed message or proof context; and
- document any curve-specific malleability or canonical-signature assumptions on which security depends.

For EIP-7951 specifically, the precompile at address `0x100` returns a 32-byte value of `1` on successful verification and empty output on failure, and does not revert. Tested Code **MUST** treat empty output as verification failure, and **MUST NOT** treat successful `STATICCALL` execution alone as proof of a valid P-256 signature.

Because secp256r1 signatures are not required to be non-malleable, Tested Code that uses a P-256 signature, or a hash of one, as a uniqueness or replay key **MUST** enforce a canonical `s` value or an equivalent application-layer non-malleability check. See also v3's `[M] No Improper Usage of Signatures for Replay Attack Protection` and `[Q] Intended Replay`.

For EIP-2537 specifically, Tested Code **MUST NOT** assume that every BLS12-381 precompile validates subgroup membership. The G1 and G2 addition precompiles check only that inputs are on the curve or the point at infinity; multi-scalar multiplication and pairing check subgroup membership. Where Tested Code combines externally supplied points using the addition precompiles, it **MUST** perform its own subgroup check, or **MUST** establish that every such point has already been subgroup-checked.

Tested Code **MUST NOT** rely on a scalar being reduced modulo the main subgroup order, and **MUST NOT** use an unreduced scalar encoding as a uniqueness key.

**Rationale (added):** the original wording ("validate precompile-specific input encoding and bounds") was too abstract to catch the specific bugs EIP-2537 and EIP-7951 create: G1ADD/G2ADD are explicitly documented as not performing subgroup checks (small-subgroup attack surface for any contract accumulating attacker-supplied points before a pairing check), MSM scalars are not required to be reduced mod the subgroup order (two distinct encodings can produce the same result), and secp256r1/P-256 signatures are not required to be non-malleable per NIST FIPS 186-5, which matters for any passkey/WebAuthn-style replay key.

References:
- https://eips.ethereum.org/EIPS/eip-2537
- https://eips.ethereum.org/EIPS/eip-7951

---

## 2.6 Compiler and vulnerability freshness

Version 3 currently has `[GP] Check For and Address New Security Bugs`, directing reviewers to check for bugs announced after 1 November 2023. A certification standard should not make current compiler advisories optional merely because a static release cannot predict future bugs.

### [M-v4-09] Check Compiler Security Advisories at Certification Time

The reviewer **MUST** check the Solidity compiler version(s) used by the Tested Code against Solidity security-advisory and known-bugs sources, and **MUST** determine that no applicable compiler vulnerability published on or before the date of that check invalidates the certification requirements.

The certification record **MUST** identify:

- the compiler version(s) used,
- the relevant compilation configuration,
- the advisory source(s) consulted, and
- the date of consultation.

Conformance under this requirement is assessed against advisories published on or before the recorded date.

**Rationale (revised):** the original wording put a reviewer/record-keeping duty at `[S]`, the level v3 reserves for predicates an unguided static tool can evaluate; no tool can determine whether a human consulted an advisory feed. It also said "current," which makes conformance drift after certification with no code change, since the same bytecode could stop conforming the day a new advisory is published. Pinning to the certification date fixes both: the level now matches v3's own [M]-is-for-human-judgment pattern, and conformance is a fixed, re-verifiable fact about a point in time rather than a moving target. **Open question for Working Group review:** whether to also retain or extend v3's enumerated per-bug `[S]` requirements for issues found after the spec was frozen (a co-chair has proposed following the pattern of `w3c-cg/EthTrust#6` and `#8`); this candidate is compatible with keeping that list at `[S]` but does not itself resolve whether to extend it. This requirement is intended to make compiler-security review evergreen. Version-specific compiler requirements MAY remain where they provide automatable checks or useful historical coverage.

### [GP-v4-10] Reassess After Security-Relevant Network Upgrades

A previously certified contract SHOULD be reassessed when a network upgrade changes an opcode, precompile, account model, gas bound, or execution semantic on which its security assumptions depend.

A protocol upgrade that has no relevant effect on the Tested Code does not by itself require recertification.

---

# 3. Ethereum mainnet EIP impact matrix

This matrix is a **first-pass AI classification for human review**, not a final Working Group determination.

Classification:

- **Normative**, candidate change to an EthTrust requirement.
- **Guidance**, explanatory/testing/reference update likely useful; no general new requirement proposed.
- **No direct EthTrust change**, reviewed but primarily consensus/networking/scaling/client behavior outside smart-contract certification.

## 3.1 Dencun, activated 13 March 2024

Meta EIP: https://eips.ethereum.org/EIPS/eip-7569

| EIP | Change | Initial EthTrust classification | Rationale |
|---|---|---|---|
| 1153 | Transient storage (`TLOAD`/`TSTORE`) | **Normative** | New transaction-scoped state affects reentrancy, lifetime and delegated execution. |
| 4788 | Beacon block root in the EVM | Guidance | Contracts consuming consensus-layer roots must document assumptions and proof validation. Existing external-data/system-architecture requirements may cover most cases. |
| 4844 | Blob transactions | Guidance | Blob-aware applications/tooling may need updated assumptions; no general contract vulnerability introduced. |
| 5656 | `MCOPY` opcode | Guidance | Static analysis, disassemblers and inline-assembly review tooling must recognize the opcode; no generic new requirement identified. |
| 6780 | Restricted `SELFDESTRUCT` | **Normative update** | Existing v3 explanation and allowed-use path assume obsolete deletion semantics. |
| 7044 | Perpetually valid signed voluntary exits | No direct EthTrust change | Consensus-layer validator behavior. |
| 7045 | Increased max attestation inclusion slot | No direct EthTrust change | Consensus-layer behavior. |
| 7514 | Max epoch churn limit | No direct EthTrust change | Consensus-layer validator churn. |
| 7516 | `BLOBBASEFEE` opcode | Guidance | Contracts using blob base fee as an economic input should include it in documented economic/manipulation testing. |

## 3.2 Pectra, activated 7 May 2025

Meta EIP: https://eips.ethereum.org/EIPS/eip-7600

| EIP | Change | Initial EthTrust classification | Rationale |
|---|---|---|---|
| 2537 | BLS12-381 precompiles | **Normative/conditional** | Security-critical crypto callers need correct input/output and proof validation. |
| 2935 | Historical block hashes in state | Guidance | Security logic using extended historical hashes must document finality/reorg/time-window assumptions. |
| 6110 | Validator deposits supplied on chain | Guidance | Directly relevant mainly to staking/deposit infrastructure; assess under architecture and external-interaction requirements. |
| 7002 | Execution-layer triggerable exits | Guidance | Relevant to staking protocols and withdrawal-credential authority; does not justify a universal requirement. |
| 7251 | Increased validator max effective balance | No direct EthTrust change | Consensus/staking economics rather than generic contract security. |
| 7549 | Committee index moved outside attestation | No direct EthTrust change | Consensus proof format; application impact is specialized. |
| 7623 | Increased calldata cost | Guidance | Gas/DoS testing may change for calldata-heavy transactions. |
| 7685 | General-purpose execution-layer requests | Guidance | Relevant to protocol/staking integrations; no universal contract requirement identified. |
| 7691 | Increased blob throughput | No direct EthTrust change | Scaling parameter. |
| 7702 | Set Code for EOAs | **Normative** | Changes account-model, delegation, `tx.origin`, initialization, storage and authorization assumptions. |
| 7840 | Blob schedule in EL config | No direct EthTrust change | Client configuration. |

## 3.3 Fusaka, activated 3 December 2025

Meta EIP: https://eips.ethereum.org/EIPS/eip-7607

| EIP | Change | Initial EthTrust classification | Rationale |
|---|---|---|---|
| 7594 | PeerDAS | No direct EthTrust change | Data-availability/network architecture; not a generic smart-contract semantic change. |
| 7823 | Upper bounds for MODEXP | Guidance | Crypto-heavy contracts must ensure inputs remain within protocol bounds. |
| 7825 | Per-transaction gas cap | **Normative** | Can make formerly valid atomic/admin/recovery transactions impossible and create liveness/DoS risks. |
| 7883 | MODEXP gas repricing | Guidance | May break gas assumptions for crypto-heavy operations; test liveness and economic bounds. |
| 7917 | Deterministic proposer lookahead | No direct EthTrust change | Consensus/proposer information; specialized MEV applications may warrant separate review. |
| 7918 | Blob base fee bounded by execution cost | Guidance | Relevant where application economics explicitly depend on blob fees. |
| 7934 | RLP execution block size limit | No direct EthTrust change | Block-level bound; no generic contract requirement identified. |
| 7939 | `CLZ` opcode | Guidance | Tooling/static analysis must recognize new opcode; no inherent generic vulnerability. |
| 7951 | secp256r1/P-256 precompile | **Normative/conditional** | Authentication/passkey/HSM integrations require correct result and signature validation. |
| 7892 | Blob Parameter Only forks | No direct EthTrust change | Upgrade mechanism for blob parameters. |
| 7642 | `eth/69` networking change | No direct EthTrust change | P2P/client networking. |
| 7910 | `eth_config` JSON-RPC method | No direct EthTrust change | Client/RPC interface. |
| 7935 | Default gas limit ~60M | Guidance | Changes environment for gas/DoS analysis but does not override EIP-7825's per-transaction cap. |

BPO1 and BPO2 adjust blob parameters and are reviewed as no direct generic EthTrust contract-security change.

---

# 4. Existing v3 requirements requiring editorial or substantive update

The integrated v4 review SHOULD inspect at least the following v3 material:

1. **`[S] No tx.origin` and `[Q] Verify tx.origin Usage`**, update explanations and Q-level verification for EIP-7702.
2. **`[S] No selfdestruct()` and `[M] Protect Self-destruction`**, preserve the conservative prohibition but replace obsolete descriptions of code/storage deletion and `CREATE2` metamorphic behavior.
3. **`[S] No delegatecall()`, `[M] Protect External Calls`, `[Q] Verify External Calls`**, cross-reference EIP-7702 delegated execution and EIP-1153 transient storage ownership.
4. **Reentrancy / Check-Effects-Interactions material**, add transient storage as a possible reentrancy-lock mechanism while requiring correct same-transaction lifetime handling.
5. **Signature mechanisms**, extend examples/reference material for BLS12-381 and P-256 precompiles and modern account/delegation signatures.
6. **Gas and gas prices**, add the Fusaka per-transaction cap and liveness implications.
7. **Compiler bugs**, replace the late-2023 optional freshness check with a normative current-advisory check.
8. **Network upgrades / post-deployment monitoring**, add a clear trigger for reassessing certificates when a fork changes a security-relevant semantic used by the Tested Code.
9. **Testing/static-analysis tooling**, require the review toolchain to understand opcodes and precompiles active on the target fork rather than silently treating unknown instructions as benign.

## 4.1 EthTrust-public issue #7

`EntEthAlliance/EthTrust-public#7` proposed stronger coverage of fuzzing, mutation testing, integrated pre-deployment assessment, adversarial simulation, organizational/OpSec controls, reentrancy, and security terminology.

A first review indicates that **much of this material already appears in the published v3 document**, including dedicated sections on Organizational and Off-Chain Security Posture, Preempting On-Chain Adversarial Conditions, Fuzzing, Mutation Testing, Symbolic Execution and Formal Verification.

For v4 the Working Group SHOULD therefore map issue #7 item-by-item against v3, close items already incorporated, and add only genuine remaining gaps. Duplicating material already present would make the standard harder to maintain.

Issue: https://github.com/EntEthAlliance/EthTrust-public/issues/7

---

# 5. Relationship to the GBBC Capital Markets Risk Mitigation Framework (RMF)

**This section is non-normative.**

The Global Blockchain Business Council (GBBC) Capital Markets Risk Mitigation Framework is an industry-led framework facilitated by **GBBC and Oliver Wyman** for assessing and mitigating non-financial risks of blockchain infrastructure used by regulated financial institutions.

The RMF is broader than EthTrust:

- **RMF Phase 1** addresses public Layer-1 blockchain infrastructure.
- **RMF Phase 2** extends to Layer-2 infrastructure and application risks for digital payments and tokenized securities.
- **RMF Phase 3** is intended to extend to native crypto assets and remaining application areas.

EthTrust is deliberately narrower: it provides concrete smart-contract security review and certification requirements. The proposed relationship is therefore:

> **GBBC RMF = broader institutional risk framework**  
> **EEA EthTrust = implementation-level smart-contract security control reference**

Where RMF identifies smart-contract/application risks, a current reviewed version of EthTrust could be cited as one possible technical control framework. Conversely, RMF can expose institutional or operational risk categories that are outside EthTrust's scope and should remain outside it.

This relationship **does not imply GBBC endorsement of EthTrust, EEA endorsement of RMF, or endorsement by any RMF participant.** EEA is currently listed as an RMF observer.

References:
- https://rmf.gbbc.io/
- https://www.gbbc.io/media/rmf-phase2

## 5.1 Why the RMF audience matters

As of this draft date, GBBC's RMF site states that the work has **30+ contributing institutions** and is built by a cross-sector group spanning financial-market infrastructures, global systemically important banks, multilateral development banks, protocol/infrastructure organizations, market-data providers, and standards bodies.

The current public RMF site lists the following participants. This roster is included to show the institutional context for a potential EthTrust cross-reference; it is **not** a claim that these organizations reviewed or endorse this draft.

### Core Contributors

- Ava Labs
- Canton Foundation
- Cardano Foundation
- Chainlink Labs
- The Depository Trust & Clearing Corporation (DTCC)
- Digital Asset
- Euroclear Group
- Global Blockchain Business Council (GBBC)
- Hedera
- Kinexys by J.P. Morgan
- Oliver Wyman
- Ripple

### Observers

- Asian Development Bank (ADB)
- BTG Pactual
- Clearstream
- Deutsche Börse
- Digital Token Identifier Foundation (DTIF)
- Enterprise Ethereum Alliance (EEA)
- European Central Bank (ECB)
- Global Legal Entity Identifier Foundation (GLEIF)
- GK8 by Galaxy
- IDB Lab
- International Securities Services Association (ISSA)
- MIT Digital Currency Initiative (MIT DCI)
- Moody's Ratings
- State Street
- Swift
- Temasek
- VerifyVASP
- United Nations Joint Staff Pension Fund (UNJSPF)
- The World Bank
- Wyoming Stable Token Commission

### Risk Assessment Partners (RAPs)

- Blockmosaic
- Dfns
- Droit Fintech
- Kaiko
- Metrika

**Roster source:** https://rmf.gbbc.io/, checked 24 August 2026. Participant categories may change and SHOULD be refreshed before publication.

## 5.2 Proposed RMF ↔ EthTrust crosswalk

The exact RMF control IDs should be mapped against the latest Phase 2 application-risk publication during human review. At a high level, the following relationship is proposed:

| Broader RMF risk area | EthTrust contribution |
|---|---|
| Smart-contract vulnerabilities | Core `[S]`, `[M]`, `[Q]` code-review requirements |
| Privileged/admin access | Access-control, least-privilege and documentation requirements |
| Upgradeability/change management | Upgradable-contract and architecture requirements; v4 delegated-account changes |
| Key/signature/authentication risk | Signature-management requirements; v4 delegated account and cryptographic-precompile requirements |
| External dependencies/oracles | External calls, oracles, architecture and adversarial testing |
| Transaction ordering / MEV | Existing ordering and MEV security considerations |
| Operational security | Existing organizational/off-chain security posture and monitoring guidance |
| Protocol-change risk | v4 fork-baseline, compiler freshness and post-upgrade reassessment |
| Availability / execution-bound risk | Gas/DoS requirements plus v4 transaction gas-cap requirement |

**Action before external reference:** replace these broad labels with exact RMF Phase 2 risk/control identifiers and confirm the mapping with GBBC/RMF maintainers.

---

# 6. Review and publication gates

This draft SHOULD NOT become the public certification baseline until the following have occurred:

- [ ] Human security review of every proposed normative requirement.
- [ ] Validate the Dencun/Pectra/Fusaka EIP matrix against the final hardfork meta-EIPs and execution specifications.
- [ ] Review current Solidity compiler/security advisories and add any required concrete tests.
- [ ] Map `EthTrust-public#7` against v3 and resolve remaining items.
- [ ] Produce updated test/checklist language for each accepted requirement.
- [ ] Review changes with practitioners from multiple independent smart-contract security organizations.
- [ ] Publish the draft for public comment through `EntEthAlliance/EthTrust-public`.
- [ ] Resolve public comments and record disposition.
- [ ] Complete EEA Working Group / organizational approval required for publication.
- [ ] Convert this delta draft into a self-contained v4 specification and checklist.
- [ ] Build an exact GBBC RMF Phase 2 ↔ EthTrust crosswalk using RMF control identifiers.
- [ ] Ask GBBC/RMF maintainers to review the crosswalk and, if they consider it appropriate, reference the final EthTrust v4 as an external smart-contract security resource.

---

# 7. Proposed change log: v3 → v4

### New candidate requirements

- `[S-v4-01] No Account-Code Introspection`
- `[M-v4-01a] Verify Account-Code Introspection` (Overriding Requirement for `[S-v4-01]`)
- `[M-v4-03] Secure EIP-7702 Delegation Logic`
- `[M-v4-04] Protect Delegated Account Initialization and Storage`
- `[GP-v4-11] Test Counterparty Behavior for Delegated Accounts`
- `[M-v4-05] Verify Transient Storage Lifetime`
- `[M-v4-06] Verify Transient Storage Under DELEGATECALL`
- `[M-v4-07] Bound Security-Critical Operations to the Transaction Gas Cap`
- `[M-v4-08] Verify Cryptographic Precompile Results`
- `[M-v4-09] Check Compiler Security Advisories at Certification Time`
- `[GP-v4-10] Reassess After Security-Relevant Network Upgrades`

### Updated candidate requirements

- `[Q] Verify tx.origin Usage` → EIP-7702-aware verification.
- `[S] No selfdestruct()` → retain requirement; update rationale/semantics.
- `[M] Protect Self-destruction` → update allowed-use verification for EIP-6780.

### Guidance/reference updates

- Dencun, Pectra and Fusaka mainnet baseline.
- New EVM opcodes and precompiles recognized by test/static-analysis tooling.
- Gas/DoS guidance updated for the Fusaka per-transaction gas cap.
- GBBC RMF relationship and proposed crosswalk added as non-normative context.

---

# 8. Primary sources for this initial draft

- EthTrust v3: https://entethalliance.github.io/wg-ethtrust-site/spec/v3/
- Dencun meta EIP: https://eips.ethereum.org/EIPS/eip-7569
- Pectra meta EIP: https://eips.ethereum.org/EIPS/eip-7600
- Fusaka meta EIP: https://eips.ethereum.org/EIPS/eip-7607
- EIP-1153: https://eips.ethereum.org/EIPS/eip-1153
- EIP-6780: https://eips.ethereum.org/EIPS/eip-6780
- EIP-7702: https://eips.ethereum.org/EIPS/eip-7702
- EIP-7825: https://eips.ethereum.org/EIPS/eip-7825
- EIP-7951: https://eips.ethereum.org/EIPS/eip-7951
- GBBC RMF: https://rmf.gbbc.io/
- GBBC RMF Phase 2 announcement: https://www.gbbc.io/media/rmf-phase2

---

## Editorial note

This AI-generated draft is intentionally explicit about uncertainty. Requirement IDs with the `v4-` prefix are placeholders to avoid colliding with the established v3 anchors. Human editors SHOULD renumber/re-anchor accepted requirements when they are integrated into the self-contained Version 4 specification.
