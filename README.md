# Finprint CID Spec

Self-describing content-addressed identifiers for Finprint.

The Finprint CID (content identifier) is a future-proof abstraction for referencing
data stored publicly or privately in decentralized storage.
A CID provides all the information need to find, (optionally) decrypt, and
decode a piece of data.

Based on the [Multiformats CID](https://github.com/multiformats/cid) spec.

## Usage in Finprint protocol

The
[Finprint protocol](https://github.com/finprint/finprint-protocol-pact/)
currently uses CIDs in two places:
* The secret written by a writer into a lockbox should be a CID pointing
to the data being offered on the protocol. This keeps the data size of the
secret itself small.
* Each version of a lockbox on the protocol has a `secretSharesCid` attribute
provided by the writer. This should be a CID pointing to a dictionary,
mapping the account IDs of sharing nodes to their respective, encrypted shares
of the secret.

## Background and design

The Finprint CID spec is a variation on the [Multiformats CID](https://github.com/multiformats/cid)
spec used by [IPFS](https://ipfs.io) and [IPLD](https://ipld.io). Like the developers of
Multiformats, we encountered the need to store and reference content in distributed information
systems in a way that wouldn't lock us into a specific choices of data encodings and formats.

Since our technical motivations are similar, we refer you to the Multiformats CID
documentation ([[1]](https://github.com/multiformats/multiformats),
[[2]](https://github.com/multiformats/cid/blob/master/README.md),
[[3]](https://github.com/ipfs/specs/issues/130))
for further discussion of the reasoning behind the basic design of CIDs.

The original Multiformats CID offers the following features:
* “*Future proof*” in the sense that the CID definition can be easily expanded while
preserving backwards compatibility. This is achieved by using
[“self-describing”](https://github.com/multiformats/multiformats)
values, where choices affecting the format and interpreation of the data are encoded
directly into the data itself, allowing the data to be parsed without additional, outside context.
* *Content-addressed* in that the CID for a piece of data includes the data's
cryptographic hash. This provides a built-in way of verifying data integrity.
* Extensible support for one's choice of
[text encoding](https://github.com/multiformats/multibase),
[data encoding/codec](https://github.com/multiformats/multicodec),
and [hashing algorithm](https://github.com/multiformats/multihash).

Finprint CID extends the functionality of Multiformats CID by adding the following
extensible fields:
* Storage method with associated storage key
  * The storage key generalizes and takes over the role of mutihash in the
  Multiformats CID spec, allowing for more flexibility in the method and
  location of storage.
* Encryption method with associated decryption key

## Specification

A Finprint CID is the concatenation of the following fields:
* **Multibase prefix**:
according to the [multibase spec](https://github.com/multiformats/multibase)
* **CID version**: unsigned varint, equal to 0x41
* **Storage method**:
unsigned varint, index into a Finprint-specific [table](src/tables.ts)
* **Encryption method**:
unsigned varint, index into a Finprint-specific [table](src/tables.ts)
* **Content encoding**:
unsigned varint, according to the
[multicodec spec](https://github.com/multiformats/multicodec)
* **Storage key length:** unsigned varint
* **Storage key**: format depends on storage location
* **Decryption key**: format depends on encryption method

The spec follows the same general format as Multiformats CID but with different
fields. As with Multiformats, the multibase prefix is required for text-encoded
CIDs, but is optional when the CID is unambiguously in raw binary format.

## Interoperability with Multiformats CID

**CID version:** A Multiformats CID begins with a multibase prefix
(optional for binary data) followed by a CID version identifier.
That version ID is `1` for the current (and only) version of the Multiformats
CID spec. To avoid overlap, the Finprint CID spec follows the same convention for
denoting version ID, but uses `0x41 = 65` as the version ID.

**Multicodec:** Similar to Multiformats, we use “tables” to define the extensible
specification of data formats supported by the protocol. Two of these tables are
custom and specific to Finprint. The content encoding field, however, makes use
of the Multiformats [multicodec](https://github.com/multiformats/multicodec)
table. This means that code written to handle multicodecs
may be leveraged for use with Finprint CIDs.

## Finprint-specific tables

We use custom tables to define the storage and encryption
methods supported by the Finprint protocol. The spec includes precise rules
defining how these methods are to be used. New entries can be added to the table
if the protocol needs to support additional methods.

These table specifications are currently housed in [src/tables.ts](src/tables.ts).

## Implementations

* [ts-cid](https://github.com/finprint/ts-cid)
* Golang: `pkg/cid` in
[sharing-node-pact](https://github.com/finprint/sharing-node-pact/tree/master/pkg/cid)

## License

This repository is only for documents. These are licensed under a [CC-BY 4.0 International License](LICENSE).
