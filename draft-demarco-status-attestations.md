---
title: "OAuth Status Attestations"
abbrev: "OAuth Status Attestations"
category: info

docname: draft-demarco-status-attestations-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
keyword:
 - digital credentials
 - status list
 - revocation
venue:
  github: "peppelinux/draft-demarco-status-attestations"
  latest: "https://peppelinux.github.io/draft-demarco-status-attestations/draft-demarco-status-attestations.html"

author:
 -
    fullname: Giuseppe De Marco
    organization: Dipartimento per la trasformazione digitale
    email: gi.demarco@innovazione.gov.it
 -
    fullname: Francesco Marino
    organization: Istituto Poligrafico e Zecca dello Stato
    email: fa.marino@ipzs.it

normative:
  RFC7515: RFC7515
  RFC7516: RFC7516
  RFC7517: RFC7517
  RFC7519: RFC7519
  RFC7638: RFC7638
  RFC7800: RFC7800
  RFC8392: RFC8392
  RFC9126: RFC9126

informative:


--- abstract

Status Attestations are signed objects that demonstrate the validity status of a
digital credential.
These attestations are ephemeral and periodically provided
to digital credential holders, that can present these to verifiers along
with the corresponding digital credentials.
The approach outlined in this document
makes the verifiers able to check the non-revocation of a digital credential
without requiring to query any third-party systems.

--- middle

# Introduction

Status Attestations play a crucial role in maintaining the integrity and
trustworthiness of digital credentials,
since these serve as proof that a particular digital credential,
whether in JSON Web Tokens (JWT) or CBOR Web Tokens (CWT) format,
has not been revoked and is still valid.

A digital credential may be presented to a verifier long after it has been issued.
During this interval, the credential could potentially be invalidated for various reasons.
To ensure the credential's validity, the issuer provides a short-lived
Status Attestation to the credential's holder.
This attestation is bound to the credential and can be presented to a verifier,
along with the credential itself, as proof of the credential's non-revocation status.

Status Attestations are designed to preserve privacy and are essential for
enabling offline use cases, where both the wallet and the verifier are
not connected to internet during the presentation phase, thereby
ensuring the security of the digital credential system.
Status Attestations provide a balance between scalability, security, and
privacy by minimizing the status information.


~~~ ascii-art
+-----------------+                             +-------------------+
|                 | Requests Status Attestation |                   |
|                 |---------------------------->|                   |
| Wallet Instance |                             | Credential Issuer |
|                 | Status Attestation          |                   |
|                 |<----------------------------|                   |
+-----------------+                             +-------------------+


+-- ----------------+                             +----------+
|                   | Presents credential and     |          |
|  Wallet Instance  | Status Attestation          | Verifier |
|                   |---------------------------->|          |
+-------------------+                             +----------+
~~~


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Terminology

This specification uses the terms "End-User", "Entity" as defined by
OpenID Connect Core [@OpenID.Core], the term "JSON Web Token (JWT)"
defined by JSON Web Token (JWT) {{RFC7519}}.

Issuer:
: Entity that is responsible for the issuance of the Digital Credentials.
The Issuer is responsible about the lifecycle of their issued Credentials
and their validity status.

Verifier:
: Entity that relies on the validity of the Digital Credentials presented to it.
This Entity, also known as a Relying Party, needs to verify the authenticity and
validity of the Credentials, including their revocation status, before accepting them.

Wallet Instance:
: The digital Wallet in control of a User, also known as Wallet or Holder.
It securly stores the User's digital Credentials. It can present Credentials to Verifiers
and request Status Attestations from Issuers under the control of the User.

# Rationale

OAuth Status Lists [@!I-D.looker-oauth-jwt-cwt-status-list] are suitable
for specific scenarios, especially when the Verifier needs to verify the
status of a Credential at a later time after the User has presented the
Digital Credential.

