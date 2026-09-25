# EthTrust v4 exploratory technical review

**Review date:** 25 September 2026

**Baseline:** [EthTrust Security Levels v3](../v3/) (March 2025)

**Scope:** Ethereum execution changes through Dencun, Pectra and Fusaka

**Tracking:** [Technical refresh issue #6](https://github.com/EntEthAlliance/wg-ethtrust-site/issues/6)

## Status and purpose

The EthTrust Working Group is currently inactive. This is an EEA-hosted exploratory technical review. It does not reactivate the Working Group, create an EEA specification, establish a certification baseline, or represent EEA approval. Feedback may inform a future Working Group process if one is established.

This material was prepared with AI assistance (OpenAI ChatGPT/Codex). It incorporates public technical feedback but has not been adopted by the EEA or an EthTrust Working Group. Independent human security review remains necessary.

The terms **MUST**, **MUST NOT**, **SHOULD**, **MAY**, `[S]`, `[M]`, `[Q]` and `[GP]` below test possible future specification language. They have no normative force in this workspace. References to certification describe a candidate future process, not permission to certify against this draft. All IDs and levels are provisional.

[README](./README.md) explains the workspace; the [candidate checklist](./CHECKLIST-DRAFT.md) supplies review questions; the [HTML review page](./index.html) presents the same candidates. The published v3 specification remains the baseline. This document records proposed deltas rather than a complete replacement specification.

## 1. Scope and baseline

Include a protocol change only when it affects security properties, assumptions, review methods, or operational safety relevant to Tested Code. Record consensus, networking, scaling and client changes without inventing generic contract requirements for them. The through-Fusaka boundary is a fixed review scope, not a claim to cover every upgrade active at a later certification date. Record the actual target network and fork when assessing a contract.

Version 3 includes compiler-bug requirements through `SOL-2023-3` and a good practice to check bugs announced after 1 November 2023. This review retains the existing v3 requirements unless an explicit candidate update says otherwise. Dencun is relevant to that technical baseline even though it predates v3 publication.

## 2. Candidate requirements

### 2.1 Account model and EIP-7702

#### [S-v4-01] No Account-Code Introspection

Tested Code **MUST NOT** contain `EXTCODESIZE`, `EXTCODEHASH`, or the Solidity expressions `address.code.length` or `address.codehash`, unless it meets the Overriding Requirement **[M-v4-01a] Verify Account-Code Introspection**.

This keeps the proposed `[S]` predicate mechanically detectable. Judgments about purpose and trust belong in the manual override. Equivalent introspection through code copying or other mechanisms also needs the security analysis in that override; the final automated detection scope remains a review item.

#### [M-v4-01a] Verify Account-Code Introspection

Account-code introspection **MUST NOT** be used by itself to establish that an address is an EOA, non-programmable, trusted, or incapable of executing contract behavior. The reviewer **MUST** document the purpose of each use and the independent controls supporting any security decision.

An introspection result **MUST NOT** be cached or relied on across transactions as a durable account classification or authorization fact. EIP-7702 delegation can be installed, changed or cleared between transactions with a valid authority authorization; observing the indicator does not establish trust in the account or its delegate.

EIP-7702 authorization processing occurs before the execution portion of a transaction. During execution, the delegation indicator cannot be installed, changed or removed. That stability **MAY** be relied on only for a documented decision specifically about that delegation state. It **MUST NOT** be generalized to stability of all account code or behavior: a constructor can call Tested Code before runtime code exists, and code can be deployed at a previously empty address later in the same transaction. A stable indicator also does not guarantee immutable delegate behavior.

The review **MUST** test constructor-originated calls, inspection before and after same-transaction deployment, and delegation changes between transactions, with expected outcomes tied to the stated security objectives. Other documented introspection uses **MAY** be accepted when they do not substitute for the independent controls above.

This is the Overriding Requirement for **[S-v4-01] No Account-Code Introspection**. A delegated EOA exposes a 23-byte `0xef0100 || address` indicator to `EXTCODE*`; its code is visible. Zero runtime code is not proof of EOA status even without EIP-7702.

Sources: [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702), [OpenZeppelin Address documentation](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address-isContract-address-).

#### [Q-v4-02] Verify `tx.origin` Under Delegated Account Execution

This candidate supplements v3 **[Q] Verify tx.origin Usage**. Its existing security-objective obligations and the **[S] No tx.origin** baseline remain.

For Tested Code that uses `tx.origin`, each use **MUST** be reviewed under EIP-7702 semantics. The review **MUST NOT** assume that `tx.origin == msg.sender` identifies only the topmost execution frame or prevents programmable multi-call execution.

Where `tx.origin` affects authorization, reentrancy protection, flash-loan or atomic-execution restrictions, anti-bot logic, or other security-sensitive behavior, the reviewer **MUST** document and test behavior when the origin has delegated code. Equality with `msg.sender` does not establish non-programmability.

#### [M-v4-02a] Verify Delegated Account Balance and Nonce Assumptions

This review **MUST** be performed independently of whether Tested Code uses `tx.origin`. The reviewer **MUST** identify security assumptions about counterparties' native account balances and EOA nonces, including balance snapshots and nonce-derived address or ordering assumptions.

Tested Code **MUST NOT** assume that an account's balance can decrease only in a transaction originating from that account, or that an EOA's nonce cannot increase after transaction execution begins. A call to a delegated account can spend its balance; delegated code can execute a contract-creation operation that increases its nonce.

Where those assumptions affect security, the reviewer **MUST** test balance changes caused by a call from another account and nonce changes caused by contract creation during execution. If no relevant dependency exists, the review **MUST** record that finding and its basis. Absence of `tx.origin` is not grounds to skip this review.

Source for both candidates: [EIP-7702, backwards compatibility](https://eips.ethereum.org/EIPS/eip-7702#backwards-compatibility).

#### [M-v4-03] Secure EIP-7702 Delegation Logic

For Tested Code implementing, verifying or relaying delegated operations, security-sensitive operation authorizations **MUST** bind the information needed to prevent unintended execution: replay nonce, intended chain or explicit cross-chain policy, target, calldata, value, griefing-sensitive gas bounds, and the authority whose privilege is exercised, as applicable. Any intentional omission **MUST** be documented and shown compatible with the security objectives.

The EIP-7702 protocol authorization installs a code pointer; it does not itself authorize every later operation. Review the delegate's operation-signature scheme separately from the protocol authorization tuple.

#### [M-v4-04] Protect Delegated Account Initialization and Storage

For Tested Code intended to execute as EIP-7702 delegated code:

- Privileged or security-sensitive initialization **MUST** be authenticated against the account authority and protected against front-running.
- Storage layout **MUST** prevent unintended collisions with prior or future delegates, or demonstrate equivalent migration safety.
- Delegation or implementation changes **MUST** be treated as security-sensitive upgrades.
- Privileges of modules, session keys, relayers and sub-keys **MUST** follow least privilege.

Namespaced storage is one possible mitigation, not a mandated implementation. Source: [EIP-7702, security considerations](https://eips.ethereum.org/EIPS/eip-7702#security-considerations).

#### [GP-v4-11] Test Counterparty Behavior for Delegated Accounts

Tested Code that varies behavior according to code presence **SHOULD** be reviewed with delegated counterparties, including delegates that lack expected receiver interfaces. Cover `onERC721Received`, `onERC1155Received`, `onERC1155BatchReceived`, and applicable ERC-777 or custom hook paths where code detection affects dispatch.

`CODESIZE`/`CODECOPY` during delegated execution observe the delegate code; `EXTCODESIZE`/`EXTCODECOPY` on the authority observe the delegation indicator. Review assumptions that compare these observations. This is a compatibility and callback problem distinct from treating code presence as a trust boundary.

### 2.2 Transient storage, EIP-1153

#### [M-v4-05] Verify Transient Storage Lifetime

For every security-sensitive transient value, Tested Code **MUST** document its meaning and intended lifetime, account for repeated and reentrant calls in one transaction, and prevent stale values from changing later calls unexpectedly. It **MUST NOT** depend on persistence across transactions. Values intended to last only for a call **MUST** be restored or cleared before that call returns normally.

The review **MUST** account for rollback: reverting a frame undoes transient writes made since entry to that frame, including writes in its inner calls. Values established before entry survive that rollback. When a caller catches a failure, the review **MUST** establish which values survive and test both the caught-failure and propagated-revert paths.

Tested Code **MUST NOT** treat transient storage as frame-local memory or a drop-in substitute for in-memory mappings. A normal return does not clear it; later calls in the transaction can observe it. Any use for temporary data **MUST** enforce the documented lifetime and isolation explicitly.

#### [M-v4-06] Verify Transient Storage Under `DELEGATECALL`

Under `DELEGATECALL` or `CALLCODE`, transient storage belongs to the calling account, so modules executing in that account share one transient-storage namespace. Ordinary `CALL` uses the callee's namespace.

Delegated modules **MUST NOT** share transient slots unless the sharing is intended and documented. Slot derivation **MUST** be collision-resistant across modules. The review **MUST** test collisions and intended sharing across delegated modules, including rollback paths. Relate this analysis to v3's external-call and delegatecall requirements.

Source: [EIP-1153](https://eips.ethereum.org/EIPS/eip-1153).

### 2.3 Update v3 `SELFDESTRUCT` explanations and review

Retain **[S] No selfdestruct()** and its existing overriding requirements. Under EIP-6780, `SELFDESTRUCT` normally transfers the balance and halts the frame without deleting code or storage. The legacy deletion behavior is available only when the contract was created in the same transaction.

Under **[M] Protect Self-destruction**, any reviewed use **MUST NOT** rely on deletion of code/storage for a contract created in an earlier transaction, later `CREATE2` redeployment of different code following that deletion, or burning Ether by self-destructing to itself where EIP-6780 no longer provides that behavior. The review **MUST** document value-transfer and lifecycle assumptions and test both creation-transaction and later-transaction cases where relevant. Existing authorization and documentation obligations remain.

Source: [EIP-6780](https://eips.ethereum.org/EIPS/eip-6780).

### 2.4 Transaction gas bounds

#### [M-v4-07] Bound Security-Critical Operations to the Transaction Gas Cap

Where safe operation depends on an administrative, recovery, upgrade, settlement, liquidation, migration, emergency or other security-critical transaction completing atomically, the reviewer **MUST** establish that it can execute within the intended network's transaction gas cap under documented worst-case state.

If it may exceed that bound, Tested Code **MUST** provide a safe bounded/chunked alternative, or the limitation **MUST** be documented and shown not to create an unacceptable denial-of-service or asset-locking condition. Reviewers **SHOULD** test near-worst-case state rather than rely on deployment-time estimates. EIP-7825 sets the transaction gas-limit cap to `16,777,216` (`2^24`); a larger block gas limit does not override it.

Source: [EIP-7825](https://eips.ethereum.org/EIPS/eip-7825).

### 2.5 Cryptographic precompiles

#### [M-v4-08] Verify Cryptographic Precompile Results

Where security depends on a cryptographic precompile, Tested Code **MUST** validate operation-specific input encoding and bounds, handle failed calls, verify exact expected output length and value, apply replay protection and domain separation, and document curve-specific assumptions. Successful EVM call execution alone **MUST NOT** count as cryptographic verification. The review **MUST** confirm availability and semantics on each target network/fork.

For EIP-7951, P-256 verification at `0x100` takes 160 bytes and returns exactly the 32-byte integer `1` on success. Invalid input or signature produces empty output without a verification-induced revert. Tested Code **MUST** reject failed calls, empty output and any unexpected result. The precompile does not enforce low-`s`: when signatures or their hashes are used as uniqueness/replay keys, Tested Code **MUST** enforce canonical `s` or equivalent application-layer non-malleability handling. Canonicalization does not replace signed-message/nonce replay protection. Relate this to v3 **[M] No Improper Usage of Signatures for Replay Attack Protection** and **[Q] Intended Replay**.

For EIP-2537, G1/G2 addition validates encoding and points on the curve (or infinity), but does not check subgroup membership. MSM and pairing do check membership. When adding externally supplied points where subgroup membership is required by the security scheme, Tested Code **MUST** check each point or establish that each was already checked. The review **MUST** cover malformed encodings, invalid points, inappropriate infinity cases, and empty inputs (which MSM and pairing reject).

MSM accepts 32-byte scalars without requiring a canonical value below the subgroup order. Distinct scalar encodings can produce the same group result. Tested Code **MUST NOT** assume the precompile enforces canonical scalar encoding or use unreduced scalar bytes as a uniqueness guarantee.

Sources: [EIP-2537](https://eips.ethereum.org/EIPS/eip-2537), [EIP-7951](https://eips.ethereum.org/EIPS/eip-7951).

### 2.6 Compiler-advisory freshness and reassessment

#### [M-v4-09] Check Compiler Security Advisories at Certification Time

The reviewer **MUST** perform a final refresh of Solidity security-advisory and known-bugs sources immediately before certification is issued. This final issuance-stage refresh establishes one certification cutoff, recorded as a UTC date and time in the certification record. Earlier checks are preliminary and **MUST NOT** substitute for it. If issuance is delayed after that final review, the refresh and applicability assessment **MUST** be repeated and the cutoff replaced before issuance.

The reviewer **MUST** assess advisories published on or before that cutoff against every compiler version and configuration used for the Tested Code, and determine that no applicable compiler vulnerability invalidates the certification requirements. An unavailable required source or unresolved applicable advisory **MUST** prevent completion of this check; a stale snapshot **MUST NOT** be treated as a fresh review.

The certification record **MUST** contain:

- The single cutoff timestamp and the certification issuance timestamp.
- Exact compiler version(s), including build/commit identifiers where available.
- Compilation configuration, including optimizer settings/runs, IR pipeline, EVM target, libraries and other relevant settings.
- Tested source revision or content hashes and resulting artifact identifiers, sufficient to identify the reviewed build.
- Each advisory source URL and the date/time of the final consultation.
- An immutable source revision/version or archived snapshot with content hash for each source, sufficient to reproduce the advisory set reviewed at the cutoff.
- The applicable advisory IDs, applicability decisions, mitigations or corrected build, and evidence that the final build was reassessed.

Conformance under this candidate is assessed against that single recorded cutoff and evidence set. An advisory published later does not retroactively change the recorded determination; it **SHOULD** trigger reassessment when relevant to the certified code. It does not make the earlier cutoff a claim of continuing safety.

This is a manual reviewer duty at `[M]`, changed from the earlier draft's `[S-v4-09]`. It would strengthen v3's optional advisory-freshness good practice. Existing enumerated per-bug `[S]` requirements remain in the baseline. Whether and how to extend them is unresolved; this candidate does not replace those checks.

Sources: [Solidity known bugs](https://docs.soliditylang.org/en/latest/bugs.html), [bugs.json](https://github.com/argotorg/solidity/blob/develop/docs/bugs.json), [Solidity security alerts](https://www.soliditylang.org/blog/category/security-alerts/), [public feedback #18](https://github.com/EntEthAlliance/EthTrust-public/issues/18).

#### [GP-v4-10] Reassess After Security-Relevant Network Upgrades

A previously certified contract **SHOULD** be reassessed when a network upgrade changes an opcode, precompile, account model, gas bound or execution semantic on which its security assumptions depend. An upgrade without a relevant effect on the Tested Code does not by itself require recertification.

## 3. Ethereum upgrade review matrix

This is a provisional impact inventory for independent review. "Candidate" means possible future requirement language, not a normative classification today. Grouped entries avoid implying a universal control for changes relevant only to specialized applications.

| Upgrade | EIPs | Initial treatment | Security-review consequence |
|---|---|---|---|
| Dencun | 1153, 6780 | Candidate | Transient lifetime/ownership and corrected self-destruction review. |
| Dencun | 4788, 4844, 7516 | Guidance | Proof, finality, blob-data and fee assumptions where consumed by an application. |
| Dencun | 5656 | Tooling | Recognize `MCOPY` in analysis and assembly review. |
| Dencun | 7044, 7045, 7514 | No generic new control identified | Validator exits, attestations and churn; specialized integrations may need review. |
| Pectra | 7702, 2537 | Candidate | Account assumptions, delegated execution and conditional BLS verification. |
| Pectra | 2935, 6110, 7002, 7685 | Guidance | Historical hashes and staking/deposit/exit integrations. |
| Pectra | 7623 | Guidance | Calldata-heavy gas and denial-of-service analysis. |
| Pectra | 7251, 7549, 7691, 7840 | No generic new control identified | Validator/attestation changes and blob parameters/configuration. |
| Fusaka | 7825, 7951 | Candidate | Transaction liveness bounds and conditional P-256 verification. |
| Fusaka | 7823, 7883 | Guidance | MODEXP input bounds and gas repricing. |
| Fusaka | 7939 | Tooling | Recognize `CLZ`. |
| Fusaka | 7918, 7935 | Guidance | Blob fee economics and block gas assumptions; preserve the separate transaction cap. |
| Fusaka | 7594, 7917, 7934, 7892, 7642, 7910 | No generic new control identified | Data availability, proposer information, block bounds, blob-only forks, networking and RPC. |

BPO1/BPO2 blob-parameter changes do not introduce a generic contract-security control in this review. A specialized dependency still needs analysis. Validate the inventory against the final [Dencun](https://eips.ethereum.org/EIPS/eip-7569), [Pectra](https://eips.ethereum.org/EIPS/eip-7600) and [Fusaka](https://eips.ethereum.org/EIPS/eip-7607) meta-EIPs and execution specifications before any future adoption.

## 4. Reconcile with v3 and public feedback

The future integration needs a requirement-by-requirement comparison with v3: `tx.origin`, `selfdestruct()`, delegated/external calls, reentrancy, signatures, gas/DoS, compiler bugs, and post-deployment monitoring. Verify that review tools understand the target fork's opcodes, delegation indicators and precompiles. Do not silently weaken existing v3 obligations.

| Public feedback | Treatment in this review |
|---|---|
| [#15](https://github.com/EntEthAlliance/EthTrust-public/issues/15) | Keep the mechanically detectable S-v4-01 rule and manual M-v4-01a override. |
| [#16](https://github.com/EntEthAlliance/EthTrust-public/issues/16) | Preserve indicator visibility and cross-transaction changes; limit the same-transaction claim to delegation state. Add independent M-v4-02a balance/nonce review. |
| [#17](https://github.com/EntEthAlliance/EthTrust-public/issues/17) | Retain GP-v4-11 counterparty/callback compatibility and internal/external code observations. |
| [#18](https://github.com/EntEthAlliance/EthTrust-public/issues/18) | Keep M-v4-09 at Level M; require one issuance-stage cutoff and reproducible evidence. Extending per-bug S checks remains open. |
| [#19](https://github.com/EntEthAlliance/EthTrust-public/issues/19) | Retain exact precompile result handling, P-256 malleability and operation-specific BLS subgroup/scalar checks. |
| [#20](https://github.com/EntEthAlliance/EthTrust-public/issues/20) | Retain transient rollback, normal-return lifetime and shared module namespace rules. |

These comments were submitted by Ignacio Freire (Olympix), with further discussion by public participants. Incorporating feedback is not EEA or Working Group approval. [EthTrust-public #7](https://github.com/EntEthAlliance/EthTrust-public/issues/7) also needs item-by-item reconciliation: v3 already contains organizational/off-chain security, adversarial conditions, fuzzing, mutation testing, symbolic execution and formal verification material. Record any remaining gaps rather than duplicate those sections.

## 5. GBBC Capital Markets Risk Mitigation Framework

**This entire section is non-normative.** It adds no conformance, certification or endorsement condition.

The [GBBC Capital Markets Risk Mitigation Framework](https://rmf.gbbc.io/) addresses broader institutional risks of blockchain infrastructure. EthTrust can be considered as an implementation-level smart-contract security reference within that broader context. The [RMF Phase 2 materials](https://www.gbbc.io/media/rmf-phase2) cover Layer-2 infrastructure and application risks for digital payments and tokenized securities.

No GBBC participant is represented as having reviewed, approved or endorsed this workspace. This proposed relationship implies neither GBBC endorsement of EthTrust nor EEA endorsement of RMF. Consult the RMF site for its current participants; this review does not maintain a roster.

| Potential RMF risk area | Possible EthTrust contribution |
|---|---|
| Smart-contract vulnerabilities | Code review at the approved EthTrust security levels. |
| Privileged access and change management | Access controls, least privilege, upgrades and delegated-account review. |
| Authentication and signature risk | Signature handling, replay protection and precompile validation. |
| External dependencies and ordering | External-call/oracle review and adversarial testing. |
| Operational and protocol-change risk | Off-chain security posture, monitoring and reassessment guidance. |
| Availability | Gas/DoS analysis and transaction execution bounds. |

These are possible mappings, not verified RMF control mappings. Any formal cross-reference should identify the applicable RMF edition/version and exact risk/control IDs, and be reviewed with GBBC/RMF maintainers and through the appropriate future EEA process before publication. RMF crosswalk work is separate from adoption of any EthTrust technical requirements.

## 6. Work required before any future specification

- Establish or reactivate the appropriate EEA Working Group process; this workspace does neither.
- Obtain independent human security review, including reviewers from multiple security organizations.
- Reconcile every candidate with v3, validate levels and define objective tests for proposed `[S]` predicates.
- Validate the EIP inventory, target-fork semantics and compiler-advisory evidence requirements.
- Produce positive and negative tests for construction/deployment, delegation, balance/nonce changes, transient rollback/collisions, self-destruction, gas limits and precompile failures.
- Record public comment dispositions through [EthTrust-public](https://github.com/EntEthAlliance/EthTrust-public).
- Obtain EEA approval under the applicable governance process and produce a complete, self-contained specification and checklist.

None of these gates is satisfied merely by merging this review workspace. Editors should keep the README, working draft, checklist and HTML summaries consistent when candidates change. Placeholder IDs should be reconciled with final anchors only in a future adoption process.
