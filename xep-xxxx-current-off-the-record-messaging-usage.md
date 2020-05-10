# XEP-xxxx: Current Off-the-Record Messaging Usage

## Introduction

The Off-the-Record messaging protocol (OTR) was originally introduced in the
2004 paper [Off-the-Record Communication, or, Why Not To Use PGP](https://otr.cypherpunks.ca/otr-wpes.pdf) [1]
and has since become an standard for performing end-to-end encryption. OTR
provides confidentiality, deniability, authentication, forward secrecy and
post-compromise security.

The OTR protocol itself (in its latest version) is currently described by the
document: Off-the-Record Messaging Protocol version 4 [2] and will not be
described here. Instead, this document aims to describe OTR's usage in its
current version (version 4) and best practices of using it within XMPP.

## Overview

Though this document will not focus on the OTR protocol itself, a brief overview
is warranted to better understand the protocols strengths and weaknesses.

OTR (in its version 4) uses several cryptographic algorithms, such as SHAKE-256
and ChaCha20. An OTR session can be held only between two parties, meaning that
OTR is incompatible with Multi-User Chat (XEP-0045) [3] and Mediated Information
eXchange (MIX) (XEP-0369) [4]. It provides deniability in its online and offline
way. It also provides forward secrecy and post-compromise security.

## Discovery

Clients that support the OTR protocol can advertise it in different ways.
OTR can be implemented in different modes that impact how the advertisement
works.

// TODO: modes, policies, different advertisement mechanisms
