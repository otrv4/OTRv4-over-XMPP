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
to optionally decide of a mode that will be used and to generate an instance tag
(which works as a device id). They should also publish their instance tag to a
device list on the PEP node, so other devices can find it. And lastly, depending
of the mode chosen, they should publish specific prekey values.

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
*Example 1. Adding the own instance tag to the device list*

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

### Publishing Prekey values

A device MUST publish its Prekey values, which can refer to the Client Profile,
the Prekey Profile and a set of Prekey Messages. For this, an specific
unstrusted Prekey Server should be used.

#### Discovering a Prekey Server

A participant will find information about a prekey server reading the Service
Discovery specification (XEP-0030). The first lookup will be a look up of items
of the containing server:

```
*Example 2. Lookup for server*

  <iq from='alice@xmpp.org/notebook'
      id='h7ns81g'
      to='xmpp.org'
      type='get'>
    <query xmlns='http://jabber.org/protocol/disco#items'/>
  </iq>
```

The server then returns the services that are associated with it:

```
*Example 3. Server returns associated services*

  <iq from='xmpp.org'
      id='h7ns81g'
      to='alice@xmpp.org/notebook'
      type='result'>
    <query xmlns='http://jabber.org/protocol/disco#items'>
      <item jid='prekey.xmpp.org'
            name='OTR Prekey Server'/>
    </query>
  </iq>
```

In order to find an OTRv4 compliant prekey server, the device then needs to send
a info request to all items returned from the original call:

```
*Example 4. Device checks for OTRv4 compliant prekey server*

  <iq from='alice@xmpp.org/notebook'
      id='info1'
      to='prekey.xmpp.org'
      type='get'>
    <query xmlns='http://jabber.org/protocol/disco#info'/>
  </iq>
```

For a compliant server, this will return the feature
`http://jabber.org/protocol/otrv4-prekey-server` and an identity that has
category `auth` and type `otr-prekey`:

```
*Example 5. Server returns an OTRv4 compliant prekey server*

<iq type='result'
    from='prekey.xmpp.org'
    to='alice@xmpp.org/notebook'
    id='info1'>
  <query xmlns='http://jabber.org/protocol/disco#info'>
    <identity
        category='auth'
        type='otr-prekey'
        name='OTR Prekey Server'/>
    <feature var='http://jabber.org/protocol/disco#info'/>
    <feature var='http://jabber.org/protocol/disco#items'/>
    <feature var='http://jabber.org/protocol/otrv4-prekey-server'/>
  </query>
</iq>
```

Finally, before starting to use a Prekey server, you also need to lookup for the
fingerprint for this server. This can be find by doing a lookup for items on the
server:

```
*Example 6. Asking for additional items from the prekey server*

  <iq from='alice@xmpp.org/notebook'
      id='items1'
      to='prekey.xmpp.org'
      type='get'>
    <query xmlns='http://jabber.org/protocol/disco#items'/>
  </iq>
```

This should return an item where node has the value `fingerprint`, and the name
will contain the hexadecimal representation of the fingerprint:

```
*Example 7. Prekey server returns its fingerprint*

<iq type='result'
    from='prekey.xmpp.org'
    to='alice@xmpp.org/notebook'
    id='items1'>
  <query xmlns='http://jabber.org/protocol/disco#items'>
    <item jid='prekey.xmpp.org'
          node='fingerprint'
          name='3B72D580C05DE2823A14B02B682636BF58F291A7E831D237ECE8FC14DA50A187A50ACF665442AB2D140E140B813CFCCA993BC02AA4A3D35C'/>
  </query>
</iq>
```

The fingerprint will, for OTRv4 keys, always be 112 hexadecimal digits that can
be decoded into a 56-byte value, following the instructions in the OTRv4
specification.

If the server returns more than one prekey server in its list of items, anyone
should be able to use it. All prekey servers exposed by an XMPP server have to
share the same storage. Thus, a client should randomly choose one of the
returned prekey servers to connect to, in order to distribute load for the
server.

#### Publishing Prekey Values to the Server

