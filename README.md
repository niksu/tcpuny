# TCPuny: a C# implementation of TCP

Diagram showing the different entities: packet processing, active parts, user=program.
  TCP thread
  Sockets interface
  Program thread (client, server, peer, ...)

uses fixed-size segments -- no adaptation of MSS based on "distance" between peers.
uses fixed-size windows -- no flow control and congestion control
  (hmm maybe flowcontrol easily doable?)
(ssthresh, rto calculation).
deals with:
  loss (tretransmission, uses fixed timer)
  duplication
  out-of-order delivery
also doesn't support any options (e.g., mss, window scaling, fast open), and features like URG.
and ignores control messages (via ICMP)
and doesn't provide any support for host naming (to be resolved via DNS) or ARP.

timers: (e.g., lack keepalive, persist, fin-wait)
* connection-establishment: expiry after SYN (remove TCB)
* retransmission
* 2MSL to remove TCB

no Nagle algorithm (for batching sends),
and immediate ACKing (no delaying).

TCP passive open, handshake
  (we don't need an active open at this stage)
  steady-state behaviour
  flow control? (advertised window)
  congestion control? (congestion window)
  timeouts? (initially static, then make adaptive)
  RTT estimates? (initially static, then make adaptive)
ISN?


Factored the implementation:
  - protocol parsing done by the time we see the packets
  - interface with the program's thread. use Berkeley interface, since more
    familiar to programmers.
  - TCP machine, running in the TCP thread.
      lookups
        what data structures to maintain
      timers -- see what Jonny used in NAT
      interact with programs -- receiving and sending events
  - what about h2n, ARP, addressing, etc?


# Configuration
Activate IP forwarding. On OSX:
```
sysctl -w net.inet.ip.forwarding=1
```

TCP is given a bunch of parameters, see echo_wiring.json


# Example
Example programs:
  echo server -- test this from telnet client

Using the .NET Socket class:
```
$ sudo mono examples/Bin/Echo.exe --address 127.0.0.1 --port 7000 -v
```

Using TCPuny:
```
$ sudo mono ./Bin/Pax.exe examples/TCP/echo_wiring.json examples/Bin/Examples.dll
```

From another terminal, run
```
$ telnet 127.0.0.1 7000
```
