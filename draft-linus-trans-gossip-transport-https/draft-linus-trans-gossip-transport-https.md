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
  ins: L. Nordberg
  name: Linus Nordberg
  email: linus@nordu.net
  org: NORDUnet

normative:
  RFC822:
  RFC2246:
  RFC4346:
  RFC5246:
  draft-linus-trans-gossip:
    title: Transparency Gossip

--- abstract

This document specifies a {{draft-linus-trans-gossip}} transport
protocol for sending Transparency Gossip messages over https.

--- middle

# Introduction

Using web servers as "gossip pools" is expected to be helpful for
transparency gossiping, especially for {{?RFC6962}}.

Web browsers can act as an HTTPS transport, sending and receiving
gossip messages to web servers it connects to for other reasons than
gossiping.

HTTPS transports that don't have connections to web servers for other
reasons than gossiping may connect to web servers known to support
gossiping. They can be known by configuration or by other
mechanisms. This document does not specify such mechanisms.

# Sending and receiving

Gossip messages may contain sensitive information and MUST NOT be sent
over connections which are not encrypted as described in {{!RFC2817}}
or {{!RFC2818}} using TLS version 1.0 or higher. When applicable the
server SHOULD be authenticated using X.509 certificates as described
in {{!RFC2459}} or by other means.

HTTPS gossip messages are sent in {{!RFC2616}} message headers with
the field-name "TransGossip".

An HTTPS transport

- SHOULD send gossip messages to HTTP servers that have indicated that
  they accept gossip by sending an HTTP response-header
  "TransGossipEnabled" with the value "Yes"

- MAY send gossip messages to HTTP servers that haven't indicated
  willingness to accept gossip

- MUST NOT send gossip messages to HTTP servers that have indicated
  that they don't accept gossip by sending an HTTP response-header
  "TransGossipEnabled" with the value "No"

# Message format and processing {#message}

Messages are ASCII strings on the following form:

    <protocol-version>:<log-id>:<gossip-data>

'protocol-version' is the version number of the protocol in
decimal. This version is 0.

'log-id' and 'gossip-data' are as defined in the GOSSIP-MSG of
{{draft-linus-trans-gossip}}. Note that 'gossip-data' is Base64
encoded.

# Security and privacy considerations

TBD

# Open questions

TBD

# IANA considerations

TBD

# Contributors

The author would like to thank Ben Laurie for their valuable
contributions.
