# TCP delay optimization

This discussion revolves around 

- Nagle's Algorithm 

- Delayed ACK  
- TCP_CORK



### Nagle's Algorithm 

This algorithm says that TCP connection can have only one outstanding small segment that has not yet been acknoledged. No additional small segments can be sent until the acknowledgement is received. This gives the OS time to coalesce multiple data chunks into single packet.

This can be disabled with `TCP_NODELAY` flag on the network socket.PUSH flag on the packet also ignores Nagle.



### Delayed ACK

This is alternative to Nagle which got introduced at the same time indepedently. Here server waits for RTO timer to expire before sending an ACK of received data. This is in anticipation that more data might arrive then server waits before sending an ACK. `TCP_QUICKACK` will allow us to disable delayed ack.

More on RTO value of 200ms can be found [here](https://blog.titanwolf.in/a?ID=00950-621d274b-025d-4a06-af5a-e4aad169ab3b)

 

Nagle and Delayes ACK [problem](https://jvns.ca/blog/2015/11/21/why-you-should-understand-a-little-about-tcp/)

### TCP Corking

This is the mechanism which delays sending the packet until its full. Flushing the packet on the message boundries should be ideal way of dealing with it. `TCP_CORK` is the flag on the socket governs this behaviour.

TCP Corking means don't send any data (partial frames) smaller than the MSS until the application says so or until 200ms later.



TCP_NODELAY sends doesn't accumulate the logical packets before sending then as network packets, Nagle's algorithm does according the algorithm, and TCP_CORK does according to the application setting it. ([credit](https://stackoverflow.com/questions/22124098/is-there-any-significant-difference-between-tcp-cork-and-tcp-nodelay-in-this-use))





# Veyron Incident

### Disable Nagle for infra-AZ traffic

We recently observed latency spikes at the VATL layer after introduction of JLBRelay for TLS connection termination, spikes were occurring on the TCP path between JLBRelay and VATL over the loopback interface due to the combination of TCP Delayed ACK and Nagle optimization. Quick primer on related TCP optimizations

-  [TCP Delayed ACK](https://en.wikipedia.org/wiki/TCP_delayed_acknowledgment) is an optimization technique used to improve TCP throughput, where the receiver delays ACK response ( by 40ms ) in the anticipation of more data coming. This pause provides opportunity for the TCP window scaling to improve the throughput. TCP_QUICKACK socket option helps to prevent these delays to certain extent. 
- [Nagle](https://en.wikipedia.org/wiki/Nagle's_algorithm#:~:text=Nagle considers delayed ACKs a,as many small packets do.), on the other hand, has implemented sender side delays if the TCP connection's window size is smaller than TCP Maximum Segment Size (MSS). Nagle's optimization will hold off any further data send until it gets ACKs for the receiver for all the in-flight data. TCP_NODELAY socket option disables Nagle's optimization completely.

Both of these optimizations are important when the latency path is large, but the combination of both is capable of creating undesired behavior on the ultra-short latency path, like here over the loopback interface. 

**JLBRelay ↔ VATL Problem** 

- JLBRelay's socket was set up in the TCP_QUICKACK mode, but it is not a permanent mode. [By design](https://man7.org/linux/man-pages/man7/tcp.7.html#:~:text=TCP_QUICKACK (since Linux,to be portable.), socket can fall in and out of the QUICKACK mode depending on the Operating Systems' TCP optimization.
- VATL socket was set up with Nagle optimization.
- This TCP communication over loopback had a larger MTU of 65536 bytes which sets the TCP MSS to the same value.
- A latency spike used to occur when the receiver JLBRelay socket used to fall out of the QUICKACK mode and starts Delay ACK timer for 40ms, at the same time if smaller data fragments arrived at the VATL socket would trigger Nagle optimization of delaying sends. The TCP Delay timer on the JLBRelay socket was the only way out of this gridlock situation. 
- This resulted in elevated end user p99 latency stuck in the 40ms range. 


**Conclusion**
We do not have full control on disabling TCP Delayed ACK using TCP_QUICKACK socket option. But we can turn off Nagle with the TCP_NODELAY option for any regional intra-AZ network traffic to avoid degraded performance.



# MSS and MTU

- The Ethernet standard, for example, sets a maximum frame size at 1518 bytes. Ethernet headers are 18 bytes long, leaving 1500 bytes for the packet to use. Therefore, the packet’s MTU is 1500 bytes.
- MTU sets TCP MSS.
- IP4 and TCP headers usually add up to 40 bytes in total.  So, if we started with an MTU of 1500 bytes, we now have an MSS of 1460 bytes.
