---
title: Gossiping in CT
docname: draft-linus-trans-gossip-ct-01
date: 2015-03-09
category: exp
pi: [toc, sortrefs, symrefs]
ipr: trust200902
area: Security
wg: TRANS
kw: Internet-Draft

author:
  -
    ins: L. Nordberg
    name: Linus Nordberg
    email: linus@nordu.net
    org: NORDUnet
  -
    ins: D. Gillmor
    name: Daniel Kahn Gillmor
    email: dkg@fifthhorseman.net
    org: ACLU

normative:
  RFC2119:
  RFC6962:
  draft-linus-trans-gossip:
    title: Transparency Gossip
  draft-linus-trans-gossip-transport-https:
    title: Transparency Gossip HTTPS transport

--- abstract

This document describes gossiping in Certificate Transparency
{{!RFC6962}}. In order to share of SCT's in a privacy preserving
manner, web browsers send SCT's to originating web servers which share
SCT's with CT auditors and monitors. CT auditors and monitors share
STH's with each other.

--- middle

# Introduction

Gossiping in Certificate Transparency (CT) can be split up in three
pieces:

- A general gossip protocol. This document uses
  {{draft-linus-trans-gossip}} for a general gossip protocol.

- Gossip strategy and policy -- what data to gossip and how to deal
  with incoming gossip information.

- Gossiping rules, i.e. what type of data and with whom to gossip.

The scope for this document is the last point, the gossiping rules.

# Problem

Gossiping about what's known about CT logs helps solving the problem
of detecting malicious logs showing different views to different
clients, a.k.a. the partitioning attack.

The separate problem of how to disseminate information about a log
misbehaving in other ways may be helped by gossiping but poses a
potential threat to the privacy of end users. Gossiping about log data
linkable to a specific log entry and through that to a specific site
has to be constrained to using the gossiping message format and
gossiping transports for sending sensitive data only to particular
recipients.

# Who should gossip {#who}

- Web browsers
- Web servers
- CT auditors and monitors

# What kind of data to gossip about {#what}

There are two separate gossip streams

- SCT feedback, transporting SCT's from clients to auditors/monitors
- STH gossip, sharing STH's between auditors/monitors

## SCT feedback

The goal of SCT feedback is for clients to share SCT's and certificate
chains with CT auditors and monitors in a privacy preserving manner.

Web browsers SHOULD store and later send some of the SCT's they see to
some particular web servers by posting them, together with their
associated certificate chains, to a .well-known URL.

Web servers SHOULD share SCT's with CT auditors and monitors by either
posting them or making them available on a .well-known URL.

Web browsers SHOULD NOT send SCT's and cert chains directly to
auditors/monitors because of the privacy implications of doing so.

### Browser to web server

A browser (B) connects to a web server (S) serving domain (D). (B)
receives a set of SCT's as part of the TLS handshake. (B) SHOULD
discard SCT's that are not signed by a known log. (B) SHOULD store the
remaining SCT's together with their corresponding certificate chains
on disk.

When (B) later reconnects to a web server (S') serving domain (D) it
again receives a set of SCT's. (B) MUST update its store of SCT's for
(D) and SHOULD send to (S') the ones in the store that were not
received from (S').

Note that the SCT store also contains SCT's received in certificates.

An SCT MUST NOT be sent to any other web server than one serving the
domain that the certificate signed by the SCT refers to.

SCT's and corresponding certificates are POSTed to the originating web
server at the well-known URL

  example.com/.well-known/sct-feedback

The data is a JSON object {{!RFC4627}} with the following content:

- sct_feedback: An array of objects consisting of

  - x509_chain: An array of base64-encoded X.509 certificates. The
    first element is the end-entity certificate, the second chains to
    the first and so on.

  - sct_version -- Version as defined in {{!RFC6962}} section 3.2, in
    decimal

  - log_id -- LogID as defined in {{!RFC6962}} section 3.2, base64
    encoded

  - timestamp -- the SCT timestamp, in decimal

  - extensions -- CtExtensions as defined in {{!RFC6962}} section 3.2,
    base64 encoded

  - signature -- the SCT signature, base64 encoded

The 'x509_chain' element MUST contain at least the leaf certificate
and SHOULD contain the full chain to a known root.


### Web server to auditors/monitors




Web servers MUST NOT share any other data on the well known URL.


## STH gossip

CT auditors and monitors SHOULD gossip about Signed Tree Heads (STH's)
with as many other auditors and monitors as possible.

Gossiping about STH's enables detection of logs presenting more than
one view of the log.

An STH contains:
- the size of the tree being signed
- a timestamp indicating the time when the tree was signed
- the merkle tree hash of the tree being signed
- a signature made by the log

An STH received from a client may indicate the following about that
client:
- gossiping
- using CT, as late as the timestamp and tree size indicate
- talking, indirectly, to the log indicated by the tree hash
- software being used and software version

Which STH's to send and how often is part of gossiping strategy and
out of scope for this document.

\[TBD gossip about inclusion proofs and consistency proofs too?\]

STH's are sent to a preconfigured gossip service in a
{{draft-linus-trans-gossip}} GOSSIP-MSG message with 'gossip-data' as
a JSON object {{!RFC7159}} with the following content:

- sths: array of {{!RFC6962}} Signed Tree Head's

# Security considerations

- TODO expand on why gossiping STH's is ok

- TODO expand on why gossiping SCT's is bad for privacy in the general
  case

# Open questions

- TODO active vs. passive participants

# IANA considerations

TBD

# Contributors

The authors would like to thank Tom Ritter for valuable discussions.
