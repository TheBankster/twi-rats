---
title: Trustworthy Enrollment of Secure Credentials
abbrev: ESC
docname: draft-bdnr-rats-trustworthy-credentials-latest
category: info
ipr: trust200902
area: Security
wg: RATS Working Group
stand_alone: true
submissiontype: IETF
number:
date:

v: 3


keyword:
 - trustworthy workload identity
 - remote attestation
 - stable workload credentials

venue:
 group: RATS
 type: Working Group
 mail: rats@ietf.org

github: "confidential-computing/twi-rats"
latest: "https://confidential-computing.github.io/twi-rats/draft-bdnr-rats-trustworthy-credentials.html"

pi:

  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  text-list-symbols: -o*+
  docmapping: yes


author:
- ins: M. Novak
  name: Mark Novak
  org:  J.P. Morgan Chase
  email: mark.f.novak@jpmchase.com

- ins: M. Richardson
  name: Michael Richardson
  org: Sandelman Software Works
  email: mcr+ietf@sandelman.ca
  street: ""
  code: ""
  city: ""
  region: ""
  country: Canada

- ins:
  name: Henk Birkholz
  org:  Franhaufer Inst.
  email: Henk.Birkholz@ietf.contact


normative:

informative:
  RFC9334: RATS
  I-D.draft-mihalcea-seat-use-cases: SEATUSE
  I-D.draft-ietf-wimse-arch: WIMSE
  I-D.draft-ietf-wimse-identifier: WIMSEID
  TWISIGDef:
    -: TWISIGDef
    target: https://github.com/confidential-computing/twi/blob/main/TWI_Definitions.md
    title: Trustworthy Workload Identity (TWI) Special Interest Group — Definitions
    author:
      org: Confidential Computing Consortium Trustworthy Workload Identity SIG


--- abstract

To be written last

There is a large class of "RATS-Unaware" Relying Parties (RUPs) that Attesters nevertheless need to interoperate with.
Existing deployed services, which precede the introduction of Remote Attestation,
are often difficult to change/update in significant ways due to regulatory and cryptographic review policies.
Yet there are significant advantages if clients can be incrementally updated in the trustworthiness of the platform.

This document details a protocol by which the trusthworthiness of an Attesters is reviewed as part of the process of it being provided with some form of an Identity Document (a key, or a credential) to authenticate to RUPs.

This specification illustrates how the RATS Architecture can be applied to interoperate with RUPs by providing Attesters with such Identity Documents.

--- middle

# Introduction

Success of a technology is ultimately measured by its adoption. The RATS Architecture requires that RATS Relying Parties understand Attestation Results expressed using standards such as EAT and AR4SI, execute Appraisal Policy for Attestation Results, and have trust in Verifiers. Additionally, there is an unstated assumption present in the RATS Architecture that a change in Evidence may lead to a change in either the Attestation Results or Appraisal Policy for Attestation Results. This requirement may pose a significant adoption blocker.

One key requirement for successful deployment of Remote Attestation-capable workloads is minimal blast radius. When a workload is moved from a legacy to a remotely attestable (e.g. Trusted Execution) environment, including Intel SGX, AMD SEV-SNP,  ARM TrustZone, that workload can use Remote Attestation to obtain a stable and trustworthy Identity Document while its clients and servers do not notice anything different.

For that, a mechanism is required by means of which the RATS Relying Party, acting as a Credential Broker, a Key Broker, or a Credential Authority, provides the intermediation between Attestation Results, expressed using formats such as EAT and AR4SI, and the RATS-Unaware Relying Parties whose authentication and authorization policies may precede the introduction of Remotely Attestable Workloads and remain static for long periods of time.

For the RATS-Unaware Relying Parties, these adoption barriers are eliminated, as these RUPs are capable of authenticating their clients utilizing Identity Documents such as shared symmetric keys, or credentials including x.509 certificates, JWTs or WIMSE WITs. In this world, the Attester uses Remote Attestation to obtain from the RATS Relying Party a key, token or credential that is compatible with the RUP.

This document details an architecture by which legacy Identity Document Identity Document issuance mechanisms are replaced with identical Identity Documents issued, but with the additional prerequisite of successful Remote Attestation of the workloads in question.

