# OTRv4 over XMPP

This document will eventually become an XEP.

This document will be used to define what issues need to be addressed for
OTRv4 to work correctly over XMPP.

## Important issues to consider over XMPP

### Explain how multicasting/synchronization will work

#### OMEMO:

From the security audit plus some comments:

"Assume Alice wants to send an OMEMO encrypted message from her phone. She can
detect that Bob’s device(s) support OMEMO by requesting his device list with
PEP. If he does, she encrypts and authenticates her message using a randomly
generated key (`sym_k`). For every device that Alice wants to send the encrypted
message to, she fetches the entire bundle via PEP (sic: this means that a Key
Agreement is done per each device: how is the long-term secret key shared?). If
she wants to add more of her own devices in the conversation, she gets their
bundles as well from her own server. Alice creates a PreKeySignalMessage for
every device by picking a random one-time prekey from each bundle and
encrypting the randomly generated key to each device. She combines all
information in a single MessageElement: the encrypted payload (<payload/>), the
plaintext iv (<iv/>), the sender id (sid) and the encrypted
random key (<key/>) tagged with the corresponding receiver id (rid)"

1. Generate a random `sym_k`.
2. Calculate: `enc_key(32), auth_key(32), IV(16) := SHA-256(sym_k || 0x00 || "OMEMO Payload")`
3. Encrypt: `c := AES_CBC(enc_key, IV || message)`
4. Calculate: `MAC := SHA-256(auth_key || c)`
5. Concatenate: `payload := c || MAC`
6. Execute the double ratchet algorithm and generate a message key `mk`.
7. Calculate: `h_enc_key(32), auth_key(32), IV(16) := SHA-256(m_k || 0x00 || "OMEMO Message Key Material")`
8. Encrypt the payload: `h := (h_enc_key, payload)`

Since step 6, it is executed per device.

Review:

* [Private Group Messaging](https://signal.org/blog/private-groups/)
* [OMEMO: cryptographic analysis report](https://conversations.im/omemo/audit.pdf)

### Define a prekey server discovery and place

### Explain how key management will work

### Explain how online and offline versions will work

## Not so important issues

### Explain invisibility status and OTR

### How to carry the encrypted message

## To take into account

### Encrypt other things than message bodies
