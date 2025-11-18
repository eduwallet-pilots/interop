# **🧩 Functional Test Flows — OpenBadge Verifiable Credential**

**Environment:** EduBadges "development"
**Credential Type:** `eduID` (VCT definition per eduID spec)
**Issuer:** TBD - We are currently rolling out a temporary issuer until the issuer for the pilot is ready and released in the pilot environment.
**Protocols:**:
    * OIDC4VCI — issuance
    * OpenID4VP — presentation

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

## **4️⃣ Presentation — OpenID4VP (Badge with Results)**

**Goal:** Enable a teacher to request and verify a specific OpenBadge Credential containing results (marks, a number from 0-10?) from a student's Wallet.

**Protocol:** OpenID for Verifiable Presentations 1.0

**Steps**

1. Teacher uses software (Verifier application) to create a presentation request for a specific OpenBadge Credential type with "results" (assessment scores/grades).
1. The Verifier generates an Authorization Request containing:
   - `response_type=vp_token`
   - `response_mode=direct_post` (for cross-device flow)
   - `dcql_query` specifying:
     - `format`: `dc+sd-jwt` or credential format used by EduBadges
     - `meta.vct_values`: credential type for OpenBadgeCredential with results
     - `claims`: requesting specific claims including:
       - `.credentialSubject.achievement.name` (badge name)
       - `.credentialSubject.result` (assessment results)
       - `.credentialSubject.achievement.resultDescription` (assessment results)
       - Student name/identifier claims
   - `nonce`: fresh cryptographic random value for replay protection
   - `client_id`: Verifier's client identifier with appropriate prefix
1. Request is presented as QR code or deep link to student
1. Student scans QR code with Wallet, triggering OpenID4VP flow
1. Wallet processes Authorization Request and displays:
   - Verifier information (teacher/institution)
   - Requested badge name and results data
   - Consent request
1. Student authenticates and consents to share the specific badge with results
1. Wallet prepares Verifiable Presentation containing:
   - The requested OpenBadge Credential
   - Cryptographic Holder Binding proof (Key Binding JWT)
   - Bound to the `nonce` and `client_id` (audience) from request
1. Wallet sends Authorization Response via HTTP POST to Verifier's `response_uri` (direct_post mode)
1. Verifier receives and validates the VP Token:
   - Verifies Holder Binding proof
   - Validates `nonce` matches request
   - Validates `aud` (audience) matches `client_id`
   - Verifies Issuer signature (EduBadges)
   - Extracts and displays badge name, student identifier, results and resultDescription `requiredValue`
1. Teacher views verified badge information including student name and assessment results

**Expected**

* Verifier successfully creates presentation request specifying OpenBadge with results
* Student can identify the specific badge being requested
* Verifiable Presentation is cryptographically bound to transaction (nonce) and Verifier (audience)
* Verifier successfully validates:
  - Issuer signature proves badge issued by EduBadges
  - Holder Binding proves student controls the credential
  - Nonce binding prevents replay attacks
  - All requested claims (badge name, results, student identifier) are present
  - Wether or not the "result" qualifies as "passed".
* Teacher can view and trust the badge name, student name, and results as authentically issued by EduBadges

✅ **Pass:** Complete OpenID4VP flow with cryptographic verification of badge with results presented from student to teacher. The teacher can see what student has presented a badge that proves they passed the criteria set in the badge

## **5️⃣ Presentation — OpenID4VP (Student-Initiated Share via QR)**

**Goal:** Enable a student to proactively share a specific OpenBadge Credential with an HR employee by generating a shareable QR code from their Wallet.

**Protocol:** OpenID for Verifiable Presentations 1.0 (Verifier-initiated flow with student-driven selection)

**Steps**

1. HR employee asks the student to share a specific OpenBadge credential
1. Student opens their Wallet and navigates to the requested OpenBadge Credential
1. Student selects "Share" functionality for the chosen badge
1. Wallet generates a presentation-ready payload:
   - Creates an Authorization Request on behalf of the student for this specific credential
   - `response_type=vp_token`
   - `response_mode=direct_post`
   - `dcql_query` specifying the selected OpenBadge credential with all attributes:
     - `format`: credential format used by EduBadges
     - `meta.vct_values`: the specific OpenBadgeCredential type
     - `claims`: all available claims including:
       - `.credentialSubject.id` (student identifier)
       - `.credentialSubject.achievement.name` (badge name)
       - `.credentialSubject.achievement.description` (badge description)
       - `.credentialSubject.result` (if present - assessment results)
       - `.credentialSubject.achievement.resultDescription` (if present)
       - `.credentialSubject.achievement.criteria` (badge criteria)
       - All other credential attributes
   - `nonce`: fresh cryptographic random value for this share session
   - `response_uri`: endpoint where verifier should POST the response
1. Wallet presents the Authorization Request as QR code to student
1. HR employee scans QR code with Verifier application
1. Verifier processes Authorization Request and automatically initiates OpenID4VP flow
1. Student's Wallet (as the responding party) prepares Verifiable Presentation containing:
   - The complete OpenBadge Credential with all attributes
   - Cryptographic Holder Binding proof (Key Binding JWT)
   - Bound to the `nonce` and verifier identifier from request
1. Wallet sends Authorization Response via HTTP POST to the `response_uri`
1. HR employee's Verifier application receives and validates the VP Token:
   - Verifies Holder Binding proof (confirms student controls credential)
   - Validates `nonce` matches the share session
   - Verifies Issuer signature (EduBadges issuer)
   - Validates issuer is trusted
1. HR employee views the complete verified credential including:
   - Student name/identifier (credentialSubject.id)
   - Badge name and description
   - Results and result description (if applicable)
   - Badge criteria and narrative
   - Issuance date and issuer information
   - All other credential attributes

**Expected**

* Student can easily select and initiate sharing of a specific badge from their Wallet
* Wallet generates cryptographically secure presentation request
* QR code is scannable and initiates OpenID4VP flow on Verifier side
* HR employee can scan with standard Verifier application
* Verifiable Presentation includes complete credential with all attributes
* Verifier successfully validates:
  - Issuer signature proves badge authentically issued by EduBadges
  - Holder Binding proves the student presenting is the legitimate credential subject
  - Nonce binding ensures freshness of this specific share session
  - Credential has not been revoked or tampered with
* HR employee can view and trust all credential attributes:
  - Student identity confirmed
  - Badge details (name, description, criteria) visible
  - Results and assessment information visible (if present)
  - Issuance provenance verified
* Entire credential validity is verifiable

✅ **Pass:** Student successfully generates shareable QR code for specific badge, HR employee scans and verifies complete credential with all attributes, cryptographic verification confirms issuer authenticity and student as legitimate holder

