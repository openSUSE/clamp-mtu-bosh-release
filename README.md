# CLAMP MSS TO MTU

Some clouds don't use jumbo packets, which in combination with software defined networking often leads to containers with a very low MTU.

Normally the kernel does [PMTU discovery](https://en.wikipedia.org/wiki/Path_MTU_Discovery), however if ICMP fragmentation needed packets can't reach the emitting container, this will not help.
Instead we use the firewall on the container host to reduce the maximum segment size (MSS) of all forwarded connections.

As discussed in https://github.com/cloudfoundry/guardian/issues/51 and others, this is considered an infrastructure problem and probably won't get consideration in releases which use or set up these containers.

However this BOSH release can be added to an instance group, or in the [runtime config](https://bosh.io/docs/runtime-config.html). It will monitor iptables and re-insert the rule if it get's lost, i.e. by reboots, cloud-checks, etc.

In our tests the speed of `git clone` doubled, since fragmentation traffic was not necessary any more.

Using `clamp-mss-to-mtu` in `iptables` forwarding chain, has several advantes over using a fixed interface MTU:

* the actual MTU of the path to the destination IP is taken into consideration
* all traffic is affected, no need to configure individual daemons
