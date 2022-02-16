# GERTi Handshaking Protocol
This document outlines the preferred protocol for establishing connections between two programs through the GERTi networking API, also known as handshaking. 
This assumes a functional GERTi network setup.

## Definitions and Vocabulary
- Address: A GERT address, assigned to the relative device running GERTi.
- Port: This **does not** refer to modem ports, but instead to the Connection ID used by GERT.
- Socket: Streams provided by the GERTi API.
- Peer: Client 
- Server: Device hosting a service
- Rich Information Request: The Second stage of a handshake, which includes all necessary information about the Server


## P2S - Peer To Server
- Once the service has finished initialising, the program must create a listener for `"GERTConnectionID"`, and a program-local `table` that will store sockets, indexed by the address of the Peer.
  - This listener function will open a socket to every device that triggers this event, with optional (but recommended) filtering using the Port, to help ensure that the program responds only to connections that concern it. This socket will be stored in the aforementioned `table`, indexed by the address of the Peer.
- Now, a Peer must open a socket to the Server. If properly configured, the socket will be completed near instantly.
  > It is considered good practice to include a Timeout on waiting for the server to complete the connection, and a timeout of 5 seconds is deemed reasonable.

For high-trust situations, this is effectively sufficient, but not implementing the next phase of the Handshake server-side risks interfering with other programs on the network, as all servers are expected to be capable of responding to Rich Information Requests, while clients are not expected to be capable of requesting Rich Information Requests.

- In addition to all the normal methods a server