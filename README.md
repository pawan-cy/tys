#
# Trust Your Supplier – DID Specification Document



**Source** : Chainyard (An IT People Company) – 1 Copley Parkway, Morrisville, NC, 27560 USA

**Website** : [www.chainyard.com](http://www.chainyard.com), [www.trustyoursupplier.com](http://www.trustyoursupplier.com)

**Authors** :  Mohan Venkataraman, Pawan Pandey

**Editors** : Shyam Adivi, Sree Mudundi, Gary Storr

**Date** : 04/19/2019

**Status** : DRAFT V0.4

Contents

About TYS        4

Method Name        4

Method Specific Identifiers        5

Generating a unique _idstring_        5

Test Results        7

TYS DIDs        8

TYS DID Example        8

TYS DID Document (Organization)        8

Example 1        8

Example 2        9

TYS DID Document (Claim)        9

DID Transaction Process Flow        11

TYS Functions        11

Create and Register a DID        11

Read        12

Update        14

Deactivate        14

GetAllCredentialsForEntity        15

Status        16

References        16



# About TYS

&quot;Trust Your Supplier&quot; aka &quot;TYS&quot;  is actively in development at Chainyard, a leading Blockchain development firm located in RTP North Carolina. The network is undergoing beta testing and will go live in the next couple of months.

The TYS network is a cross industry source of supplier information and identity helping to simplify and accelerate the onboarding and lifecycle management process. TYS is a fit-for-purpose blockchain optimized for sharing supplier credentials in a supply chain environment. TYS DIDs may be used by Suppliers, Buyers, Verifiers, Banks and other organizations to establish identities for verifiable claims made by any party.

TYS is implemented on Hyperledger Fabric, a permissioned blockchain technology under the Linux Foundation&#39;s Hyperledger Project.  The &quot;Smart Contract&quot; Functions are written in &quot;Golang&quot; and all client APIs are provided as REST APIs written in &quot;Javascript&quot; running on &quot;NodeJS&quot;.

This document specifies the &quot;Trust Your Supplier&quot; [DID Method](https://w3c-ccg.github.io/did-spec/#specific-did-method-schemes) [did:tys].

This specification conforms to the requirements specified in the DID specification currently published by the W3C Credentials Community Group. For more information about DIDs and DID method specifications, please see [DID Primer](https://git.io/did-primer) and [DID Specification](https://w3c-ccg.github.io/did-spec).

**TYS Security**

TYS as an application supports authentication and authorization schemes based on certificates issued by the network Certificate Authority. It also supports multi-factor authentication. TYS transactions are executed by smart contracts that are based on business rules and policies. If a particular policy requires re-authentication, the specific smart contract function will address those needs. Key management services are provided by cloud based HSMs. The TYS ledger currently uses the &quot;Kafka&quot; based ordering service for consensus. It will soon be transitioned to the modified version of the RAFT protocol supported by the Hyperledger Fabric as its consensus algorithm when generally available.

All user interactions with TYS are logged for auditability. The application restricts a user to have only a single session at any given time.

# Method Name

The &quot;namestring&quot; that shall identify this DID method is:  **tys.**

A DID that uses this method  **MUST**  begin with the following prefix: did:tys:. Per the DID specification, this prefix MUST be in lowercase. The format of remainder of the DID, after this prefix, is specified below in the section on [Method Specific Identifiers](https://github.com/ockam-network/did-method-spec/blob/master/README.md#method-specific-identifiers).

# Method Specific Identifiers

TYS DIDs will conform with [the Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme) described in the DID spec. The format of the tys-specific-idstring is described below in [ABNF](https://tools.ietf.org/html/rfc5234):

tys-did               = &quot;did:tys:&quot; tys-specific-idstring

idstring                = 40\*HEXDIG (base58 encoded)

base58char              = &quot;1&quot; / &quot;2&quot; / &quot;3&quot; / &quot;4&quot; / &quot;5&quot; / &quot;6&quot; / &quot;7&quot; / &quot;8&quot; / &quot;9&quot; / &quot;A&quot; / &quot;B&quot; / &quot;C&quot;

                          / &quot;D&quot; / &quot;E&quot; / &quot;F&quot; / &quot;G&quot; / &quot;H&quot; / &quot;J&quot; / &quot;K&quot; / &quot;L&quot; / &quot;M&quot; / &quot;N&quot; / &quot;P&quot; / &quot;Q&quot;

                          / &quot;R&quot; / &quot;S&quot; / &quot;T&quot; / &quot;U&quot; / &quot;V&quot; / &quot;W&quot; / &quot;X&quot; / &quot;Y&quot; / &quot;Z&quot; / &quot;a&quot; / &quot;b&quot; / &quot;c&quot;

                          / &quot;d&quot; / &quot;e&quot; / &quot;f&quot; / &quot;g&quot; / &quot;h&quot; / &quot;i&quot; / &quot;j&quot; / &quot;k&quot; / &quot;m&quot; / &quot;n&quot; / &quot;o&quot; / &quot;p&quot;

                          / &quot;q&quot; / &quot;r&quot; / &quot;s&quot; / &quot;t&quot; / &quot;u&quot; / &quot;v&quot; / &quot;w&quot; / &quot;x&quot; / &quot;y&quot; / &quot;z&quot;

# Generating a unique _idstring_

A unique idstring is created as follows:

1. Generate a public/private keypair, using one of the methods in the [Linked Data Cryptographic Suite Registry](https://w3c-ccg.github.io/ld-cryptosuite-registry/).
2. Hash the public key from step 1 using one of the hashing algorithms supported by [multihash](https://multiformats.io/multihash/).
3. Truncate the hash from step 2 to the lower 20 bytes – This will be compatible with account IDs in Ethereum and other blockchains.
4. The [multihash prefix](https://github.com/multiformats/multicodec/blob/master/table.csv) for the algorithm chosen in step 2  and the length are not included in the current version. Currently the algorithm (0x12) and the length (0x14) part of a multihash is not included because length of the hashed value is always 20 bytes per step 3 and the hash algorithm is sha2-256.
5. [Base58](https://en.wikipedia.org/wiki/Base58) encode the value from step 4 using the [Bitcoin alphabet](https://en.bitcoinwiki.org/wiki/Base58#Alphabet_Base58).

The following Golang code illustrates this process of generating a unique TYS DID. This code will be part of the identity smart contract within the TYS Network. The random seed will be generated externally and passed in as a transient payload.

package main

import (

        &quot;crypto/rand&quot;

        &quot;crypto/sha256&quot;

        &quot;crypto/ecdsa&quot;

        &quot;crypto/elliptic&quot;

        &quot;github.com/btcsuite/btcutil/base58&quot;

        &quot;fmt&quot;

        &quot;os&quot;

)

func main() {

        // ECDSA Elliptical Curve Pub/Private Keys

        pubkeyCurve := elliptic.P256() //see http://golang.org/pkg/crypto/elliptic/#P256

        prKey := new(ecdsa.PrivateKey)

        prKey, err := ecdsa.GenerateKey(pubkeyCurve, rand.Reader) // this generates a public &amp; private key pair

        if err != nil {

                fmt.Println(err)

                os.Exit(1)

        }

        var pukey ecdsa.PublicKey

        pukey = prKey.PublicKey

        pubKeyStr := pukey.X.String()+pukey.Y.String()

        pubKeyHex := fmt.Sprintf(&quot;%x%x&quot;, pukey.X, pukey.Y)

        // Generate Base58 Encoded DID

        sum := sha256.Sum256([]byte(pubKeyHex))

        sum\_base58 := base58.Encode(sum[0:20])

        tys\_did\_base58enc := fmt.Sprintf(&quot;did:tys:&quot;+ sum\_base58)

        // Generate standard unencoded DID

        tys\_did := fmt.Sprintf(&quot;did:tys:%x&quot;, sum[0:20])

        fmt.Println(&quot;Private Key :&quot;)

        fmt.Printf(&quot;%d \n&quot;, prKey)

        fmt.Println()

        fmt.Println(&quot;Public Key :&quot;)

        fmt.Printf(&quot;%d \n&quot;, pukey)

        fmt.Printf(&quot;%s \n&quot;, pubKeyStr)

        fmt.Printf(&quot;%s \n&quot;, pubKeyHex );

        fmt.Println()

        fmt.Println(&quot;hash of Public Key  in HEX:&quot;)

        fmt.Printf(&quot;%x \n&quot;, sum)

        fmt.Println()

        fmt.Println(&quot;First 20 Bytes in HEX&quot;)

        fmt.Printf(&quot;%x \n&quot;, sum[0:20])

        fmt.Println()

        fmt.Println(&quot;DID in Base58 Encoding and without Encoding&quot;)

        fmt.Println(tys\_did\_base58enc)

        fmt.Println(tys\_did)

}

### Test Results

**Private Key :**

&amp;{{{842350552256} 22001963804334027204127086025456691471972245705670468283780584265832903672895 68783280147399643216889661967638391658473045558227023134602849600796510992165} 84402287888165422353075677022858628482056011430115186165685582319392847963632}

**Public Key :**

{{842350552256} 22001963804334027204127086025456691471972245705670468283780584265832903672895 68783280147399643216889661967638391658473045558227023134602849600796510992165}

2200196380433402720412708602545669147197224570567046828378058426583290367289568783280147399643216889661967638391658473045558227023134602849600796510992165

30a4ab92b3cf09e0980f7162a2cef5152c9caf84046bc19599f3968ad42f043f9811f4f9df35564903e040fd0dacecaf72e2ce68fd927aa05230e5bb24d53725

**hash of Public Key in HEX:**

6dc5d7c55790988004373b35b124eb2745c49cd4865318a9fe6c4db5e1990c15

**First 20 Bytes in HEX**

6dc5d7c55790988004373b35b124eb2745c49cd4

**DID in Base58 Encoding and without Encoding**

did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd

did:tys:6dc5d7c55790988004373b35b124eb2745c49cd4

# TYS DIDs

In TYS, any of the clients can submit a verifiable claim. All TYS member Organizations will be issued Root Certificates to join the business network. Organizations can generate credentials to operate nodes or admit their users to exercise the application. TYS Node Operators can run as many nodes as they desire with approval from the TYS Governance Body. Nodes could be _endorser_ nodes or _commiter_ nodes.

Suppliers are the holders of the credentials and buyers are considered as the relying parties. Any client with the role of an issuer can issue claims, but only a verifier can create a TYS DID after verifying the claim. In TYS a issuer can also play the role of a verifier. The following is an example of a TYS DID:

## TYS DID Example

did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd

## TYS DID Document (Organization)

In TYS, a DID can be issued to a Business Entity such as a &quot;Supplier&quot; or to a verifiable claim.  A Business Entity DID will point to a collection of DIDs that represent verifiable Claims.  The Public Key of the Supplier can be used to securely request details of the verifiable claim such as document metadata and uploaded documents.

The following is an example of DID associated with a &quot;Business Entity&quot;.

#### Example 1

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Invoking Create Method

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

DID Document:

{

  &quot;@context&quot;: &quot;https://w3id.org/did/v1&quot;,

  &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq&quot;,

  &quot;name&quot;: &quot;Alice Corp&quot;,

  &quot;owner&quot;: {

    &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq&quot;,

    &quot;type&quot;: &quot;id&quot;

  },

  &quot;dc:created&quot;: {

    &quot;dc:created&quot;: &quot;&quot;,

    &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq&quot;,

    &quot;type&quot;: &quot;xsd:dateTime&quot;

  },

  &quot;publicKey&quot;: [

    {

      &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq#key1&quot;,

      &quot;sec&quot;: &quot;93f02d9a495018403fd5dd43d96a880c7b6ebc221f176f9456f0df95fa6b6a3360d4e4c22e66f29cb67f234e7d783013873155211b4b2f16276d5f180a8d8&quot;,

      &quot;type&quot;: &quot;ECDSA&quot;

    }

  ],

  &quot;authentication&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq#key1&quot;,

  &quot;sec:revoked&quot;: {

    &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq&quot;,

    &quot;sec&quot;: &quot;&quot;,

    &quot;type&quot;: &quot;xsd:dateTime&quot;

  },

  &quot;type&quot;: &quot;SUPPLIER&quot;

}

#### Example 2

{

    &quot;@context&quot;: [https://w3id.org/did/v1](https://w3id.org/did/v1),

    &quot; **id**&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd&quot;,

    &quot; **publicKey**&quot;: [{

        &quot;id&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd#keys-1&quot;,

        &quot;type&quot;: [&quot;ECDSA&quot;, &quot;secp256r1&quot;],

        &quot;controller&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd&quot;,

        &quot;publicKeyHex&quot;:

               &quot;30a4ab92b3cf09e0980f7162a2cef5152c9caf84046bc19599f3968ad42f043f9811f4f9df35564903e040fd0dacecaf72e2ce68fd927aa05230e5bb24d53725&quot;

    }],

    &quot; **authentication**&quot;: [{

        // This key is referenced and described above

        &quot;type&quot;: [&quot;ECDSA&quot;, &quot;secp256r1&quot;],

        &quot;publicKey&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd#keys-1&quot;

    }],

    &quot; **credentials**&quot;: [{

        &quot;type&quot;: &quot;credential&quot;,

        &quot;id&quot;: &quot;did:tys:2BfdfxCGMpz7MHEKBwbadCZd6aBd#claim&quot;

    },

    {

        &quot;type&quot;: &quot;credential&quot;,

        &quot;id&quot;: &quot;did:tys:3ZydfxCGMpz7MHEKBwbadCZd6aBd#claim&quot;

    }],

}

## TYS DID Document (Claim)

The following DID document represents a verifiable claim. The attributes of this type of document include

1. Credential  Owner
2. Credential Expiration Date
3. Credential Issuance Date
4. Credential Subject
5. Credential Issuer
6. Credential Pubic Key
7. Signature of Verifier
8. Service Endpoints
9. Credential Type

DID Document:

{

  **&quot;@context&quot;:** [

    &quot;[https://www.w3.org/2018/credentials/v1](https://www.w3.org/2018/credentials/v1)&quot;,

    &quot;[https://w3id.org/did/v1](https://w3id.org/did/v1)&quot;

  ],

  &quot; **owner**&quot;: {

    &quot;id&quot;: &quot; did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd&quot;,

    &quot;type&quot;: &quot;id&quot;

  },

  &quot; **verifiableCredential**&quot;: {

    &quot; **cred:expirationDate**&quot;: {

      &quot;expirationDate&quot;: &quot;31/01/2020&quot;,

      &quot;type&quot;: &quot;xsd:dateTime&quot;

    },

    &quot; **cred:issuanceDate**&quot;: {

      &quot;issuanceDate&quot;: &quot;01/02/2019&quot;,

      &quot;type&quot;: &quot;xsd:dateTime&quot;

    },

    &quot; **credentialSubject**&quot;: &quot;Insurance Certificate of USD 1 Million&quot;,

    &quot;id&quot;: &quot;did:tys:39ryrZi9nuaRCyBxPKZiCMi3Yzge&quot;,

    &quot;issuer&quot;: {

      &quot;cred:issuer&quot;: &quot;AIA General Insurance&quot;,

      &quot;id&quot;: &quot;did:tys:12345678&quot;

    },

    &quot; **publicKey**&quot;: [

      {

        &quot;id&quot;: &quot;did:tys:39ryrZi9nuaRCyBxPKZiCMi3Yzge#key1&quot;,

        &quot;sec&quot;: &quot;59821f30bd123879b32098e6fbdf9a020d0d836bd629e7ca82c4138275af15bdbd3cc9ce0731319ab22dad8c7516df6c76928d623f13344258b88e087f50f158&quot;,

        &quot;type&quot;: &quot;ECDSA&quot;

      }

    ],

    &quot; **serviceEndpoint**&quot;: {

      &quot;serviceEndpoint&quot;: &quot;[https://www.tys.com/documents/insdoc.ecr](https://www.tys.com/documents/insdoc.ecr)&quot;

    },

    &quot; **signature**&quot;: {

      &quot;sec:signingAlgorithm&quot;: &quot;RsaSignature2018&quot;,

      &quot;signatureValue&quot;: &quot;4048574891045760770709703117946836604384822674299714246985631999965462522002985594532011646191982200179610655476059934788095400135508507503376790195335512&quot;

    },

    &quot; **type**&quot;: [

      &quot;Credential&quot;,

      &quot;InsuranceCredential&quot;

    ]

  }

}

# DID Transaction Process Flow

//TODO

# TYS Functions

### Create and Register a DID

TYS Clients can create/register an entity in the TYS Network by submitting a [Verifiable Claim](https://www.w3.org/TR/verifiable-claims-data-model) as a transaction. The [issuer](https://www.w3.org/TR/verifiable-claims-data-model) and the [subject](https://www.w3.org/TR/verifiable-claims-data-model) of this claim are the same DID that is being registered.

Here is an example of such a claim:

{

  **&quot;@context&quot;:** [

    &quot;[https://www.w3.org/2018/credentials/v1](https://www.w3.org/2018/credentials/v1)&quot;,

    &quot;[https://w3id.org/did/v1](https://w3id.org/did/v1)&quot;

  ],

  &quot; **owner**&quot;: {

    &quot;id&quot;: &quot; did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd&quot;,

    &quot;type&quot;: &quot;id&quot;

  },

  &quot; **verifiableCredential**&quot;: {

    &quot; **cred:expirationDate**&quot;: {

      &quot;expirationDate&quot;: &quot;31/01/2020&quot;,

      &quot;type&quot;: &quot;xsd:dateTime&quot;

    },

    &quot; **cred:issuanceDate**&quot;: {

      &quot;issuanceDate&quot;: &quot;01/02/2019&quot;,

      &quot;type&quot;: &quot;xsd:dateTime&quot;

    },

    &quot; **credentialSubject**&quot;: &quot;Insurance Certificate of USD 1 Million&quot;,

    &quot;id&quot;: &quot;did:tys:39ryrZi9nuaRCyBxPKZiCMi3Yzge&quot;,

    &quot;issuer&quot;: {

      &quot;cred:issuer&quot;: &quot;AIA General Insurance&quot;,

      &quot;id&quot;: &quot;did:tys:12345678&quot;

    },

    &quot; **publicKey**&quot;: [

      {

        &quot;id&quot;: &quot;did:tys:39ryrZi9nuaRCyBxPKZiCMi3Yzge#key1&quot;,

        &quot;sec&quot;: &quot;59821f30bd123879b32098e6fbdf9a020d0d836bd629e7ca82c4138275af15bdbd3cc9ce0731319ab22dad8c7516df6c76928d623f13344258b88e087f50f158&quot;,

        &quot;type&quot;: &quot;ECDSA&quot;

      }

    ],

    &quot; **serviceEndpoint**&quot;: {

      &quot;serviceEndpoint&quot;: &quot;[https://www.tys.com/documents/insdoc.ecr](https://www.tys.com/documents/insdoc.ecr)&quot;

    },

    &quot; **signature**&quot;: {

      &quot;sec:signingAlgorithm&quot;: &quot;RsaSignature2018&quot;,

      &quot;signatureValue&quot;: &quot;4048574891045760770709703117946836604384822674299714246985631999965462522002985594532011646191982200179610655476059934788095400135508507503376790195335512&quot;

    },

    &quot; **type**&quot;: [

      &quot;Credential&quot;,

      &quot;InsuranceCredential&quot;

    ]

  }

}

The [DID Document](https://w3c-ccg.github.io/did-spec/#did-documents) representing the entity is included in the body of the claim.

This transaction is approved and the entity is registered if all of these conditions are true:

1. if the did in the issuer, id and document.id fields match
2. if the document.publicKey contains at least one public key that when run through the process described in the section on [Generating a unique idstring](https://github.com/ockam-network/did-method-spec/blob/master/README.md#generating-a-unique-idstring) results in the exact DID fromthe issuer, id and document.id
3. if the signature on the claim is by one of the public keys mentioned in the DID document as approved for authentication.

### Read

TYS Clients can read a DID document by sending a query request for a DID.

For example a query for did:tys:2Mm9pLRQwueo7FJUvBoDW7QKGBXTX would return:

{

  **&quot;@context&quot;:** [

    &quot;[https://www.w3.org/2018/credentials/v1](https://www.w3.org/2018/credentials/v1)&quot;,

    &quot;[https://w3id.org/did/v1](https://w3id.org/did/v1)&quot;

  ],

  &quot; **owner**&quot;: {

    &quot;id&quot;: &quot; did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd&quot;,

    &quot;type&quot;: &quot;id&quot;

  },

  &quot; **verifiableCredential**&quot;: {

    &quot; **cred:expirationDate**&quot;: {

      &quot;expirationDate&quot;: &quot;31/01/2020&quot;,

      &quot;type&quot;: &quot;xsd:dateTime&quot;

    },

    &quot; **cred:issuanceDate**&quot;: {

      &quot;issuanceDate&quot;: &quot;01/02/2019&quot;,

      &quot;type&quot;: &quot;xsd:dateTime&quot;

    },

    &quot; **credentialSubject**&quot;: &quot;Insurance Certificate of USD 1 Million&quot;,

    &quot;id&quot;: &quot;did:tys:39ryrZi9nuaRCyBxPKZiCMi3Yzge&quot;,

    &quot;issuer&quot;: {

      &quot;cred:issuer&quot;: &quot;AIA General Insurance&quot;,

      &quot;id&quot;: &quot;did:tys:12345678&quot;

    },

    &quot; **publicKey**&quot;: [

      {

        &quot;id&quot;: &quot;did:tys:39ryrZi9nuaRCyBxPKZiCMi3Yzge#key1&quot;,

        &quot;sec&quot;: &quot;59821f30bd123879b32098e6fbdf9a020d0d836bd629e7ca82c4138275af15bdbd3cc9ce0731319ab22dad8c7516df6c76928d623f13344258b88e087f50f158&quot;,

        &quot;type&quot;: &quot;ECDSA&quot;

      }

    ],

    &quot; **serviceEndpoint**&quot;: {

      &quot;serviceEndpoint&quot;: &quot;[https://www.tys.com/documents/insdoc.ecr](https://www.tys.com/documents/insdoc.ecr)&quot;

    },

    &quot; **signature**&quot;: {

      &quot;sec:signingAlgorithm&quot;: &quot;RsaSignature2018&quot;,

      &quot;signatureValue&quot;: &quot;4048574891045760770709703117946836604384822674299714246985631999965462522002985594532011646191982200179610655476059934788095400135508507503376790195335512&quot;

    },

    &quot; **type**&quot;: [

      &quot;Credential&quot;,

      &quot;InsuranceCredential&quot;

    ]

  }

}

### Update

TYS Clients can update a DID document by submitting a [Verifiable Claim](https://www.w3.org/TR/verifiable-claims-data-model) as a transaction. The [issuer](https://www.w3.org/TR/verifiable-claims-data-model) and the [subject](https://www.w3.org/TR/verifiable-claims-data-model) of this claim are the same DID that is being updated.

{

  &quot;@context&quot;: &quot;https://w3id.org/did/v1&quot;,

  &quot;UpdateDidDescription&quot;: {

    &quot;UpdateDidDescription&quot;: &quot;old\_value=xxxx:new\_value=yyyy&quot;

  },

  &quot;authentication&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq#key1&quot;,

  &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq&quot;,

  &quot;name&quot;: &quot;Alice Corp&quot;,

  &quot;nonce&quot;: &quot;1&quot;,

  &quot;publicKey&quot;: [

    {

      &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq#key1&quot;,

      &quot;sec&quot;: &quot;93f02d9a495018403fd5dd43d96a880c7b6ebc221f176f9456f0df95fa6b6a3360d4e4c22e66f29cb67f234e7d783013873155211b4b2f16276d5f180a8d8&quot;,

      &quot;type&quot;: &quot;ECDSA&quot;

    }

  ],

  &quot;type&quot;: &quot;SUPPLIER&quot;

}

This transaction is approved, and the entity is updated if all these conditions are true:

1. if the did in the issuer, claim.id and claim.document.id fields match.
2. if the signature on the claim is by one of the public keys mentioned in the DID document as approved for authentication.

### Deactivate

Tys Clients can revoke a DID document by submitting a [Verifiable Claim](https://www.w3.org/TR/verifiable-claims-data-model) as a transaction. The [issuer](https://www.w3.org/TR/verifiable-claims-data-model) and the [subject](https://www.w3.org/TR/verifiable-claims-data-model) of this claim are the same DID that is being revoked.

The document field is set to null.

{

  &quot;@context&quot;: &quot;https://w3id.org/did/v1&quot;,

  &quot; **UpdateDidDescription**&quot;: {

    &quot;UpdateDidDescription&quot;: &quot;revoked&quot;

  },

  &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq&quot;,

  &quot; **name**&quot;: &quot;Alice Corp&quot;,

  &quot;nonce&quot;: &quot;1&quot;,

  &quot; **publicKey**&quot;: [

    {

      &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq#key1&quot;,

      &quot;sec&quot;: &quot;&quot;,

      &quot;type&quot;: &quot;ECDSA&quot;

    }

  ],

  &quot; **sec:revoked**&quot;: {

    &quot;id&quot;: &quot;did:tys:DaW1iKJEMmyvjpyHJi9pw6rcCeq&quot;,

    &quot;sec&quot;: &quot;18/04/2019&quot;,

    &quot;type&quot;: &quot;xsd:dateTime&quot;

  },

  &quot; **type**&quot;: &quot;SUPPLIER&quot;

}

This transaction is approved and the entity is deactivated if all these conditions are true:

1. if the did in the issuer and id fields match.
2. if the signature on the claim is by one of the public keys mentioned in the DID document as approved for authentication.

### GetAllCredentialsForEntity

This service queries retrieves all dids associated with the holder. It is typically based on access privileges. The access privileges are provided to buyer, auditor and regulator.

{

    &quot;@context&quot;: [https://w3id.org/did/v1](https://w3id.org/did/v1),

    &quot; **id**&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd&quot;,

    &quot; **publicKey**&quot;: [{

        &quot;id&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd#keys-1&quot;,

        &quot;type&quot;: [&quot;ECDSA&quot;, &quot;secp256r1&quot;],

        &quot;controller&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd&quot;,

        &quot;publicKeyHex&quot;:

               &quot;30a4ab92b3cf09e0980f7162a2cef5152c9caf84046bc19599f3968ad42f043f9811f4f9df35564903e040fd0dacecaf72e2ce68fd927aa05230e5bb24d53725&quot;

    }],

    &quot; **authentication**&quot;: [{

        // This key is referenced and described above

        &quot;type&quot;: [&quot;ECDSA&quot;, &quot;secp256r1&quot;],

        &quot;publicKey&quot;: &quot;did:tys:2XhdfxCGMpz7MHEKBwbadCZd6aBd#keys-1&quot;

    }],

    &quot; **credentials**&quot;: [{

        &quot;type&quot;: &quot;credential&quot;,

        &quot;id&quot;: &quot;did:tys:2BfdfxCGMpz7MHEKBwbadCZd6aBd#claim&quot;

    },

    {

        &quot;type&quot;: &quot;credential&quot;,

        &quot;id&quot;: &quot;did:tys:3ZydfxCGMpz7MHEKBwbadCZd6aBd#claim&quot;

    }],

}

# Status

This document is a work in progress draft.

# References

1. TYS URL http://www.trustyoursupplier.com
2. Decentralized Identifiers (DIDs) v0.11 [https://w3c-ccg.github.io/did-spec](https://w3c-ccg.github.io/did-spec)
3. ABNF [https://tools.ietf.org/html/rfc5234](https://tools.ietf.org/html/rfc5234)
4. Multihash - Self-describing hashes [https://multiformats.io/multihash/](https://multiformats.io/multihash/)
5. The Multihash Data Format [https://tools.ietf.org/html/draft-multiformats-multihash-00](https://tools.ietf.org/html/draft-multiformats-multihash-00)
6. Multihash Labels [https://github.com/multiformats/multicodec/blob/master/table.csv](https://github.com/multiformats/multicodec/blob/master/table.csv)
7. Base58 Encoding [https://en.wikipedia.org/wiki/Base58](https://en.wikipedia.org/wiki/Base58)
8. Bitcoin Base58 Alphabet [https://en.bitcoinwiki.org/wiki/Base58#Alphabet\_Base58](https://en.bitcoinwiki.org/wiki/Base58#Alphabet_Base58)
9. Linked Data Cryptographic Suite Registry [https://w3c-ccg.github.io/ld-cryptosuite-registry](https://w3c-ccg.github.io/ld-cryptosuite-registry)
10. Verifiable Claims [https://www.w3.org/TR/verifiable-claims-data-model](https://www.w3.org/TR/verifiable-claims-data-model)
11. JSON-LD 1.0 - A JSON-based Serialization for Linked Data [https://www.w3.org/TR/json-ld](https://www.w3.org/TR/json-ld)