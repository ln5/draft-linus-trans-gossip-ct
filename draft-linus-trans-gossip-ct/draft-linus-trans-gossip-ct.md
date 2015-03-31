---
title: Gossiping in CT
docname: draft-linus-trans-gossip-ct-02-dev
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

--- abstract

This document describes two gossiping mechanisms for Certificate
Transparency {{!RFC6962}}; Feedback and STH gossip. In order for
HTTPS clients to share SCTs and STHs with CT auditors in a privacy-preserving
manner they send SCTs and STHs to originating HTTP servers which in turn share
them with CT auditors. CT auditors and monitors share STHs among
each other.

--- middle

# Introduction

Public append-only untrusted logs have to be monitored for
consistency, i.e. that they should never rewrite
history. Monitors and other log clients need to exchange information
about monitored logs in order to be able to detect a partitioning
attack.

A partitioning attack is when a log serves different views of the log
to different clients. Each client would be able to verify the
append-only nature of the log while in the extreme case being the
only client seeing this particular view.

Gossiping about what's known about logs helps solving the problem of
detecting malicious or compromised logs mounting such a partitioning
attack. We want some side of the partitioned tree, and ideally both
sides, to see the other side.

Disseminating known information about a log poses a potential threat
to the privacy of end users. Gossiping about data which is linkable to
a specific log entry and by that to a specific site has to take
privacy considerations into account in order not to leak sensitive
information.

However, there is no loss in privacy if a client sends SCTs for a
given site to the site named in the SCT, since the site's access logs
would already indicate that the client is accessing that site. In this
way a site can accumulate records of SCTs that have been issued by
various logs for that site, providing a consolidated repository of
SCTs which can be queried by auditors.

# Terminology

This document relies on terminology and data structures defined in
{{!RFC6962}}, including STH, SCT, Version, LogID, SCT timestamp,
CtExtensions, SCT signature, Merkle Tree Hash.

# Who should gossip {#who}

- HTTPS clients and servers (Feedback)
- HTTPS servers and CT auditors (Feedback)
- CT auditors and monitors (STH gossip)

# What to gossip about and how {#whathow}

There are two separate gossip streams:

- Feedback, transporting SCTs and STHs from clients to auditors
- STH gossip, sharing STHs between auditors/monitors

## Feedback

The goal of Feedback is for clients to share SCTs, certificate chains,
and STHs with CT auditors and monitors in a privacy-preserving manner.

HTTPS clients store SCTs, certificate chains, and STHs they see and later
send them to originating HTTPS servers by posting them to a
.well-known URL. This is described in {{feedback-clisrv}}. Note
that clients send the same SCTs, chains, and STHs to servers multiple times
with the assumption that a potential man-in-the-middle attack
eventually will cease so that an honest server will receive collected
malicious SCTs, chains, and STHs.

HTTPS servers store SCTs, certificate chains, and STHs received from clients
and later share them with CT auditors by either posting them or making
them available on a .well-known URL. This is described in
{{feedback-srvaud}}.

HTTPS clients MAY send SCTs, cert chains, and STHs directly to auditors. Note
that there are privacy implications of doing so, outlined in
{{privacy-SCT}}.

### HTTPS client to server {#feedback-clisrv}

An HTTPS client connects to an HTTPS server for a particular domain. The
client receives a set of SCTs as part of the TLS handshake. The
client MUST discard SCTs that are not signed by a known log and
SHOULD store the remaining SCTs together with the corresponding
certificate chain for later retrieval.

An HTTPS client may also acquire STHs by asking its supported logs for
the current STH directly, or via some other (currently unspecified)
mechanism.

When the client later reconnects to any HTTPS server for the same
domain it again receives a set of SCTs. The client MUST update its
store of SCTs for the domain and MUST send to the server the ones in
its store that were not received from that server.

Note that the SCT store also contains SCTs received in certificates.

The client MUST NOT send the same set of SCTs to the same server more
often than TBD.

An SCT MUST NOT be sent to any other HTTPS server than one serving the
domain that the certificate signed by the SCT refers to.

SCTs and corresponding certificates are POSTed to the originating
HTTPS server at the well-known URL:

  https://\<domain\>/.well-known/ct/v1/feedback

The data sent in the POST is defined in {{feedback-dataformat}}.

HTTPS servers perform a number of sanity checks on SCTs from clients
before storing them:

  1. if a bit-wise compare of the SCT matches one already in the
  store, the SCT MAY be discarded

  2. if the SCT can't be verified to be a valid SCT for the
  accompanying leaf cert, issued by a known log, the SCT MUST be
  discarded

  3. if the leaf cert is not for a domain that the server is
  authoritative for, the SCT MUST be discarded

