[![Build Status](https://travis-ci.com/mysto/java-fpe.svg?branch=main)](https://travis-ci.com/mysto/java-fpe)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Downloads](https://pepy.tech/badge/ff3)](https://pepy.tech/project/ff3)

# ff3 - Format Preserving Encryption in Java

An implementation of the NIST approved Format Preserving Encryption (FPE) FF3 algorithm in Java.

This package follows the FF3 algorithm for Format Preserving Encryption as described in the March 2016 NIST publication 800-38G _Methods for Format-Preserving Encryption_, 
and revised on February 28th, 2019 with a draft update for FF3-1.

* [NIST Recommendation SP 800-38G (FF3)](http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-38G.pdf)
* [NIST Recommendation SP 800-38G Revision 1 (FF3-1)](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-38Gr1-draft.pdf)

Changes to minimum domain size and revised tweak length have been partially implemented in this package with updates to domain size. It is expected that the final standard will provide new test vectors necessary to change the tweak lengths to 56 bits.  Currently, tweaks remain set to 64 bits.

## Requires

This project was built and tested with Java 11.  It uses the javax.crypto for AES encryption in ECB mode.

## Build & Testing

Build this project with gradle:

`gradle build`

There are official [test vectors](https://csrc.nist.gov/csrc/media/projects/cryptographic-standards-and-guidelines/documents/examples/ff3samples.pdf) for FF3 provided by NIST, which are used for testing in this package.

To run unit tests on this implementation, including all test vectors from the NIST specification, run the command:

  `gradle test`

## Usage

FF3 is a Feistel cipher, and Feistel ciphers are initialized with a radix representing an alphabet. The number of 
characters in an alphabet is called the _radix_.
Practical radix limits of 36 in Java means the following radix values are typical:
* radix 10: digits 0..9
* radix 36: alphanumeric 0..9, a-z

Special characters and international character sets, such as those found in UTF-8, would require a larger radix, and are not supported.
Also, all elements in a plaintext string share the same radix. Thus, an identification number that consists of a letter followed
by 6 digits (e.g. A123456) cannot be correctly encrypted by FPE while preserving this convention.

Input plaintext has maximum length restrictions based upon the chosen radix (2 * floor(96/log2(radix))):
* radix 10: 56
* radix 36: 36

To work around string length, its possible to encode longer text in chunks.

As with any cryptographic package, managing and protecting the key(s) is crucial. The tweak is generally not kept secret.
This package does not protect the key in memory.

## Code Example

The example code below can help you get started.

Using default domain [0-9]

```jshell
   jshell --class-path build/libs/java-fpe-X.X-SNAPSHOT.jar

    import com.privacylogistics.FF3Cipher;
    FF3Cipher c = new FF3Cipher("EF4359D8D580AA4F7F036D6F04FC6A94", "D8E7920AFA330A73");
    String pt = "4000001234567899";
    String ciphertext = c.encrypt(pt);
    String plaintext = c.decrypt(ciphertext);
    pt;ciphertext;plaintext
```

## The FF3 Algorithm

The FF3 algorithm is a tweakable block cipher based on an eight round Feistel cipher. A block cipher operates on fixed-length groups of bits, called blocks. A Feistel Cipher is not a specific cipher,
but a design model.  This FF3 Feistel encryption consisting of eight rounds of processing
the plaintext. Each round applies an internal function or _round function_, followed by transformation steps.

The FF3 round function uses AES encryption in ECB mode, which is performed each iteration 
on alternating halves of the text being encrypted. The *key* value is used only to initialize the AES cipher. Thereafter
the *tweak* is used together with the intermediate encrypted text as input to the round function.

## Other FPE Algorithms

Only FF1 and FF3 have been approved by NIST for format preserving encryption. There are patent claims on FF1 which allegedly include open source implementations. Given the issues raised in ["The Curse of Small Domains: New Attacks on Format-Preserving Encryption"](https://eprint.iacr.org/2018/556.pdf) by Hoang, Tessaro and Trieu in 2018, it is prudent to be very cautious about using any FPE that isn't a standard and hasn't stood up to public scrutiny.

## Implementation Notes

This implementation follows the algorithm as outlined in the NIST specification as closely as possible, including naming.

FPE can be used for sensitive data tokenization, especially with PCI and cryptographically reversible tokens. This implementation does not provide any guarantees regarding PCI DSS or other validation.

While all test vectors pass, this package has not otherwise been extensively tested.

Java's [BigInteger](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigInteger.html) supports radices/bases up to 36. Therefore, this package also supports a max base of 36, which can contain numeric digits 0-9 and lowercase alphabetic characters a-z.

FF3 uses a single-block encryption with an IV of 0, which is effectively ECB mode. AES ECB is the only block cipher function which matches the requirement of the FF3 spec.

The domain size was revised in FF3-1 to radix<sup>minLen</sup> >= 1,000,000 and is represented by the constant `DOMAIN_MIN` in `FF3Cipher.java`. FF3-1 is in draft status and updated 56-bit test vectors are not yet available.

The tweak is required in the initial `FF3Cipher` constructor, but can optionally be overridden in each `encrypt` and `decrypt` call. This is similar to passing an IV or nonce when creating an encryptor object.

## Author

Brad Schoening

## License

This project is licensed under the terms of the [Apache 2.0 license](https://www.apache.org/licenses/LICENSE-2.0).
