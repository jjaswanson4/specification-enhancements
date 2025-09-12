# Specification Update Proposal

## Owner

Armand Craig
Nilanjan Samajder

## Summary

This SUP is focused on finalizing the technical details pertaining to our previously approved REST API, for usage between the Device and the Workload Fleet Manager. 
    - see the following [Link](https://github.com/margo/specification/issues/21) for a history on when the REST API was approved for the following functionalities
        - Onboarding of the API Interface 
        - Providing WFM with Device Capabilities information
        - Providing WFM with workload deployment status updates

Technical Details the SUP aims to finalize in the specification:

- Secure certificates utilized / format
- API Authentication Mechanism
- API Security / Encryption strategy
- How unique identifiers are produced and who is responsible for providing them/maintaining them. 
- Required Ports to enable the edge to cloud communication
- TLS Protocol version
- API definition / documentation strategy

What this SUP does not cover:
- How the API is onboarded in an automated fashion
- How the desired state is retrieved via the edge device
- Multi-vendor trusted CA strategy for Margo

## Reason for proposal

The main reason for this proposal is to complete and gain community consensus on the technical details pertaining to the decided API mechanism for Devices to communicate with the WFM. 

## Requirements alignment acknowledgement

This SUP is aligned with the following Technical Feature. 
- https://github.com/margo/specification/issues/101


## Technical proposal

> The SUP's technical details. There must be enough technical details that someone can take the information in this section and implement it on their own.

### General REST API information 
- This proposal recommends a server-side TLS-enabled REST API operating over HTTP1.1.
    - The motivation to utilize HTTP1.1 is to ensure maximum support for existing infrastructure within our install base. 
    - Server-side tls is utilized instead of mTLS due to possible issues with TLS terminating HTTPS load-balancer or a HTTPS proxy doing lawful inspection. See the API Security / Encryption section for more details.
- This REST API should utilize a known root CA the client can download, which enables the TLS handshake and other onboarding credentials to be details out in a separate SUP.  
### Secure certificates utilized / format
- This proposal recommends the usage of x.509 certificates to represent both parties within the REST API construction.
    - These certificates are utilized to prove each participants identity, establish secure TLS session, and securely transport information in secure envelopes. 
### API Authentication Mechanism
- Initial Trust via TLS
    - The device establishes a secure HTTPS connection using server-side TLS.
    - It validates the server’s identity using the public root CA certificate.
> Note: Further onboarding details will be provided in a separate SUP submission.
### API Security / Encryption strategy
- To ensure completeness in the description, I have copied over a section from the specification shown below
    - Due to the limitations of utilizing mTLS with common OT infrastructure components, such as TLS terminating HTTPS load-balancer or a HTTPS proxy doing lawful inspection, Margo has adopted a certificate-based payload signing approach to protect payloads from being tampered with. By utilizing the certificates to create payload envelopes, the device's management client can ensure secure transport between the device's management client and the Workload Fleet Management's web service.
    - Message Envelope Details
        - Once the edge device has a message prepared for the Workload Fleet Management's web service, it completes the following to secure the message.
        - The device's management client calculates a digest and signature of the payload
        - The device's management client adds an envelope around it that has:
            - actual payload
            - SHA of the payload, signed by the device certificate
            - Identifier for the certificate that corresponds to the private key used to sign it. 
                - This identifier MUST be the UUID provided by the WFM server. 
        - The envelope is sent as the payload to the Workload Fleet Management's web service. 
        - The Workload Fleet Management's web service treats the request's payload as envelope structure, and receives the certificate identifier.
        - The Workload Fleet Management's web service computes digest from the payload, and verifies the signature using the device certification.
        - The payload is then processed by the Workload Fleet Management's web service.
    - Signing details
        1. Generate a SHA-256 hash value for the request's body
        2. Create a digital signature by using the message source certificates's private key to encrypt the the hash value
        3. Base-64 encode the certificate's public key and the digital signature in the format of `<public key>;<digital signature>`
        3. Include the base-64 encoded string in the request's `X-Payload-Signature` header
    - Verifying Signed Payloads
        1. Retrieve the public key from the `X-Payload-Signature` header
        2. Decrypt the digital signature using the public key to get the original hash value
        3. Generate a SHA-256 hash value for the requests's body
        4. Ensure the generated hash value matches the hash value from the message
### Unique Identifiers
>Note: this section is still being formulated.
- This proposal recommends the WFM create a client id to uniquely identify each client within the architecture. 
    - This client ID MUST be in the format of UUIDv4
### Management Interface Ports
Required Ports to enable the edge to cloud communication
    - The goal of this API is to minimize the ports required for a customer to enable cloud to edge communication. 
        - This Rest API must ONLY utilize port 443 for it's traffic. 

### API definition / documentation strategy
- Initially this SUP proposes the usage of Open API REST definitions. 
### Working Prototype
The development team is underway creating the Code First Sandbox. This strategy is being implemented by the dev team, and will be available for review in the coming weeks. 

## Alternatives considered (optional)

> List any alternative solutions considered while working on the SUP and the reason for not choosing them. If the SUP owner knows that there is a risk of a competing SUP, this section can be used to make their case ahead of any potential votes on why their solution is better.
> 
> Complete as part of Phase 3: SUP Technical Development

## Rejection reason

> If a SUP is rejected, indicate the reason why it was rejected.
> 
> Complete if SUP is rejected at Phase 2: Proposal Creation or Phase 4: Final Decision 
