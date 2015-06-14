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

This document describes three gossiping mechanisms for Certificate
Transparency {{!RFC6962}}; SCT Feedback, STH Pollination and Trusted
Auditor Relationship.

SCT Feedback enables HTTPS clients to share SCTs with CT auditors in a
privacy-preserving manner by sending SCTs to originating HTTPS servers
which in turn share them with CT auditors.

In STH Pollination, HTTPS clients use HTTPS servers as STH pools
sharing STHs with connecting clients in the hope that STHs will find
their way to auditors and monitors.

HTTPS clients in a Trusted Auditor Relationship share SCTs and STHs
with trusted auditor or monitors directly with expectations of privacy
sensitive data being handled according to whatever privacy policy
agreed on between client and trusted party.

--- middle

# Introduction

Public append-only untrusted logs have to be monitored for
consistency, i.e. that they should never rewrite history. Monitors and
other log clients need to exchange information about monitored logs in
order to be able to detect a partitioning attack.

A partitioning attack is when a log serves different views of the log
to different clients. Each client would be able to verify the
append-only nature of the log while in the extreme case being the only
client seeing this particular view.

Gossiping about what's known about logs helps solving the problem of
detecting malicious or compromised logs mounting such a partitioning
attack. We want some side of the partitioned tree, and ideally both
sides, to see the other side.

Disseminating known information about a log poses a potential threat
to the privacy of end users. Gossiping about data which is linkable to
a specific log entry and by that to a specific site has to take
privacy considerations into account in order not to leak sensitive
information. Sharing STHs can be problematic even if they don't link
to a specific log entry -- tracking by fingerprinting through rare
STHs is one potential attack.

However, there is no loss in privacy if a client sends SCTs for a
given site to the site named in the SCT, since the site's access logs
would already indicate that the client is accessing that site. In this
way a site can accumulate records of SCTs that have been issued by
various logs for that site, providing a consolidated repository of
SCTs which can be queried by auditors.

Sharing an STH is considered reasonably safe from a privacy
perspective as long as the same STH is shared by a large number of
other clients. This is solved by requiring a certain freshness for
STHs in order to be accepted and limiting the log STH issuing
frequency.

# Terminology and overview

This document relies on terminology and data structures defined in
{{!RFC6962}}, including STH, SCT, Version, LogID, SCT timestamp,
CtExtensions, SCT signature, Merkle Tree Hash.

~~~~
   +- Cert ---- +----------+
   |            |    CA    | ----------+
   |   + SCT -> +----------+           |
   v   |                          Cert & SCT
+----------+                           |
|   Log    |                           |
+----------+                           v
  |  ^                          +----------+
  |  |          SCT & cert ---- | Website  |
  |  |[1]           |           +----------+
  |  |[2]          STH            ^     |
  |  |[3]           v             |     |
  |  |          +----------+      |     |
  |  +--------> | Auditor  |      |  HTTPS traffic
  |             +----------+      |     |
  |             /    ^            |     |
  |            /      \     SCT & cert  |
Log entries   /        \          |     |
  |          /      (Trusted     STH   STH
  v         /[4]      Auditor     |     |
+----------+           Stream)    |     v
| Monitor  |               \    +----------+
+----------+                +---| Browser  |
                                +----------+

*   Auditor                        Log
[1] |--- get-sth ------------------->|
    |<-- STH ------------------------|
[2] |--- leaf hash + tree size ----->|
    |<-- index + inclusion proof --->|
[3] |--- tree size 1 + tree size 2 ->|
    |<-- consistency proof ----------|
[4] SCT, cert and STH among multiple Auditors and Monitors
~~~~

# Who should gossip {#who}

- HTTPS clients and servers (SCT Feedback and STH Pollination)
- HTTPS servers and CT auditors (SCT Feedback)
- HTTPS clients and CT auditors (Trusted Auditor Relationship)
- CT auditors and monitors (Trusted Auditor Relationship)

# What to gossip about and how {#whathow}

There are three separate gossip streams:

- SCT Feedback, transporting SCTs and certificate chains from HTTPS
  clients to CT auditors/monitors via HTTPS servers.

- STH Pollination, HTTPS clients and CT auditors/monitors using HTTPS
  servers as STH pools for exchanging STHs.

- Trusted Auditor Stream, HTTPS clients communicating directly with
  trusted CT auditors/monitors sharing SCTs, certificate chains and
  STHs.

## SCT Feedback

The goal of SCT Feedback is for clients to share SCTs and certificate
chains with CT auditors and monitors in a privacy-preserving manner.

