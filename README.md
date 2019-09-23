# Toy Protocol

This repository contains a few increasingly complex exercises that should help you get familiar with [Tamarin](https://tamarin-prover.github.io/), a verification tool for security protocols. The protocol you are going to model over the course of these exercises is basically a simplified version of the WPA2 four-way handshake. Along with the exercises, the repository contains possible solutions, but note that those solutions are just suggestions and that there are several ways to model one and the same protocol in Tamarin. Note that the exercises here require a basic understanding of Tamarin, which you can obtain by reading the [Tamarin Manual](https://tamarin-prover.github.io/manual/index.html).

## Initial Protocol

Your goal is to model a simple protocol (which has several flaws) in Tamarin and then step-by-step improve the protocol to deal with its flaws. The initial protocol is as follows:

There are two parties, Alice and Bob, who want to compute a session key so they can communicate with each other secretly. To do so, they proceed as follows:

1. Alice computes a nonce and sends it to Bob (A -> B: ANonce)
2. When Bob receives Alice's nonce, he computes his own nonce and sends it to Alice. (B -> A: BNonce)
3. When Alice receives Bob's nonce, she does two things:
   - She installs a session key SK, which she computes from ANonce and BNonce by applying a key derivation function to them (i.e., SK = kdf(ANonce, BNonce)). 
   - Once the session key is installed, she sends a message with the string "ACK" to Bob (A -> B: "ACK") and switches to a 'DONE' state to indicate that the protocol has been executed successfully.
4. When Bob receives the "ACK" message, he also computes and installs the session key SK = kdf(ANonce, BNonce) and switches to a 'DONE' state.

### Questions

Can you model this protocol in Tamarin? Note that there is not "the one perfect solution" but that the protocol can be modeled in several different ways.

Once you have a model of the protocol, can you formulate a lemma ("exists-trace") in Tamarin that says that both Alice and Bob can reach the 'DONE' state?

Can you formulate a lemma in Tamarin that says "When Bob is in the 'DONE' state, an adversary cannot know the session key"? Can Tamarin prove this statement? If not, what's the problem?

Possible solution: toy_protocol_1.spthy

## Extension: Preshared Master Key

In the above protocol, one problem was that a person-in-the-middle could just obtain the ANonce and the BNonce and then compute the session key herself. We could mitigate this problem by requiring that Alice and Bob have shared a secret (a permanent "master key") before they run the protocol (we just assume there is some master key that has been shared). When they compute the session key, they then incorporate the master key (let's call it MK) as follows: SK = kdf(MK, ANonce, BNonce).

### Questions

Can you modify the initial model to incorporate the master-key mechanism?

Can you again formulate a lemma that says that both Alice and Bob can reach the 'DONE' state?

Can you again formulate a lemma that says "When Bob is in the 'DONE' state, an adversary cannot know the session key"? Can Tamarin prove this lemma?

Can you also formulate and prove a lemma that says "When Alice is in the 'DONE' state, an adversary cannot know the session key"?

Is it possible that Bob reaches the 'DONE' state but Alice does not? (Maybe Tamarin can help with answering this question!?) If so, why is that? What could we do to deal with this?

Possible solution: toy_protocol_2_master_key.spthy

## Extension: Message Authentication Code

In the previous version of our protocol, the problem was that an adversary could send the "ACK" message to Bob and thus trick Bob into believing that Alice has successfully installed the session key. To avoid this, we can require Alice to compute a message authentication code (with the session key SK) for the "ACK" message and send this MAC to Bob together with the message. When Bob receives the message and the MAC, he then first verifies the MAC before switching to the 'DONE' state. This should prevent an adversary from forging the "ACK" message.

### Questions

Can you incorporate this message-authentication mechanism into the Tamarin model?

Can you now formulate and prove the lemma that says "When Bob reaches the 'DONE' state, Alice must have reached the 'DONE' state too (even before Bob)"?

Possible solution: toy_protocol_3_mac.spthy

## Extended Protocol: Resending Messages

In our current protocol, Alice and Bob will only send their nonces (and the "ACK" message) once. This could cause problems when messages get lost. To deal with this, we allow Alice to send her nonce again (arbitrarily many times) in case she hasn't received a response from Bob.

### Questions

Can you incorporate this into the Tamarin model?

What happens now if you want to prove the lemma that says "When Alice is in the 'DONE' state, an adversary cannot know the session key"? Why might Tamarin fail to terminate? How could you help Tamarin prove the lemma?

Possible solution: toy_protocol_4_resend_anonce.spthy
