# Verasion-Proposal
Proposal for upcoming implementaion of enhancing CORIM support for [Verasion/services](https://github.com/veraison/services)
# Overview

I have tried to include the general implementation of steps in the [Proposal]() section

**Step 1: Analyze and Modify CoRIM Generation**

Current State:Veraison services does not support multi-signatures

**Step 2: Implement Signature Validation in Veraison services**

Current State: Veraison's existing services for provisioning and attestation verification need to be extended to support multi-signatures.

**Step 3: Updating Cocli**

Current State: Cocli doest not handle the multiple signatures

**Step 4: Ensure Secure Key Management and Configuration**

Current State: Secure management of keys is crucial for the multi-signature process.


**Note**

I just searched up the steps in the proposal and for this one and try to practically think if this will work but i was not able to make any sense for this. May need to discuss with mentors for this.


**Step 5: Update Documentation**

Current State: Clear documentation of the new functionality is crucial for users and developers.


**Step 6: Review Security Implications**

Current State: The implementation is done in 4th step but need to be revised

# Proposal

****
**STEP-1**

Actions:

**[Modify CoRIM Schema to Support Multi-Signatures]()**


   Update CoRIM schema generation in the [verasion/corim](https://github.com/veraison/corim)


   Include a signatures[ ] field in CoRIM schema generator. 
   Each element in this array should hold a COSE signature structure, which consists of three parts: the protected header (including cryptographic algorithm info), unprotected fields (e.g., kid for key identifier), and the signature field containing the actual cryptographic signature.

   Example of a single Signature

      Single Signature
      
      This example uses the following:
      
      o  Signature Algorithm: ECDSA w/ SHA-256, Curve P-256
      
      Size of binary file is 103 bytes
      
         98(
     [
       / protected / h'',
       / unprotected / {},
       / payload / 'This is the content.',
       / signatures / [
         [
           / protected / h'a10126' / {
               \ alg \ 1:-7 \ ECDSA 256 \
             } / ,
           / unprotected / {
             / kid / 4:'11'
           },
           / signature / h'e2aeafd40d69d19dfe6e52077c5d7ff4e408282cbefb
      5d06cbf414af2e19d982ac45ac98b8544c908b4507de1e90b717c3d34816fe926a2b
      98f53afd2fa0f30a'
            ]
          ]
        ]
      )   


****
**STEP-2**

Actions:

**[Integrate COSE Libraries]()**


We can then use the go-cose library to handle the signing and verification processes for multiple signatures. This will require modifying the CoRIM generation logic to support the inclusion of multiple keys and output a CoRIM manifest that includes the multi-signature structure in the [veraison/corim](https://github.com/veraison/corim).

Once that is done, we need to update the attestation verification process in [veraison/services](https://github.com/veraison/services) repository to handle these multi-signed CoRIM files, ensuring that each signature is validated using the appropriate public key.


****
**STEP-3**

Actions:

**[Modify cocli to Handle Multi-Signatures]()**


Update cocli tool in [veraison/cocli](https://github.com/veraison/cocli) to accept multiple signing keys from different authorities.
Modify CoRIM creation logic to generate CoRIMs with embedded multi-signatures.
Output CoRIM that contains the required multi-signature structure for verification.

****
**STEP-4**

Actions:

**[Secure Key Management and Configuration]()**

**4.1: Implement Secure Key Storage Mechanisms**

Choose a Secure Key Storage Solution:

Use solutions such as Hardware Security Modules (HSMs), Cloud Key Management Systems (KMS), or Key Vaults to securely store cryptographic keys.
These solutions provide encryption at rest and access control to manage sensitive keys, ensuring that only authorized entities can access them.
Recommended Solutions:

Cloud Providers: AWS KMS, Azure Key Vault, Google Cloud KMS.
On-Premise Options: HashiCorp Vault, Fortanix, or an internal HSM.
Actions:

Integrate the key vault or HSM API into your Veraison services to access signing keys securely.
Ensure that the key storage is non-exportable for private keys, preventing leakage.
Key Access Control:

Define which entities have access to signing keys. For example, only certain services or administrators should be allowed to perform signing operations.
Implement role-based access control (RBAC) to grant or restrict access based on the user's role.
Actions:

Set up RBAC in your key storage solution to ensure that only the designated entities can access or sign using private keys.

**4.2: Define Key Usage Policies**

Key usage policies are critical in the context of multi-signatures to enforce rules around signing thresholds and key usage, ensuring the security of the system.

Threshold Signature Policy (e.g., 2-out-of-3):

Define the number of signatures required to consider a CoRIM valid. For example, if you have three signers, you may require two-out-of-three signatures.
Actions:

Create a policy for threshold signatures in your key management system and Veraison service configuration (e.g., config.yaml), indicating the minimum number of signatures required for validation.
Example: If the key management system stores keys for three signers, define that at least two signatures are required for a CoRIM manifest to be considered valid.
Signature Algorithms Policy:

Enforce the use of secure and standardized algorithms for signing, such as ECDSA or EdDSA.
This ensures consistency in the signing process and prevents weak algorithms from being used in your multi-signature process.
Actions:

Configure Veraison to accept only signatures generated with pre-approved algorithms.
Define supported algorithms in your Veraison configuration (e.g., signature_algorithms: [ECDSA, EdDSA]).
Key Rotation and Expiry Policies:

Implement policies to rotate signing keys periodically and manage key expiry to avoid potential security risks.
Define key expiration dates and automate the renewal of keys once they reach the expiration limit.
Actions:

Set up automated key rotation in the key management solution to ensure signing keys are periodically replaced with new keys.
Implement a policy in Veraison's services to check the expiration of keys during the verification process, rejecting CoRIM files signed by expired keys.

**4.3: Secure Configuration and Integration into Veraison Services**

After defining and implementing the key management policies, you will need to integrate them into Veraison's services repo.

Service Configuration for Key Access:

Update the Veraison service configuration (e.g., config.yaml) to include parameters for integrating with the secure key management solution (e.g., HSM, KMS).
Define the path to the key storage and required access credentials to securely fetch the signing keys for the multi-signature process.
Actions:

Add key management configurations to Veraison’s service configuration file:

      key_management:
        provider: AWS_KMS
        region: us-west-2
        key_id: example-key-id
        Key Signing Integration:

Update the signing process in the CoRIM generation logic to fetch signing keys securely from the key storage system.
When generating a CoRIM file, access the private keys from the storage solution to sign the manifest. Implement checks for valid key usage according to the defined key policies.
Actions:

Implement a function in Veraison to interact with the secure key management system and retrieve the private key for signing:

      func fetchKeyFromKMS(keyID string) (privateKey []byte, err error) {
      // Use AWS SDK to fetch private key from KMS
      }

Multi-Signature Validation:

During attestation verification, ensure that the service can validate multi-signatures by checking the number of valid signatures against the threshold signature policy.
Implement a verification function that compares the signatures in the CoRIM manifest against the public keys in your key management system, validating that at least the required number of signatures are present.
Actions:

Update the CoRIM verification process to handle multi-signature validation by iterating over the signatures[] array and confirming that the required threshold is met.
Use the go-cose library to verify the signatures:

      func verifySignatures(coRim CoRIMManifest) (bool, error) {
         validSignatures := 0
         for _, signature := range coRim.Signatures {
            if verifySignature(signature) {
            validSignatures++
               }
         }
      return validSignatures >= threshold, nil
      }

****
**STEP-5**

Actions:

**[Update Documents]()**

General changes that can be reflected in documentation can be -

The CoRIM schema was updated to support multi-signature functionality, enabling multiple entities to securely sign a CoRIM, enhancing the integrity verification process. The schema now includes a signatures[] field, where each signature contains metadata (e.g., algorithm and key identifier) and the signature itself. This allows for CoRIMs to be validated with multiple signatures from different signers, ensuring higher security and compliance with threshold signature policies.

For the service, Veraison was updated to integrate multi-signature validation. The service now checks that the CoRIM meets the required number of valid signatures, as specified in the configuration (e.g., 2-out-of-3 threshold). This was achieved by modifying Veraison's signing logic to handle the multi-signature process and integrating it with the go-cose library for signing and verification.

The cocli tool was modified to support the generation of CoRIM files with multiple signatures. It accepts multiple signing keys and outputs a CoRIM file with the necessary signature structure.

To ensure the security of the signing process, key management was addressed with secure key storage mechanisms like hardware security modules (HSM) or cloud key management services (KMS). Additionally, key usage policies were defined to enforce thresholds and ensure that only authorized signers can sign the CoRIM. This guarantees that the generated CoRIM files are secure and meet the integrity standards required for sensitive data verification.


****
**STEP-6**

Actions:

**[Revisiting the current security implementation]()**

I was thinking of maybe  following an approach where we could generate different scenarios and check the new security measures

And then keep updating them as per the testing. This will be the best way to achieve the most simple and reliable security layer

****
      

# Goals

1. **Enhanced Security through Multi-Signature Support:**

Integrated multi-signature functionality into CoRIM schema, allowing multiple signers to contribute to the integrity of CoRIM files.
Supported threshold signature policies (e.g., 2-out-of-3 signatures) to ensure that CoRIMs are only considered valid when the required number of signatures are present.

2. **Improved Integrity Verification:**

Updated Veraison services to support the validation of multi-signature CoRIMs, ensuring the authenticity of the manifest is verified by multiple independent signers.
Utilized the go-cose library to handle cryptographic signatures, providing a secure and standardized method of signing and verifying CoRIMs.

3. **Secure Key Management:**

Implemented secure key storage through HSM or Cloud KMS, ensuring signing keys are stored securely and can only be accessed by authorized entities.
Defined key usage policies to control who can sign CoRIMs, with threshold signature rules ensuring that the required number of signatures is validated before accepting a CoRIM.

4. **Increased Compliance and Auditability:**

Incorporated comprehensive logging for key usage, signing actions, and key rotation, ensuring compliance with security protocols.
Audited CoRIM generation and signing activities to ensure all actions are recorded for traceability.

5. **Streamlined CoRIM Generation and Signing Process:**

Updated the cocli tool to generate multi-signature CoRIMs, allowing for seamless creation of manifests signed by multiple authorities.
Automated the signing process with multiple keys, reducing manual intervention and streamlining the workflow for CoRIM generation.

6. **Scalability for Future Security Enhancements:**

Designed the system to easily accommodate more signers and support additional cryptographic algorithms, ensuring scalability as security requirements evolve.

# Resources
[CBOR Object Signing and Encryption (COSE)](https://datatracker.ietf.org/doc/html/rfc8152)

[Concise Reference Integrity Manifest (CoRIM)](https://github.com/veraison/corim)

[CBOR (Concise Binary Object Representation](https://datatracker.ietf.org/doc/html/rfc8152)

[Veraison Trusted Services Architecture](https://github.com/veraison/services)

[I-D.ietf-rats-endorsements](https://datatracker.ietf.org/doc/draft-ietf-rats-endorsements/)

[CoRIM presentation](https://people.linaro.org/~thomas.fossati/preso/corim@coco-06272024.pdf)

[cocli](https://github.com/veraison/cocli)

[veraison/services](https://github.com/veraison/services)