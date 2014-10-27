---
title: Gossiping in CT
docname: draft-linus-trans-gossip-ct-00
date: 2014-10-27
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
    ins: N. N
    name: Your Name Here
    email: n @ domain
    org: org
  -
    ins: N. N
    name: Your Name Here
    email: n @ domain
    org: org

normative:
  RFC6962:
  draft-linus-trans-gossip:
    title: Transparency Gossip
  draft-linus-trans-gossip-transport-https:
    title: Transparency Gossip HTTPS transport

--- abstract

This document describes gossiping in Certificate Transparency.

\[FIXME: more -- 5-10 lines, max 20\]

--- middle

# Introduction

- separation between gossip protocol and strategy for gossiping
- defining the scope -- gossiping in CT has three pieces
    - general gossip protocol {{draft-linus-trans-gossip}}
    - gossip-ct -- what type of data and with whom to gossip (this draft)
    - strategy/policy -- what data to gossip about and how to deal
      with incoming gossip
  - scope for this document is gossip-ct

# Problem

Gossiping can help CT solving two separate problems.

1. Detection of malicious logs showing different views to different
   clients, a.k.a. the partitioning attack.

2. Dissemination of information about illegitimate STH's and SCT's
   (and corresponding X.509) \[FIXME: define illegitimate\]

# Who should gossip {#who}

- TLS clients using PKIX (i.e. web browsers, MTA:s, MUA:s, XMPP
  clients)
- CT auditors and CT monitors

# What to gossip {#what}

This section describes what log data to gossip.

## Signed Tree Heads {#STH}

All CT clients SHOULD gossip about Signed Tree Heads (STH's) with as
many other CT clients as possible.

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

\[FIXME include consistency proofs too?\]

### Web browsers

Web browsers SHOULD send STH's to web servers using Transparency
Gossiping {{draft-linus-trans-gossip}} by sending GOSSIP-MSG messages
to a gossip service. Web browsers SHOULD use the
{{draft-linus-trans-gossip-transport-https}} transport and MAY use
other transports as well.

Which web servers to send STH's to is determined by which web servers
are connected to by the transport and those web servers capability and
willingness to convey gossip. This is handled by the gossip transport.

Web browsers MAY register as a gosisp transports themselves and
perform the sending and receiving of gossip mesages.

### CT monitors

CT monitors SHOULD send STH's to web servers using Transparency
Gossiping {{draft-linus-trans-gossip}} by sending GOSSIP-MSG messages
to a gossip service.

CT monitors SHOULD use as many transports as possible.

### MTA:s

TBD

### MUA:s

TBD

### XMPP clients

TBD

## Invalid Signed Certificate Timestamps

TLS clients MAY send illegitimate Signed Certficate Timestamps (SCT's)
with corresponding X.509 certificate to the server it got the SCT
from. The {{draft-linus-trans-gossip-transport-https}} messaging
format SHOULD be used for this.

# Security and privacy considerations

- TODO why STH's are ok
- TODO why SCT's are bad for privacy in the general case

# Open questions

- TODO active vs. passive participants

# IANA considerations
TBD

# Contributors
TBD
