---
title: Transparency Gossip HTTPS transport
docname: draft-linus-trans-gossip-transport-https-01
date: 2015-03-09
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
  RFC2119:
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
or {{!RFC2818}} using TLS version 1.2 or higher. When applicable the
server SHOULD be authenticated using X.509 certificates as described
in {{!RFC2459}} or by other means.

Gossip messages can contain data that are sensitive or not. This is
indicated in the messagee. An example of sensitive data is SCT's. An
example of non sensitive data is STH's.

HTTPS gossip messages are sent as HTTPS GET or POST requests.

# Message format and processing {#message}

Messages are strings of US-ASCII data on the following form:

    <protocol-version>:<log-id>:<gossip-data>

'protocol-version' is the version number of the protocol in
decimal. This version is 0.

'log-id' and 'gossip-data' are as defined in the GOSSIP-MSG of
{{draft-linus-trans-gossip}}. Note that 'gossip-data' is
base64-encoded.

Messages MUST be processed according to {{draft-linus-trans-gossip}}.

\[FIXME are there any http specific processing rules to be added?\]

# Security considerations

TBD

# IANA considerations

TBD

# Contributors

The author would like to thank Ben Laurie for their valuable
contributions.
