# 概念

## 物理节点 Physics nodes

Gaspa 中的节点在部署时分为两种：`businessNode` `networkNode`

其中 `bussinessNode` 是指普通的区块节点，包括事务、存储和所有网络功能。（必须）
而 `networkNode` 是固定的，只提供网络拓补和网桥功能。（可选，仅当 businessNode 由于拓补问题在物理上就没有办法直接实现双向通信时，需要部署一个或几个所有 bussinessNode 都能访问的 networkNode。且 networkNode 会优先被作为 scheduler）

## 网络街区 networkBlock

一个 networkBlock 类似于 PaaS 中的一个 subnetwork，但是它是由 Gaspa 自动探测得的，把能够实现双向通信的相邻节点全部归为一个 block （即求物理层的连通块），作为一个 networkBlock。

networkBlock 具有层级性（即一个街区内可包含另外的街区）。

## 大师、调度器 Master and Scheduler

所有的节点分为三类： `master` `scheduler` `normal`

`normal` 节点是普通节点，仅负责数据存储、拓补存储、业务处理等职能。

`scheduler` 节点除 `normal` 功能外，对应一个 networkBlock 的所有节点的拓补管理以及带有 `schedule` 属性的 `message` 的 `loadBalance`。

`master` 节点负责所有 `sheduler` 的拓补管理以及带有 `schedule` 属性的 `message` 的 `loadBalance`。

`scheduler` 和 `master` 由内部根据网络拓补结构选举产生。

`master` 和 `scheduler` 区别：`scheduler` 只能对应管理 `normal`，`master` 只能对应管理 `scheduler`，而任何 node 可以同时作为 `scheduler` 和 `master`。

## 自动拓补细节

话不多说，上图：（!B 代表 bussinessNode, !N 代表 networkNode）

- Router M (OpenWRT, !B)
  - Router A (OpenWRT !N)
    - Server A1 (Linux, !B)
    - Server A2 (Linux)
      - Container A1.1 (Linux, !B)
      - Container A1.2 (Linux, !B)
      - Container A1.3 (Linux, !B)
      - Jail A1[network] (Linux, !N)
    - Server A3 (Windows, !B)
    - Server A4 (CoreOS, !N)
      - VM A4-1 (CoreOS, !B)
        - Container A4-1.1 (CoreOS, !B)
      - VM A4-2 (CoreOS, !B)
    - Server A5 (Redox, !B)
  - Router B (TP-Link)
    - PC B1 (Windows, !B)
    - NAS B2 (Linux, !N)
    - RaspberryPi B3 (Linux, !B)
  - Server X (Linux, !B)

这是一个示例结构。

Master: Router M, Router A, Server A4

Scheduler: Server A1, Jail A1[network], Server A3, VM A4-1, Server A5, NAS B2, Server X

## 通信：广播、低语和调度 Broadcast and Whisper and Schedule

`broadcast` 、 `whisper` 和 `schedule` 都是多个 `node` 之间通信的基本方式。他们都是一个 `node` 对其他 `node` 传送信息的方式，都是单向的。

`broadcast` 是指广播，可以发送给单独的 node，也可以同时发送给多个 node，虽然具有可靠性，保证节点能够接收到，但是数据结构共享，直接对应一个临时 chain，且 chain 完全共享，也就是说，除了被 notify 的节点能够保证收到数据，其他未被列入 broadcast queue 的所有 node 也都可以通过一定手段获得数据。`broadcast` 不具有双向通信的本领，也就是说，不保证有统一的 `messageID`。但是，在传播大数据给大量节点的场景，由于数据共享，效率比 whisper 高得多。

`whisper` 是指低语，也就是 node 与 node 的通信。即直接将数据传输给另外一个 node，数据是直接通过网络传输的，仅接受者能够获得。但是在传输大量的数据给大量节点的场景，数据会被重复发送和存储，效率较低。`whisper` 保证提供单独、统一的 `messageID`，所以可以用来进行双向通信。

`schedule` 是调度，基于 `whisper`，但是接收者不固定，由 `master` 和 `scheduler` 共同选举产生最佳接收者，包含 `loadBalance` 功能。

## 数据结构 Data Structure

// TODO: @xtlsoft: @LemonHX, you decide this point.

待定。
