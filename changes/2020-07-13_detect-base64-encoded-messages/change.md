[//]: # "Copyright Amazon.com Inc. or its affiliates. All Rights Reserved."
[//]: # "SPDX-License-Identifier: CC-BY-SA-4.0"

# Detect Base64-encoded Messages

## Affected Features

This serves as a reference of all features that this change affects.

| Feature                                                                                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------- |
| [Decrypt](https://github.com/awslabs/aws-encryption-sdk-specification/blob/623992d999db0b309d8a8adbd664f0d72feee813/client-apis/decrypt.md) |

## Affected Specifications

This serves as a reference of all specification documents that this change affects.

| Specification                           |
| --------------------------------------- |
| [Decrypt](../../client-apis/decrypt.md) |

## Affected Implementations

This serves as a reference for all implementations that this change affects.

| Language   | Repository                                                                            |
| ---------- | ------------------------------------------------------------------------------------- |
| C          | [aws-encryption-sdk-c](https://github.com/aws/aws-encryption-sdk-c)                   |
| Java       | [aws-encryption-sdk-c](https://github.com/aws/aws-encryption-sdk-java)                |
| Javascript | [aws-encryption-sdk-javascript](https://github.com/aws/aws-encryption-sdk-javascript) |
| Python     | [aws-encryption-sdk-javascript](https://github.com/aws/aws-encryption-sdk-python)     |

## Definitions

### Conventions used in this document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

## Summary

When a decryption attempt is made on the Base64-encoding of a valid message,
implementations SHOULD detect this and provide a more specific error message.

## Motivation

The message format is a binary format, but it is a common mistake for users to attempt decryption on the Base64 encoding of this data instead.
Because the first two bytes of the message format have a very limited set of possible values
(currently they are in fact fixed),
the first two bytes of the Base64 encoding of a valid message are also simple to recognize.

## Drawbacks

This feature technically means the version value
cannot be incremented beyond the hex value `40`,
since `41` would then conflict with the Base64-encoded value for `1`.
This does not seem worth worrying about
given how rarely we intended to change the version, however.

## Security Implications

This change SHOULD NOT have any security implications.

## Operational Implications

This change should have positive operational impact,
as it should reduce the number of support engagements with customers
necessary to help diagnose failures due to this root cause.

## Guide-level Explanation

When a decryption attempt is made on the Base64-encoding of a valid message,
implementations SHOULD detect this and provide a more specific error message on a best-effort basis.

## Reference-level Explanation

Implementations SHOULD detect the first two bytes of the Base64 encoding of any supported message [versions](../data-format/message-header.md#version-1)
and [types](../data-format/message-header.md#type)
and fail with a more specific error message.
In particular, the hex values to detect for the current set of versions and types are:

| Version and type (hex) | Base64 encoding (ascii) | Base64 encoding (hex) |
| ---------------------- | ----------------------- | --------------------- |
| 01 80                  | A Y ...                 | 41 59 ...             |

Note that the bytes that follow the initial two in the Base64 encoding
partially depend on subsequent bytes in the binary message format
and hence are not as predictable.