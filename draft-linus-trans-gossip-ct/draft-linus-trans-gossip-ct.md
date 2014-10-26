---
title: Gossiping in CT
docname: draft-linus-trans-gossip-ct-00
date: 2014-10-26
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

--- abstract

\[5-10 lines, max 20\]

--- middle

# Introduction

- separation between gossip protocol and strategy for gossiping
- defining the scope -- gossiping in CT has three pieces
    - general gossip protocol [draft-linus-trans-gossip]
    - gossip-ct -- what type of data and with whom to gossip
    - strategy/policy -- what data to gossip about and how to deal
      with incoming gossip
  - scope for this document is gossip-ct

# Problem

Gossiping in CT can help with solving two separate problems:

- detection of malicious logs showing different views to different
  clients, aka the partitioning attack

- how to spread information about illegitimate certificates, SCT's,
  STH's and proofs

# Who should gossip {#who}

- TLS clients using PKIX (i.e. web browsers, MTA:s, MUA:s, XMPP clients)
- CT auditors and CT monitors

# What to gossip {#what}

What kind of data should be gossiped about?

## Signed Tree Heads {#STH}

CT clients SHOULD send STH's to all servers willing to convey gossip.

Which STH's to send and how often is part of gossiping strategy and
out of scope for this document.

\[FIXME include consistency proofs too?\]

### Web browsers

Web browsers SHOULD send STH's to web servers they connect to.

Which web servers to send STH's to is determined by the web servers
capability and willingness to convey gossip. This is indicated by http
header \[FIXME\].

### CT monitors
SHOULD xxx

### MTA:s
SHOULD xxx

### MUA:s
SHOULD xxx

### XMPP clients
SHOULD xxx

## Invalid Signed Certificate Timestamps

- send invalid SCT's to _originating_ web server

# How to gossip {#how}

- pointer to [draft-linus-trans-gossip]

# Open questions

- active vs. passive participants

# Security and privacy considerations

- why STH's are ok
- why SCT's generally are bad for privacy
- pointer to [draft-linus-trans-gossip]

# IANA considerations
TBD

# Contributors
TBD
