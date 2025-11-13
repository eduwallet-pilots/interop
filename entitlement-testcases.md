# **🧩 Functional Test Flows — Entitlement Verifiable Credential (VCT-based)**

**Environment:** DIIP v4 Compatible Testbed (2025)  
 **Credential Type:** `entitlement`  
 **Issuer:** `https://issuer.dev.eduid.nl` → `did:web:issuer.dev.eduid.nl`  
 **Protocols:**

* OIDC4VCI — issuance

* OpenID4VP — presentation

* DCQL — claim-level constraint and retrieval queries

**Actors:**

* **Issuer:** `issuer.dev.eduid.nl`

* **Wallet:** Holder / researcher / student wallet

* **Verifier:** Relying party requesting proof of entitlement to resources

---

## **1️⃣ Trust & Metadata Discovery**

**Goal:** Validate that Wallet and Verifier discover and recognize the `entitlement` credential type.

**Steps**

1. Wallet retrieves issuer metadata from  
    `https://issuer.dev.eduid.nl/.well-known/openid-credential-issuer`.

2. Wallet finds `credential_type=entitlement`.

3. VCT schema for `entitlement` is fetched and parsed.

4. Wallet and Verifier resolve DID: `did:web:issuer.dev.eduid.nl`.

**Expected**

* Issuer DID resolves successfully.

* VCT contains one claim path: `entitlement`.

* VCT display matches color `#8f0f12` and eduID logo.

✅ **Pass:** VCT is recognized and correctly rendered.

---

## **2️⃣ Issuance — OIDC4VCI (Entitlement)**

**Goal:** Ensure the Wallet can complete an issuance of an `entitlement` VC.

**Steps**

1. User authenticates via the eduID IdP.

Issuer constructs issuance request:

 {  
  "credential\_type": "entitlement",  
  "format": "vc+sd-jwt"  
}

2.   
3. Wallet performs OIDC4VCI token exchange.

Issuer issues credential payload:

 {  
  "entitlement": "urn:mace:surf.nl:invite.test.surfconext.nl:907a56f7-ca42-4cec-954f-b199db541588:eduwallet\_pilot\_deelnemer\_dev"  
}

4.   
5. Wallet verifies signature against DID.

6. Wallet stores credential and displays it per VCT render settings.

**Expected**

* Credential is signed and valid.

* Wallet display uses localized name:

  * English: “Entitlement”

  * Dutch: “Recht”

* Claim `"entitlement"` is `"sd": "mandatory"` (must be disclosed in all valid presentations).

✅ **Pass:** Credential issued, verified, and stored.

---

## **3️⃣ Wallet Storage and Claim Policy**

**Goal:** Ensure Wallet enforces VCT-specified `"sd": "mandatory"` disclosure behavior.

**Steps**

1. Wallet lists entitlement credentials.

2. Inspect claim policy.

3. Attempt selective disclosure.

**Expected**

* Wallet **must always** disclose the `"entitlement"` claim.

* No option to withhold or redact the field.

* Claim value matches the issued URN string.

✅ **Pass:** Wallet enforces mandatory disclosure as defined.

---

## **4️⃣ Presentation — OpenID4VP (Entitlement)**

**Goal:** Verify that the Wallet can present a valid `entitlement` VC to a Verifier.

**Steps**

1. Verifier issues presentation request for:

   * `credential_type: entitlement`

   * Claim: `"entitlement"`

2. Wallet generates a Verifiable Presentation (VP).

3. VP includes the mandatory `entitlement` value.

4. Wallet signs VP using holder DID.

5. Verifier validates signature and issuer DID.

**Expected**

* VP accepted.

* Claim `"entitlement"` present and matches original value.

* Proof chain valid: `Holder → Issuer`.

✅ **Pass:** Valid presentation accepted by Verifier.

---

## **5️⃣ Revocation & Status Check**

**Goal:** Verify revocation handling for entitlement credentials.

**Steps**

1. Issuer revokes a credential using DIIP status endpoint.

2. Wallet attempts to present revoked VC.

3. Verifier checks `credentialStatus`.

**Expected**

* Verifier detects revocation and rejects VP.

✅ **Pass:** Revoked entitlement cannot be reused.

---

## **6️⃣ DCQL Query — Specific Entitlement Value**

**Goal:** Ensure Verifier can query and confirm a precise entitlement value via DCQL.

### **Scenario 6A — Check for Specific Entitlement**

**Objective:** Confirm user holds the exact entitlement:

urn:mace:surf.nl:invite.test.surfconext.nl:907a56f7-ca42-4cec-954f-b199db541588:eduwallet\_pilot\_deelnemer\_dev

**DCQL Query Example:**

{  
  "select": \["entitlement"\],  
  "where": {  
    "entitlement": {  
      "eq": "urn:mace:surf.nl:invite.test.surfconext.nl:907a56f7-ca42-4cec-954f-b199db541588:eduwallet\_pilot\_deelnemer\_dev"  
    }  
  }  
}

**Steps**

1. Verifier sends the above DCQL query in a presentation definition.

2. Wallet evaluates stored entitlement VC.

3. Claim matches query condition.

4. Wallet returns VP disclosing full `"entitlement"` claim and proof.

**Expected**

* DCQL evaluation returns true.

* Verifier receives disclosed value and cryptographic proof bound to issuer.

* Any mismatch value yields `no_match` response.

✅ **Pass:** Verifier confirms exact entitlement via DCQL.

---

## **Scenario 6B — Negative DCQL Query — Nonexistent Entitlement**

**Goal:** Validate correct response when entitlement value does not match.

**DCQL Query Example:**

{  
  "select": \["entitlement"\],  
  "where": {  
    "entitlement": {  
      "eq": "urn:mace:surf.nl:invite.test.surfconext.nl:00000000-0000-0000-0000-000000000000:invalid"  
    }  
  }  
}

**Expected**

* Wallet evaluates VC.

* No match found.

Verifier receives:

 { "result": "no\_match" }

* 

✅ **Pass:** Query correctly returns no match.