HTTPS clients store SCTs and certificate chains they see and later
send them to originating HTTPS servers by posting them to a
.well-known URL. This is described in {{feedback-clisrv}}. Note that
clients send the same SCTs and chains to servers multiple times with
the assumption that a potential man-in-the-middle attack eventually
will cease so that an honest server will receive collected malicious
SCTs and chains.

HTTPS servers store SCTs and certificate chains received from clients
and later share them with CT auditors by either posting them or making
them available on a .well-known URL. This is described in
{{feedback-srvaud}}.

### HTTPS client to server {#feedback-clisrv}

An HTTPS client connects to an HTTPS server for a particular
domain. The client receives a set of SCTs as part of the TLS
handshake. The client MUST discard SCTs that are not signed by a known
log and SHOULD store the remaining SCTs together with the
corresponding certificate chain for later retrieval.

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

    https://\<domain\>/.well-known/ct/v1/sct-feedback

The data sent in the POST is defined in {{feedback-dataformat}}.

HTTPS servers perform a number of sanity checks on SCTs from clients
before storing them:

  1. if a bit-wise compare of the SCT matches one already in the
  store, the SCT MAY be discarded

  1. if the SCT can't be verified to be a valid SCT for the
  accompanying leaf cert, issued by a known log, the SCT MUST be
  discarded

  1. if the leaf cert is not for a domain that the server is
  authoritative for, the SCT MUST be discarded

Check number 1 is a pure optimisation. Check number 2 is to prevent
spamming and attacks where an adversary can fill up the store prior to
attacking a client. Check number 3 is to help misbehaving clients from
leaking what sites they visit.

### HTTPS server to auditors {#feedback-srvaud}

HTTPS servers receiving SCTs from clients SHOULD share SCTs and
certificate chains with CT auditors by either providing the well-known
URL:

    https://\<domain\>/.well-known/ct/v1/collected-sct-feedback

or by HTTPS POSTing them to a number of preconfigured auditors.

The data received in a GET of the well-known URL or sent in the POST
is defined in {{feedback-dataformat}}.

HTTPS servers SHOULD share all SCTs and certificate data they see that
pass the checks above, but MAY as an optimisation choose to not share
SCTs that the operator considers legitimate. An example of a
legitimate SCT might be one that was received from a CA as part of
acquisition of a certificate. Another example is an SCT received
directly from a CT log when submitting a certificate chain.

HTTPS servers MUST NOT share any other data that they may learn from
the submission of feedback by HTTP clients.

Auditors SHOULD provide the following URL accepting HTTPS POSTing of
SCT feedback data:

    https://\<auditor\>/ct/v1/sct-feedback

Auditors SHOULD regularly poll HTTPS servers at the well-known
collected-sct-feedback URL. How to determine which domains to poll is
outside the scope of this document but the selection MUST NOT be
influenced by potential HTTPS clients connecting directly to the
auditor.

### SCT Feedback data format {#feedback-dataformat}

The data shared between HTTPS clients and servers as well as between
HTTPS servers and CT auditors/monitors is a JSON object {{!RFC7159}}
with the following content:

- sct_feedback: An array of objects consisting of

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

The 'x509_chain' element MUST contain at least the leaf certificate
and SHOULD contain the full chain to a known root.

## STH pollination

The goal of sharing Signed Tree Heads (STHs) through pollination is to
share STHs between HTTPS clients and CT auditors and monitors in a
privacy-preserving manner.

HTTPS servers supporting the protocol act as STH pools. HTTPS clients
and others in the possesion of STHs should pollinate STH pools by
sending STHs to them. CT auditors and monitors should retrieve STHs
from pools by downloading STHs from them.

