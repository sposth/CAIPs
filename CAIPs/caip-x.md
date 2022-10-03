---
# Every document starts with a front matter in YAML enclosed by triple dashes.
# See https://jekyllrb.com/docs/front-matter/ to learn more about this concept.
caip: <to be assigned>
title: 3ID - Private composible identity
author: Joel Thorstensson <joel@3box.io>
discussions-to: <URL>
status: Draft
type: Standard
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
updated: <date last updated, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <CAIP number(s)>
replaces (*optional): <CAIP number(s)>
---

<!--You can leave these HTML comments in your merged EIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EIPs. Note that an EIP number will be assigned by an editor. When opening a pull request to submit your EIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`. The title should be 44 characters or less.-->
This is the suggested template for new CAIPs.

Note that an CAIP number will be assigned by an editor. When opening a pull request to submit your CAIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`.

The title should be 42 characters or less.

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the CAIP.-->
If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the CAIP.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
A short (~200 word) description of the technical issue being addressed.
A self-certifying revocation regitry.

## Motivation
<!--The motivation is critical for CAIP. It should clearly explain why the state of the art is inadequate to address the problem that the CAIP solves. CAIP submissions without sufficient motivation may be rejected outright.-->
The motivation is critical for CAIP. It should clearly explain why the state of the art is inadequate to address the problem that the CAIP solves. CAIP submissions without sufficient motivation may be rejected outright.

## Specification


<!--The technical specification should describe the standard in detail. The specification should be detailed enough to allow competing, interoperable implementations. -->
The technical specification should describe the standard in detail. The specification should be detailed enough to allow competing, interoperable implementations.

### DID Method Name

The name string that shall identify this DID method is: `3`.

A DID that uses this method MUST begin with the following prefix: `did:3`. Per the [DID specification](https://w3c.github.io/did-core/), this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

### Method Specific Identifier

`did:3:<capHash>`

Where `<capHash>` is a multihash over the CID of an object capability, e.g. a CACAO. The multibase MUST be encoded using `base64url`

##### Example

TODO - example needs to be multihash

`did:3:ui9TTAHdWzDSMRyZIkefeu6Wujcg2YFkjXSBvpmO4jL0`

### CRUD Operation Definitions

#### Create



#### Read

Deterministic based on the `<capHash>`. 

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
  ]
  "id": "did:3:<capHash>",
  "authentication": [{
    "id": "did:3:<capHash>#0",
    "type": "capHash",
    "controller": "did:3:<capHash>",
    "multihash": "<capHash>"
  }]
}
```



**Example:**

`did:3:ui9TTAHdWzDSMRyZIkefeu6Wujcg2YFkjXSBvpmO4jL0`

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
  ]
  "id": "did:3:ui9TTAHdWzDSMRyZIkefeu6Wujcg2YFkjXSBvpmO4jL0",
  "authentication": [{
    "id": "did:3:ui9TTAHdWzDSMRyZIkefeu6Wujcg2YFkjXSBvpmO4jL0#0",
    "type": "capHash",
    "controller": "did:3:ui9TTAHdWzDSMRyZIkefeu6Wujcg2YFkjXSBvpmO4jL0",
    "multihash": "ui9TTAHdWzDSMRyZIkefeu6Wujcg2YFkjXSBvpmO4jL0"
  }]
}
```

#### 

#### Update



#### Deactivate

In order to deactivate a 3ID all capabilities will need to be revoked. This can be done in a single update. Note however that for an observer resolving the DID it's not possible to determine if the DID is deactivated or not since it's impossible to know if there are capabilites that have yet to be revoked.

### Object Capabilities 

Uses CACAO + CapGrok.

All capabilities MUST include the `did:3:self?resource=vnd.ipld.car` caveat.

All capabilities MUST be over resource `did:3:<capHash>`, with the exception of the genesis capability since it is used to produce `<capHash>`.

Using the capgrok namespace `3`.

```
urn:capability:3:eyJkZWZhdWx0QWN0aW9ucyI6WyJyZWFkIl0sInRhcmdldGVkQWN0aW9ucyI6eyJteS5yZXNvdXJjZS4xIjpbImFwcGVuZCIsImRlbGV0ZSJdLCJteS5yZXNvdXJjZS4yIjpbImFwcGVuZCJdLCJteS5yZXNvdXJjZS4zIjpbImFwcGVuZCJdfX0
```



```json
{
  "def": [ "write" ]
  "ext":{
    "pcap": "bafybeigk7ly3pog6uupxku3b6bubirr434ib6tfaymvox6gotaaaaaaaaa",
    "revreg": "did:3:self?resource=vnd.ipld.car"
  }
}
```

When this capability is invoked the `revreg` caveat MUST be used to check if the capability chain used in for invocation was valid at the point of invocation. 

The `revreg` DID URL [dereferences](https://wiki.trustoverip.org/display/HOME/DID+URL+Resource+Parameter+Specification) to a key value store (IPLD HAMT ) where a key is a *capHash* and the value is of the following format:

```ipldsch
type type CapHash = Bytes

// The write event is a DagJWS where the payload CID
// is of type HAMT<CapHash:Value>.
type WriteEvent = DagJOSE

type AnchorProof struct {
  chainId String
  txHash Link
  root Link
}

type AnchorEvent struct {
  id Link
  prev &WriteEvent
  proof &AnchorProof
  path String
}

type TimeProof struct {
  time Integer
  event &AnchorEvent
} representation tuple

type Value struct {
  revoked Bool
  createdAt nullable TimeProof
  revokedAt nullable TimeProof
} representation tuple

type HAMT<CapHash:Value>

```

1. create capHash genesis -> h0
2. Put h0 into HAMT (build kv-store with only h0)
3. grant new capHash -> h1
4. Put h1 into HAMT (add h1 to kv-store)
5. Create writeEvent for HAMT root
6. Anchor write event
7. Update h1 value in HAMT to include `createTime`

If there are two writes before anchor:

* If write-2 includes all updates to kv-store from write-2 both updated kv-pairs would get the anchor of write-2
* If there are two conflicting writes, both writes would be considered valid and the kv-store HAMT can be merged after the anchors

### Registrations

#### Multihash verification method property

For did-spec-registries registration

#### CapHash

For did-spec-registries registration

#### CapGrok "3" namespace



#### StreamId code





## Rationale

<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All CAIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CAIP must explain how the author proposes to deal with these incompatibilities. CAIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->
All CAIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CAIP must explain how the author proposes to deal with these incompatibilities. CAIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases
<!--Please add test cases here if applicable.-->
Please add test cases here if applicable.

## Links
<!--Links to external resources that help understanding the CAIP better. This can e.g. be links to existing implementations.-->
Links to external resources that help understanding the CAIP better. This can e.g. be links to existing implementations.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

