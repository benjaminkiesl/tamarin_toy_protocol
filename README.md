# Tamarin Toy Protocol

This repository contains exercises that should help you get familiar with [Tamarin](https://tamarin-prover.github.io/), a tool for formally analyzing security protocols. The protocol you are going to model over the course of these exercises is a simplified version of the [WPA2 four-way handshake](https://benjaminkiesl.github.io/publications/a_formal_analysis_of_ieees_wpa2_cremers_kiesl_medinger.pdf). Along with the exercises, the repository contains possible solutions, but keep in mind that there are several ways to model one and the same protocol in Tamarin. The exercises here require a basic understanding of Tamarin, which you can obtain by reading the [Tamarin Manual](https://tamarin-prover.github.io/manual/index.html).

## Initial Protocol

Your goal is to model a simple protocol (which has several flaws) in Tamarin and then step-by-step improve the protocol to deal with its flaws. The initial protocol is as follows:

There are two parties, Alex and Blake, who want to establish a [session key](https://en.wikipedia.org/wiki/Session_key) (which they can then use to encrypt messages and communicate with each other secretly). To do so, they proceed as follows:

1. Alex computes a [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) and sends it to Blake. (*A* -> *B*: *ANonce*)
2. When Blake receives Alex's nonce, Blake computes their own nonce and sends it to Alex. (*B* -> *A*: *BNonce*)
3. When Alex receives Blake's nonce, Alex does two things:
   1. Alex installs a session key *SK*, which is derived from *ANonce* and *BNonce* by applying a [key derivation function](https://en.wikipedia.org/wiki/Key_derivation_function) (i.e., *SK* = *kdf*(*ANonce*, *BNonce*)). 
   2. Once the session key is installed, Alex sends a message with the string "ACK" to Blake (*A* -> *B*: "ACK") and switches to a 'DONE' state to indicate that the protocol has been executed successfully on Alex's side.
4. When Blake receives the "ACK" message, Blake also computes the session key *SK* = *kdf*(*ANonce*, *BNonce*), installs it and switches to a 'DONE' state.

### Exercises

1. Can you model this protocol in Tamarin? Note that there is not "the one perfect solution" but that the protocol can be modeled in several different ways.

2. Once you have a model of the protocol, can you formulate a lemma ("exists-trace") in Tamarin that says that both Alex and Blake can reach the 'DONE' state?

3. Can you formulate a lemma in Tamarin that says "When Blake is in the 'DONE' state, an attacker cannot know the session key"? Can Tamarin prove this statement? If not, what's the problem?

Possible solution: [toy_protocol_1.spthy](toy_protocol_1.spthy)

## Extension: Preshared Master Key

In the above protocol, one problem was that a [person-in-the-middle attacker](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) could just obtain the *ANonce* and the *BNonce* and then use them to compute the session key. We could mitigate this problem by requiring that Alex and Blake must have shared a secret (a permanent "master key") before they run the protocol (we just *assume* there is some master key that they have shared; in practice, this master key could be a simple password that Alex and Blake agreed on before running the protocol). When they compute the session key, they then incorporate the master key (let's call it *MK*) as follows: *SK* = *kdf*(*MK*, *ANonce*, *BNonce*).

### Exercises

1. Can you modify the initial model to incorporate the master-key mechanism?

2. Can you again formulate a lemma that says that both Alex and Blake can reach the 'DONE' state?

3. Can you again formulate a lemma that says "When Blake is in the 'DONE' state, an attacker cannot know the session key"? Can Tamarin prove this lemma?

4. Can you also formulate and prove a lemma that says "When Alex is in the 'DONE' state, an attacker cannot know the session key"?

5. Is it possible that Blake reaches the 'DONE' state but Alex does not? (Maybe Tamarin can help with answering this question!?) If so, why is that? What could we do to deal with this?

Possible solution: [toy_protocol_2_master_key.spthy](toy_protocol_2_master_key.spthy)

## Extension: Message Authentication Code

In the previous version of our protocol, the problem was that an attacker could send the "ACK" message to Blake and thus trick Blake into believing that Alex has successfully installed the session key. To avoid this, we can require Alex to compute a [message authentication code](https://en.wikipedia.org/wiki/Message_authentication_code) (with the session key *SK*) for the "ACK" message and send this MAC to Blake together with the message. When Blake receives the message and the MAC, Blake then first verifies the MAC before switching to the 'DONE' state. This should prevent an attacker from forging the "ACK" message.

### Exercises

1. Can you incorporate this message-authentication mechanism into the Tamarin model?

2. Can you now formulate and prove the lemma that says "When Blake reaches the 'DONE' state, Alex must have reached the 'DONE' state too (even before Blake)"?

Possible solution: [toy_protocol_3_mac.spthy](toy_protocol_3_mac.spthy)

## Extended Protocol: Resending Messages

In our current protocol, Alex and Blake will only send their nonces (and the "ACK" message) once. This could cause problems when messages get lost. To deal with this, we allow Alex to send their nonce (*ANonce*) again (arbitrarily many times) in case they haven't received a response from Blake.

### Exercises

1. Can you incorporate this retransmission mechanism into the Tamarin model?

2. What happens now if you want to prove the lemma that says "When Alex is in the 'DONE' state, an attacker cannot know the session key"? Why might Tamarin fail to terminate? How could you help Tamarin prove the lemma?

Possible solution: [toy_protocol_4_resend_anonce.spthy](toy_protocol_4_resend_anonce.spthy)
