# start here: how to use rdma-forensics

This repository is an engineering investigation portfolio built from the perspective of an
experienced network architect. Beginning with routed fabrics, RoCEv2, PFC, ECN, DCQCN, and
BGP, then traces the transport deeper into the host-side verbs layer.

The goal is to investigate what happens within the application, pyverbs, kernel driver, and
NIC before a RoCEv2 packet reaches the wire. Each conclusion should be supported by code,
packet captures, counter data, and measured results.



## Repository components

| Label | Meaning | Examples |
|---|---|---|
| Provided plumbing | Working support code that is not the learning exercise | `verbs/handshake.py`, plotting, packet parsing, setup scripts |
| Student implementation | Student-written code that exercises the verbs API | `RCConnection` transitions, posting work requests, polling, counter diffing |
| pyverbs object | A Python wrapper around an RDMA resource or attribute structure | `Context`, `PD`, `MR`, `CQ`, `QP`, `QPAttr` |
| Driver or NIC value | A value assigned or reported below the student program | GID table entry, QP number, memory key, completion status |
| Investigation evidence | An output that supports a portfolio claim | pcap, counter delta, benchmark CSV, plot, casefile finding |

Each TODO is resolved by first identifying the Python object, the values it needs,
the side effect it causes, and how that side effect is verified.

## Reading order

1. Read this page.
2. Read [python_for_rdma.md](python_for_rdma.md).
3. Read manual Part I, sections 1 through 4.
4. Read `verbs/handshake.py`. The connection code calls it.
5. Follow [investigation_01_connection.md](investigation_01_connection.md) one checkpoint at a time.
6. Continue with the remaining investigation guides only after the previous casefile is complete.

| Order | Guide | Question answered |
|---|---|---|
| 1 | [Investigation 1](investigation_01_connection.md) | How does an RC QP acquire local resources, exchange peer coordinates, and reach RTS? |
| 2 | [Investigation 2](investigation_02_write_read.md) | How do one-sided READ and WRITE target remote memory, and what appears in the RETH? |
| 3 | [Investigation 3](investigation_03_atomics.md) | How does the remote HCA serialize atomic operations on an eight-byte value? |
| 4 | [Investigation 4](investigation_04_multiqp.md) | How do many QPs share host resources and appear as flows to the fabric? |

## RC connection sequence

Every file in the repository should describe this same order:

```text
RCConnection.open()
    Create Context, PD, CQ, MR, and QP
    QP state after creation: RESET

RCConnection.to_init()
    Bind the QP to the local port and declare allowed remote operations
    QP state after success: INIT

post_recv(...)
    Supply receive buffers before the peer can send

RCConnection.local_record()
    Query local GID and collect local QPN, PSN, RKey, and buffer address

handshake.exchange(...)
    Exchange ConnRecord objects over an ordinary TCP control connection
    Returned value: the peer's ConnRecord

RCConnection.to_rtr(remote)
    Install the peer QPN, peer PSN, and routed address vector
    QP state after success: RTR

RCConnection.to_rts()
    Install the local send PSN and retry behavior
    QP state after success: RTS
```

`ConnRecord` is created near RESET to INIT because all local values already exist.
The peer record is consumed by INIT to RTR.

## Control path and data path

The handshake and the data path are separate:

| Path | Protocol | Purpose |
|---|---|---|
| Bootstrap or control path | TCP socket in this lab | Exchange QPN, PSN, GID, RKey, and buffer address |
| RDMA data path | RoCEv2 over UDP destination port 4791 | Carry SEND, WRITE, READ, atomic, ACK, and NAK packets |

The TCP port selected with `--port` is a rendezvous port the two processes use to exchange
coordinates before their QPs can communicate.

This lab protocol is intentionally simple and is not a production connection manager. It has
no authentication, authorization, encryption, replay protection, or lifecycle management.
Production systems commonly use RDMA CM or an authenticated application rendezvous service.

## Completing a checkpoint

For every TODO:

1. State the current QP state or resource state.
2. List each input and its Python type.
3. Mark each value as local, remote, library constant, or generated.
4. Write the smallest implementation that can be verified.
5. Run the checkpoint command from the investigation guide.
6. Record the observed output before proceeding.

If an operation fails, capture the exact exception, QP state before the call, attribute mask,
device name, port number, GID index, backend (`rxe` or `mlx5`), and rdma-core version.

## Casefile contents

Each completed investigation includes a casefile with:

- the hypothesis and why it matters to AI infrastructure;
- the topology and exact software and hardware environment;
- the commands and parameters used;
- a packet or counter prediction made before the run;
- actual packet, counter, latency, or bandwidth evidence;
- a finding that distinguishes application, host, NIC, and fabric causes;
- limitations and the next experiment that would falsify the finding.

## Skills exercised

- Routed fabric architecture: RoCEv2, PFC, ECN, DCQCN, and BGP
- Verbs layer implementation  with pyverbs
- Packet capture / counter analysis
- Latency / bandwidth measurement
- Fault isolation across application, host, NIC,  fabric
