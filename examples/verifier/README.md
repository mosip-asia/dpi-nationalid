# 🔍 Decentralized Reference Verifier — Relying Party Integration Guide

> **Part of the MOSIP Asia Digital Public Infrastructure (DPI) Suite**  
> **Agency A:** Department of Provincial Administration (DOPA) Reference Implementations  
> **Target Audience:** Relying Parties (Banks, Universities, Telecoms, Border Control, Hackathon Submission Portals)

---

## 💡 The Core Paradigm: Decentralized "Zero-Phone-Home" Verification

In legacy digital identity systems, verifying a citizen's credential required the relying party (e.g., a commercial bank or university) to make an API call back to the identity issuer (DOPA):

```
❌ THE LEGACY CENTRALIZED TRAP (Phone-Home):
┌────────────────┐                     ┌───────────────────────────┐
│  Relying Party │────(API Request)───▶│ DOPA Central Verification │
│ (Bank/School)  │◀───(Citizen Data)───│          Server           │
└────────────────┘                     └───────────────────────────┘
• Mass citizen tracking & surveillance risk
• Centralized bottleneck & single point of failure
• Network dependency & downtime propagation
```

Under the **W3C Verifiable Credentials & DPI Trust Framework**, verification is **100% decentralized**:

```
✅ DECENTRALIZED TRUST FRAMEWORK (Zero-Phone-Home):
┌──────────────────────────┐             ┌─────────────────────────────┐
│ 1. Citizen Public Wallet │             │      2. Relying Party       │
│  (dpi-wallet PWA)        │────(VP)────▶│   (Bank / University / App) │
└──────────────────────────┘             └──────────────┬──────────────┘
                                                        │
                      ┌─────────────────────────────────┴─────────────────────────────────┐
                      │ Decentralized Verification (NO contact with DOPA!)                │
                      ▼                                                                   ▼
       ┌───────────────────────────────┐                                   ┌───────────────────────────────┐
       │   dpi-trust: VDR did:web      │                                   │  dpi-trust: StatusList2021    │
       │  (Resolves DOPA Public Key)   │                                   │ (Checks Revocation Bit Array) │
       └───────────────────────────────┘                                   └───────────────────────────────┘
```

### Why Zero-Phone-Home Matters:
1. **Privacy-Preserving:** DOPA never knows where, when, or why a citizen presents their identity.
2. **High Availability:** Relying parties can verify identities even during ISP outages, natural disasters, or DOPA server maintenance.
3. **Infinite Horizontal Scale:** Verification throughput is limited only by the relying party's local compute; DOPA servers experience zero verification load.

---

## 🚀 Running the Reference Verifier

The included script [`verify.py`](./verify.py) is a dependency-free, standard Python 3 reference verifier.

### 1. Offline Verification (Using Bundled Fixtures)
```bash
python3 verify.py --offline
```

### 2. Live Verification Against Sandbox Trust Framework
```bash
python3 verify.py --domain dpi.ait.ac.th --vc fixtures/national-id-vc.json
```

---

## 🔬 Step-by-Step Verification Protocol

Any relying party implementing verification must execute these 5 deterministic checks:

### Step 1: Structural & Schema Validation
* Ensure `@context` includes `https://www.w3.org/2018/credentials/v1` and the national profile context.
* Confirm `type` array includes `VerifiableCredential` and `ThaiNationalIDCredential`.

### Step 2: Issuer Public Key Resolution (VDR)
* Read the `issuer` field: `did:web:vdr.trust.dpi.ait.ac.th:issuers:dopa`.
* Resolve the DID document from `https://vdr.trust.dpi.ait.ac.th/issuers/dopa/did.json` (or cached locally).
* Extract the verification key referenced by `proof.verificationMethod`.

### Step 3: Temporal Validity Window
* Validate that `issuanceDate` $\le$ `current_utc_time` $\le$ `expirationDate`.

### Step 4: Revocation Status (W3C StatusList2021)
* Read `credentialStatus` from the VC:
  ```json
  "credentialStatus": {
    "type": "StatusList2021Entry",
    "statusPurpose": "revocation",
    "statusListIndex": "100",
    "statusListCredential": "https://vdr.trust.dpi.ait.ac.th/status/status-list-2021.json"
  }
  ```
* Fetch or read cached `statusListCredential`.
* Base64-decode and GZIP-decompress `encodedList`.
* Check byte index `100 // 8` and bit `100 % 8`:
  * If bit is `0` $\rightarrow$ **Active** (Valid).
  * If bit is `1` $\rightarrow$ **Revoked** (Reject immediately).

### Step 5: Accreditation Check (Trusted Issuers List)
* Check that DOPA's DID is present in `https://registry.trust.dpi.ait.ac.th/api/v1/issuers`.

---

## 📁 Fixture Files

| File | Purpose |
|---|---|
| [`fixtures/national-id-vc.json`](./fixtures/national-id-vc.json) | Valid Thai National ID Verifiable Credential issued by DOPA |
| [`fixtures/did.json`](./fixtures/did.json) | DOPA's public DID Document hosted on `vdr.trust` |
| [`fixtures/status-list-2021.json`](./fixtures/status-list-2021.json) | StatusList2021 credential containing bitstring revocation flags |
