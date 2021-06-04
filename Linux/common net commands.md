# Introduction

This document lists some common network commands in Linux.

##### How to check what driver is used by a NIC

```\
test-server:[~]$ ethtool -i eno1
driver: igb
version: 5.6.0-k
firmware-version: 1.67, 0x80000faa, 19.5.12
expansion-rom-version:
bus-info: 0000:18:00.0
supports-statistics: yes
supports-test: yes
supports-eeprom-access: yes
supports-register-dump: yes
supports-priv-flags: yes
```

other information like firmware version and bus-info (the same as the output of `lspci` command) is also included.

##### How to check features supported by NIC

```bash
test-server:[~]$ ethtool -k eno1
Features for eno1:
rx-checksumming: on
tx-checksumming: on
        tx-checksum-ipv4: on
        tx-checksum-ip-generic: off [fixed]
        tx-checksum-ipv6: on
        tx-checksum-fcoe-crc: off [fixed]
        tx-checksum-sctp: off [fixed]
scatter-gather: on
        tx-scatter-gather: on
        tx-scatter-gather-fraglist: off [fixed]
tcp-segmentation-offload: on
        tx-tcp-segmentation: on
        tx-tcp-ecn-segmentation: on
        tx-tcp-mangleid-segmentation: off
        tx-tcp6-segmentation: on
udp-fragmentation-offload: off
generic-segmentation-offload: on
generic-receive-offload: on
large-receive-offload: off [fixed]
rx-vlan-offload: on [fixed]
tx-vlan-offload: on [fixed]
ntuple-filters: off [fixed]
receive-hashing: off [fixed]
highdma: on
rx-vlan-filter: off [fixed]
vlan-challenged: off [fixed]
tx-lockless: off [fixed]
netns-local: off [fixed]
tx-gso-robust: off [fixed]
tx-fcoe-segmentation: off [fixed]
tx-gre-segmentation: off [fixed]
tx-gre-csum-segmentation: off [fixed]
tx-ipxip4-segmentation: off [fixed]
tx-ipxip6-segmentation: off [fixed]
tx-udp_tnl-segmentation: off [fixed]
tx-udp_tnl-csum-segmentation: off [fixed]
tx-gso-partial: off [fixed]
tx-sctp-segmentation: off [fixed]
tx-esp-segmentation: off [fixed]
tx-udp-segmentation: off [fixed]
fcoe-mtu: off [fixed]
tx-nocache-copy: off
loopback: off [fixed]
rx-fcs: off [fixed]
rx-all: off [fixed]
tx-vlan-stag-hw-insert: off [fixed]
rx-vlan-stag-hw-parse: off [fixed]
rx-vlan-stag-filter: off [fixed]
l2-fwd-offload: off [fixed]
hw-tc-offload: off [fixed]
esp-hw-offload: off [fixed]
esp-tx-csum-hw-offload: off [fixed]
rx-udp_tunnel-port-offload: off [fixed]
tls-hw-tx-offload: off [fixed]
tls-hw-rx-offload: off [fixed]
rx-gro-hw: off [fixed]
tls-hw-record: off [fixed]
```

##### How to turn on/off NIC features

```bash
test-server:[~]$ sudo ethtool -K eno1 sg off
Actual changes:
scatter-gather: off
        tx-scatter-gather: off
generic-segmentation-offload: off [requested on]
```

Note that NIC features have fixed abbreviations, e.g., sg is short for scatter-gather. For more abbreviations, please refer the *manpage* of `ethtool`, abbreviations are listed under the -K item.  

##### How to check TCP metrics cache

Linux will cache TCP metrics (e.g., cwnd, rtt and rttvar) in IP granularity in the routing table, if you want to check those metrics, you can use:

```bash
[101.6.30.141]-[guest@fit-server-141]-[~]$ ip tcp_metrics show
34.76.80.167 age 1228727.064sec source 101.6.30.141
91.189.91.49 age 939401.980sec source 101.6.30.141
101.6.8.193 age 40093.888sec ssthresh 2 cwnd 1 rtt 8069us rttvar 10709us source 101.6.30.141
45.79.85.237 age 858580.412sec cwnd 10 rtt 178594us rttvar 178594us source 101.6.30.141
104.16.21.35 age 1384296.376sec cwnd 10 rtt 164164us rttvar 164164us source 101.6.30.141
......
```

if you want to delete TCP metrics of a specific IP address, you can use:

```bash
[101.6.30.141]-[guest@fit-server-141]-[~]$ sudo ip tcp_metrics delete 101.6.8.193
```

or you can delete all caches with:

```bash
[101.6.30.141]-[guest@fit-server-141]-[~]$ sudo ip tcp_metrics flush
```

