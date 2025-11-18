# **🧩 Functional Test Flows — OpenBadge Verifiable Credential**

**Environment:** EduBadges "development"
**Credential Type:** `eduID` (VCT definition per eduID spec)
**Issuer:** TBD - We are currently rolling out a temporary issuer until the issuer for the pilot is ready and released in the pilot environment.
**Protocols:**:
    * OIDC4VCI — issuance

## **1️⃣ Trust & Metadata Discovery**

**Goal:** Verify the Wallet and Verifier discover and validate the EduCredential credential configuration, issuer metadata, and DID.

**Steps**

1. Wallet requests issuer metadata:  
    `https://TBD/.well-known/openid-credential-issuer`

2. Wallet retrieves VCT definition (`EduCredential`) and parses `display` and `claims` arrays.

3. Wallet and Verifier resolve DID (web):
    `did:web:TBD`

4. Cache issuer signing keys and capabilities.

**Expected**

* Metadata exposes supported `credential_configurations` with a configuration of `EduCredential`, its `credential_type` is a.o. `OpenBadgeCredential`

* DID document resolves successfully with verification keys.

* VCT definition includes localized display and claim definitions (as provided).

* The TBD issuer is "trusted" by the wallet so that no extra warning dialogs must be clicked through.

✅ **Pass:** Wallet and Verifier both recognize issuer and supported credential type.


## **2️⃣ Issuance — OIDC4VCI (EduCredential)**

**Goal:** Ensure Wallet can complete an issuance flow for an eduID credential from `TBD`.

**Steps**

1. Student logs in on EduBadges. This is called "their backpack". 
1. Student sees several "Badges". Each Badge offers the posssibility to "import in a wallet". Select the first.
1. Starting the "import in wallet", creates a OID4VCI offer (with pre-authorized code flow) for this credential and presents it as QR.
1. Wallet scans this QR (or copy pastes it in dev mode) and starts the OID4VCI flow with `TBD` issuer.
1. Wallet initiates OIDC4VCI token exchange at `TBD` issuer.
1. Wallet verifies signature via Issuer DID document.
1. Wallet stores and renders credential per EduCredential `display` configuration.

**Expected**

* Credential issued, verifiable, and adheres to claim definitions.
* Credential shows in the wallet using the OpenBadge name and description as provided in the *claims* (i.e. `.credentialSubject.achievement.name` etc)

✅ **Pass:** Credential stored and rendered correctly using the name for this badge

---

## **3️⃣ Wallet Storage and rendering**

**Goal:** Wallet stores multiple Credentials of type `OpenBadgeCredential` following the structure of the `EduCredential` credential_configuration.

**Steps**

1. Wallet user walks through steps in *issuance* multiple times for *distinct* badges. 
1. Wallet stores multiple badges. At least 2, but 3 or more is preferred. 10 is a realistic amount in practice.
1. Wallet user can distinguish all the OpenBadges it recieved:
  1. Amongst other credential types
  1. Amongst eachother

**Expected**

* An ever growing amount of badges can be navigated, interpreted and displayed by and to a human user.
* Each distinct edubadge credential is visually distinctive so that the various badges can be uniquely identified. 
* Concepts like pagination, search or filtering may be used to help here.

✅ **Pass:** Wallet shows OpenBadge Credentials in the wallet in way that uniquely distinguishes that specific credential by e.g. its name, narrative, image or other distinguishable attributes
❌ **Fail** Wallet shows a long list badges with repeated names like "EduCredential" and visual identities like the generic "display"