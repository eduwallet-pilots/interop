# eduWallet Interop tests

This repositories provides various testcases for validating interoperability in the context of the eduWallet pilots

To be able to test, various resources are needed, depending on the test case.

# Generic resources

(all currently in test or development)

* [eduID (dev) Issuer](https://issuer.dev.eduid.nl/) \- Issuer (DIIP v4 credential issuer for SD-JWT eduID credentials)  
* [eduID (test)](https://login.test.eduid.nl) \- SAML IdP and OIDC OP  
* [mijn eduID (test)](https://mijn.test.eduid.nl/) \- Self service environment to allow users/tests to manage eduIDs

* [eduWallet Pilot portal](https://portal.dev.eduwallet.nl) \- Pilot participant enrollment portal  
* SURFconext [invite](https://invite.test.surfconext.nl/home) \- Pilot participant invitation portal

* Edubadges \- platform for digital certificates, including microcredentials, for the Dutch education sector using OpenBadges. Has three stages:
  * [Development](https://edubadges.dev.sdp.surf.nl/) \- Unstable, behind EduVPN.
  * [Playground](https://edubadges.playground.sdp.surf.nl/) - Stable, not fully functional.
  * [Demo](https://demo.edubadges.nl/) - Stable, production ready, only issuance.
* [OpenBadge credential examples](https://www.educredentials.eu/obv3-examples/) \- Structure, JSON, documentation and issuance of _EduCredentials_, OpenBadge version 3 implementation with our business-logic

# Pilot Specific resources

# Credentials

* **eduID**  
  [https://github.com/eduwallet-pilots/interop/blob/main/credentials/eduid.vct](https://github.com/eduwallet-pilots/interop/blob/main/credentials/eduid.vct)  
* **entitlement** [https://github.com/eduwallet-pilots/interop/blob/main/credentials/entitlement.vct](https://github.com/eduwallet-pilots/interop/blob/main/credentials/entitlement.vct)

# Test cases

* **eduID** [https://github.com/eduwallet-pilots/interop/blob/main/testcases/eduID-testcases.md](https://github.com/eduwallet-pilots/interop/blob/main/testcases/eduID-testcases.md)  
* **entitlement** [https://github.com/eduwallet-pilots/interop/blob/main/testcases/entitlement-testcases.md](https://github.com/eduwallet-pilots/interop/blob/main/testcases/entitlement-testcases.md)
* **EduBadges** [https://github.com/eduwallet-pilots/interop/blob/main/testcases/openBadge-testcases.md](https://github.com/eduwallet-pilots/interop/blob/main/testcases/openBadge-testcases.md)

# OpenID Federation Testbed
* Testbed: https://testbed.dev.oidf.lab.surfconext.nl/
* TA: https://testbed.dev.oidf.lab.surfconext.nl
* Leafs: https://ta.dev.oidf.lab.surfconext.nl/list
* Leaf example:
  * **eduID**: https://leafs.dev.oidf.lab.surfconext.nl/leafs/a6261dc05c50195a641d635a8ac218dba4863b8d/
