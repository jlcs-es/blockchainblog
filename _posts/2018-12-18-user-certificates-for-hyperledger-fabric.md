---
layout: post
published: false
title: User certificates for Hyperledger Fabric without Fabric CA
subtitle: Load user certificates in the NodeJS SDK client
date: '2018-12-18'
---

In Hyperledger Fabric we have two BCCSP (Blockchain Cryptographic Service Provider) options: Software or PKCS#11.

This is intended so that if you have an HSM (Hardware Security Module) in your organization, you can create your private keys inside the HSM, therefore never leaving a secure environment. But if you don't have such a device, you still can use files to store your keys and certificates.

Hyperledger Fabric also offers its own CA implementation: [Hyperledger Fabric CA](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/).

Fabric CA can take care of the certificates lifecycle in a Hyperledger Fabric friendly way, i.e. it is going to set everything up to work just fine with your peers and clients.

Nonetheless, this is yet another node to set up and maintain. What if your organization already has its own protocols and policies to manage certificates? What if they don't trust this CA implementation?

In this article, I'll show how to generate **Hyperledger Fabric compatible** certificates using **OpenSSL**, as well as to how generate them inside an **HSM** so your peers and clients can actually use them.



## Software Based Certificates

The current NodeJS SDK (for Fabric v1.4), only seems to [admit](https://github.com/hyperledger/fabric-sdk-node/blob/release-1.4/fabric-client/lib/impl/CryptoSuite_ECDSA_AES.js#L166) PEM [ECDSA](https://github.com/hyperledger/fabric-sdk-node/blob/release-1.4/fabric-client/lib/impl/ecdsa/key.js#L15) certificates. More specifically, the library used underneath is [jsrsasign](https://kjur.github.io/jsrsasign/), but only the ECDSA version.

Therefore, if we are going to use our own certificates, first, we need to store them in compatible formats, or transform them when passing the certificate and private key to the SDK. jsrasign is compatible with PEM formatted PKCS#1/5/8 private/public keys and X.509 certificates.

PEM format and private keys in [PKCS8](https://en.wikipedia.org/wiki/PKCS_8) format.

### OpenSSL scripts

These are **not production** intended scripts, as in the same function the private key and the parent CA signature take place. These are just for testing and inspiration purposes, so you can adapt to your companies policies from a minimal OpenSSL command.

The following bash script defines three functions that generate valid H. Fabric certificates:

<script src="https://gist.github.com/jlcs-es/90e94bde5ea8cfe1a84723646d23bc19.js"></script>

NOTES:
* These scripts use the Elliptic Curve Prime256v1, althouth Fabric is compatible with other curves.
* All certificates are issued with ~10 years of validity.
* Instead of modifying your default OpenSSL configuration file, these commands load the minimal configuration for H. Fabric, using the bash `<( command_that_prints_something)` utility. This saves the printed output of the inner command to a temporal file, and replaces all the `<(...)` with the temporal file path.