However, there are cases where the Verifier only needs
to check the revocation status of a Digital Credential at the time of
presentation, or situations where the Verifier should not be allowed to
check the status of a credential over time due to some privacy constraints,
in compliance to national privacy regulations.

In scenarios where the Verifier, Credential Issuer, and OAuth Status List
Provider are all part of the same domain or operate within a context where
a high level of trust exists between them and the End-User, the OAuth
Status List is the optimal solution; while there might be other cases
where the OAuth Status List facilitates the exposure to the following
privacy risks:

- An OAuth Status List provider might known the association between a specific
list and a Credential Issuer, especially if the latter issues a
single type of Credential. This could inadvertently reveal to the
Status List provider which list corresponds to which Credential.
- A Verifier retrieves an OAuth Status List by establishing a TCP/IP connection
with an OAuth Status List provider. This allows the OAuth Status List provider to
obtain the IP address of the Verifier and potentially link it to a specific
Credential type and Issuer associated with that OAuth Status List. A malicious
OAuth Status List provider could use internet diagnostic tools, such as Whois
or GeoIP lookup, to gather additional information about the Verifier.
This could inadvertently disclose to the OAuth Status List provider which
Credential the requestor is using and from which Credential Issuer,
information that should remain confidential.

Status Attestations differ significantly from OAuth Status Lists in several ways:

1. **Privacy**: Status Attestations are designed to be privacy-preserving.
They do not require the Verifier to gather any additional information
from third-party systems, thus preventing potential privacy leaks.

2. **Static Verification**: Status Attestations are designed to be
statically provided to Verifiers by Wallet Instance.
Once an Attestation is issued, it can be verified without any further
communication with the Issuer or any other party.

3. **Digital Credentials Formats**: Status Attestations are agnostic from the
Digital Credential format to which they are bound.

4. **Trust Model**: Status Attestations operate under a model where
the Verifier trusts the Issuer to provide accurate status information,
while the OAuth Status Lists operate under a model where the Verifier
trusts the Status List Provider to maintain an accurate and up-to-date
list of statuses.

5. **Offline flow**: OAuth Status List can be accessed by a Verifier when
an internet connection is present. At the same time,
OAuth Status List defines
how to provide a static Status List Token, to be included within a
Digital Credential. This requires the Wallet Instance to acquire a
new Digital Credential for a specific presentation. Even if similar to
the OAuth Status List Token, the Status Attestations enable the User to
persistently use their preexistent Digital Credentials, as long as
the linked Status Attestation is available and presented to the
Verifier, and not expired.


# Requirements

In this section are listed the general requirements that must be satisfied
when the Status Attestation are implemented. The Status Attestation:

- MUST be presented in conjunction with the Digital Credential.
The Status Attestation MUST be timestamped with its issuance datetime,
always referring to a previous period to the presentation time.
- MUST contain the expiration datetime after which the Digital Credential
MUST NOT be considered valid anymore. The expiration datetime MUST be
superior of the issuance datetime.
- enables offline use cases as it MUST be validated using
cryptographic signature and the cryptographic public key of the Issuer.

Please note: in this specification the examples and the normative properties
of attestations are reported in accordance with the JWT standard, while
for the purposes of this specification any credential or attestation
format format may be used, as long as all attributes and requirements
defined in this specification are satisfied, even using equivalent names
or values.


# Status Attestation Request

The Issuer provides the Wallet Instance with a Status Attestation,
which is bound to a Credential.
This allows the Wallet Instance to present it, along with the Credential itself,
to a Verifier as proof of the Credential's non-revocation status.

The following diagram shows the Wallet Instance requesting a
Status Attestation to an Issuer,
related to a specific Credential issued by the same Issuer.


~~~ ascii-art
+-------------------+                         +--------------------+
|                   |                         |                    |
|  Wallet Instance  |                         | Credential Issuer  |
|                   |                         |                    |
+--------+----------+                         +----------+---------+
         |                                               |
         | HTTP POST /status                             |
         |  credential_pop = $CredentialPoPJWT           |
         +----------------------------------------------->
         |                                               |
         |  Response with Status Attestation JWT         |
         <-----------------------------------------------+
         |                                               |
