# Network



## Ethernet and NICs







## TCP

This discussion revolves around

* Nagle's Algorithm
* Delayed ACK
* TCP\_CORK

### Nagle's Algorithm

This algorithm says that TCP connection can have only one outstanding small segment that has not yet been acknoledged. No additional small segments can be sent until the acknowledgement is received. This gives the OS time to coalesce multiple data chunks into single packet.

This can be disabled with `TCP_NODELAY` flag on the network socket.PUSH flag on the packet also ignores Nagle.

### Delayed ACK

This is alternative to Nagle which got introduced at the same time indepedently. Here server waits for RTO timer to expire before sending an ACK of received data. This is in anticipation that more data might arrive then server waits before sending an ACK. `TCP_QUICKACK` will allow us to disable delayed ack.

More on RTO value of 200ms can be found [here](https://blog.titanwolf.in/a?ID=00950-621d274b-025d-4a06-af5a-e4aad169ab3b)

### TCP Corking

This is the mechanism which delays sending the packet until its full. Flushing the packet on the message boundries should be ideal way of dealing with it. `TCP_CORK` is the flag on the socket governs this behaviour.
