使用 CAS 原子指令来处理对数据的并发访问，这是非阻塞算法得以实现的基础。
head/tail 节点都允许滞后，也就是说它们并非总是指向队列的头/尾节点，这是因为并不是每次操作队列都更新 head/tail，和 LinkedTransferQueue 一样，使用了一个“松弛阀值（2）”， 当前指针距离
head/tail 节点大于2时才会更新 head/tail，这也是一种优化方式，减少了CAS指令的执行次数。
由于队列有时会处于不一致状态。为此，ConcurrentLinkedQueue 对节点使用了“不变性”和“可变性”来约束非阻塞算法的正确性（后面会详细说明）。
使用“自链接”方式管理出队节点，这样一个自链接节点意味着需要从head向后推进

从tail节点向后自旋查找 next 为 null 的节点，也就是最后一个节点（因为 tail 节点并不是每次都更新，所以我们取到的 tail 节点有可能并不是最后一个节点），然后CAS插入新增节点

上面我们提到过：并不是每次操作都会更新 head/tail 节点，而是使用了一个“松弛阀值”，这个“松弛阀值”就体现在上面源码中if (p != t)这一行，p初始是等于tail的，如果向后查找了一次以上才找到最后一个节点，再加上新增的节点，说明tail已经跳跃了两个（或以上）节点，此时才会CAS更新tail，这里也算是一种编程技巧
