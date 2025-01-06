# Verasion-Proposal
Proposal for upcoming implementaion of enhancing CORIM support for [Verasion/services](https://github.com/veraison/services)
# Overview

**Step 1: Analyze and Modify CoRIM Generation**

Current State: CoRIM generation in Veraison does not support multi-signatures. You'll need to modify the existing cocli tool (used for generating CoRIMs).
Actions:

Update CoRIM Schema: Modify the CoRIM schema to accept multiple signatures, ensuring it adheres to COSE standards for multi-signatures.
Use COSE Multi-Signature Structure (COSE_Multi_Signature) for signatures, where the list of signatures is embedded as part of the CoRIM metadata.
Implement the signatures[] field to store the multiple signatures.
Integrate COSE Libraries: Utilize Go libraries like veraison/go-cose for signing and verifying multi-signatures.
Modify cocli to Handle Multi-Signatures:
Modify the tool’s CoRIM creation logic to accept and generate multi-signatures.
Accept multiple signing keys (from different entities or authorities).
Output a CoRIM with the necessary multi-signature structure.

**Step 2: Implement Signature Validation in Veraison Plugin**

Current State: Veraison's existing plugins for attestation verification need to be extended to support multi-signatures.
Actions:

Modify the Plugin for CoRIM Verification:
Extend the current Veraison plugin architecture to verify multi-signatures.
Leverage COSE multi-signature verification methods (such as verifying the set of public keys corresponding to the signatures).
Ensure the plugin can validate the integrity of each component and the aggregation of the signatures, as per COSE standards.
Signature Handling:
Implement a signature aggregation logic to handle cases where the signatures need to be verified across multiple components in the CoRIM.
Ensure the integrity of the CoRIM’s content is validated against all signatures before trust is established.

**Step 3: Integration of COSE Multi-Signature Handling**

Current State: Veraison does not yet support multi-signature in COSE format.
Actions:

Adapt COSE for Multi-Signatures:
COSE supports multi-signature in a format where a signature is generated over the concatenation of the message and signatures.
Implement COSE_Multi_Signature to capture and verify multiple signatures for a CoRIM.
Public Key Handling:
Define and implement a method to handle the public keys of the signers in a multi-signature scenario.
Ensure public keys are encoded according to COSE standards and matched with the correct signatures.
Ensure COSE Compliance:
Validate that the multi-signature structure complies with COSE standards for signing and verifying multiple signatures in the context of CoRIM.

**Step 4: Ensure Secure Key Management and Configuration**

Current State: Secure management of keys is crucial for the multi-signature process.
Actions:

Key Management:
Implement secure key storage mechanisms (e.g., using hardware security modules or secure key vaults).
Implement access control to manage the signing keys, ensuring only authorized signers can sign CoRIMs.
Define Key Usage Policies:
Create rules for key usage in the context of multi-signatures (e.g., 2-out-of-3, threshold signatures).
Enforce policies on signature validation to ensure at least the required number of signatures are validated for the CoRIM to be considered valid.

**Step 5: Test CoRIM with Multi-Signature Support**

Current State: Testing the integration of multi-signatures in CoRIMs and Veraison services is essential.
Actions:

Unit Testing for CoRIM Generation:
Test the new CoRIM generation logic by generating CoRIMs with multiple signatures and verifying them using the cocli tool.
Unit Testing for Plugin:
Write unit tests for the Veraison plugin that verify the new multi-signature handling capabilities.
Integration Testing:
Test the complete process: generate a multi-signed CoRIM, verify the signatures through Veraison, and check that the integrity of the data is maintained.

**Step 6: Update Documentation**

Current State: Clear documentation of the new functionality is crucial for users and developers.
Actions:

Update CoRIM Generation Docs:
Document the changes to the CoRIM generation process, explaining how multi-signatures are added and verified.
Provide examples of multi-signed CoRIMs.
Update Veraison Plugin Docs:
Include details on the new signature verification methods and configurations for multi-signature handling in Veraison plugins.
Provide sample configuration files and troubleshooting tips for multi-signature verification.

**Step 7: Review Security Implications**

Current State: The implementation of multi-signatures requires robust security practices to protect the integrity of the system.
Actions:

Security Audits:
Perform code reviews and security audits on the multi-signature handling logic.
Penetration Testing:
Simulate attacks on the multi-signature implementation to ensure there are no vulnerabilities such as signature spoofing or key exposure risks.
Compliance Checks:
Ensure the implementation complies with relevant security standards (e.g., FIPS 140-2 for cryptographic modules).

**Step 8: Deploy to Staging and Production**

Current State: The solution needs to be deployed and monitored.
Actions:

Deploy to Staging:
Deploy the updated Veraison services to a staging environment for further validation.
Monitor for Issues:
Monitor for any issues related to multi-signature verification, performance, or security breaches.
Use logging to capture detailed error reports during CoRIM generation and verification.


# Proposal

# Goals

1. Enhanced Trust and Security
   Proposal Alignment:
   The plan emphasizes implementing multi-signature mechanisms where multiple entities sign the CoRIM, distributing trust and reducing reliance on a single entity.
   It integrates COSE’s tamper-resistance features, ensuring CoRIMs are invalidated if altered.
   The inclusion of rigorous testing (unit, integration, and compliance) ensures robustness and trustworthiness.


2. Standards Compliance
   Proposal Alignment:
   The plan explicitly focuses on adhering to COSE standards (RFC 8152), ensuring cryptographic operations and data structures are compliant.
   It leverages COSE-defined structures like COSE_Sign and COSE_Signature, maintaining compatibility with other systems and tools.


3. Scalability and Flexibility
   Proposal Alignment:
   The extended CoRIM schema allows for multi-party endorsements, addressing complex and distributed use cases such as federated IoT networks.
   By maintaining backward compatibility, the system ensures smooth adoption for current users while preparing for future scalability needs.


4. Robust Auditing and Traceability
   Proposal Alignment:
   The plan includes enhanced logging and audit mechanisms, recording all multi-signature processing activities for transparency and diagnostics.
   Key management practices, including rotation and revocation, further reinforce traceability and accountability.


5. Seamless Integration
   Proposal Alignment:
   The plan ensures backward compatibility by designing multi-signature support to coexist with single-signature workflows.
   Provisioning and verification APIs are upgraded to handle multi-signature operations without disrupting current functionality.


6. Enhanced Collaboration
   Proposal Alignment:
   Multi-signature CoRIMs enable endorsements from multiple stakeholders, fostering collaboration between manufacturers, service providers, and regulatory bodies.
   The plan’s focus on standards-based interoperability ensures trust across diverse and federated systems.

# Resources
[CBOR Object Signing and Encryption (COSE)](https://datatracker.ietf.org/doc/html/rfc8152)

[Concise Reference Integrity Manifest (CoRIM)]()

[CBOR (Concise Binary Object Representation]()

[Veraison Trusted Services Architecture](https://github.com/veraison/services)