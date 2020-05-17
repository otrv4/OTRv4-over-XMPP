# XEP-xxxx: Current Off-the-Record Messaging Usage

## Introduction

The Off-the-Record messaging protocol (OTR) was originally introduced in the
2004 paper 'Off-the-Record Communication, or, Why Not To Use PGP' [\[1\]](#references)
and has since become an standard for performing end-to-end encryption. OTR
provides confidentiality, deniability, authentication, forward secrecy and
post-compromise security.

The OTR protocol itself (in its latest version) is currently described by the
document: Off-the-Record Messaging Protocol version 4 [\[2\]](#references) and
will not be described in detail over here. Instead, this document aims to
describe OTR's usage in its current version (version 4) and best practices of
using it within XMPP.

## Overview

Though this document will not focus on the OTR protocol itself, a brief overview
is warranted to better understand the protocols strengths and weaknesses.

OTR provides several cryptographic properties depending of what kind of
conversation in established. In terms of deniability, OTR (in its version 4)
provides offline deniability for any kind of conversations, but only online
deniability for both parties during online conversations (it provides online
deniability for only one party -the initiator- during offline conversations).
It also provides confidentiality, integrity, and authentication (in a deniable
matter). Furthermore, it provides forward secrecy and post-compromise security.
To achieve all of these properties, OTR uses several cryptographic algorithms,
such as DAKEs, SHAKE-256, ChaCha20, SMP, and more.

An OTR session can be held only between two parties, regardless of them being
online or offline. Because of this, OTR is incompatible with Multi-User Chat
(XEP-0045) [\[3\]](#references) and Mediated Information eXchange (MIX)
(XEP-0369) [\[4\]](#references); but OTR might support group chat in future
versions.

Take into account that OTR, in its version 4, can be implemented over clients
following a selection of modes: 'OTRv3-compatible mode', 'OTRv4-standalone mode',
and 'OTRv4-interactive-only'. Information about these modes can be found on
the main protocol [\[5\]](#references).

## Discovery

Clients that support the OTR protocol can advertise it in different ways.
OTR can be implemented in different modes that impact how the advertisement
works.

// TODO: modes, policies, different advertisement mechanisms

Must contain:

* Discovering if peer is online or offline
* Routing and multidevice
* Sever discovery
* Publishing information

Additional:
* Toolkit?


## References

1. Borisov, N., Goldberg, I. and Brewer, E. (20004). *Off-the-Record Communication,
   or, Why Not To Use PGP*, WPES’04. Available at:
   https://otr.cypherpunks.ca/otr-wpes.pdf
