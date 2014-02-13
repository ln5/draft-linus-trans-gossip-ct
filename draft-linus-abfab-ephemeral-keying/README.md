Ephemeral keying for ABFAB
==========================

This document describes how EAP-GSS provides forward secrecy by
encrypting each session in an ephemeral key generated in the initial
state of the context establishment. This Diffie-Hellman key is shared
by the initiator (EAP peer) and acceptor (EAP authenticator).

The goal is to stop a passive attacker with access to the traffic
between an ABFAB user and the service she uses (Relying Party), from
getting access to key material and information linkable to the user or
from being able to fingerprint the user.
