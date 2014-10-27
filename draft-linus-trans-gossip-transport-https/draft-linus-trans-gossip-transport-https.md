---
title: Transparency Gossip HTTPS transport
docname: draft-linus-trans-gossip-transport-https-00
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
  RFC822:
  RFC2616:
  draft-linus-trans-gossip:
    title: Transparency Gossip

--- abstract

TBD \[5-10 lines, max 20\]

--- middle

# Introduction

This document specifies a {{draft-linus-trans-gossip}} transport
protocol for sending gossip messages over https.

- web server willingness to gossip is indicated by http header TBD

- gossip messages

# Problem

TBD

# Sending and receiving

Gossip messages MUST NOT be sent over a connection which is not
encrypted as described in {{!RFC2818}} using TLS version 1.0 or higher
({{!RFC2246}}, {{!RFC4346}}, {{!RFC5246}}.

# Message format and processing {#message}

HTTPS gossip messages MUST be sent in {{RFC2616}} message headers with
the field-name \[TBD\].

- request-header (section 5.3)
- response-header (section 6.2)

Messages are ASCII strings on the following format:

    <timestamp>:<log-id>:<gossip-data>

'timestamp', 'log-id' and 'gossip-data' are as defined in a GOSSIP-MSG
of {{draft-linus-trans-gossip}}.

Note that the timestamp indicates when the message was received by the
sending part. A transport sending a message received from a gossip
service MUST use the timestamp from the GOSSIP-MSG. A transport
sending a message constructed from another source MUST use its best
approximation for when the source of the message was received.

# Security and privacy considerations

# Open questions

# IANA considerations

TBD

# Contributors

TBD
