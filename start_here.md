# start here: how to use rdma-forensics

This repository is an engineering investigation portfolio built from the perspective of an
experienced network architect. Beginning with routed fabrics, RoCEv2, PFC, ECN, DCQCN, and
BGP, then traces the transport deeper into the host-side verbs layer.

The goal is to investigate what happens within the application, pyverbs, kernel driver, and
NIC before a RoCEv2 packet reaches the wire. Each conclusion should be supported by code,
packet captures, counter data, and measured results.

