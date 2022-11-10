---
caip: <to be assigned>
title: Authenticated IPLD Append-Only Log
author: Zach Ferland <@zachferland>, Joel Thorstensson (@oed)
discussions-to: <>
status: Draft
type: Standard
created: 2022-11-10
updated: 2022-11-10
requires: <>
---

## Simple Summary 

Authenticated IPLD Append-Only Logs provide the best of IPLD and an authenticated immutable underlying log with the ability to model mutable databases and data structures on top. 

## Abstract

This specification combines IPLD and DagJWS (with optional use of CAIP-X DagJWS CACAO), with a minimal log based data structure. A log based data structure being a simple to define subset of a general IPLD DAG. 

## Motivation

Append-only logs are a frequently used underlying immutable data structure in distributed systems to improve data integrity, consistency, performance, history, etc. Open distributed systems use hash linked lists/logs, to allow others to verify the integrity of any data. IPLD is a natural way to define an immutable append-only log. Combined with DagJWS and "CAIP-X DagJWS CACAO" it allows authenticated writes to these logs using blockchain accounts. Providing a common data base layer for users and applications besides more expensive on-chain data or centralized and siloed databases. 

A minimally defined log structure allows a base level of interoperability while allowing diverse implementations of mutable databases and data structures on top. Base levels of interoperability could include log transport, update syncing, consensus, etc.

## Specification

Logs are made up of events. A genesis event being the origination of a new log and typically used to reference or name a log. Every additional “update” appended is a data event. This specification provides minimal definition of an IPLD log, parameters in both the headers and body can be added by specific implementations or protocols. Formats and types are described using [IPLD schema language](https://ipld.io/docs/schemas/).

Being a CAIP, with the optional use of "CAIP-X DagJWS CACAO", keys and references to signers are expected to be did:pkh or DIDs. General specifications would not necessarily rely on did:pkh or DIDs.

All events are DAGJWS payloads encoded in IPLD using the [DAG-JOSE codec](https://ipld.io/specs/codecs/dag-jose/spec/).

### Genesis Event

A log is initialized with a genesis event. The CID of this event can be used to reference or name this log. 

```tsx
type GenesisHeader struct {
  controllers [String]
}
type GenesisEvent struct {
  header GenesisHeader
  data Any | &Any
}
```

Parameters defined as follows: 

- `controllers` - an array of DID strings that defines which DIDs can write events to the log, when using CACAO, the did is expected to be the issuer of the CACAO
- `data` - data is can be any type or a CID (Link) to data of any type

### Data Event

Log updates are data events. Data events are appended in the log to a genesis event or prior data events. 

```tsx
type DataHeader struct {
  controllers optional [String]
}

type DataEvent struct {
  id Link
  prev Link
  header optional DataHeader
  data Any | &Any
}
```

Additional parameters defined as follows, controllers and data defined same as above. 

- `id` -  CID (Link) to the genesis event of the log
- `prev` -  CID (Link) to prior event in log
- `header` -  Optional header, included here only if changing header parameter value (controllers) from prior event. Other header values may be included outside this specification.

### Log Verification

A valid log is one that includes data events as defined above and traversing the log resolves to an originating genesis event as defined above. Each event is valid when it includes the required parameters above and the DAGJWS signature is valid for the given event `controller` did or valid as defined by "CAIP-X DagJWS CACAO". There will likely be additional verification steps specific to any protocol or implementation. 

### Log Syncing

Logs are named or reference using their genesis events, but you need the latest data event to sync the entire log and resolve the path in IPLD from “latest” event to genesis event. The “latest” event can be reference as the log “tip”.  The “latest” or tip selection and sharing is left up to any higher level specification or protocol, as this mostly deals with the data structure. Logs can have multiple tips when there is forks in the log, “tip” selection becomes a consensus problem, to be determined by specific protocols or implementations building on this specification. 

### Log Extensions

This being a minimally define log on IPLD, later specifications or protocols can add additional parameters to both genesis and data events and their headers as needed. For example "CAIP-X IPLD Timestamp Proof" can be added as a data event to add time stamping to logs.

```tsx
type AnchorDataEvent struct {
  id Link
  prev Link
  header optional DataHeader
	proof Link
  path String
}
```

## Links
- [IPLD](https://ipld.io/)
- [IPLD schema language](https://ipld.io/docs/schemas/)
- [dag-jose](https://ipld.io/specs/codecs/dag-jose/spec/)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
