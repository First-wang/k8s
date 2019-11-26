# etcd

> A highly-available key value store for shared configuration and service discovery.

etcd 是一个分布式的，一致性的 key-value 存储。

etcd 采用 Raft 作为保证分布式系统强一致性的算法。

etcd 能力：
- Put(key, value) / Del(key)
- Get(key) / Get(keyFrom, keyEnd)
- Watch(key/keyPrefix)
- Transactions(if/then/else ops).Commit()
- Leases:Grant/Revoke/KeepAlive 

#### etcd 数据版本

- term 全局单调递增，64bit，代表 master 节点更改次数
- revision 全局单调递增，64bit，代表全局数据更改次数
- keyValue:
    - create_revision 创建此 k-v 的revision
    - mod_revision 修改此 k-v 的revision
    - version 相当于修改次数

etcd 的 Watch 机制就通过数据版本变化发现。


