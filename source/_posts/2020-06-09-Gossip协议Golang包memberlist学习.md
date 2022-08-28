---
title: Gossip协议Golang包memberlist学习
date: 2020-06-09 10:32:58
categories:	Go
tags:
---

> memberlist是一个Go库，它使用基于Gossip的协议管理集群成员和成员故障检测。

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=34218938&auto=1&height=32"></iframe>
Config 配置文件：

```go
type Config struct {
    // 节点名称，在集群中必须唯一
    Name string
    // Transport提供节点间通讯的基础服务，tcp、udp等
    Transport Transport
    // gossip peer绑定的地址，新节点可以通过任意一个节点的BindAddr
  	// 默认"0.0.0.0"、"7946"
    BindAddr string
    BindPort int

    // 这对ip、port是集群其他节点与自己通讯用的, 也就是外部可以访问到的ip和端口
    // cluster members. Used for nat traversal.
    AdvertiseAddr string
    AdvertisePort int

    // ProtocolVersion is the configured protocol version that we
    // will _speak_. This must be between ProtocolVersionMin and
    // ProtocolVersionMax.
    ProtocolVersion uint8

    // TCPTimeout is the timeout for establishing a stream connection with
    // a remote node for a full state sync, and for stream read and write
    // operations. This is a legacy name for backwards compatibility, but
    // should really be called StreamTimeout now that we have generalized
    // the transport.
    TCPTimeout time.Duration

    // IndirectChecks is the number of nodes that will be asked to perform
    // an indirect probe of a node in the case a direct probe fails. Memberlist
    // waits for an ack from any single indirect node, so increasing this
    // number will increase the likelihood that an indirect probe will succeed
    // at the expense of bandwidth.
    IndirectChecks int

    // RetransmitMult is the multiplier for the number of retransmissions
    // that are attempted for messages broadcasted over gossip. The actual
    // count of retransmissions is calculated using the formula:
    //
    //   Retransmits = RetransmitMult * log(N+1)
    //
    // This allows the retransmits to scale properly with cluster size. The
    // higher the multiplier, the more likely a failed broadcast is to converge
    // at the expense of increased bandwidth.
    RetransmitMult int

    // SuspicionMult is the multiplier for determining the time an
    // inaccessible node is considered suspect before declaring it dead.
    // The actual timeout is calculated using the formula:
    //
    //   SuspicionTimeout = SuspicionMult * log(N+1) * ProbeInterval
    //
    // This allows the timeout to scale properly with expected propagation
    // delay with a larger cluster size. The higher the multiplier, the longer
    // an inaccessible node is considered part of the cluster before declaring
    // it dead, giving that suspect node more time to refute if it is indeed
    // still alive.
    SuspicionMult int

    // SuspicionMaxTimeoutMult is the multiplier applied to the
    // SuspicionTimeout used as an upper bound on detection time. This max
    // timeout is calculated using the formula:
    //
    // SuspicionMaxTimeout = SuspicionMaxTimeoutMult * SuspicionTimeout
    //
    // If everything is working properly, confirmations from other nodes will
    // accelerate suspicion timers in a manner which will cause the timeout
    // to reach the base SuspicionTimeout before that elapses, so this value
    // will typically only come into play if a node is experiencing issues
    // communicating with other nodes. It should be set to a something fairly
    // large so that a node having problems will have a lot of chances to
    // recover before falsely declaring other nodes as failed, but short
    // enough for a legitimately isolated node to still make progress marking
    // nodes failed in a reasonable amount of time.
    SuspicionMaxTimeoutMult int

    // PushPullInterval is the interval between complete state syncs.
    // Complete state syncs are done with a single node over TCP and are
    // quite expensive relative to standard gossiped messages. Setting this
    // to zero will disable state push/pull syncs completely.
    //
    // Setting this interval lower (more frequent) will increase convergence
    // speeds across larger clusters at the expense of increased bandwidth
    // usage.
    PushPullInterval time.Duration

    // ProbeInterval and ProbeTimeout are used to configure probing
    // behavior for memberlist.
    //
    // ProbeInterval is the interval between random node probes. Setting
    // this lower (more frequent) will cause the memberlist cluster to detect
    // failed nodes more quickly at the expense of increased bandwidth usage.
    //
    // ProbeTimeout is the timeout to wait for an ack from a probed node
    // before assuming it is unhealthy. This should be set to 99-percentile
    // of RTT (round-trip time) on your network.
    ProbeInterval time.Duration
    ProbeTimeout  time.Duration

    // DisableTcpPings will turn off the fallback TCP pings that are attempted
    // if the direct UDP ping fails. These get pipelined along with the
    // indirect UDP pings.
    DisableTcpPings bool

    // DisableTcpPingsForNode is like DisableTcpPings, but lets you control
    // whether to perform TCP pings on a node-by-node basis.
    DisableTcpPingsForNode func(nodeName string) bool

    // AwarenessMaxMultiplier will increase the probe interval if the node
    // becomes aware that it might be degraded and not meeting the soft real
    // time requirements to reliably probe other nodes.
    AwarenessMaxMultiplier int

    // GossipInterval and GossipNodes are used to configure the gossip
    // behavior of memberlist.
    //
    // GossipInterval is the interval between sending messages that need
    // to be gossiped that haven't been able to piggyback on probing messages.
    // If this is set to zero, non-piggyback gossip is disabled. By lowering
    // this value (more frequent) gossip messages are propagated across
    // the cluster more quickly at the expense of increased bandwidth.
    //
    // GossipNodes is the number of random nodes to send gossip messages to
    // per GossipInterval. Increasing this number causes the gossip messages
    // to propagate across the cluster more quickly at the expense of
    // increased bandwidth.
    //
    // GossipToTheDeadTime is the interval after which a node has died that
    // we will still try to gossip to it. This gives it a chance to refute.
    GossipInterval      time.Duration
    GossipNodes         int
    GossipToTheDeadTime time.Duration

    // GossipVerifyIncoming controls whether to enforce encryption for incoming
    // gossip. It is used for upshifting from unencrypted to encrypted gossip on
    // a running cluster.
    GossipVerifyIncoming bool

    // GossipVerifyOutgoing controls whether to enforce encryption for outgoing
    // gossip. It is used for upshifting from unencrypted to encrypted gossip on
    // a running cluster.
    GossipVerifyOutgoing bool

    // EnableCompression is used to control message compression. This can
    // be used to reduce bandwidth usage at the cost of slightly more CPU
    // utilization. This is only available starting at protocol version 1.
    EnableCompression bool

    // SecretKey is used to initialize the primary encryption key in a keyring.
    // The primary encryption key is the only key used to encrypt messages and
    // the first key used while attempting to decrypt messages. Providing a
    // value for this primary key will enable message-level encryption and
    // verification, and automatically install the key onto the keyring.
    // The value should be either 16, 24, or 32 bytes to select AES-128,
    // AES-192, or AES-256.
    SecretKey []byte

    // The keyring holds all of the encryption keys used internally. It is
    // automatically initialized using the SecretKey and SecretKeys values.
    Keyring *Keyring

    // Delegate and Events are delegates for receiving and providing
    // data to memberlist via callback mechanisms. For Delegate, see
    // the Delegate interface. For Events, see the EventDelegate interface.
    //
    // The DelegateProtocolMin/Max are used to guarantee protocol-compatibility
    // for any custom messages that the delegate might do (broadcasts,
    // local/remote state, etc.). If you don't set these, then the protocol
    // versions will just be zero, and version compliance won't be done.
    Delegate                Delegate
    DelegateProtocolVersion uint8
    DelegateProtocolMin     uint8
    DelegateProtocolMax     uint8
    Events                  EventDelegate
    Conflict                ConflictDelegate
    Merge                   MergeDelegate
    Ping                    PingDelegate
    Alive                   AliveDelegate

    // DNSConfigPath points to the system's DNS config file, usually located
    // at /etc/resolv.conf. It can be overridden via config for easier testing.
    DNSConfigPath string

    // LogOutput is the writer where logs should be sent. If this is not
    // set, logging will go to stderr by default. You cannot specify both LogOutput
    // and Logger at the same time.
    LogOutput io.Writer

    // Logger is a custom logger which you provide. If Logger is set, it will use
    // this for the internal logger. If Logger is not set, it will fall back to the
    // behavior for using LogOutput. You cannot specify both LogOutput and Logger
    // at the same time.
    Logger *log.Logger

    // Size of Memberlist's internal channel which handles UDP messages. The
    // size of this determines the size of the queue which Memberlist will keep
    // while UDP messages are handled.
    HandoffQueueDepth int

    // Maximum number of bytes that memberlist will put in a packet (this
    // will be for UDP packets by default with a NetTransport). A safe value
    // for this is typically 1400 bytes (which is the default). However,
    // depending on your network's MTU (Maximum Transmission Unit) you may
    // be able to increase this to get more content into each gossip packet.
    // This is a legacy name for backward compatibility but should really be
    // called PacketBufferSize now that we have generalized the transport.
    UDPBufferSize int

    // DeadNodeReclaimTime controls the time before a dead node's name can be
    // reclaimed by one with a different address or port. By default, this is 0,
    // meaning nodes cannot be reclaimed this way.
    DeadNodeReclaimTime time.Duration

    // RequireNodeNames controls if the name of a node is required when sending
    // a message to that node.
    RequireNodeNames bool
    // CIDRsAllowed If nil, allow any connection (default), otherwise specify all networks
    // allowed to connect (you must specify IPv6/IPv4 separately)
    // Using [] will block all connections.
    CIDRsAllowed []net.IPNet
}
```