+--------+----------+                         +----------+---------+
|                   |                         |                    |
|  Wallet Instance  |                         | Credential Issuer  |
|                   |                         |                    |
+-------------------+                         +--------------------+
~~~

The Wallet Instance sends the Status Attestation request to the Issuer.
The request MUST contain the Digital Credential, for which the Status Attestation
is requested, and enveloped in a signed object as proof of possession.
The proof of possession MUST be signed with the private key corresponding
to the public key attested by the Issuer and contained within the Credential.

~~~
POST /status HTTP/1.1
Host: issuer.example.org
Content-Type: application/x-www-form-urlencoded

credential_pop=$CredentialPoPJWT
~~~

To validate that the Wallet Instance is entitled to request its Status Attestation,
the following requirements must be satisfied:

- The Issuer verifies the signature of the `credential_pop` object using
the public key contained in the Credential;
- the Issuer MUST verify that it is the legitimate Issuer of the Credential.

The technical and details about the `credential_pop` object
are defined in the next section.


## Digital Credential Proof of Possession

The Wallet that holds a Digital Credential, when requests a Status Attestation,
MUST demonstrate the proof of possession of the Credential to the Credential Issuer.

Ther proof of possession is made by enclosing the Credential in an
object (JWT) signed with the key referenced in the Credential.

Below a non-normative example of a Credential proof of possession with
the JWT headers and payload represented without applying signature and
encoding, for better readability:

~~~
{
    "alg": "ES256",
    "typ": "status-attestation-request+jwt",
    "kid": $WIA-CNF-JWKID

}
.
{
    "iss": "0b434530-e151-4c40-98b7-74c75a5ef760",
    "aud": "https://issuer.example.org/status-attestation-endpoint",
    "iat": 1698744039,
    "exp": 1698834139,
    "jti": "6f204f7e-e453-4dfd-814e-9d155319408c",
    "credential_format": "vc+sd-jwt",
    "credential": $Issuer-Signed-JWT
}
~~~


When the JWT format is used, the JWT MUST contain the parameters defined in the following table.

| JOSE Header | Description | Reference |
| --- | --- | --- |
| **typ** | It MUST be set to `status-attestation-request+jwt` | {{RFC7516}} Section 4.1.1 |
| **alg** | A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry. It MUST NOT be set to `none` or any symmetric algorithm (MAC) identifier. | {{RFC7516}} Section 4.1.1 |
| **kid** | Unique identifier of the JWK used for the signature of the Status Attestation Request, it MUST match the one contained in the Credential `cnf.jwk`. | {{RFC7515}} |

| JOSE Payload | Description | Reference |
| --- | --- | --- |
| **iss** | Wallet identifier. | {{RFC9126}}, {{RFC7519}} |
| **aud** | It MUST be set to the identifier of the Credential Issuer. | {{RFC9126}}, {{RFC7519}} |
| **exp** | UNIX Timestamp with the expiration time of the JWT. | {{RFC9126}}, {{RFC7519}} |
| **iat** | UNIX Timestamp with the time of JWT issuance. | {{RFC9126}}, {{RFC7519}} |
| **jti** | Unique identifier for the JWT.  | {{RFC7519}} Section 4.1.7 |
| **credential_format** | The data format of the Credential. Eg: `vc+sd-jwt` for SD-JWT, `vc+mdoc` for ISO/IEC 18013-5 MDOC CBOR [@ISO.18013-5] | this specification |
| **credential** | It MUST contain the Credential according to the data format given in the `format` claim. | this specification |


# Status Attestation

When a Status Attestation is requested to an Issuer, the
Issuer checks the status of the Credential and creates a Status Attestation bound to it.

If the Credential is valid, the Issuer creates a new Status Attestation, which a non-normative example is given below.

