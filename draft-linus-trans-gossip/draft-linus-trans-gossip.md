---
title: Gossiping
docname: draft-linus-trans-gossip-00
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

--- abstract

\[5-10 lines, max 20\]

--- middle

# Introduction

- separation between general gossip protocol and specifics for
  different transparency systems

- defining the scope -- gossiping in any *T has two distinct pieces
    - general gossip protocol
      - message format
      - validation
      - transport
    - gossip for any specific *T system
      - which types of data
      - with whom
      - what data (strategy/policy)
      - how to act on gossip (strategy/policy)
  - scope for this document is general gossip protocol

# Problem

Gossiping can help to solve two separate problems:

- detection of malicious logs showing different views to different
  clients, aka the partitioning attack

- how to spread information about illegitimate log entry content, like
  certificates in the CT case

# Message format and processing {#message}

- message description

- validation of received message

# Transports



# Open questions

# Security and privacy considerations
- protection against bad actors
  - flooding attack
- gossiping being blocked

# IANA considerations
TBD

# Contributors
TBD
