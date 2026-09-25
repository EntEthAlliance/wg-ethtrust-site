# EthTrust v4 exploratory review workspace

The EthTrust Working Group is currently inactive. This is an EEA-hosted exploratory technical review. It does not reactivate the Working Group, create an EEA specification, establish a certification baseline, or represent EEA approval. Feedback may inform a future Working Group process if one is established.

Prepared with AI assistance (OpenAI ChatGPT/Codex), this workspace incorporates public feedback and still requires independent human security review. The published [EthTrust v3](../v3/) remains the baseline. Candidate terms such as `MUST`, `[S]`, `[M]`, `[Q]` and `[GP]` test possible future language and have no normative force here.

- [Working draft](./WORKING-DRAFT.md): technical analysis and provisional candidate requirements.
- [Candidate checklist](./CHECKLIST-DRAFT.md): review questions and evidence, not an approved certification checklist.
- [HTML review page](./index.html): public presentation of the same review.
- [Tracking issue #6](https://github.com/EntEthAlliance/wg-ethtrust-site/issues/6).
- [Public feedback](https://github.com/EntEthAlliance/EthTrust-public).

The review uses v3 as its baseline and considers Ethereum execution changes through Dencun, Pectra and Fusaka. This is a fixed review scope, not a claim to cover every later active fork. The [feedback disposition](./WORKING-DRAFT.md#4-reconcile-with-v3-and-public-feedback) records the technical corrections from EthTrust-public #15-#20.

Account-code introspection cannot by itself prove EOA status, non-programmability or trust. Delegation-indicator stability during transaction execution is distinct from constructor and same-transaction deployment behavior. Balance/nonce review has its own M-v4-02a candidate, independent of `tx.origin`. M-v4-09 remains Level M and uses one reproducible certification cutoff established by a final advisory refresh immediately before issuance, with compiler, configuration, build, source, timestamp and snapshot/version evidence.

The [GBBC RMF section](./WORKING-DRAFT.md#5-gbbc-capital-markets-risk-mitigation-framework) is explicitly non-normative. It proposes areas for mapping, implies no endorsement, and requires exact RMF edition/control references and maintainer review before any formal cross-reference.

Keep these four documents aligned when editing. A future specification would require an appropriate EEA process, independent technical review, v3 reconciliation, objective tests, public comment disposition, EEA approval, and a complete specification and checklist. Merging this workspace does not satisfy those steps.