An entity authenticates to the server through an interactive DAKE. DAKE
messages are sent in "message" stanzas. This follows the prekey server
specification for OTR [\[6\]](#references).

When calculating the `phi` value for XMPP, the bare JID of the publisher and the
bare jid of the server has to be used:

```
*Example 8. Calculating Phi*

  phi = DATA("alice@xmpp.org") || DATA("prekey.xmpp.org")
```

An entity starts the DAKE by sending the first encoded message in the body
of a message:

```
*Example 9. First DAKE message*

  <message
      from='alice@xmpp.org/notebook'
      id='nzd143v8'
      to='prekey.xmpp.org'>
    <body>AAQ1...</body>
  </message>
```

The server responds with the subsequent DAKE message:

```
*Example 10. Second DAKE message*

  <message
      from='prekey.xmpp.org'
      id='13fd16378'
      to='alice@xmpp.org/notebook'>
    <body>AAQ2...</body>
  </message>
```

And the entity terminates the DAKE and sends the prekey values attached to the
last DAKE message:

```
*Example 11. Thrid DAKE message and submission*

  <message
      from='alice@xmpp.org/notebook'
      id='kud87ghduy'
      to='prekey.xmpp.org'>
    <body>AAQ3...</body>
  </message>
```

And the Prekey Server responds with a "Success" message:

```
*Example 12. Success message*

  <message
      from='prekey.xmpp.org'
      id='0kdytsmslkd'
      to='alice@xmpp.org/notebook'>
    <body>AAQG...</body>
  </message>
```

#### Obtaining Information about Prekey Messages from the Server

An entity authenticates to the server through a DAKE. DAKE messages are send
in "message" stanzas.

It sends the same DAKE messages as the previous section, except for the attached
message in the last DAKE-3 message.

And the entity terminates the DAKE and asks for storage information:

```
*Example 13. Thrid DAKE message and ask for storage information*

  <message
      from='alice@xmpp.org/notebook'
      id='kud87ghduy'
      to='prekey.xmpp.org'>
    <body>AAQ3...</body>
  </message>
```

And the Prekey Server responds with a "Storage Status" message:

```
*Example 14. Storage status message*

  <message
      from='prekey.xmpp.org'
      id='0kdytsmslkd'
      to='alice@xmpp.org/notebook'>
    <body>AAQL...</body>
  </message>
```

## Discovery

Clients that support the OTR protocol can advertise it in different ways, as
OTR can be implemented in different modes that impact how the advertisement
works.

There are two main mechanisms used:

1. Plaintext advertisement
2. DAKE advertisement.

### Plaintext advertisement

If you are implementing 'OTRv3 Compatible Mode' and 'Interactive-Only Mode',
or starting an online conversations, you can use these kind of initialization
messages, which use OTR's own discovery mechanisms.

1. Whitespace Tag

If a client wishes to indicate support for OTR they include a special whitespace
tag in their messages. This tag can appear anywhere in the body of the message
stanza, but it is most often found at the end. The OTR tag comprises the
following bytes:

```
*Example 14. OTR tag*

\x20\x09\x20\x20\x09\x09\x09\x09 \x20\x09\x20\x09\x20\x09\x20\x20
```

and is followed by one or more of the following sequences to indicate the
version of OTR which the client supports:

```
*Example 15. OTR tag version 3*

\x20\x20\x09\x09\x20\x20\x09\x09
```

```
*Example 16. OTR tag version 3*

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
*Example 17. OTR query*

?OTR?v34?
```
Any message which begins with the afforementioned string (note that the version
number[s] may be different) should be treated as an OTR message. The
initialization message can contain a payload, which should not refer to the
identity of any participant.

### DAKE advertisement

If you are implementing 'Interactive-Only Mode', 'Standalone Mode',
or starting offline conversations, you can use these kind of initialization
messages, which use OTR's XMPP own discovery mechanisms.

In order to determine whether a given contact has devices that support OTRv4,
the devices node in PEP is consulted. Devices MUST subscribe to
urn:xmpp:otr:1:devices via PEP, so that they are informed whenever their
contacts add a new device. They MUST cache the most up-to-date version of the
device list.

```
*Example 18. Devicelist update received by subscribed clients*

<message from='ahab@otr.im'
         to='ishmael@otr.im'
         type='headline'
         id='update_01'>
  <event xmlns='http://jabber.org/protocol/pubsub#event'>
    <items node='urn:xmpp:otr:1:devices'>
      <item id='current'>
        <devices xmlns='urn:xmpp:omemo:1'>
          <device id='7c91e4ce' />
          <device id='c8257518' label='Gajim Client' />
        </devices>
      </item>
    </items>
  </event>
</message>
```

After the device list has been received, and the client have decided to
which device to send an initialization message (which depends on the policy
and instance tag behaviour defined. It can be to all of them, to the
trusted one, to the most recent one, etc.), an identity message or a
Non-Interactive Auth message can be sent (for online conversations and offline
conversations respectively). Note that prior to sending any of these messages,
Prekey values should be asked for.

## Building a session

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
