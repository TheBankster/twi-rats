**Working draft – everything is subject to change. Please keep change tracking on.**

# Abstract

There is a large class of "RATS-Unaware" Relying Parties (RUPs) that Attesters nevertheless need to interoperate with. RUPs are existing deployed services, which precede the introduction of Remote Attestation, are assumed to be unchangeable, and therefore cannot parse or process Attestation Results, or execute Appraisal Policy for Attestation Results. Neither do they understand the concept of, or can be configured to have trust in, Verifiers.  
Attesters require some form of an Identity Document (a key, or a credential) to authenticate to RUPs. This specification illustrates how the RATS Architecture can be applied to interoperate with RUPs by providing Attesters with such Identity Documents.

# Introduction

Success of a technology is ultimately measured by its adoption. The RATS Architecture requires that RATS Relying Parties understand Attestation Results expressed using standards such as EAT and AR4SI, execute Appraisal Policy for Attestation Results, and have trust in Verifiers. Additionally, there is an unstated assumption present in the RATS Architecture that a change in Evidence may lead to a change in either the Attestation Results or Appraisal Policy for Attestation Results. This requirement may pose a significant adoption blocker.

One key requirement for successful deployment of Remote Attestation-capable workloads is minimal blast radius. When a workload is moved from a legacy to a remotely attestable (e.g. Trusted Execution) environment, including Intel SGX, AMD SEV-SNP,  ARM TrustZone, that workload can use Remote Attestation to obtain a stable and trustworthy Identity Document while its clients and servers do not notice anything different.

For that, a mechanism is required by means of which the RATS Relying Party, acting as a Credential Broker, a Key Broker, or a Credential Authority, provides the intermediation between Attestation Results, expressed using formats such as EAT and AR4SI, and the RATS-Unaware Relying Parties whose authentication and authorization policies may precede the introduction of Remotely Attestable Workloads and remain static for long periods of time.

For the RATS-Unaware Relying Parties, these adoption barriers are eliminated, as these RUPs are capable of authenticating their clients utilizing Identity Documents such as shared symmetric keys, or credentials including x.509 certificates, JWTs or WIMSE WITs. In this world, the Attester uses Remote Attestation to obtain from the RATS Relying Party a key, token or credential that is compatible with the RUP.

This document details an architecture by which legacy Identity Document Identity Document issuance mechanisms are replaced with identical Identity Documents issued, but with the additional prerequisite of successful Remote Attestation of the workloads in question.

# Conventions and Terminology

\<TODO\>  
Broker \- something that deals out pre-existing keys or credential, vs. Credential Authority which mints new credentials  
RUP?  
Workload Owner – tasked with key/credential bootstrapping before Workload is launched

Assumptions around Duration of workload need to be explained.  
Add “proof-of-possession credential”  
Assumptions about bootstrapping

# RATS Architecture Extension for RATS-Unaware Relying Parties

TODO: talk about “credential bootstrapping”, separate into RATS RP being a Credential Authority, or a Key Store, etc…  
In order for an Attester to interoperate with a RUP, the RATS Relying Party requires a mechanism for returning to the Attester either:

1) (Key Broker mode) A cryptographic key encrypted to an attested, Attester-held asymmetric Key Encryption Key  
2) (Credential Broker mode) A proof-of-possession credential and a corresponding Credential Signing Key encrypted to an attested, Attester-held asymmetric Key Encryption Key  
3) (Credential Authority mode) A freshly minted proof-of-possession credential matching an attested, Attester-held asymmetric Credential Signing Key, or a newly minted short-lived bearer token encrypted to an attested, Attester-held asymmetric Token Encryption Key

The RATS Relying Party can employ either the Passport or Background Check model. The top part of the diagram below is intentionally illustrated as matching the RFC9334 RATS Architecture, because no architectural changes are required to RATS itself – only that the RATS-Unaware Relying Party becomes the final Relying Party following the Remote Attestation exchange.

┌──────────┐   ┌───────────┐  ┌──────────┐  ┌─────────┐  
│ Endorser ├─┐ │ Reference │  │ Verifier │  │ Relying │  
└──────────┘ │ │   Value   │  │  Owner   │  │  Party  │  
             │ │ Provider  │  └─┬────────┘  │  Owner  │  
             │ └─┬─────────┘    │           └───────┬─┘  
             │   │              │ Appraisal         │    
             │   │ Reference    │ Policy for        │    
             │   │ Values       │ Evidence          │    
             │   │              │                   │    
           ┌─▼───▼──────────────▼─┐       Appraisal │    
      ┌────►       Verifier       ├───┐  Policy for │    
      │    └──────────────────────┘   │ Attestation │    
      │                   Attestation │     Results │    
      │ Evidence              Results │             │    
      │                             ┌─▼─────────────▼─┐  