Check number 1 is a pure optimisation. Check number 2 is to prevent
spamming and attacks where an adversary can fill up the store prior to
attacking a client. Check number 3 is to help misbehaving clients from
leaking what sites they visit.

### HTTPS server to auditors {#feedback-srvaud}

HTTPS servers receiving SCTs from clients SHOULD share SCTs and
certificate chains with CT auditors by either providing the well-known
URL:

  https://\<domain\>/.well-known/ct/v1/collected-feedback

or by HTTPS POSTing them to a number of preconfigured auditors.

The data received in a GET of the well-known URL or sent in the POST
is defined in {{feedback-dataformat}}.

HTTPS servers SHOULD share all SCTs, certificate data, and STHs they see
that pass the checks above, but MAY as an optimisation choose to not
share SCTs that the operator consider legitimate. An example of a
legitimate SCT might be one that was received from a CA as part of
acquisition of a certificate. Another example is an SCT received
directly from a CT log when submitting a certificate chain.

[TBD if a server chooses to not share an SCT that the operator
     considers legitimate, should they share the STH associated with
     that log in their collected-feedback?  if so, how would they do
     that, since in feedback STHs are associated with an SCT? ]

HTTPS servers MUST NOT share any other data that they may learn from
the submission of feedback by HTTP clients.

Auditors SHOULD provide the following URL accepting HTTPS POSTing of
SCT feedback data:

  https://\<auditor\>/ct/v1/feedback-gossip

Auditors SHOULD regularly poll HTTPS servers at the well-known
collected-feedback URL. How to determine which domains to poll is outside
the scope of this document but the selection MUST NOT be influenced by
potential HTTPS clients connecting directly to the auditor.

### Feedback data format {#feedback-dataformat}

The data shared between HTTPS clients and servers as well as between
HTTPS servers and CT auditors/monitors is a JSON object {{!RFC7159}}
with the following content:

- feedback: An array of objects consisting of

  - x509_chain: An array of base64-encoded X.509 certificates. The
    first element is the end-entity certificate, the second chains to
    the first and so on.

  - sct_data: An array of objects consisting of

    - sct_version: Version as defined in {{!RFC6962}} Section 3.2, as
      a number.

    - log_id: LogID as defined in {{!RFC6962}} Section 3.2, as a
      base64 encoded string.

    - timestamp: The SCT timestamp as defined in {{!RFC6962}}
      Section 3.2, as a number.

    - extensions -- CtExtensions as defined in {{!RFC6962}}
      Section 3.2, as a base64 encoded string.

    - signature -- The SCT signature as defined in {{!RFC6962}}
      Section 3.2, as a base64 encoded string.

    - sths -- an array of 0 or more recent STH objects recently
      collected from the log associated with log_id.  Each of these
      objects consists of

      - sth_version: Version as defined in {{!RFC6962}} Section 3.2,
        as a number. The version of the protocol to which the
        sth_gossip object conforms.

      - tree_size: The size of the tree, in entries, as a number.

      - timestamp: The timestamp of the STH as defined in {{!RFC6962}}
        Section 3.2, as a number.

      - sha256_root_hash: The Merkle Tree Hash of the tree as defined
        in {{!RFC6962}} Section 2.1, as a base64 encoded string.

      - tree_head_signature: A TreeHeadSignature as defined in
        {{!RFC6962}} Section 3.5 for the above data, as a base64
        encoded string.


The 'x509_chain' element MUST contain at least the leaf certificate
and SHOULD contain the full chain to a known root.

## STH gossip

The goal of gossiping about STHs is to detect logs that are
presenting different (inconsistent) views of the log to different
parties. CT auditors and monitors SHOULD gossip about Signed Tree
Heads (STHs) with as many other auditors and monitors as possible.

\[TBD gossip about inclusion proofs and consistency proofs too?\]

Which STHs to share and how often gossip should happen is regarded as
policy and out of scope for this document.

Auditors and monitors SHOULD provide the following URL accepting GET
requests returning STHs:

  https://\<auditor-or-monitor\>/ct/v1/sth-gossip

The data returned is a JSON object {{!RFC7159}} with the following
content:

