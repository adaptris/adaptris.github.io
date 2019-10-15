---
title:              "The new PGP services"
description:        "A collection of services that provide PGP encryption, decryption, signing, and verification."
author:             albinoloverats
date:               2019-10-15
published:          true
tags:               [interlok, interlok-pgp, gpg, pgp, cryptography]
categories:         [interlok, interlok-pgp, cryptography]
keywords:           "interlok, interlok-pgp, gpg, pgp, encrypt, decrypt, sign, verify, public key, private key, cryptography, bouncy castle"
excerpt_separator:  <!-- more -->
---

Interlok PGP is a collection of services that provide an easy way to
perform the major PGP functions: encryption, decryption, signing, and
signature verification. It uses [Bouncy Castle] to do the heavy lifting.

<!-- more -->

It's actually a little surprising that it's taken until now for PGP
functionality to be requested, but here we are. It is by no means
complete, as the choice of algorithms are currently hard-coded to
AES-256 for encryption and SHA-256 for signing, both of which could be
made available to the end user as advanced configuration options.

Likewise with signatures, the current options are only clearsigned and
detached, and not the traditional signed document. This would be an
ideal enhancement for when the original message is binary data and the
end user doesn't want a detached signature, as the clearsign option
mixes the ASCII signature with the binary message.

Below are some example service XML configurations for all of the four
services, with a few comments to indicate where cleartext and ciphertext
are coming from and going to each time.

Obviously the services can be chained together so that a document can be
signed, to prove where the message has come from, and then encrypted for
the intended recipient. Or conversely, decrypted with the cleartext then
verified as coming from the expected origin.

## Encryption

This service provides a way to encrypt messages with GPG/PGP. It
requires a public key or the intended recipient, and a message to
encrypt. Optionally it will ASCII armor encode the cipher text
(default), and include extra integrity checks (default).

```xml
    <pgp-encrypt>
        <unique-id>mad-lalande</unique-id>
        <public-key class="constant-data-input-parameter">
            <value>-----BEGIN PGP PUBLIC KEY BLOCK-----

    mQENBF2ckxABCAC5Kfu39ky3OIXkxwWOJx70G2dLRYvDMHXf3ZraUPNRMIhh3ZGx
    -----END PGP PUBLIC KEY BLOCK-----</value>
        </public-key>
        <clear-text class="stream-payload-input-parameter"/>             <!-- clear text comes from message payload -->
        <cipher-text class="stream-payload-output-parameter"/>           <!-- cipher text goes back into the message payload -->
        <armor-encoding>true</armor-encoding>
        <integrity-check>true</integrity-check>
    </pgp-encrypt>
```

## Decryption

This service provides a way to decrypt GPG/PGP encrypted messages. It
requires a private key, the passphrase to unlock the key, and an
encrypted message.

```xml
    <pgp-decrypt>
        <unique-id>trusting-mayer</unique-id>
        <private-key class="constant-data-input-parameter">
            <value>-----BEGIN PGP PRIVATE KEY BLOCK-----

    lQPGBF2ckxABCAC5Kfu39ky3OIXkxwWOJx70G2dLRYvDMHXf3ZraUPNRMIhh3ZGx
    -----END PGP PRIVATE KEY BLOCK-----</value>
        </private-key>
        <passphrase class="constant-data-input-parameter">
            <value>my5ecr3tP455w0rd</value>
        </passphrase>
        <cipher-text class="stream-payload-input-parameter"/>            <!-- cipher text comes from message payload -->
        <clear-text class="stream-payload-output-parameter"/>            <!-- clear text goes back into the message payload -->
    </pgp-decrypt>
```

## Signatures

This service provides a way to sign messages via GPG/PGP. It requires a
private key, the passphrase to unlock the key, and a message to sign.
Optionally it will ASCII armor encode the signature (default) and create
a detached signature (default).

```xml
    <pgp-sign>
        <unique-id>nostalgic-golick</unique-id>
        <private-key class="constant-data-input-parameter">
            <value>-----BEGIN PGP PRIVATE KEY BLOCK-----

    lQPGBF2ckxABCAC5Kfu39ky3OIXkxwWOJx70G2dLRYvDMHXf3ZraUPNRMIhh3ZGx
    -----END PGP PRIVATE KEY BLOCK-----</value>
        </private-key>
        <passphrase class="constant-data-input-parameter">
            <value>my5ecr3tP455w0rd</value>
        </passphrase>
        <clearText class="stream-payload-input-parameter"/>              <!-- clear text comes from message payload -->
        <armor-encoding>true</armor-encoding>
        <detached-signature>true</detached-signature>
        <signature class="metadata-stream-output-parameter">             <!-- detached signature goes into message metadata -->
            <metadata-key>signature</metadata-key>
        </signature>
    </pgp-sign>
```

## Verification

This service provides a way to verify GPG/PGP signed messages. It
requires the public key of whom signed the message, the signed message,
and (if the signature is detached) the signature. It will will also
optionally return the original/unsigned message (especially useful if
the signature was not detached).

```xml
    <pgp-verify>
        <unique-id>jovial-elion</unique-id>
        <public-key class="constant-data-input-parameter">
            <value>-----BEGIN PGP PUBLIC KEY BLOCK-----

    mQENBF2ckxABCAC5Kfu39ky3OIXkxwWOJx70G2dLRYvDMHXf3ZraUPNRMIhh3ZGx
    -----END PGP PUBLIC KEY BLOCK-----</value>
        </public-key>
        <signed-message class="stream-payload-input-parameter"/>         <!-- signed message (without signature, as it's detached) -->
        <signature class="metadata-stream-input-parameter">              <!-- detached signature comes into message metadata -->
            <metadata-key>signature</metadata-key>
        </signature>
        <original-message class="string-payload-data-output-parameter"/> <!-- optional original message, without signature -->
    </pgp-verify>
```

[Bouncy Castle]: https://bouncycastle.org/