┌─────┴────┐                        │  RATS Relying   │  
│ Attester ◄────────────────────────┤     Party       │  
└─────┬────┘       Token, Key,      │(Cred/Key Broker)│  
      │           or Credential     └─────────────────┘                     
      │                             ┌─────────────────┐  
      │         Authentication      │   RATS-Unaware  │  
      └─────────────────────────────►  Relying Party  │  
                                    └───────────────▲─┘  
                                       RATS-Unaware │    
                                     Authentication │    
                                             Policy │    
                                     ┌──────────────┴─┐  
                                     │  RATS-Unaware  │  
                                     │ Relying Party  │  
                                     │     Owner      │  
                                     └────────────────┘

# Details of Protocol

## Summary

| Variant | Attester Generates and Attests | RATS Relying Party Returns |
| :---- | :---- | :---- |
| **1a: Key Broker** for proof-of-possession keys (\*) | Asymmetric Key Encryption Key (KEK) | Credential Signing key CSK matching PPC, encrypted with the KEK |
| **1b: Key Broker** for shared keys | Asymmetric Key Encryption Key (KEK) | Secret Preshared Key SPK, encrypted with the KEK |
| **2: Credential Broker** for proof-of-possession credentials | Asymmetric Key Encryption Key (KEK) | Proof-of-Possession Credential (PPC) and Credential Signing Key CSK for PPC, Encrypted with the KEK |
| **3a: Credential Authority** for proof-of-possession credentials | Asymmetric Credential Signing Key (CSK) | Newly minted proof-of-possession credential matching the CSK |
| **3b: Credential Authority** for short-lived bearer tokens | Asymmetric Token Encryption Key (TEK) | Newly minted short-lived bearer token encrypted with the TEK |

(\*) In this case the Proof-of-Possession Credential (PPC) is pre-provisioned to the Attester.

The details of each variant above are explained below:  
TODO: To each variant below, add discussion of Background Check vs. Passport Model suitability.

## Variant 1: Key Broker Mode

The Attester generates an asymmetric Key Encryption Key (KEK), and includes its public portion in Evidence during Remote Attestation. The Attester receives from the RATS Relying Party a Secret Key encrypted to the KEK. That Secret Key could be an asymmetric Credential Signing Key for a proof-of-possession credential, such as an x.509 certificate, that is pre-provisioned to the Attester (Variant 1a), or a symmetric key for preshared key scenarios (Variant 1b).

## Variant 2: Credential Broker Mode

The Attester generates an asymmetric Key Encryption Key (KEK), and includes its public portion in Evidence during Remote Attestation. The Attester receives from the RATS Relying Party a pre-provisioned Credential (a WIT or an x.509 certificate) together with its corresponding Credential Signing Key encrypted to the KEK.

## Variant 3: Credential Authority Mode

In Variant 3a, which is similar to the Attested CSR protocol, the Attester generates an asymmetric Credential Signing Key (CSK), and includes its public portion in Evidence during Remote Attestation. The RATS Relying Party, acting as a Credential Authority, mints a brand new proof-of-possession credential and returns it to the Attester.

In Variant 3b, the Attester generates an asymmetric Token Encryption Key (TEK), and includes its public portion in Evidence during Remote Attestation. The RATS Relying Party, acting as a Credential Authority, mints a brand new short-lived bearer token (e.g. a JWT), encrypts it with the TEK, and returns the encrypted bearer token to the Attester.

## Bootstrapping

TODO  
Use Cases

- Includes who needs this, who is going to implement this?  
- Who are the RUPs and Attesters that are brownfield / legacy deployment.  
- There are significant blockers to CCC, regulated institutions will do little until regulators demand it.  
  - There are high-risk locations, and the host country is not trusted  
  - There are Regulators in the world, existing reverse proxy MUST run software from regulator, with access to incoming traffic, nginx would run in trusted environment, regulator software elsewhere.    
- Attester is an outside jurisdiction thing that wants to connect to the (Bank’s) reverse proxy.  
  - Money transfers across national borders. (Why is SWIFT not good enough? Is this SWIFT v2?)  
  - 

# Security Considerations

\<TODO: see how much of the TWI Replica threat model needs to be placed here\>  
TODO: Add a clarification that use of bearer tokens, even if obtained via remote attestation, does not stop them from being, well, bearer tokens :) Same for shared keys, only worse as there’s no expiration time (if used instead of distributing asymmetric signing keys in Variant 1\)

# IANA Considerations

\<TODO: Anything at all?\>