STH Pollination is carried out by sending STHs to HTTPS servers
supporting the protocol. In the case of HTTPS clients, STHs are sent
in an already established TLS session. This makes it hard for an
attacker to disrupt STH gossiping without also disturbing ordinary
secure browsing (https://).

STHs are sent by POSTing them at the .well-known URL:

    https://\<domain\>/.well-known/ct/v1/sth-pollination

The data sent in the POST is defined in {{sth-pollination-dataformat}}.

The response contains zero or more STHs in the same format, described
in {{sth-pollination-dataformat}}.

An HTTPS client may acquire STHs by several methods:

- in replies to pollination POSTs;
- asking its supported logs for the current STH directly;
- via some other (currently unspecified) mechanism.

HTTPS clients which have STHs and CT auditors and monitors SHOULD
pollinate STH pools with STHs. Which STHs to send and how often
pollination should happen is regarded as policy and out of scope for
this document.

In order to avoid being tracked by an attacker controlling an STH
pool, HTTPS clients silently ignore STHs which are not fresh. An STH
is considered fresh iff its timestamp is less than 14 days in the
past. Given a maximum STH issuance rate of one per hour, an attacker
has 336 unique STHs per log for tracking. Even when multiplied by the
number of logs that a client accepts STHs for, this number of unique
STHs seems to stay small enough for the risk of a successful
fingerprinting attack through STHs to be acceptable.

\[TBD urge HTTPS clients to store STHs retrieved in responses?\]

\[TBD share inclusion proofs and consistency proofs too?\]


### STH Pollination data format {#sth-pollination-dataformat}

The data sent from HTTPS clients and CT monitors and auditors to HTTPS
servers is a JSON object {{!RFC7159}} with the following content:

- sths -- an array of 0 or more fresh STH objects
  \[XXX recently collected\] from the log associated with log_id. Each
  of these objects consists of

  - sth_version: Version as defined in {{!RFC6962}} Section 3.2, as a
    number. The version of the protocol to which the sth_gossip object
    conforms.

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

\[XXX An STH is considered recently collected iff TBD.\]

## Trusted Auditor Stream

HTTPS clients MAY send SCTs and cert chains as well as STHs directly
to auditors. Note that there are privacy implications of doing so,
outlined in {{privacy-SCT}} and {{privacy-trusted-auditors}}.

### Trusted Auditor data format

\[TBD specify something here or leave this for others?\]


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

### Privacy in SCT Feedback {#privacy-feedback}

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
SCT for targeted log client(s). A colluding log and HTTPS server
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
STH from a CT log that it accepts SCTs from. An HTTPS client should
regularly request an STH from all logs it is willing to accept, even
if it has seen no SCTs from that log.

### Privacy in STH Pollination

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
  way that the relationship between HTTPS clients and HTTPS servers
  is;

- because auditors regularly exchange STHs with each other, the
  re-appearance of a targeted STH from some auditor does not imply
  that the auditor was the original one targeted by the log;

- an HTTPS client's relationship to a log is not sensitive in the way
  that its relationship to an HTTPS server is. As long as the client
  does not query the log for more than individual STHs, the client
  should not leak anything else to the log itself. However, a log and
  an HTTPS server which are collaborating could use this technique to
  fingerprint a targeted HTTPS client.

Note that an HTTPS client in the configuration described in this
document doesn't make direct use of the STH itself. Its fetching of
the STH and reporting via STH Pollination provides a benefit to the CT
ecosystem as a whole by providing oversight on logs, but the HTTPS
client itself will not necessarily derive direct benefit.

### Trusted Auditors for HTTPS Clients {#privacy-trusted-auditors}

Some HTTPS clients may choose to use a trusted auditor. This trust
relationship leaks a certain amount of information from the client to
the auditor. In particular, it is likely to identify the web sites
that the client has visited to the auditor. Some clients may already
share this information to a third party, for example, when using a
server to synchronize browser history across devices in a
server-visible way, or when doing DNS lookups through a trusted DNS
resolver. For clients with such a relationship already established,
sending SCT Feedback to the same organization does not appear to leak
any additional information to the trusted third party.

Clients who wish to contact an auditor without associating their
identities with their SCT Feedback may wish to use an anonymizing
network like Tor to submit SCT Feedback to the auditor. Auditors
SHOULD accept SCT Feedback that arrives over such anonymizing
networks.

Clients sending feedback to an auditor may prefer to reduce the
temporal granularity of the history leakage to the auditor by caching
and delaying their SCT Feedback reports. This strategy is only as
effective as the granularity of the timestamps embedded in the SCTs
and STHs.

### HTTPS Clients as Auditors

Some HTTPS Clients may choose to act as Auditors themselves. A Client
taking on this role needs to consider the following:

- an Auditing HTTPS Client potentially leaks their history to the logs
  that they query. Querying the log through a cache or a proxy with
  many other users may avoid this leakage, but may leak information to
  the cache or proxy, in the same way that an non-Auditing HTTPS
  Client leaks information to a trusted Auditor.

- an effective Auditor needs a strategy about what to do in the event
  that it discovers misbehavior from a log. Misbehavior from a log
  involves the log being unable to provide either (a) a consistency
  proof between two valid STHs or (b) an inclusion proof between an
  SCT and an STH any time after the log's MMD has elapsed from the
  issuance of the SCT. The log's inability to provide either proof
  will not be externally cryptographically-verifiable, as it may be
  indistinguishable from a network error.

# IANA considerations

TBD

# Contributors

The authors would like to thank the following contributors for
valuable suggestions: Ben Laurie, Benjamin Kaduk, Magnus Ahltorp, Tom
Ritter.

# ChangeLog

## Changes between -01 and -02

- Add STH Pollination.

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