~~~
{
    "alg": "ES256",
    "typ": "status-attestation+jwt",
    "kid": $ISSUER-JWKID
}
.
{
    "iss": "https://issuer.example.org",
    "iat": 1504699136,
    "exp": 1504700136,
    "credential_hash": $CREDENTIAL-HASH,
    "credential_hash_alg": "sha-256",
    "cnf": {
        "jwk": {...}
    }
}
~~~

The Status Attestation MUST contain the following claims when the JWT format is used.

| JOSE Header | Description | Reference |
| --- | --- | --- |
| **alg** | A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry. It MUST NOT be set to `none` or to a symmetric algorithm (MAC) identifier. | {{RFC7515}}, {{RFC7517}} |
| **typ** | It MUST be set to `status-attestation+jwt`. | {{RFC7515}}, {{RFC7517}} and this specification |
| **kid** | Unique identifier of the Issuer JWK. | {{RFC7515}} |

| JOSE Payload | Description | Reference |
| --- | --- | --- |
| **iss** | It MUST be set to the identifier of the Issuer. | {{RFC9126}}, {{RFC7519}} |
| **iat** | UNIX Timestamp with the time of the Status Attestation issuance. | {{RFC9126}}, {{RFC7519}} |
| **exp** | UNIX Timestamp with the expiry time of the Status Attestation. | {{RFC9126}}, {{RFC7519}} |
| **credential_hash** | Hash value of the Credential the Status Attestation is bound to. | this specification |
| **credential_hash_alg** | The Algorithm used of hashing the Credential to which the Status Attestation is bound. The value SHOULD be set to `S256`. | this specification |
| **cnf** | JSON object containing the cryptographic key binding. The `cnf.jwk` value MUST match with the one provided within the related Credential. | {{RFC7800}} Section 3.1 |


# Status Attestation Response

If the Status Attestation is requested for a non-existent, expired, revoked or invalid Credential, the
Credential Issuer MUST respond with an HTTP Response with the status code set to 404.

If the Credential is valid, the Issuer then returns the Status Attestation to the Wallet Instance,
as in the following non-normative example.

~~~
HTTP/1.1 201 OK
Content-Type: application/json

{
    "status_attestation": "eyJhbGciOiJFUzI1Ni ...",
}
~~~


# Issuers Supporting Status Attestations


## Issuer Metadata

The Issuers that uses the Status Attestations MUST include in their
OpenID4VCI [@!OpenID.VCI] metadata the claim `status_attestation_endpoint`,
which its value MUST be an HTTPs URL, indicating the endpoint where
the Wallet Instances can request Status Attestations.


## Issued Credentials

The Issuers that uses the Status Attestations SHOULD include in the
issued Credentials the object `status` with the
JSON member `status_attestation` set to a JSON Object containing the following
member:

- `credential_hash_alg`. REQUIRED. The Algorithm used of hashing the Credential to which the Status Attestation is bound. The value SHOULD be set to `S256`.


The non-normative example of an unsecured payload of
an SD-JWT VC is shown below.

> TODO: alignments with the OAuth2 Status List schema and IANA registration.

~~~
{
 "vct": "https://credentials.example.com/identity_credential",
 "given_name": "John",
 "family_name": "Doe",
 "email": "johndoe@example.com",
 "phone_number": "+1-202-555-0101",
 "address": {
   "street_address": "123 Main St",
   "locality": "Anytown",
   "region": "Anystate",
   "country": "US"
 },
 "birthdate": "1940-01-01",
 "is_over_18": true,
 "is_over_21": true,
 "is_over_65": true,
 "status": {
    "status_attestation": {
        "credential_hash_alg": "S256",
    }
 }
}
~~~

# Presenting Status Attestations

The Wallet Instance that provides the Status Attestations MUST include in the
`vp_token` JSON array, as defined in [@OpenID4VP], the Status Attestation along with the
related Digital Credential.

