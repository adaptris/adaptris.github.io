---
title:             "Interlok + AWS KMS"
description:       "Fun and games with AWS KMS interoperability"
published:         true
categories:        [interlok, aws]
tags:              [interlok, aws]
author:            [quotidian-ennui]
keywords:          "interlok, aws, kms, encrypt, decrypt, sign, verify, public key, private key, cryptography, bouncy castle"
billboard: /billboards/icon-open-source.png
excerpt_separator: <!-- more -->
---

As of 3.10.1 (the next release due Q2 2020); there is support for AWS KMS within Interlok for their key operations, namely encryption; decryption; signing; and signature verification. The AWS SDK is very easy to use, so producing the code itself didn't take long; it took longer to write the unit tests as usual. When writing a new optional component, we normally make the decision that we expose the configuration to the user, so the user can decide how they want to interact with the system.

<!-- more -->

While KMS abstracts a lot of the complexity for you, cryptography is _hard to get right_; that means we as integration specialists need to understand at a non-superficial level a lot of the technologies and what tools we can use to investigate and debug. I'm going to focus on the signing / signature verification since this is most relevant to the use-case that we're doing.

KMS is pure key management; by this I mean you have public and private keys (if you're using asymmetric operations) but no concept of X.509 certificates or similar (this becomes important a bit later). Any asymmetric operations you do via KMS assume that you have access to both the public + private key; this means that your customer managed key cannot _just be the public key_. This then has a cascading effect because if you want to verify a document from your business partner; you will only have access to their public key, which means that you can't use KMS to do signature verification.

Our use case is pretty simple it's basically :

1. Create a digital signature of a document.
1. Send the document to a REST endpoint; along with the signature.
1. Receive a document via a webhook
1. Verify the signature of the document.


However, of course, the devil is in the details; and we faced a few problems on the way.

## Getting the public key to our partner.

In order to deliver our public key to the partner, we had to create a PKCS10 Certificate signing request and send it to them. This is effectively a trivial problem to solve with code, but I genuinely thought that there would be *something out there already that could do this*. Sadly either my google-fu wasn't up to scratch or there genuninely wasn't. As a result I knocked up a quick bit of code that does the work for me so I guess now there is. It's just a case of using the AWS SDK in conjunction with the bouncycastle Crypto APIs : [https://github.com/quotidian-ennui/aws-kms-csr](https://github.com/quotidian-ennui/aws-kms-csr).

## Deciding on the KMS signing mode RAW or DIGEST

This is point where _knowing enough about enough things to get yourself into trouble_ is really useful. It's _quite obvious now_ but it wasn't when the delivery team first knocked at my door. If you look at the steps above; then it's apparent that you can mock out the remote endpoint by using an Interlok instance with 2 workflows (i.e. the first workflow does steps 1+2; then second workflow does 3+4) or some variant thereof. That means you can test the plumbing without hitting an external service, incurring costs or banging against security policies. However, of course, for step 4 in the execution chain, we would only have access to a public key, which means that we can't use the `verify(VerifyRequest)` method. This isn't really a big deal since we can just write some custom code using `java.security.Signature` to verify the document.

What was not obvious in this case was how the _MessageType_ changes the behaviour. Since we can't guarantee the size of the message, they were computing the hash, using the `DIGEST` mode when signing, passing in the computed hash as the message parameter for the SignRequest. This actually meant that steps 3+4 didn't work; however it did work if we changed the mode to `RAW`.

Of course, this means some digging, and filling out some of the gaps in our understanding of the KMS documentation. It all really starts with openssl **because it always does** since this is the reference implementation for everything crypto; if you can make it work with openssl then you just have to understand what you're doing in the language of your choice, since almost everything useful will ultimately expose something similar to openssl.

In the following extracts, `digest-raw.sig` is the KMS signature generated in DIGEST mode (not base64 encoded); `raw-raw.sig` is the KMS generated signature using RAW mode. We can ask `openssl rsautl` to do what it can with the files :

```console
$ openssl rsautl -verify -in raw-raw.sig -inkey public-key.pem -pubin -hexdump
0000 - 30 31 30 0d 06 09 60 86-48 01 65 03 04 02 01 05   010...`.H.e.....
0010 - 00 04 20 7b be ad f3 68-80 0c 98 c1 62 72 d2 7b   .. {...h....br.{
0020 - e3 e9 52 49 83 e2 30 6f-50 78 88 5c a5 20 d0 99   ..RI..0oPx.\. ..
0030 - b4 c5 fa
```

```console
$ openssl rsautl -verify -in digest-raw.sig -inkey public-key.pem -pubin -hexdump
0000 - 30 31 30 0d 06 09 60 86-48 01 65 03 04 02 01 05   010...`.H.e.....
0010 - 00 04 20 6d 09 e4 29 04-59 09 26 2e 56 81 b7 29   .. m..).Y.&.V..)
0020 - 70 c0 3e 39 dd dd f0 91-01 f4 86 3d 5f c8 e8 19   p.>9.......=_...
0030 - 5b fc 77                                          [.w
```

We know that the hash of the payload is always part of the signature; and since those operations give you slightly different results then the signatures must contain a different *hash*; which means we switch to `openssl dgst` to try and verify the file.

```console
$ openssl dgst -verify public-key.pem -hex -signature raw-raw.sig orig.txt
Verification Failure
$ openssl dgst -verify public-key.pem -hex -signature digest-raw.sig orig.txt
Verified OK
```

I'm not going to pretend that I'm a crypto expert or anything; and google led me to [StackOverflow](https://stackoverflow.com/questions/9951559/difference-between-openssl-rsautl-and-dgst) where the illuminating statement is : _The simple answer is that `dgst -sign` creates a hash, ASN1 encodes it, and then signs the ASN1 encoded hash, whereas `rsautl -sign` just signs the input without hashing or ASN1 encoding_. This gives us enough clues to think about what KMS is doing, but first of all we can compute the hash of the of the document just to confirm.

```console
$ cat orig.txt | sha256sum | xxd -r -p | hexdump -C
00000000  6d 09 e4 29 04 59 09 26  2e 56 81 b7 29 70 c0 3e  |m..).Y.&.V..)p.>|
00000010  39 dd dd f0 91 01 f4 86  3d 5f c8 e8 19 5b fc 77  |9.......=_...[.w|
00000020
```

This tells us that _digest-raw.sig_ contains the correct hash, and _raw-raw.sig_ contains an different hash entirely. The right question to ask now is: _What is the difference between RAW & DIGEST mode in AWS KMS?_ I'm sure that the answer to this question is available in their copious documentation but it can be summed up as :

* RAW mode re-computes the hash of your message parameter and signs it -> effectively doing openssl dgst -sign (this is actually demonstrated by the hash in _raw-raw.sig_; it appears to be `SHA256(SHA256(orig.txt))`)
* DIGEST just uses the ByteBuffer that you've sent *as part of the signature* -> effectively doing openssl rsautl -sign

_What does this mean for Interlok/java.security.Signature?_ We know that `Signature.verify()` must compute the hash of the whatever bytes you've used with `update()` when checking the signature, so we need to adjust our thinking in terms what we're verifying.

* If the "hash of the document was signed in DIGEST mode" -> it's the actual payload that you want to verify not the hash of the document.
* If the "hash of the document was signed in RAW mode" -> then it is the hash that you want to verify, because that is the real document.

None of this required any code changes to our custom services in this instance. Because they were previously hashing the payload before invoking KMS, they were computing the hash and using that as part of signature verification step; all they needed to do was to pass in the raw payload to the custom signature-verification service. Brute force and ignorance would have got them to the same place but understanding why is important.


## Bonus Chatter

The custom signature verification service is very simple; the key parts are to *read the public key from a PEM file* and then to verify the signature. We're using `MessageWrapper<byte[]>` for `signature` and `data-to-be-verified` which is why it was easy to switch configuration without any changes; it is repeated here; again, it uses the bouncycastle crypto APIs.

```
  @Override
  protected void initService() throws CoreException {
    try (PEMParser parser = new PEMParser(new FileReader(publicKeyFile))) {
      SubjectPublicKeyInfo keyInfo = (SubjectPublicKeyInfo) parser.readObject();
      publicKey = new JcaPEMKeyConverter().getPublicKey(keyInfo);
    } catch (Exception e) {
      throw ExceptionHelper.wrapCoreException(e);
    }
  }

  @Override
  public void doService(AdaptrisMessage msg) throws ServiceException {
    try {
      Signature sig = Signature.getInstance(signingAlgorithm(), BouncyCastleProvider.PROVIDER_NAME);
      sig.initVerify(publicKey);
      byte[] toBeVerified = getDataToBeVerified().wrap(msg);
      byte[] theSignature = getSignature().wrap(msg);
      sig.update(toBeVerified);
      if (!sig.verify(theSignature)) {
        throw new ServiceException("Signature was not verified; Signature#verify returned false");
      }
    } catch (Exception e) {
      throw ExceptionHelper.wrapServiceException(e);
    }
  }
```
