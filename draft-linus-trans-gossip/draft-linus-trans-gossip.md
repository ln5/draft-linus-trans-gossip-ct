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

This document describes a gossip service. Gossip peers connect to it
and start sending and receiving gossip messages. Gossip transports
register with it and transfer messages to other gossip services and
peers.

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

- terminology
  - gossip peer
  - gossip service
  - gossip transport

# Problem

Gossiping can help solving two separate problems:

First, detection of malicious logs showing different views to
different clients, a.k.a. the partitioning attack.

Second, how to spread information about illegitimate log entry content
or other log data. In CT this would be X.509 certificates, SCT's,
STH's and proofs.

# Message format and processing

## Message description {#message}

### gossip-msg

- protocol-version = v0

v0 = 0

- timestamp = 64bit integer

time of arrival to gossip service, milliseconds since the epoch

- direction = "outgoing" \| "incoming"

- destination = transports \| "all" \| ""

transports = transport [transports]

which transports to send an outgoing message over

"all" means that the message should be sent to all registered
transports

an incoming message has destination = ""

- source = transport \| ""

an incoming message has source set to the transport it came in over

an outgoing message has source = ""

- transport = a string containing the name of a registered transport

- log-id = SHA-256 hash of a log's public key, 32 octets

the log id of the transparency log gossiped about

- datum = opaque blob containing the payload of the message

## Validation of received messages {#validation}

# Transports

# Open questions

- replace strings in the protocol with registered values

# Examples

This section describes how a gossiping system could be implemented for
CT.

        web-browser   ct-monitor              <-- gossip peer
               \       /
            gossip-daemon                     <-- gossip service
               /       \
     socket-tansport   http-transport         <-- gossip transport
             |            |
       web-browser[0]   tls-lib

The daemon listens to two separate sockets, one for peers and one
for transports.

Peers (top row) connect to the daemon client socket and send and
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
(think IMAP) to messages would make it possible for peers to match
sent and received messages, making an echo transport useful for
programs like web browsers which already have the transport ready for
use.)

# Security and privacy considerations

- protection against bad actors
  - flooding attack
- gossiping being blocked

# IANA considerations

TBD

# Contributors

TBD