The Verifier that receive a Digital Credential supporting the Status Attestation,
SHOULD:

- Decode and validate the Digital Credential;
- check the presence of `status.status_attestation`, if present the Verifier SHOULD:
  - produce the hash of the Digital Credential using the hashing algoritm defined in `status.status_attestation`;
  - decode all the Status Attestations provided in the presentation, by mathing the JWS Header parameter `typ` set to `status-attestation+jwt` and looking for the `credential_hash` value that match with the hash produced at the previous point;
  - evaluate the validity of the Status Attestation.


# Security Considerations

TODO Security

--- back

# IANA Considerations

## JSON Web Token Claims Registration

This specification requests registration of the following Claims in the
IANA "JSON Web Token Claims" registry [@IANA.JWT] established by {{RFC7519}}.

*  Claim Name: `credential_format`
*  Claim Description: The Digital Credential format the Status Attestation is bound to.
*  Change Controller: IETF
*  Specification Document(s):  [[ (#digital-credential-proof-of-possession) of this specification ]]

<br/>

*  Claim Name: `credential`
*  Claim Description: The Digital Credential the Status Attestation is bound to.
*  Change Controller: IETF
*  Specification Document(s):  [[ (#digital-credential-proof-of-possession) of this specification ]]

<br/>

*  Claim Name: `credential_hash`
*  Claim Description: Hash value of the Digital Credential the Status Attestation is bound to.
*  Change Controller: IETF
*  Specification Document(s):  [[ (#status-attestation) of this specification ]]

<br/>

*  Claim Name: `credential_hash_alg`
*  Claim Description: The Algorithm used of hashing the Digital Credential to which the Status Attestation is bound.
*  Change Controller: IETF
*  Specification Document(s):  [[ (#status-attestation) of this specification ]]

## Media Type Registration

This section requests registration of the following media types [@RFC2046] in
the "Media Types" registry [@IANA.MediaTypes] in the manner described
in [@RFC6838].

To indicate that the content is an JWT-based Status List:

  * Type name: application
  * Subtype name: status-attestation-request+jwt
  * Required parameters: n/a
  * Optional parameters: n/a
  * Encoding considerations: binary; A JWT-based Status Attestation Request object is a JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  * Security considerations: See (#Security) of [[ this specification ]]
  * Interoperability considerations: n/a
  * Published specification: [[ this specification ]]
  * Applications that use this media type: Applications using [[ this specification ]] for updated status information of tokens
  * Fragment identifier considerations: n/a
  * Additional information:
    * File extension(s): n/a
    * Macintosh file type code(s): n/a
  * Person &amp; email address to contact for further information: Giuseppe De Marco, gi.demarco@innovazione.gov.it
  * Intended usage: COMMON
  * Restrictions on usage: none
  * Author: Giuseppe De Marco, gi.demarco@innovazione.gov.it
  * Change controller: IETF
  * Provisional registration? No

To indicate that the content is an CWT-based Status List:

  * Type name: application
  * Subtype name: status-attestation+jwt
  * Required parameters: n/a
  * Optional parameters: n/a
  * Encoding considerations: binary
  * Security considerations: See (#Security) of [[ this specification ]]
  * Interoperability considerations: n/a
  * Published specification: [[ this specification ]]
  * Applications that use this media type: Applications using [[ this specification ]] for status attestation of tokens and Digital Credentials
  * Fragment identifier considerations: n/a
  * Additional information:
    * File extension(s): n/a
    * Macintosh file type code(s): n/a
  * Person &amp; email address to contact for further information: Giuseppe De Marco, gi.demarco@innovazione.gov.it
  * Intended usage: COMMON
  * Restrictions on usage: none
  * Author: Giuseppe De Marco, gi.demarco@innovazione.gov.it
  * Change controller: IETF
  * Provisional registration? No


# Acknowledgments
{:numbered="false"}

TODO acknowledge.


# Document History

TODO changelog.
