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
 github: "mcr/twi-rats"

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
  RFC7030: EST

informative:
  RFC9334: RATS
  I-D.mihalcea-seat-use-cases: SEATUSE
  I-D.ietf-wimse-arch: WIMSE
  I-D.ietf-wimse-identifier: WIMSEID
  TWISIGDef:
    -: TWISIGDef
    target: https://github.com/confidential-computing/twi/blob/main/TWI_Definitions.md
    title: Trustworthy Workload Identity (TWI) Special Interest Group — Definitions
    author:
      org: Confidential Computing Consortium Trustworthy Workload Identity SIG


--- abstract

There is a large class of "RATS-Unaware" Relying Parties (RUPs) that Attesters nevertheless need to interoperate with.
Existing deployed services, which precede the introduction of Remote Attestation,
are often difficult to change/update in significant ways due to, among other reasons, organizational friction, technological inertia, and regulatory policies.
Yet there are significant advantages if workloads can be incrementally updated in the trustworthiness of the platform, without disrupting their clients and servers.

This document details a protocol by which Remote Attestattion of Attesters is incorporated into the process of them being provided with Identity Documents (keys or credentials) to authenticate to RUPs.

This specification illustrates how the RATS Architecture can be applied to interoperate with RUPs by providing Attesters with such Identity Documents.

--- middle

# Introduction

Success of a technology is ultimately measured by its adoption.
The RATS Architecture requires that RATS Relying Parties understand Attestation Results expressed using standards such as EAT and AR4SI, execute Appraisal Policy for Attestation Results, and have trust in Verifiers.
Additionally, there is an unstated assumption present in the RATS Architecture that a change in Evidence may lead to a change in either the Attestation Results or Appraisal Policy for Attestation Results. This requirement may pose a significant adoption blocker.

One key requirement for successful deployment of Remote Attestation-capable workloads is minimal blast radius.
When a workload is moved from a legacy to a remotely attestable (e.g. Trusted Execution) environment, including Intel SGX, AMD SEV-SNP,  ARM TrustZone, that workload can use Remote Attestation to obtain a stable and trustworthy Identity Document while its clients and servers do not notice anything different.

For that, a mechanism is required by means of which a Credential Broker, a Key Broker, or a Credential Authority takes on the role of RATS Relying Party.
This provides an intermediation between Attestation Results, expressed using formats such as EAT and AR4SI, and the RATS-Unaware Relying Parties whose authentication and authorization policies may precede the introduction of Remotely Attestable Workloads and remain static for long periods of time.

For the RATS-Unaware Relying Parties, these adoption barriers are eliminated, as these RUPs are capable of authenticating their clients utilizing appropriate Identity Documents.
This includes shared symmetric keys, bearer tokens, credentials including PKIX certificates {{!RFC5280}}, JWTs {{!RFC7515}}, or WIMSE WITs {{!I-D.ietf-wimse-workload-creds}}.
In this world, the Attester uses Remote Attestation to obtain from the RATS Relying Party a key, token or credential that is compatible with the RUP.

This document details an architecture by which legacy Identity Document issuance mechanisms are replaced with identical Identity Documents issued, but with the additional prerequisite of successful Remote Attestation of the workloads in question.

## Reasons for RATS Unaware Relying Party Immutability

The most important and most common scenario addressed here is that of a workload that employs Remote Attestation but whose Relying Party has no capacity to process Attestation Results or execute Appraisal Policy for Attestation Results.
This RATS Unaware Relying Party is typically unable to make the corresponding changes for a number of reasons:

* It may be a compiled object or container provided by a third party.
* Or it may be implemented in a language not easily changed or upgraded with new capabilities.
* Or, as an extreme example, it could be an ancient COBOL program compiled into a WASM object, perhaps connected to the network via virtual paper-tape and virtual printer interfaces.
* Further, such a system may require extensive and significant review by an authority before changes to the core algorithm can be made.
* Or, finally, the reluctance to change may come from organizational friction within an enterprise where the remotely attesting workload is organizationally separate from its Relying Party and different priorities of different parts of organization prevent them moving in lockstep.

In all of these cases, it is assumed that the remotely attesting workload can make the necessary changes to perform remote attestation, and that interoperability with the RUP will be preserved so long as the key, token, or credential obtained by the workload following Remote Attestation matches that expected by the RUP.

