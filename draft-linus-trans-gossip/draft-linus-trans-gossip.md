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

This document describes a gossip service. Gossip peers connect to a
gossip service and start sending and receiving gossip messages. Gossip
transports register with a gossip service and transfer messages
between other gossip services and local peers.

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

### GOSSIP-MSG

- protocol-version = "gossip-msg-0"

- timestamp = 64bit integer

NTP time of message receival by gossip service, milliseconds since the
epoch

- direction = "outgoing" \| "incoming"

outgoing messages go from a peer to one or more registered transports

incoming messages go from a transport to all connected peers

- destination = transports \| "all" \| ""

transports = transport [transports]

an incoming message has destination = ""

destination denotes which transport(s) to send an outgoing message
over

"all" means that the message should be sent to all registered
transports

- source = transport \| ""

an incoming message has source set to the transport it came in over

an outgoing message has source = ""

transport = a string containing the name of a registered transport

- log-id = SHA-256 hash of a log's public key, 32 octets

the log id of the transparency log gossiped about

- datum = opaque blob

datum contains the payload of the message. Its contents depend on what
kind of data is being gossiped.

## Validation of received messages {#validation}

Peers MUST validate received gossip messages by verifying the
GOSSIP-MSG 'datum' according to the rules of the log indicated by
'log-id'. Messages that don't check out correctly MUST be discarded.

Transports MUST validate gossip messages received over its external
protocol, i.e. not received from the gossip service.

\[FIXME: Include this? Transports SHOULD validate gossip messages
received from the gossip service if it's serving a gossip service
allowing untrusted peers.\]

Peers MAY respond to an incoming message by sending one or more
messages back to the transport it was received from.

# Transports

A gossip transport is responsible for sending and receiving gossip
messages over a specific protocol, like HTTP or XMPP.

A transport registers with a gossip service, either by being compiled
into the service, being dynamically loaded into it or by connecting to
it over a socket endpoint, local or remote. In the case of a
connecting transport, the transport has to authenticate with the
service by sending it a shared secret in an AUTHENTICATE-REQUEST
message.

## Defined transports

- gossip-transport-https = "gossip-transport-https"

## Transport protocol

### AUTHENTICATE-REQUEST

- protocol-version = "gossip-authenticate-request-0"

- shared-secret = string

### AUTHENTICATE-RESPONSE

- protocol-version = "gossip-authenticate-response-0"

- response = "ok" \| "bad-auth"

# Open questions

- replace strings in the protocol with registered values?

# Examples

This section describes how a gossiping system could be implemented for
CT.

## CT clients

        web-browser   ct-monitor              <-- gossip peer
               \       /
            gossip-daemon                     <-- gossip service
               /       \
     socket-transport   https-transport       <-- gossip transport
             |            |
       web-browser[0]   tls-lib

\[0\] same instance as the gossip peer "web-browser"

              web-server                      <-- gossip peer
                  |
            gossip-daemon                     <-- gossip service
                  | 
            socket-transport                  <-- gossip transport
                  |
             web-server[1]

\[1\] same instance as the gossip peer "web-server"

The daemon listens to two separate sockets, one for peers and one for
transports.

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
- gossiping messages being blocked
- transports authenticating with gossip services

# IANA considerations

TBD

# Contributors

- benl -- the daemon idea
