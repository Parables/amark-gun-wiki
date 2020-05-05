# Introduction

## Goal of this Guide

This document will detail different ways to introduce Gun into various existing technology stacks. It will also explain Gun concepts by relating it to other technology.

## Audience for this Guide

### Developers trying to Learn Gun

Some developers may be trying to conceptually understand how Gun works, but they understand web apps best in terms of the technology stack that they are most familiar with.

For example, you've been working with [LAMP stacks](#from-a-lamp-stack) for 20 years and you want to know what parts of the stack Gun can actually replace or enhance.

### Developers with Existing Apps

While it is nice to create a brand new app from scratch, some developers may be wondering how Gun can benefit their existing apps.

For example, maybe you just want to replace your user authentication with Gun and nothing else. Or maybe you want to keep your user authentication but you want to replace a specific component of your website like a user comment system.

# From a LAMP Stack

## Components

* Linux operating system
* Apache HTTP Server
* MySQL relational database management system (RDBMS)
* PHP programming language. 

## Where and How to Introduce Gun

### Example

# From a MEAN Stack

## Components

* MongoDB, a NoSQL database
* Express.js, a web application framework that runs on Node.js
* Angular.js or Angular, JavaScript MVC frameworks that run in browser JavaScript engines
* Node.js, an execution environment for event-driven server-side and networking applications

## Where and How to Introduce Gun

Gun can replace the MongoDB portion of this stack in regards to user authentication and data storage.

### Example

TODO: Give example code in a MEAN stack where you would handle login switching, etc and what parts of code you would replace with Gun API calls

# Component-based frameworks: React, Vue, Angular, etc

Specifically in a flux-based architecture (e.g. React & Vue), you should consider replacing uses of `redux` and async handlers like thunks/epics/observables, since the functionality of Gun completely overlaps with these, and any integration efforts will lead to needless duplication. React-Redux examples here: https://stackoverflow.com/a/57541198

Gun can be used for both your application state, as well as backend synchronization, but you have to keep in mind that Gun is inherently peer-2-peer from the ground up. When hosting your app, you have the choice of running your own Gun server or connecting to a shared network (see [AXE](https://gun.eco/docs/AXE) for more on this).

You can of course use multiple backends in your app, meaning that some components get data from Gun while others fetch from elsewhere like a GraphQL or REST backend, but just be aware you may introduce technical debt that comes back to bite you later on. You can also consider integrating Gun server-side, e.g. connecting to your existing database inside of a nodejs Gun server. This would treat Gun as more of an API frontend than a standalone database, and implicates other technical considerations.

Framework actively under development:
* https://github.com/amark/gun/wiki/React#under-development
* https://github.com/rm-rf-etc/weir

# From React + GraphQL

## Components

## Where and How to Introduce Gun

### Example

# From Ruby on Rails

## Components

## Where and How to Introduce Gun

### Example
