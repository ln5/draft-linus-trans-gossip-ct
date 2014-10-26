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

Gossiping can help solving two separate problems:

First, detection of malicious logs showing different views to
different clients, a.k.a. the partitioning attack.

Second, how to spread information about illegitimate log entry content
or other log data. In CT this would be X.509 certificates, SCT's,
STH's and proofs.

# Informal description of the proposed gossiping system

This section describes how a gossiping system could be implemented for
CT.

        web-browser   ct-monitor              <-- gossip client
               \       /
            gossip-daemon
               /       \
     socket-tansport   http-transport         <-- gossip transport
             |            |
       web-browser[0]   tls-lib

The daemon listens to two separate sockets, one for clients and one
for transports.

Clients (top row) connect to the daemon client socket and send and
receive gossip messages. A message contains one or more transport ids
and gossip data. The transport ids in outgoing messages are used to
select which transport(s) the gossip data is allowed to be sent
over. Transport ids in incoming messages reflect which transport they
were received over.

Transports are either built into the daemon or registered with the
daemon by connecting to the daemon transport socket and identifying
itself with a string (the transport id). This registering process will
require authentication.

(A less complex alternative to such a registering scheme would be to
have built-in transports only, one of which is "echo". Adding a tag
(think IMAP) to messages would make it possible for clients to match
sent and received messages, making an echo transport useful for
programs like web browsers which already have the transport ready for
use.)

# Message format and processing

## Message description {#message}

## Validation of received messages {#validation}

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
