### Introduction

Contributions are always welcome. Generally the core of the library is written for performance and smallness and hence readability is not a focus. There is many adapters that are written a bit more verbose by contributors and can serve as templates for other contributors.


### Code Style / Module Definition

#### Code Style

GUN's code base is non-linear, functional, reactive, event-driven, and it only operates against abstract data types, so there is very little code that does "anything"  traditional (the code is extremely boring in what it does, and tedious the number of edge cases you have to predict), its pretty much just a state machine that recurses in on itself (think more like a compiler, or AST parser), which usually makes no sense at all unless you have concrete datasets to test it against.

#### Module Definitions

All modules are modeled as below:
```
USE( IIFE )(USE, './modulename')
```

The IIFE is usually structured in 5 sections:

1 - Requirements 
2 - Module variables
3 - Function Flow (Usually data flows vertically from Top to Bottom)
4 - Entry Code ( Data arrives here and starts flow) 
5 - Export

### Structure of the library

***

#### Gun.js - Core Library
  
-- Type Module - Optimized Type Checking

-- Onto Module - Event System

-- HAM Module - Hypothetical Amnesia Machine CRDT Algorithm

-- VAL Module - Data validation System

-- Node Module - Node Definition (aka Vertex in Graph)

-- State Module - State System (Machine State Generation etc)

-- Graph Module - Graph Definition

-- Ask Module - Request / Response Module

-- Dup Module - De-duplication System

-- Root Module - Definition for Get, Put and Chain API

-- Back Module - Definition for going up in graphs

-- Chain Module - Internal Chaining helpers

-- Get Module - Get Definition

-- Put Module - Put Definition

-- Index Module - Put above Definitions into Gun Root

-- On Module - On / Once / Off Definitions

-- Map Module - Map Definition

-- Set Module - Set Definition

-- Adapter LocalStorage - Storing into LocalStorage (BROWSER)

-- Adapter Mesh - Meshing Functions (NETWORK)

-- Adapter Websocket - Websocket Wrapper for Mesh (NETWORK)

***

#### SEA.js - Security, Encryption and Authorization (BETA)

--> BROWSER uses webcrypto of the browser

--> Node.js uses peculiar/webcrypto (BETA - Experimental)

-- Root Module

-- HTTPS Module - Elevate to https

-- Base64 Module - Node.js btoa atob shim

-- Array Module - Array Buffer Implementation

-- Buffer Module - Safe Buffer Implementation

-- Shim Module - Add peculiar/webcrypto if in Node.js

-- Settings Module - Crypto Settings

--- PDBKDF2 Hash, Iteration, KS

--- ECDSA Pair Gen (P-256)

--- ECDSA Sign (SHA-256)

--- ECDH (P-256)

-- SHA256 Module - Shim for Node.js

-- SHA1 Module - Shim for Node.js (KeyID generator)

-- Work Module - PoW Implementation

-- Pair Module - Key Pair Generation Implementation

-- Sign Module - Data Signing Implementation

-- Verify Module - Data/Author Verification Implementation

-- AESKey Module - Shim for AES (SHA256)

-- Encrypt Module - Data Encryption (AES-GCM)

-- Decrypt Module - Data Decryption (AES-GCM)

-- Secret Module - Shared Secret Derivation (AES-GCM)

-- SEA Module - Core Module

-- Then Module - Promise Wrapper over Gun Chain

-- User Module - Chain User Definition

-- Create Module - User Creation, Authorization, Adding Trust etc

-- Index Module - Verification, Signature Checking, AutoSigning for transport Code

***

### Common Variable Names and Meaning

p = path
fn = function
n = number
t = text or time
s = string
l = length (sometimes also ll / lle )
c = character
o = object or operator
k = key
v = value
u = undefined
tag = subscriber
_ = metadata
as = what to write the result to (@amark to clarify)
at = location in the graph, usually reference to the node you are in
# = soul == uuid in metadata
cb = callback
g = graph
msg = message that is sent to other peers (or received)
ctx = context (usually when passing this around)
cat = (context at ...) metadata short cut