- sth_gossip: An array of objects consisting of

  - sth_version: Version as defined in {{!RFC6962}} Section 3.2, as
    a number. It's the version of the protocol to which the
    sth_gossip object conforms.

  - tree_size: The size of the tree, in entries, as a number.

  - timestamp: The timestamp of the STH as defined in {{!RFC6962}}
    Section 3.2, as a number.

  - sha256_root_hash: The Merkle Tree Hash of the tree as defined in
    {{!RFC6962}} Section 2.1, as a base64 encoded string.

  - tree_head_signature: A TreeHeadSignature as defined in
    {{!RFC6962}} Section 3.5 for the above data, as a base64 encoded
    string.

  - log_id: LogID as defined in {{!RFC6962}} Section 3.2, as a base64
    encoded string.

# Security considerations

## Privacy considerations

The most sensitive relationships in the CT ecosystem are the
relationships between HTTPS clients and HTTPS servers. Client-server
relationships can be aggregated into a network graph with potentially
serious implications for correlative de-anonymisation of clients and
relationship-mapping or clustering of servers or of clients.

### Privacy and SCTs {#privacy-SCT}

SCTs contain information that typically links it to a particular web
site. Because the client-server relationship is sensitive, gossip
between clients and servers about unrelated SCTs is risky. Therefore,
a client with an SCT for a given server should transmit that
information in only two channels: to a server associated with the SCT
itself; and to a trusted CT auditor, if one exists.

### Privacy in feedback {#privacy-feedback}

HTTPS clients which allow users to clear history or cookies associated
with an origin MUST clear stored SCTs associated with the origin as
well.

Auditors should treat all SCTs as sensitive data. SCTs received
directly from an HTTPS client are especially sensitive, since the
auditor is a trusted by the client to not reveal their associations
with servers. Auditors MUST NOT share such SCTs in any way, including
sending them to an external log, without first mixing them with
multiple other SCTs learned through submissions from multiple other
clients. The details of mixing SCTs are TBD.

There is a possible fingerprinting attack where a log issues a unique
SCT for targeted log client(s).  A colluding log and HTTPS server
operator could therefore be a threat to the privacy of an HTTPS
client. Given all the other opportunities for HTTPS servers to
fingerprint clients -- TLS session tickets, HPKP and HSTS headers,
HTTP Cookies, etc. -- this is acceptable.

The fingerprinting attack described above could be avoided by
requiring that logs i) MUST return the same SCT for a given cert chain
({{!RFC6962}} Section 3) and ii) use a deterministic signature scheme
when signing the SCT ({{!RFC6962}} Section 2.1.4).

### Privacy for HTTPS clients requesting STHs

An HTTPS client that does not act as an auditor should only request an
STH from a CT log that it accepts SCTs from.  An HTTPS client should
regularly request an STH from all logs it is willing to accept, even
if it has seen no SCTs from that log.

### Privacy in STH gossip

An STH linked to an HTTPS client may indicate the following about that
client:

- that the client gossips;

- that the client been using CT at least until the time that the
  timestamp and the tree size indicate;

- that the client is talking, possibly indirectly, to the log
  indicated by the tree hash;

- which software and software version is being used.

There is a possible fingerprinting attack where a log issues a unique
STH for a targeted log auditor or HTTPS client. This is similar to the
fingerprinting attack described in {{privacy-feedback}}, but it is
mitigated by the following factors:

- the relationship between auditors and logs is not sensitive in the
  way that the relationship between clients and servers is;

- because auditors regularly exchange STHs with each other, the
  re-appearance of a targeted STH from some auditor does not imply
  that the auditor was the original one targeted by the log.

- An HTTPS client's relationship to a log is not sensitive in the way
  that its relationship to a server is.  As long as the client does
  not query the log for more than individual STHs, the client should
  not leak anything else to the log itself.  However, a collaborating
  log and server could use this technique to fingerprint a targeted
  HTTPS client.

Note that an HTTPS client in the configuration described in this
document doesn't make direct use of the STH itself.  Its fetching of
the STH and reporting via feedback provides a benefit to the CT
ecosystem as a whole by providing oversight on logs, but the HTTPS
client itself will not necessarily derive direct benefit.

# IANA considerations

TBD

# Contributors

The authors would like to thank Tom Ritter, Magnus Ahltorp and
Benjamin Kaduk for valuable contributions.

# ChangeLog

## Changes between -01 and -02

- add STHs to feedback mechanism

## Changes between -00 and -01

- Add the SCT feedback mechanism: Clients send SCTs to originating web
  server which shares them with auditors.
- Stop assuming that clients see STHs.
- Don't use HTTP headers but instead .well-known URL's -- avoid that
  battle.
- Stop referring to trans-gossip and trans-gossip-transport-https --
  too complicated.
- Remove all protocols but HTTPS in order to simplify -- let's come
  back and add more later.
- Add more reasoning about privacy.
- Do specify data formats.
