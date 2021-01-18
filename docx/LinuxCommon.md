### 僵尸进程、孤儿进程
- 僵尸进程：一个进程用fork创建一个子进程，、如果子进程退出时父进程未调用wait或waitpid来获取子进程的状态信息，导致子进程状描述符信息一直被留在系统中，进程号一直被占用（干掉子进程或者干掉父进程让子进程变成孤儿进程由init处理）
- 孤儿进程：父进程退出后，其子进程就变成孤儿进程，孤儿进程将由init进程来管理

### select/poll/epoll
可以监听多个设备的文件描述符（等待队列），遍历等待队列，当有任意一个描述符满足条件（可读、可写）时，select/poll返回，否则进行休眠等待有可读或者可写的文件描述符时被唤醒。

### cgroups、ns
#### cgroups（controller system sources for groups）
- Resource Limitation：限制资源使用（内存上限、文件缓存等）
- Prioritization：优先级控制（CPU的使用、磁盘IO）
- Accounting：一些审计或者统计（主要为了计费）
- Control：挂起进程、恢复进程执行

#### namespaces
提供一种资源隔离方案，ns下的资源对其他ns不可见，多个ns下可以同时出现相同的pid的进程