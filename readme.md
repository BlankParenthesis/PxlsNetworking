Pxls Networking API
===================

This repository contains the specifications defining the communication of clients and the server of [Pxls](https://github.com/pxlsspace).

Third-party tool developers can use this as a reference when authoring their tools. 
Client and server developers can use this as a reference for implementing their respective half of the protocol.

The specification is broken up into a [core specification](./core.md) and [extensions](./extensions/) which build on this core to add features.
All client and server implementations should support the core specification.

Implementations are expected to be able to operate correctly with the intersection of both sides implemented extensions.
This means that a server that has implemented all extensions should still function for a client which only implements the core specification.
Likewise, a client that has implemented all extensions should connect and function without issue to a server implementing only the core specification.

This specification is currently a work in progress and no implementations of it exist. Once this specification has stabilized, the intent is to have the current [Pxls client and server](https://github.com/pxlsspace/Pxls) be the reference implementations.