# Tamarin Toy Protocol

This repository contains a few increasingly complex exercises that should help you get familiar with [Tamarin](https://tamarin-prover.github.io/), a tool for formally analyzing security protocols. The protocol you are going to model over the course of these exercises is a simplified version of the [WPA2 four-way handshake](https://benjaminkiesl.github.io/publications/a_formal_analysis_of_ieees_wpa2_cremers_kiesl_medinger.pdf). Along with the exercises, the repository contains possible solutions, but keep in mind that these solutions are just suggestions, and that there are several ways to model one and the same protocol in Tamarin. The exercises here require a basic understanding of Tamarin, which you can obtain by reading the [Tamarin Manual](https://tamarin-prover.github.io/manual/index.html).

## Initial Protocol

Your goal is to model a simple protocol (which has several flaws) in Tamarin and then step-by-step improve the protocol to deal with its flaws. The initial protocol is as follows:

There are two parties, Alice and Bob, who want to establish a [session key](https://en.wikipedia.org/wiki/Session_key) (that they can then use to encrypt messages and communicate with each other secretly). To do so, they proceed as follows:

1. Alice computes a [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) and sends it to Bob. (*A* -> *B*: *ANonce*)
2. When Bob receives Alice's nonce, he computes his own nonce and sends it to Alice. (*B* -> *A*: *BNonce*)
3. When Alice receives Bob's nonce, she does two things:
   1. She installs a session key *SK*, which she computes from *ANonce* and *BNonce* by applying a [key derivation function](https://en.wikipedia.org/wiki/Key_derivation_function) (i.e., *SK* = *kdf*(*ANonce*, *BNonce*)). 
   2. Once the session key is installed, she sends a message with the string "ACK" to Bob (*A* -> *B*: "ACK") and switches to a 'DONE' state to indicate that the protocol has been executed successfully on her side.
4. When Bob receives the "ACK" message, he also computes and installs the session key *SK* = *kdf*(*ANonce*, *BNonce*) and switches to a 'DONE' state.

### Exercises

1. Can you model this protocol in Tamarin? Note that there is not "the one perfect solution" but that the protocol can be modeled in several different ways.

2. Once you have a model of the protocol, can you formulate a lemma ("exists-trace") in Tamarin that says that both Alice and Bob can reach the 'DONE' state?

3. Can you formulate a lemma in Tamarin that says "When Bob is in the 'DONE' state, an attacker cannot know the session key"? Can Tamarin prove this statement? If not, what's the problem?

Possible solution: [toy_protocol_1.spthy](toy_protocol_1.spthy)

## Extension: Preshared Master Key

In the above protocol, one problem was that a [person-in-the-middle attacker](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) could just obtain the *ANonce* and the *BNonce* and then compute the session key herself. We could mitigate this problem by requiring that Alice and Bob have shared a secret (a permanent "master key") before they run the protocol (we just *assume* there is some master key that has been shared; in practice, this master key could be a simple password that Alice and Bob agreed on before running the protocol). When they compute the session key, they then incorporate the master key (let's call it *MK*) as follows: *SK* = *kdf*(*MK*, *ANonce*, *BNonce*).

### Exercises

1. Can you modify the initial model to incorporate the master-key mechanism?

2. Can you again formulate a lemma that says that both Alice and Bob can reach the 'DONE' state?

3. Can you again formulate a lemma that says "When Bob is in the 'DONE' state, an attacker cannot know the session key"? Can Tamarin prove this lemma?

4. Can you also formulate and prove a lemma that says "When Alice is in the 'DONE' state, an attacker cannot know the session key"?

5. Is it possible that Bob reaches the 'DONE' state but Alice does not? (Maybe Tamarin can help with answering this question!?) If so, why is that? What could we do to deal with this?

Possible solution: [toy_protocol_2_master_key.spthy](toy_protocol_2_master_key.spthy)

## Extension: Message Authentication Code

In the previous version of our protocol, the problem was that an attacker could send the "ACK" message to Bob and thus trick Bob into believing that Alice has successfully installed the session key. To avoid this, we can require Alice to compute a [message authentication code](https://en.wikipedia.org/wiki/Message_authentication_code) (with the session key *SK*) for the "ACK" message and send this MAC to Bob together with the message. When Bob receives the message and the MAC, he then first verifies the MAC before switching to the 'DONE' state. This should prevent an attacker from forging the "ACK" message.

### Exercises

1. Can you incorporate this message-authentication mechanism into the Tamarin model?

2. Can you now formulate and prove the lemma that says "When Bob reaches the 'DONE' state, Alice must have reached the 'DONE' state too (even before Bob)"?

Possible solution: [toy_protocol_3_mac.spthy](toy_protocol_3_mac.spthy)

## Extended Protocol: Resending Messages

In our current protocol, Alice and Bob will only send their nonces (and the "ACK" message) once. This could cause problems when messages get lost. To deal with this, we allow Alice to send her nonce again (arbitrarily many times) in case she hasn't received a response from Bob.

### Exercises

1. Can you incorporate this retransmission mechanism into the Tamarin model?

2. What happens now if you want to prove the lemma that says "When Alice is in the 'DONE' state, an attacker cannot know the session key"? Why might Tamarin fail to terminate? How could you help Tamarin prove the lemma?

Possible solution: [toy_protocol_4_resend_anonce.spthy](toy_protocol_4_resend_anonce.spthy)
