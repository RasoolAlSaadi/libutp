# libutp-mod - A modified version of the uTorrent Transport Protocol library.

This modified version of libutp fixes some congestion control (LEDBAT) related
implementation problems of the original libutp.

## Congestion Control Related Problems of libutp

libutp has problems that cause fairness and performance issues, which are:
1) Does not backoff cwnd more frequently than once every 100ms regardless of RTT.
2) MAX_CWND_INCREASE_BYTES_PER_RTT (i.e. G × MSS) is effectively set to
   3000 bytes, allowing the congestion window growth by roughly two MSS rather
   than one MSS per RTT (faster than it would with standard TCP).
3) The fast recovery in libutp is unable to recover from retransmission losses
   (loss of the retransmitted packets) without relying on the retransmission
   timeout (RTO). This problem causes sending stalls (for one second) which
   reduces the throughput significantly in heavy packet-loss scenarios such as
   in AQM-enabled bottlenecks and wireless networks.

## libutp-mod Fixes

We fixed the issues above by modifying libutp as follows:
1) Use libutp’s existing smoothed RTT estimate to allow cwnd to backoff once
   per RTT (per RFC6817) on packet loss.
2) Modify MAX_CWND_INCREASE_BYTES_PER_RTT to ensure cwnd grows by only one MSS
   per RTT.
3) Allow lost retransmissions to themselves be retransmitted long before an RTO
   is triggered by utilising the selective ACK option and packet transmission time.


# libutp - The uTorrent Transport Protocol library.
Copyright (c) 2010 BitTorrent, Inc.

uTP is a TCP-like implementation of [LEDBAT][ledbat] documented as a BitTorrent
extension in [BEP-29][bep29]. uTP provides provides reliable, ordered delivery
while maintaining minimum extra delay. It is implemented on top of UDP to be
cross-platform and functional today. As a result, uTP is the primary transport
for uTorrent peer-to-peer connections.

uTP is written in C++, but the external interface is strictly C (ANSI C89).

## The Interface

The uTP socket interface is a bit different from the Berkeley socket API to
avoid the need for our own select() implementation, and to make it easier to
write event-based code with minimal buffering.

When you create a uTP socket, you register a set of callbacks. Most notably, the
on_read callback is a reactive callback which occurs when bytes arrive off the
network. The write side of the socket is proactive, and you call UTP_Write to
indicate the number of bytes you wish to write. As packets are created, the
on_write callback is called for each packet, so you can fill the buffers with
data.

The libutp interface is not thread-safe. It was designed for use in a
single-threaded asyncronous context, although with proper synchronization
it may be used from a multi-threaded environment as well.

See utp.h for more details and other API documentation.

## Example

See ucat.c. Build with:
 
   make ucat

## Building

uTP has been known to build on Windows with MSVC and on linux and OS X with gcc.
On Windows, use the MSVC project files (utp.sln, and friends). On other platforms,
building the shared library is as simple as:

    make

To build one of the examples, which will statically link in everything it needs
from libutp:

    cd utp_test && make

## Packaging and API

The libutp API is considered unstable, and probably always will be. We encourage
you to test with the version of libutp you have, and be mindful when upgrading.
For this reason, it is probably also a good idea to bundle libutp with your
application.

## License

libutp is released under the [MIT][lic] license.

## Related Work

Research and analysis of congestion control mechanisms can be found [here.][survey]

[ledbat]: http://datatracker.ietf.org/wg/ledbat/charter/
[bep29]: http://www.bittorrent.org/beps/bep_0029.html
[lic]: http://www.opensource.org/licenses/mit-license.php
[survey]: http://datatracker.ietf.org/doc/draft-ietf-ledbat-survey/
