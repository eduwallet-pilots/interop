# **🧩 Functional Test Flows — eduID Verifiable Credential (VCT-based)**

**Environment:** DIIP v4 compatible Testbed (2025)  
 **Credential Type:** `eduID` (VCT definition per eduID spec)  
 **Issuer:** `https://issuer.dev.eduid.nl` → `did:web:issuer.dev.eduid.nl`  
 **Protocols:**

* OIDC4VCI — issuance

* OpenID4VP — presentation

* DCQL — claim-level querying / constraint language (per DIIP v4)

**Actors:**

* **Issuer:** `issuer.dev.eduid.nl` (eduID Test Issuer)

* **Wallet:** Learner / Student Wallet (VC Holder)

* **Verifier:** Service Provider / relying party validating eduID credentials

---

## **1️⃣ Trust & Metadata Discovery**

**Goal:** Verify the Wallet and Verifier discover and validate the eduID credential type, issuer metadata, and DID.

**Steps**

1. Wallet requests issuer metadata:  
    `https://issuer.dev.eduid.nl/.well-known/openid-credential-issuer`

2. Wallet retrieves VCT definition (`eduID`) and parses `display` and `claims` arrays.

3. Wallet and Verifier resolve DID:  
    `did:web:issuer.dev.eduid.nl`

4. Cache issuer signing keys and capabilities.

**Expected**

* Metadata exposes supported `credential_type = eduID`.

* DID document resolves successfully with verification keys.

* VCT definition includes localized display and claim definitions (as provided).

✅ **Pass:** Wallet and Verifier both recognize issuer and supported credential type.

---

## **2️⃣ Issuance — OIDC4VCI (eduID)**

**Goal:** Ensure Wallet can complete an issuance flow for an eduID credential from `issuer.dev.eduid.nl`.

**Steps**

1. Student authenticates via eduID IdP.

Issuer constructs issuance request with:
```
 {  
  "credential_type": "eduID",  
  "format": "vc+sd-jwt"  
}
```
2.   
3. Wallet initiates OIDC4VCI token exchange.

Issuer issues credential (example):
```
 {  
  "name": "Alice Nguyen",  
  "email": "alice@exampleuniversity.nl",  
  "schac_home_organization": "exampleuniversity.nl",  
  "eduperson_scoped_affiliation": "student@exampleuniversity.nl",  
  "eduperson_assurance": "https://eduid.nl/assurance/email-validated",  
  "is_student": true  
}
```
4. Wallet verifies signature via Issuer DID document.

5. Wallet stores and renders credential per VCT `display` configuration (color `#8f0f12`, eduID logo).

**Expected**

* Credential issued, verifiable, and adheres to VCT claim definitions.

* All `sd: "never"` claims (e.g., `eduperson_assurance`) are included but marked non-disclosable.

✅ **Pass:** Credential stored and rendered correctly.

---

## **3️⃣ Wallet Storage and Claim Management**

**Goal:** Validate Wallet’s storage and interpretation of eduID claim data and SD permissions.

**Steps**

1. Wallet lists all credentials from `issuer.dev.eduid.nl`.

2. Confirm claim mapping aligns with VCT claim paths.

3. Attempt to export credential (JWT or SD-JWT format).

**Expected**

* All `"sd": "allowed"` claims visible for possible disclosure.

* `"sd": "never"` claim (assurance) cannot be selected for disclosure.

* Wallet uses VCT-provided display labels in the correct locale.

✅ **Pass:** Wallet honors `sd` policy and displays correct localized info.

---

## **4️⃣ Presentation — OpenID4VP (eduID)**

**Goal:** Validate full selective-disclosure presentation to a Verifier.

**Preconditions:**

* Verifier supports OpenID4VP and presentation definition referencing `credential_type=eduID`.

**Steps**

1. Verifier issues presentation request asking for:

   * `name`

   * `is_student`

   * `schac_home_organization`

2. Wallet locates stored eduID VC matching presentation definition.

3. Wallet generates Verifiable Presentation (VP) disclosing only requested `sd=allowed` claims.

4. Wallet signs VP with holder DID.

5. Verifier verifies VP, Issuer signature, and proof chain.

**Expected**

* VP includes only requested claims.

* Signatures and proofs validate.

* `"eduperson_assurance"` remains undisclosed.

✅ **Pass:** Verifier accepts valid presentation, rejects excess or forbidden disclosures.

---

## **5️⃣ Revocation & Status Check**

**Goal:** Confirm status-based revocation detection.

**Steps**

1. Issuer revokes a credential (via DIIP testbed API or `statusList2021` endpoint).

2. Wallet attempts to present revoked VC.

3. Verifier checks credential’s `credentialStatus` URL.

**Expected**

* Revoked status \= true → VP rejected.

* Verifier provides `credential_revoked` error.

✅ **Pass:** Revoked credential cannot be reused.

---

## **6️⃣ Selective Disclosure Scenarios**

**Goal:** Verify selective attribute release for eduID claims.

**Scenario A — Library Walk-In**  
 Verifier requests only `is_library-walk-in`.

**Scenario B — Staff Portal**  
 Verifier requests `is_staff`, `schac_home_organization`.

**Scenario C — Alumni Access**  
 Verifier requests `is_alum`, `name`.

**Expected**

* Wallet discloses only claims marked `"sd":"allowed"`.

* `"eduperson_assurance"` and `"eduperson_scoped_affiliation"` withheld unless explicitly requested and allowed.

✅ **Pass:** Correct selective disclosure per VCT rules.

---

## **7️⃣ DCQL Query Scenario — eduID Claim Value Retrieval**

**Goal:** Validate Verifier’s use of **Data Credential Query Language (DCQL)** to assert or request specific eduID claim values.

### **Scenario 7A — Scoped Affiliation Match**

**Objective:** Verify that the eduID VC contains a specific `eduperson_scoped_affiliation` value.

**DCQL Example:**
```
{  
  "select": ["eduperson_scoped_affiliation"],  
  "where": {  
    "eduperson_scoped_affiliation": {  
      "eq": "student@exampleuniversity.nl"  
    }  
  }  
}
```
**Steps**

1. Verifier issues a DCQL query to the Wallet in a presentation request.

2. Wallet evaluates the VC against the query.

3. Wallet discloses matching value if found and allowed.

4. Verifier validates returned value and proof of inclusion.

**Expected**

* Query resolves successfully.

* Disclosed value \= `"student@exampleuniversity.nl"`.

* Proof cryptographically bound to original credential.

✅ **Pass:** Wallet returns only the matching value with valid disclosure proof.

---

### **Scenario 7B — Assurance Level Check**

**Objective:** Verify assurance claim matches a defined eduID assurance URI.

**DCQL Example:**
```{  
  "select": ["eduperson_assurance"],  
  "where": {  
    "eduperson_assurance": {  
      "eq": "https://eduid.nl/validated/email-validated"  
    }  
  }  
}
```
**Steps**

1. Verifier constructs DCQL query for assurance level.

2. Wallet evaluates VC.

3. Since `eduperson_assurance` is `"sd": "never"`, Wallet does **not** disclose raw value, but can return a *verified boolean* (`true`) if allowed by profile.

4. Verifier accepts boolean assertion if signed by Wallet and proof derived from Issuer signature.

**Expected**

* Wallet either returns non-disclosing proof of compliance or explicit denial (`not_disclosable`).

* If `true` response allowed, proof chain verifies.

✅ **Pass:** Verifier obtains assurance check result securely without violating SD policy.

