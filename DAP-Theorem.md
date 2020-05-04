Given our work on decentralized algorithms we have come across common problem loops, and would like to add to the literature so that others do not get stuck in the same circles. This article attempts to bring light to what we call the DAP Theorem, which is similar in principle to the CAP Theorem. It suggests you can only have 2 out of the 3 properties, however it has not yet been proven. We ask others to participate in this discussion and add to the literature to either confirm or deny our proposition.

### Introduction

An ideal system would be fully decentralized, anonymous, and private. However, after many attempts to build such a system and failing we have concluded it is impossible to achieve all three. Instead you must choose the two that you want to support and sacrifice some if not all of the third. First, let us review some definitions to clarify the terms and then go over existing examples.

### Definitions

 - **D**ecentralized, means to have no central authority. The system can survive in a completely peer to peer manner, requiring no leader to do look ups or make decisions. Its advantages are in resilience and robustness to failure.

 - **A**nonymous, means having no identifying information that is traceable. Two points of data in the system are seen as being independent, having no correlation despite the fact that the same author might be behind them. Its advantages is that the content has to be judged on its own merit and enabling a whistle-blower to be protected from backlash.

 - **P**rivate, means reliably communicating with a specific endpoint without bits of the information being snooped on or altered in transit. Its advantage is that the recipient knows the information has been vetted by a source they trust, allowing them to filter out anything else as noise.

### Examples

 - **Bitcoin** is not anonymous, despite popular perception. There might be anonymity between the online identity and the face behind the transactions, but all activity of a particular wallet can be traced. In fact, this is the strength of a Block Chain Ledger, it is verifiable and consistent across a consensus of peers. Such metadata on every transaction can be used to identify the intents of the suspected wallet. However bitcoin has the virtues of not requiring any central leader and is capable of reliably transferring money to a specific identity.

... WIP ...