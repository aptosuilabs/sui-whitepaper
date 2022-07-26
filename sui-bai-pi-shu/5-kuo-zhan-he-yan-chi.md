# 5 扩展和延迟

Sui 系统允许通过将更多资源（即一台机器内或多台机器上的 CPU、内存、网络和存储）用于处理事务的权限进行扩展。更多的资源会提高处理交易的能力，从而增加为这些资源提供资金的费用收入。更多的资源也会导致更低的延迟，因为无需等待必要的资源可用就可以执行操作。

吞吐量。为了确保更多资源以准线性方式增加容量，Sui 设计积极减少了瓶颈和需要在权限内全局锁定的同步点。处理事务被明确地分为两个阶段，即（1）确保事务对特定版本的拥有或共享对象具有独占访问权，以及（2）随后执行事务并提交其效果。

阶段 (1) 要求事务以对象的粒度获取分布式锁。对于拥有的对象，这是通过可靠的广播原语执行的，不需要权限内的全局同步，因此可以通过 ObjID 对多台机器的锁管理进行分片来扩展。对于涉及共享对象的事务，需要使用共识协议进行排序，该协议确实对这些事务施加了全局顺序，并且有可能成为瓶颈。然而，工程高吞吐量共识协议 \[9] 的最新进展表明，顺序执行是状态机复制的瓶颈，而不是排序。在 Sui 中，排序仅用于确定输入共享对象的版本，即递增对象版本号并将其与事务摘要相关联，而不是执行顺序执行。

阶段 (2) 发生在所有输入对象的版本为权威所知（并且在权威之间安全地达成一致）并且涉及 Move 交易的执行及其效果的承诺。一旦知道输入对象的版本，就可以完全并行执行。移动多核或物理机上的虚拟机读取版本化的输入对象、执行并将生成的对象写入存储。对象和事务的存储（除了顺序锁映射）的一致性要求非常宽松，允许每个权限内部使用可扩展的分布式键值存储。执行是幂等的，即使处理执行的组件发生崩溃或硬件故障也很容易恢复。

因此，彼此没有因果关系的事务的执行可以并行进行。因此，智能合约设计者可以在其合约中设计对象和操作的数据模型，以利用这种并行性。

检查点和状态承诺是根据关键事务处理路径计算的，不会阻止新事务的处理。这些涉及对已提交数据的读取操作，而不是在事务达到最终确定之前需要计算和协议。因此，它们不会影响处理新事务的延迟或吞吐量，并且它们本身可以分布在可用资源中。

读取可以从非常积极的、可扩展的缓存中受益。权威机构签署并提供轻客户端读取所需的所有数据，这些数据可以由分布式存储作为静态数据提供。证书作为交易和对象的完整因果历史的信任根。状态承诺进一步允许整个系统对所有状态和处理的交易具有定期的全球信任根，至少在每个时期或更频繁地处理。

潜伏。智能合约设计者可以灵活地控制他们定义的操作的延迟，具体取决于它们是否涉及拥有或共享的对象。拥有的对象在执行和提交之前依赖于可靠的广播，这需要两次往返到法定人数才能达到最终确定性。另一方面，涉及共享对象的操作需要一致的广播来创建证书，然后在共识协议中进行处理，从而导致延迟增加（截至 \[9] 需要 4 到 8 次往返仲裁）。