# Conventions and Definitions
{: #definitions }
{::boilerplate bcp14-tagged}

This document uses terms and concepts defined by the WIMSE and RATS architectures, as well as the terms defined by the Trustworthy Workload Identity Special Interest Group at the Confidential Computing Consortium.
For a complete glossary, see {{Section 4 of -RATS}}, {{-WIMSE}} & {{-TWISIGDef}}.

The definitions of terms like Trustworthy Workload Identity and Workload Credential match those specified by the TWI SIG Definitions {{-TWISIGDef}}.

Broker:
: an entity that deals out pre-existing keys or credentials. Constrast to a Credential Authority which mints new credentials.

Identity Document:
: a catch-all term for any type of key, token, or credential that is used by the attesting workload to authenticate to the RATS Unaware Relying party.

Identity Document Service:
: a catch-all term for a service that acts as a Key/Token Broker, Credential Broker, or Credential Authority, that the attesting workload uses to obtain its identity documents following successful remote attestation.

RUP:
: The RATS Unaware Relying Party (RUP).  A target service that interacts with many clients based upon credentials provided. This is sometimes called the Collaborating Party.

Workload:
: {{-WIMSE}} defines 'Workload' as "an instance of software executing for a specific purpose". Here we restrict that definition to the portions of the deployed software and its configuration that are subject to Remote Attestation.

Workload Duration:
: the lifespan of the workload.   While some workloads can be very long lived, but many workloads are created for a brief period of time, often added on demand to support rising demand, and persisting for only minutes to a small fraction of a day.

Workload Owner:
: the entity that manages a workload, arranging to provision it with appropriate Workload Credentials before Workload is launched

Workload Credential:
: an ephemeral identity document containing an identity and a number of additional claims, that can be short-lived or long-lived, and that is used to access a service

Proof of possession credential:
: this is a credential, such as an x.509 certificate or a WIMSE WIT, that requires proof of possession (typically an asymmetric signing key) to us. It is considered public information -- the secrecy is in the associated signing key.

Collaborating Party:
: see RUP.

Verifier:
: an entity performing the role of Attestation Verification, as documented in {{Section 4 of -RATS}}

# Overview of Mechanism

A newly created workload connects to the Identity Document Service (IDS) to obtain a set of Identity Documents to be used to perform its functions.

The proposed mechanism works equally well with Background Check and Passport models of RATS.
The IDS is acting as a RATS Relying Party, the workload is the Attester.

Figure {{credential-arch}} extends the {{-RATS}} architecture to show how the workload and IDS take on the roles of Attester and Relying Party.
Note in particular that no changes are made to the RATS Architecture; the only change is that the RATS Relying Party is now acting as the IDS,
and not the ultimate destination for workload's authentication.

If the Attestation Result is acceptable, then the IDS returns the Identity Document that the workload needs to accomplish its task.

~~~ aasvg
{::include credential_architecture.txt}
~~~
{: #credential-arch artwork-align="center" title="Credential Enrollment Architecture"}

## Types of Credentials

There are three kinds of credentials that can be involved.
Some workloads might use a few of each, possibly with each one being used with a different RATS Unaware Party.

1. The IDS is also an Identity Provider (IdP), and acts as an Registration Authority (possibly including the Certification Authority). It issues new credentials in the form of PKIX certificates or WIMSE WITs to each trustworthy workload, based on attested workload-held private keys.

2. The IDS is a respository for Identity Documents issued by another Identity Provider (IdP).
The IDS has both the private key (encrypted) and the proof-of-possession credential, and it discloses these to trustworthy workloads by returning them in a manner that ensures that only authorized workloads can decrypt the private key, typically by encrypting them to an attested, attester-held asymmetric encryption key.

The use of a shared assymetric private keys as identity documents is unorthodox.
This architecture is justified by the need to rapidly scale the number of workers and to recover from hardware or network failures.
The alternatives is that external Identity Providers would need to be willing to respond to spikes of hundreds of credential requests within a small period of time.
This would look like a denial of service attack, and it may also require additional human authorization for each.

The patterns of communication shown in figure {{credential-arch}} are designed specifically such that as few modifications are required to the workload, and no changes to the Collaborating Party are required.

## Deployment of Credentials

Workloads are expected to include a (virtual) Trusted Platform Module (TPM), a Trusted Execution Environment, or equivalent, by which they will generate collect and sign Evidence to be used in the Remote Attestation process.
Workloads are also assumed capable of generating and attesting (including in Evidence) asymmetric encryption or signing keys, the private portions of which never leave the workload, and thus guarantee that no other workload can make make use of these secrets.

# Details of protocol

As there are three major types of credentials that may be used, it is not unreasonable that they may get provisioned in different ways, using different protocols.

However, when time comes for the workload to request the Identity Documents, one of three possibilities arise:

1. (Key Broker mode) A cryptographic key encrypted to an attested, Attester-held asymmetric Key Encryption Key
2. (Credential Broker mode) A proof-of-possession credential and a corresponding Credential Signing Key encrypted to an attested, Attester-held asymmetric Key Encryption Key
3. (Identity Provider mode) A freshly minted proof-of-possession credential matching an attested, Attester-held asymmetric Credential Signing Key, or a newly minted short-lived bearer token encrypted to an attested, Attester-held asymmetric Token Encryption Key

| Variant | Workload Generates and Attests | IDS Returns |
|---------|--------------------------------|-------------|
| 1a: Key Broker for proof-of-possession Keys (PPC) | Asymmetric Key Encryption Key (KEK) | Credential Signing Key (CSK) matching PPC, encyprted with the KEK |
| 1b: Key Broker for shared keys | Asymmetric Key Encryption Key (KEK) | Secret Preshared Key (SPK), encrypted with the KEK |
| 2: Credential Broker for proof-of-possession credentials | Asymmetric Key Encryption Key (KEK) | Proof-of-Possession Credential (PPC) and Credential Signing Key (CSK) for PPC, Encrypted with the KEK |
| 3a: Credential Authority for proof-of-possession credentials | Asymmetric Credential Signing Key (CSK) | Newly minted proof-of-possession credential matching the CSK |
| 3b: Credential Authority for short-lived bearer tokens | Asymmetric Token Encryption Key (TEK) | Newly minted short-lived bearer token encrypted with the TEK |

These are explained in more detail below:

## Variant 1: Key Broker Mode

The Attester generates an asymmetric Key Encryption Key (KEK), and includes its public portion in Evidence during Remote Attestation. The Attester receives from the RATS Relying Party a Secret Key encrypted to the KEK. That Secret Key could be an asymmetric Credential Signing Key for a proof-of-possession credential, such as an x.509 certificate, that is pre-provisioned to the Attester (Variant 1a), or a symmetric key for preshared key scenarios (Variant 1b).

## Variant 2: Credential Broker Mode

The Attester generates an asymmetric Key Encryption Key (KEK), and includes its public portion in Evidence during Remote Attestation. The Attester receives from the RATS Relying Party a pre-provisioned Credential (a WIT or an x.509 certificate) together with its corresponding Credential Signing Key encrypted to the KEK.

## Variant 3: Credential Authority Mode

In Variant 3a, which is similar to the Attested CSR protocol, the Attester generates an asymmetric Credential Signing Key (CSK), and includes its public portion in Evidence during Remote Attestation. The RATS Relying Party, acting as a Credential Authority, mints a brand new proof-of-possession credential and returns it to the Attester.

In Variant 3b, the Attester generates an asymmetric Token Encryption Key (TEK), and includes its public portion in Evidence during Remote Attestation. The RATS Relying Party, acting as a Credential Authority, mints a brand new short-lived bearer token (e.g. a JWT), encrypts it with the TEK, and returns the encrypted bearer token to the Attester.

## Use of Enrollment over Secure Transport (EST)

EST ({{RFC7030}}) describes a mechanism to obtain a credential and/or a corresponding credential signing key using TLS over HTTP.
A few changes are needed to make EST suitable for handling workloads capable of Remote Attestation.
In all cases below the EST client hosts the workload, but is not assumed to be inside the workload's TCB.
This EST client might be the workload's hypervisor, a container orchestrator, or some other workload hosting environment.
Use of authentication by this EST client is OPTIONAL.
Even if used, client authentication MUST NOT be used to establish the identity of the workload for purposes of deciding which key, token or credential to return to it.
The EST server acts as a RATS Relying Party in Background Check mode.
The Evidence is submitted by the workload to the EST server by the EST client acting on its behalf.
There are three ways to submit the Evidence:

1. Via a new, Remote Attestation extension to EST

2. Using {{!I-D.ietf-lamps-csr-attestation}} extensions to the CSR itself

3. Within TLS itself, using for instance, {{?I-D.fossati-seat-expat}}, or whichever protocol the SEAT WG standardizes (in this case, the workload becomes the EST client)

This Evidence is passed by the EST server to a Verifier, and Attestation Results returned by the Verifier are used to establish the identity of the workload.

### Using EST to Mint New Proof-of-Possession Credentials

The EST /simpleenroll mechanism is used.

### Using EST to Obtain Existing Keys, Tokens, or Proof-of-Possession Credentials

The EST /serverkeygen mechanism is used.
The server does not generate a fresh key, but rather retrieves the keypair (private key and certificate) from the store.
This is encrypted back to the client, however, mechanisms described in RFC7030 need to be modified as any secrets (keys, bearer tokens) must be returned to an attested, attester (workload)-held asymmetric encryption key, not to the EST client directly.
(TBD: This needs more detail, particularly for the mTLS used in the EST)

### Credential Broker as short-term Bearer Token issuer

EST is not appropriate for this use case.
Another protocol or an extension to EST will be required.

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

