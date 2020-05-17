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

This document will refer to OTR in its current version -4-, and only take into
account behaviour for version 3 as well, as version 2 and 1 have been deprecated.

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

## Setup

To participate of an OTR conversation, clients need to set up an OTR library,
decide of a mode that will be used and generate an instance tag (which works as
a device id). They should also publish their instance tag to a device list on
the PEP node, so other devices can find it.

### Adding a instance tag to the device list

In order for other devices to be able to initiate a session with a given device
(for offline conversations, and when not wanting to use a query message or
whitespace tag), it first has to announce itself by adding its instace tag to
the devices PEP node.

It is REQUIRED to set the access model of the urn:xmpp:otr:1:devices node to
'open' to give entities without presence subscription read access to the devices
and allow them to establish an OTR session. Not having presence subscription is
a common occurrence on the first few messages between two contacts.

Devices MUST check that their own device id is contained in the list whenever
they receive a PEP update from their own account. If they have been removed,
they MUST reannounce themselves.

The device element MAY contain an attribute called label, which is a user
defined string describing a device that supports OTR. It is
RECOMMENDED to keep the length of the label under 53 Unicode code points. It
MUST not contain location, identity or private information, and only refer to
the name of the client.

```
*Example 1. Adding the own device id to the list*

<iq from='ahab@otr.im' type='set' id='announce1'>
  <pubsub xmlns='http://jabber.org/protocol/pubsub'>
    <publish node='urn:xmpp:otr:1:devices'>
      <item id='current'>
        <devices xmlns='urn:xmpp:otr:1'>
          <device id='c45b041c' label='Beagle Client'/>
          <device id='c1bb7b7a' />
          <device id='7c91e4ce' label='ChatSecure Client' />
        </devices>
      </item>
    </publish>
    <publish-options>
      <x xmlns='jabber:x:data' type='submit'>
        <field var='FORM_TYPE' type='hidden'>
          <value>http://jabber.org/protocol/pubsub#publish-options</value>
        </field>
        <field var='pubsub#access_model'>
          <value>open</value>
        </field>
      </x>
    </publish-options>
  </pubsub>
</iq>
```

## Discovery

Clients that support the OTR protocol can advertise it in different ways, as
OTR can be implemented in different modes that impact how the advertisement
works.

There are two main mechanisms used: 1. plaintext advertisement, 2. DAKE
advertisement.

### Plaintext advertisement

If you are implementing 'OTRv3 Compatible Mode' and 'Interactive-Only Mode',
or starting an online conversations, you can use these kind of initialization
messages, depending on if you want to use OTR's own discovery mechanisms or
OTR's XMPP own discovery mechanisms.

- OTR own discovery mechanisms:

1. Whitespace Tag

If a client wishes to indicate support for OTR they include a special whitespace
tag in their messages. This tag can appear anywhere in the body of the message
stanza, but it is most often found at the end. The OTR tag comprises the
following bytes:

```
*Example 1. OTR tag*

\x20\x09\x20\x20\x09\x09\x09\x09 \x20\x09\x20\x09\x20\x09\x20\x20
```

and is followed by one or more of the following sequences to indicate the
version of OTR which the client supports:

```
*Example 2. OTR tag version 3*

\x20\x20\x09\x09\x20\x20\x09\x09
```

```
*Example 2. OTR tag version 3*

\x20\x20\x09\x09\x20\x09\x20\x20
```

When a client sees this special string in the body of a message stanza it may
choose to start an OTR session immediately, or merely indicate support to the
user and allow the user to manually start a session. This is done by sending a
message stanza containing an OTR query message in the body which indicates the
supported versions of OTR.

2. Query message

As stated above, this is a message stanza containing an OTR query message in the
body which indicates the supported versions of OTR (currently only 3 and 4 are
supported).

```
*Example 3. OTR query*
?OTR?v34?
```
Any message which begins with the afforementioned string (note that the version
number[s] may be different) should be treated as an OTR message. The
initialization message can contain a payload, which should not refer to the
identity of any participant.

- XMPP-OTR own discovery mechanisms:

For online conversations, an Identity message can be used to directly start
an OTR conversation. This only works for version 4.

In order to determine whether a given contact has devices that support OTRv4,
the devices node in PEP is consulted. Devices MUST subscribe to
urn:xmpp:otr:1:devices via PEP, so that they are informed whenever their
contacts add a new device. They MUST cache the most up-to-date version of the
device list.

// TODO: modes, policies

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
