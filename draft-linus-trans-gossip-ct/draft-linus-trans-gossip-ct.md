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

--- abstract

This document describes gossiping in Certificate Transparency
{{!RFC6962}}. In order to share SCT's in a privacy preserving manner,
HTTP clients send SCT's to originating HTTP servers which in turn
share SCT's with CT auditors. CT auditors and monitors share STH's
with each other.

--- middle

# Introduction

TBD

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

- HTTP clients and servers (SCT feedback)
- HTTP servers and CT auditors (SCT feedback)
- CT auditors and monitors among themselves (STH gossip)

# What kind of data to gossip about {#what}

There are two separate gossip streams

- SCT feedback, transporting SCT's from clients to auditors
- STH gossip, sharing STH's between auditors/monitors

## SCT feedback

The goal of SCT feedback is for clients to share SCT's and certificate
chains with CT auditors and monitors in a privacy preserving manner.

HTTP clients SHOULD store and later send some of the SCT's they see to
some particular HTTP servers by posting them, together with their
associated certificate chains, to a .well-known URL.

HTTP servers SHOULD store and later share SCT's with CT auditors by
either posting them or making them available on a .well-known URL.

HTTP clients MAY send SCT's and cert chains directly to auditors. It
should be noted that there are major privacy implications of doing so.

### HTTP client to server

A browser (B) connects to an HTTP server (S) serving domain (D). (B)
receives a set of SCT's as part of the TLS handshake. (B) SHOULD
discard SCT's that are not signed by a known log. (B) SHOULD store the
remaining SCT's together with their corresponding certificate chains
on disk.

When (B) later reconnects to an HTTP server (S') serving domain (D) it
again receives a set of SCT's. (B) MUST update its store of SCT's for
(D) and SHOULD send to (S') the ones in the store that were not
received from (S').

Note that the SCT store also contains SCT's received in certificates.

An SCT MUST NOT be sent to any other HTTP server than one serving the
domain that the certificate signed by the SCT refers to.

SCT's and corresponding certificates are POSTed to the originating
HTTP server at the well-known URL

  https://<domain>/.well-known/ct/v1/sct-feedback

The data sent in the POST is defined in {{#dataformat}}.

HTTP servers SHOULD perform a number of sanity checks on SCT's from
clients before storing them:

  1. if a bit-wise compare of the SCT matches one already in the
  store, discard

  2. if the SCT can't be verified to be a valid SCT for the
  accompanying leaf cert, issued by a known log, discard

  3. if the leaf cert is not for a domain that the server is
  authoritative for, discard

Check number 1 is a pure optimisation. Check number 2 is to prevent
spamming and attacks where an adversary can fill up the store prior to
attacking a client. Check number 3 is to help misbehaving clients from
leaking what sites they visit.

### HTTP server to auditors

HTTP servers that are receiving SCT's from clients SHOULD share SCT's
and certificate chains with CT auditors by either providing the
.well-known URL

  https://<domain>/.well-known/ct/v1/sct-feedback

or by POSTing them to a number of preconfigured auditors.

The data received in a GET of the well-known URL or sent in the POST
is defined in {{#dataformat}}.

HTTP servers SHOULD share all SCT's and certificate data but MAY as an
optimisation chose to not send SCT's that the operator consider
legitimate. An example of a legitimate SCT might be one that was
received from a CA as part of acquisition of a certificate. Another
example is an SCT received directly from a CT log.

HTTP servers MUST NOT share any other data or meta data learned from
the submission of SCT's from clients.

CT auditors SHOULD provide the following URL accepting POSTing of SCT
feedback data:

  https://<<auditor>/ct/v1/sct-feedback

CT auditors SHOULD regularly poll HTTP servers on the internet at the
well-known sct-feedback URL. How to determine which domains to poll is
outside the scope of this document but it MUST NOT be influenced by
potential HTTP clients connecting directly to the auditor.

### SCT feedback data format {#dataformat}

The data shared between HTTP clients and servers as well as between
HTTP servers and CT auditors/monitors is a JSON object {{!RFC7159}}
with the following content:

- sct_feedback: An array of objects consisting of

  - x509_chain: An array of base64-encoded X.509 certificates. The
    first element is the end-entity certificate, the second chains to
    the first and so on.

  - sct_version -- Version as defined in {{!RFC6962}} section 3.2, in
    decimal.

  - log_id -- LogID as defined in {{!RFC6962}} section 3.2, base64
    encoded.

  - timestamp -- The SCT timestamp, in decimal.

  - extensions -- CtExtensions as defined in {{!RFC6962}} section 3.2,
    base64 encoded.

  - signature -- The SCT signature, base64 encoded.

The 'x509_chain' element MUST contain at least the leaf certificate
and SHOULD contain the full chain to a known root.

## STH gossip

The goal of gossiping about STH's is to detect logs that are
presenting more than one view of the log.

CT auditors and monitors SHOULD gossip about Signed Tree Heads (STH's)
with as many other auditors and monitors as possible. 

Which STH's to send and how often is part of gossiping strategy and
out of scope for this document.

\[TBD gossip about inclusion proofs and consistency proofs too?\]

STH's are sent to FIXME as a JSON object {{!RFC7159}} with the
following content:

- sth-gossip: An array of consisting of

  - sth_version -- Version as defined in {{!RFC6962}} section 3.2,
    base64 encoded. It's the version of the protocol to which the
    signature conforms.

  - tree_size: The size of the tree, in entries, in decimal.

  - timestamp: The timestamp, in decimal.

  - sha256_root_hash: The Merkle Tree Hash of the tree, in base64.

  - tree_head_signature: A TreeHeadSignature as defined in
    {{!RFC6962}} section 3.5 for the above data.

# Security considerations

- TODO expand on why gossiping STH's is ok

- TODO expand on why gossiping SCT's is bad for privacy in the general
  case

An STH contains:
- the size of the tree being signed
- a timestamp indicating the time when the tree was signed
- the merkle tree hash of the tree being signed
- a signature made by the log

An STH linked to a client may indicate the following about that
client:
- it's gossiping
- it's been using CT at least until the timestamp and tree size
  indicate
- it's talking, indirectly, to the log indicated by the tree hash
- which software and software version is being used

# Open questions

- TODO active vs. passive participants

# IANA considerations

TBD

# Contributors

The authors would like to thank Tom Ritter for valuable discussions.
