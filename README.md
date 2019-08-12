# Toy Protocol

This repository contains several (increasingly complex) Tamarin models of a simple protocol (basically a simplified version of the WPA2 four-way handshake).

## Initial Protocol

Your goal is to model a simple protocol (which has several flaws) in Tamarin and then successively improve the protocol to deal with the flaws. The initial protocol is as follows:

There are two parties, Alice and Bob, who want to compute a session key so they can communicate with each other secretly. To do so, they proceed as follows:

1. Alice computes a nonce and sends it to Bob (A -> B: ANonce)
2. When Bob receives Alice's nonce, he computes his own nonce and sends it to Alice. (B -> A: BNonce)
3. When Alice receives Bob's nonce, she does two things:
	(i) She installs a session key SK, which she computes from ANonce and BNonce by applying a key derivation function to them (i.e., SK = kdf(ANonce, BNonce)). 
	(ii) Once the session key is installed, she sends a message with the string "ACK" to Bob (A -> B: "ACK") and switches to a 'DONE' state to indicate that the protocol has been executed successfully.
4. When Bob receives the "ACK" message, he also computes and installs the session key SK = kdf(ANonce, BNonce) and switches to a 'DONE' state.

### Questions

Can you model this protocol in Tamarin? Note that there is not "the one perfect solution" but that the protocol can be modeled in several different ways.

Once you have a model of the protocol, can you formulate a lemma ("exists-trace") in Tamarin that says that both Alice and Bob can reach the 'DONE' state?

Can you formulate a lemma in Tamarin that says "When Bob is in the 'DONE' state, an adversary cannot know the session key"? Can Tamarin prove this statement? If not, what's the problem?

## Extension: Preshared Master Key

In the above protocol, one problem was that a person-in-the-middle could just obtain the ANonce and the BNonce and then compute the session key herself. We could mitigate this problem by requiring that Alice and Bob have shared a secret (a permanent "master key") before they run the protocol (we just assume there is some master key that has been shared). When they compute the session key, they then incorporate the master key (let's call it MK) as follows: SK = kdf(MK, ANonce, BNonce).

### Questions

Can you modify the initial model to incorporate the master-key mechanism?

Can you again formulate a lemma that says that both Alice and Bob can reach the 'DONE' state?

Can you again formulate a lemma that says "When Bob is in the 'DONE' state, an adversary cannot know the session key"? Can Tamarin prove this lemma?

Is it possible that Bob reaches the 'DONE' state but Alice does not? (Maybe Tamarin can help with answering this question!?) If so, why is that? What could we do to deal with this?

## Extension: Message Authentication Code

In the previous version of our protocol, the problem was that an adversary could send the "ACK" message to Bob and thus trick Bob into believing that Alice has successfully installed the session key. To avoid this, we can require Alice to compute a message authentication code for the "ACK" message using the session key SK. This should prevent an adversary from forging the "ACK" message.

### Questions

Can you incorporate this MAC mechanism into the Tamarin model?

Can you now formulate and prove the lemma that says "When Bob reaches the 'DONE' state, Alice must have reached the 'DONE' state too (even before Bob)"?

## Extended Protocol: Resending Messages

In our current protocol, Alice and Bob will only send their nonces (and the "ACK" message) once. This could cause problems when messages get lost. To deal with this, we allow Alice to send her nonce again in case she hasn't received a response from Bob.

### Questions

Can you incorporate this into the Tamarin model?

What happens now if you want to prove the lemma that says "When Bob is in the 'DONE' state, an adversary cannot know the session key"? Why might Tamarin fail to terminate? How could you help Tamarin prove the lemma?
