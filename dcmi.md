---
title: DCMI Protocol
layout: default
wip: true
copyrights:
  -
    name: "Daniel Oaks"
    org: "ircdocs"
    org_link: "http://ircdocs.horse/"
    email: "daniel@danieloaks.net"
    editor: true
---


# DCMI Protocol

This is an attempt to create a mesh-based IRC server-to-server protocol to replace the existing spanning-tree protocols in use. In doing this, DCMI aims to minimise the impact of links between servers going down, reduce the possibility of netsplits and improve the speed of unicast messages through the network.


---


# Introduction

Currently, IRC servers link together in a spanning-tree layout. This is useful as it ensures no network loops exist with very low overhead and makes for very simple, linear processing of commands. However, this spanning-tree layout also makes it very noisy and obvious to end users when single links between servers go down, or any server not on the edge of the network itself. The noise of netsplits also help accentuate network attacks, and creates an 'end goal' for users attacking the network and attempting to disrupt ordinary network operations.

DCMI is an attempt to create an IRC server-to-server protocol using a mesh topology. DCMI attempts to use links in such a way that ensures backup links are available (if configured) and can be hot-swapped to. In this way, DCMI also attempts to reduce the possibility of netsplits by adding resiliency to the architecture of networks and to the S2S protocol itself.

DCMI is based off [Protocol Buffers](https://developers.google.com/protocol-buffers/), and the protobuf specification file goes into more detail about the messages that exist and their exact semantics.


---


# Design

DCMI draws heavily from the IP and routing protocols themselves. In incorporates elements such as link-state routing, failover links, and timestamps (similar to TS6) to ensure state is kept consistent across the network.


## Network State

In regular IRC exchanging state information is very simple. You send the IRC message to your neighbours, your neighbours evaluate and execute the messages one by one (as they are received in order). Conflicts are generally handled with a timestamp solution (as in TS6).

Since DCMI aims to avoid netsplits through automatic failover, network state must be exchanged in a different way. That is, failing over from an active link to a secondary link, without requiring a full re-sync of network information, requires a more carefully-designed method of exchanging state.

Various parts of the design of this blob of DCMI has been thrown around before in calls to make S2S protocols more efficient than the current text-based protocols we use. However, there are changes that must be made to accommodate the mesh-based topology of the network.


# Routing State

In regular spanning-tree topologies, route state is relatively simple. A link is either up or down (pinged occasionally to check this), and if it goes down everyone on the other side of the link disconnects from the network.

Routing state in DCMI requires a more in-depth protocol since it has to handle failover of downed links and avoiding network loops.

DCMI aims to take lessons from routing protocols in how this is designed.


## Routing Unicast Traffic

There are two ways of routing unicast traffic with DCMI (or specifically, two ways of deciding which interface to send a message out on). It does not matter which is chosen, so long as the entire network uses the same method. This is checked and enforced at link time.

### Spanning-Tree Unicast

The first is through following the spanning-tree model. This is very simple and replicates how IRC servers route traffic today. Essentially:

1. Which of my neighbours has the server I'm trying to reach? (via the created spanning tree)
2. Send the message to that neighbour.

### Meshy Unicast

The second method is through following a weighting algorithm such as dijkstra's algorithm. This method should be more quick, since it is not constricted to the spanning-tree model unnecessarily (so long as every server in the network reaches the same conclusion about which server is the right one to route to). However, it's more complex to route traffix with this model. Implementations may find it useful to precalculate a lookup table of servers on the network and which of their neighbours they should be routing it to (and recalculate as necessary whenever routing changes happen).
