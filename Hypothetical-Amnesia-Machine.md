## Introduction

When conversations happen face-to-face, the participants can tell what comments proceed and follow their own.  When conversations are spread out over space, that clear synchronicity is absent.  GUN's Hypothetical Amnesia Machine (HAM) uses a variety of mechanisms to algorithmically provide digital synchronicity to data.

There are a variety of mechanisms that can be used to determine the order of data, each of which has drawbacks.  

## Existing mechanisms and challenges

### Timestamps

One such mechanism is to the use of timestamps.  In this case, users' data would include a timestamp and a rough approximation of order can be based on these timestamps.  But there are several problems when using timestamps.  One problem is that individual users' computers may have slight variations in their clocks, while these may seem miniscule, when dealing with real-time data they can cause significant errors.  Another problem is that some users may intentionally try to alter their data's timestamp.

### Vector Clocks

An alternative mechanism is the use of vector clocks<sup>1</sup>.  In this case, messages are given a "state" measurement that indicates the order of operations.  For example, if Alice starts up a chat server and sends out a message, it might be given the state "A:1", while her very next message would be given the state "A:2".  But before Alice can send a third message, she receives a message from Bob.  In Bob's message, he indicates that had received Alice's state "A:1" message.  While we now know that Alice's "A:1" message was first, we don't know whether Alice's "A:2" or Bob's "B:2" message was second or third.  However, if there are enough peers that are collecting and comparing states, we can deduce a large percentage of order.  For example, if Charlie was listening to Alice and Bob, he may be able to indicate that Bob's "B:2" message was received before Alice's "A:2" message.  Unfortunately, just like humans in transit, data takes time to travel and the distance between peers can be highly variable, especially on the internet where each packet of data can take a different path.

|          | Timestamps                              | Vector Clocks                |
|:--------:|:---------------------------------------:|:----------------------------:|
| **pros** | built-in to users' machines             | not dependent on clocks      |
| **cons** | inconsistencies between users' machines | high probability of conflict |
|          | subject to intentional user alteration  |                              |

### Conflicts

In addition to the actual ordering of data, it is possible, and in a system with enough data being transmitted, highly probable, that data will in fact be sent at precisely the same moment.

## GUN's Hypothetical Amnesia Machine (HAM)

GUN's HAM combines timestamps, vector clocks, and a conflict resolution algorithm, to maximize the ordering of data and consistently resolve conflicts that do occur.

## Footnotes
<sup>1</sup> Wikipedia entry on [Vector clocks](https://en.wikipedia.org/wiki/Vector_clock)