# Conventions and Definitions
{: #definitions }
{::boilerplate bcp14-tagged}

This document uses terms and concepts defined by the WIMSE and RATS architectures, as well as the terms defined by the Trustworthy Workload Identity Special Interest Group at the Confidential Computing Consortium.
For a complete glossary, see {{Section 4 of -RATS}} , {{-WIMSE}} & {{-TWISIGDef}}.

The definitions of terms like Trustworthy Workload Identity and Workload Credential match those specified by the TWI SIG Definitions {{-TWISIGDef}}.

Workload:

: {{-WIMSE}} defines 'Workload' as "an instance of software executing for a specific purpose". Here we restrict that definition to the portions of the deployed software and its configuration that are subject to Remote Attestation.

Workload Credential:

: an ephemeral identity document containing an identity and a number of additional claims, that can be short-lived or long-lived, and that is used to access a service

Collaborating Party:
: a service that the workload needs to communicate with in order to accomplish its task.  For example, an image processor would need access to read images (photos) from the end-user's photo store (e.g., flikr, Google Photos, iPhoto), and then a place to store the results.  In OAUTH2, this would be the Resource.

Credential Broker:

: an entity trusted to obtain and/or distribute Workload Credentials

Verifier:

: an entity performing the role of Attestation Verification, as documented in {{Section 4 of -RATS}}

# Overview of Mechanism

A newly created workload connects to the Credential Broker to obtain a set of credentials to be used to perform it's functions.

Within this connection, Evidence is transferred to the Credential Broker to demonstrate the workloads' trusthworthiness.
The Credential Broker is acting as a RATS Relying Party, the workload is the Attester.
The Credential Broker contacts (using the background check model), a Verifier that it trusts in order to evaluate the Evidence, obtaining an Attestation Result.

Figure {{credential-arch}} extends the {{-RATS}} architecture to show how the workload and credential broker take on the roles of Attester and Relying Party.

If the Attestation Result is acceptable, then the Credential Broker provides the set of credentials that the workload needs to accomplish it's task.

~~~ aasvg
{::include credential_architecture.txt}
~~~
{: #credential-arch artwork-align="center" title="Credential Enrollment Architecture"}

## Types of Credentials

There are three kinds of credentials that can be involved.
Some workloads might use a few of each.

1. The Credential Broker is also an Identity Provider (IdP), and acts as an Registration Authority (possibly including the Certification Authority).  It issues new credentials in the form of PKIX certificates to each trustworthy workload.

2. The Credential Broker is a respository for a credential issued by another Identity Provider (IdP).  The Credential Broker has both the private key and the certificate, and it discloses these to trustworthy workloads by encrypting these to an identity that identifies the workload.

3. The Credential Broker is a resposity for a bearer token issued by a Resource Owner, or a Workload Identity Tokens (WITs) defined in Section 3.1 of {{-WIMSEID}}.
The Credential Broker discloses this to trustworthy workloads by encrypting these to an identity that identifies the workload.

The use of a shared private key is unorthodox.
This architecture is justified by the need to rapidly scale the number of workers and to recover from hardware or network failures.
The alternatives is that external Identity Providers would need to be willing to respond to spikes of hundreds of credential requests within a small period of time.
This would look like a denial of service attack, and it may also require additional human authorization for each.

The patterns of communication shown in figure {{credential-arch}} are designed specifically such that as few modifications are required to the workload, and no changes to the Collaborating Party are required.

## Deployment of Credentials

Workloads are expected to include a (virtual) Trusted Platform Module (TPM) (or equivalent) by which they will collect and sign Evidence to be used in the Remote Attestation process.

The credential that will be shared by the Credential Broker will be encrypted to a key involved in the Remote Attestation process.
The most natural mechanism is to encrypt to a key that is available only to the TPM.
The credential are then decrypted by the TPM, and the keypair can then be made available to the workload, while never permitting the workload to ever see the key.

In this way, a workload that be designed to do mutual TLS using a client-certificate, and for which the location of the private key can be configured to be in a TPM, can be adapted to the mechanism described in this document without any significant change to the workload itself.

# Details of protocol

TBD.


# Security Considerations

All communications between entities (Workload to Credential Authority, Workload to Verifier etc) MUST be secured using mutually authenticated, confidential, and integrity-protected channels (e.g., TLS).

In addition to the considerations herein, Verifier, which is a central point of anchor for Trustworthy Workload Identifer MUST follow the security guidance detailed in the "Security and Privacy considerations" as detailed in the RATS Architecture {{Section 11 and Section 12 of -RATS}}.

The credential key MUST always be stored securely at all time, for example in a secure element of the underlying platform running the Workload.

There is a risk that a live Workload Migration may render some of the claims about the Workload invalid (e.g., live-migrating a Workload between Germany and France may incorrectly preserve the "Country=Germany" claim, but correctly preserve the "Region=Europe" claim).

# Privacy Considerations

Remote Attestation of a Workload requires exchange of attestation related messages, for example, Evidence and Attestation Results. This can potentially leak sensitive information about the Workload.

Confidentiality: Encryption could be used to prevent unauthorised parties from accessing sensitive information from Evidence or Attestation Results.
This is crucial in multi-tenant environments.
The Credential Key to be released to a Workload MUST always be encrypted to avoid potential leakage to unauthorised actors.

# IANA Considerations

This document has no IANA actions (yet).


--- back

# Acknowledgments
{:numbered="false"}

