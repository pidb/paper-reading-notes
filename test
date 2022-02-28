#paper #b-tree #b-tree并发控制
## 1. 介绍
本论文研究了B*-tree (后文简称B-tree) 的并发控制协议. 第2节定义了本文中使用的术语. 在第3街, 介绍了并发问题和基本的解决方案. 在第4节, 提出了要研究的一般模式. 在第5节中, 证明了一般模式是无死锁的. 第6街对该模式定量分析, 以调整参数来优化模式的性能. 在第7节简要讨论了该模式的一些扩展.

## 2. B*-trees 定义
每个 entry 是一个 pair (entry key, associated information).  key 通常是排序好的, 本论文不讨论关联信息. B-tree 有以下属性:
1.  所有的 entries 存储在叶节点. 每个叶节点包含 $\mu$ 个entries.
2. 从根节点 root 到任意点叶节点路径相同
3. 所有的非叶节点 (internal nodes) 包含 $p_0, r_1,p_1, r_2,...r_{\mu}, p_{\mu}$ 元素,  其中 $p_i$ 指向直接后代节点, $r_i$ 是可以比较的 key (也成 *reference key*), 用于分隔后代节点, $p_i$ 指向的子树的 $key$ 都小于 $r_i$, $p_{i+1}$ 指向的子树的 $key$ 都大于等于 $r_i$, 
4. 除了根节点, 所有节点都满足 $k \le \mu \le 2k$.

### B*-trees操作
操作分为三种
- search
- insert
- delete
search 操作定义为 reader. search 操作不更改树. -insert 和 delete 称为 updater. 
B-tree 结构重要的事实：
- 当更新程序尝试 insert/delete 和 扫描节点 $n$ 时，可以很轻松的检测 $n$ 上的一个充分条件：任何祖先节点都不会受到insert/delete的影响
- 如果条件满足，$n$ 是 **safe** 的，否则是 **unsafe的**。

### 确定所操作节点的安全性
B-tree 确定节点是否安全的标准非常简单：
- 在 insert 时，如果 $\mu < 2k$ , 节点是安全的  
- 在 delete 时，如果 $\mu > k$ ，节点时安全的
![[Pasted image 20220227155604.png]]

例如 Fig.1 插入 (186) 时, 如何根据定义确定节点 $n$ 的安全性:
- 首先读取根节点 (节点1)，并且确定这个节点是 safe的，然后继续向下
- 到节点3，此时 $\mu = 2k$, 检测到该节点是 unsafe 的
- 最后移动到节点 11，该节点是 safe 的

## 3. 基本问题
最简单的并发控制是严格串行（strictly serialize）：整个 B-tree 使用 latch 锁住，但这样会降低并发性，使得操作成为串行的。
论文提出了 B-tree 三种并发访问的解决方案。对于每一种解决方案，都会给 *reader* 和 *updater* 设计一个协议（protocol）。
锁由调度程序根据进程的请求授予。论文假设 （除了方案3），锁调度程序以 *FIFO* 的顺序处理锁请求。该顺序是通过为树中的每个节点 $n$ 设置一个 *FIFO* 队列来维护的。请求锁定某个节点的进
程将被放在该节点队列的末尾，而锁调度程序将为队列开头的进程提供服务。

### 锁的相容性 （compatibility）
- 任意两个节点之间的边意味着两个不同的进程可以能使在一个节点 $n$ 上持有这些锁
- 没有边表明两个不同的进程不能同时在一个节点上持有这些锁
如 Fig.2, $p$  锁和 $\xi$  是不相容的，因为它们之间没有边连接。
![[Pasted image 20220227201304.png]]
### Solution 1
该方案基本上是由 Metzger [11] 提出的, 它派生自最简单的并发控制协议。该方案使用两种类型的锁：
1. $p\text{-lock}$ 或者说是读锁（read lock）
2. $\xi \text{-lock}$ 或者说是排他锁（exclusive lock）

> $p\text{-lock}$ 和 $\xi \text{-lock}$ 是不相容的，也就是说任意的两个进程（Process）不能在一个节点 $n$ 上同时持有它们。这个约束由锁调度器（lock scheduler）强制执行。

**reader 的协议为：**
			0) Place $p\text{-lock}$ on root;
			1) Get root and make it the current node;
			2) **While** current node is not a leaf node **do**
					   {Exactly one p-lock is held by process}
					   **begin**
						3) Place p-lock on appropriate son of current node;
						4) Release p-lock on current node;
						5) Get son of current node and make it current;
				     **end** **mainloop**

通过执行这个协议，reader 可以扫描 B-tree, 从 root 节点开始，移动到 leaf 节点。

**updater 的协议为：**
		0) Place $\xi \text{-lock}$ on root;
			1) Get root and make it the current node;
			2) **While** current node is not a leaf node **do**
					   {number of $\xi \text{-locks}$ held $\ge$ 1}
					   **begin**
						3) Place $\xi \text{-locks}$  on appropriate son of current node;
						4) Get son and make it the current node;
						5) **if** current node is safe
								**then** release all locks held on ancestors of current node
						**end mainloop**

Fig.3 解释了 updater 协议的骨架。$a,b$ 是 safe，$c$ 是unsafe。开始执行主循环前，$\xi \text{-lock}$ 设置在 $a$ 上。
根据协议，会发生以下事件：
1. [Step 3] A ~-lock is requested on node b.
2. [Step 4] After ~-lock is granted, node b is retrieved.
3. [Step 5] Since node b is safe, the i-lock on node a is released, thereby, allowing other updaters or readers to access node a.
4. [Step 3] A i-lock is requested on node c.
5. [Step 4] After i-lock is granted, node c is retrieved.
6. [Step 5] Since node c is unsafe, the i-lock on node b is kept.
7. [Step 3] A i-lock is requested on node d.
8. [Step 4] After ~-lock is granted, node d is retrieved.
9. [Step 5] Since node d is safe, the i-locks on nodes b and c can be released.

**方案1协议的优缺点：**
- 优点：只需要一个简单的协议（reader protocl 和 updater protocol）就可以比开始描述的协议获得更合理的并发性
- 缺点 ：当更新子树时，updater 首先使用 $\xi \text{-lock}$ 锁住root 节点（从而阻止其他进程对这个子树的访问），即使大多数情况下，更新对这个 root 根本没有影响。这样限制了并发性，在更高的并发访问下性能不佳。

为了实现更高的并发性，可以让 updater 表现得像 reader 一样。这就引出了下一个解决方案。

### Solution2
该方案使用和方案1一样的锁，reader的协议和一也是一样的，区别在于 updaer 协议：
		0) Place $p \text{-lock}$ on root;
			1) Get root and make it the current node;
			2) **While** current node is not a leaf node **do**
					   {number of $\xi \text{-locks}$ held $\ge$ 1}
					   **begin**
						3)  **if**  son is not a leaf node
								**then** place $p \text{-lock}$ on appropriate son
								**else** place $\xi \text{-lock} on appropriate son;
						4) Release lock on current node;
						5) Get son and make it the current node
						**end mainloop**
			6) {A leaf node has been reached}
				   **If** current node is unsafe
				    **then** release all locks and repeat access to the tree, this time
							using the protocol for an updater as in Solution 1;

通过该协议，updater 像 reader 一样从root节点沿着树向下执行，直到叶节点，该过程中，是以 $p\text{-lock}$ 锁定的。
但是，如果发现更新会影响树中更高的节点，那么到目前为止所做的所有分析都将丢失，需要使用方案1中的 updater 协议重启 （注意，这产生了一个 reader 扫描树的开销）。

**方案2协议的优缺点：**
- 优点：方案2 是乐观执行的，这带来了更高的并发性（在冲突很少的情况下）。根据 B-tree 的性质，大约每 $k$ 次对树的更新才会发生一次，而实际系统中 $k$ 总是选择很大的值 [2d]。
- 缺点 ：如果发生冲突，需要重启，然后使用悲观的方法重试（方案1中reader的协议），这产生了一个扫描周期的开销。

如果产生的开销比较重要（例如在非常深的树），那么方案3更有吸引力。

### Solution3
该方案使用三种类型的锁：$p\text{-lock}$， $\alpha\text{-lock}$,  $\xi \text{-lock}$. 
![[Pasted image 20220227224925.png]]
锁的相容性如 Fig.4.  在Fig.4 中，虚线表示可以从 $\alpha$ 转换为 $\xi$ ， 即锁可以转换。

从$\alpha$ 锁转换到 $\xi$ 类型的锁，进程需要首先在节点上持有 $\alpha$ 类型的锁。当发出转换请求时，进程将被放在相关节点队列的开头（即锁转换请求在其他请求之前先处理）。如果可以转换（许可），则该进程现在在该节点上持有第二种类型的锁。如果锁转换请求是不相容的锁，则转换不允许，并且请求锁转换的进程在该节点队列的开头被设置为等待状态。

该方案的 reader 协议和方案1一样，updaer 遵循下面的协议：
		0) Place $\alpha \text{-lock}$ on root;
			1) Get root and make it the current node;
			2) **While** current node is not a leaf node **do**
					   {number of $\alpha \text{-locks}$ held $\ge$ 1}
					   **begin**
						3)  Place an $\alpha \text{-locks}$ on appropriate son of current node;
						4)  Get son and make it the current node;
								**If** current node is safe
								**then** release all locks held on ancestors of current node
						**end mainloop**
			6) {A leaf node has been reached. At this time we can determine if update can be successfully completed.}
				   **If** the update will be successful
				    **then** convert, top-down, all $\alpha \text{-locks}$ into $\xi \text{-locks}$;


**方案3优缺点**
- 优点：
	- 相比方案1，该协议使用 $\alpha \text{-lock}$ 而不是 $\xi \text{-lock}$, 这样做的好处时允许 reader 共享 updater 在节点上放置的 $\alpha \text{-locks}$, 从而增加了并发性（方案1中，只有 $\xi \text{-lock}$, reader 只能等待）。
	- 相比方案2，该协议在 Step6 时，只存在两种情况：
		1. 更新可以成功，因此自顶向下的将  $\alpha \text{-locks}$ 转换为 $\xi \text{-locks}$，此时所有需要修改的节点在 Step6 后被排他锁锁定。
		2. 更新失败，其他的进程更快或者说抢先完成了锁转换，该进程推出，不需要像方案2那样在重启然后重试（因为 $\alpha \text{-lock}$ 隐式的告诉了我们有其他 updater 抢先在更新了，重启是无意义的，而在方案2中不是这样的，因为方案2没有额外的信息知道有其他 updater 抢先更新了，因此它只能选择重启再次尝试，因为再次尝试会遇到 $\xi \text{-lock}$ 的情况，这时他就知道被其他 updater 抢先了，而存在 $\alpha \text{-lock}$ 时包含了这个情况）。
	- $\xi \text{-lock}$  仅放置在那些被修改的节点上（到达叶节点是才会将 $\alpha \text{-lock}$ 进行转换），因此防止了 reader 仅检查最小可能的节点集合。
- 缺点：
	- 和方案1一样，updater 进程可能会组织其他 updater 扫描某个节点，即使该节点不受到更新的影响。
	- 锁转换会花费额外是实际

通过修改协议，在叶节点上之间设置 $\xi \text{-lock}$ 而不是 $\alpha \text{-lock}$, 可以消除叶节点这一层级所需的锁转换。

但是，如果更新影响 higher node, 且节点持有 $\alpha \text{-lock}$ ，则需要首先将叶节点上的 $\xi \text{-lock}$ 转换为 $\alpha \text{-lock}$，然后进行 $\alpha \text{-lock}$ 到 $\xi \text{-lock}$的转换。因此，叶结点上 行 $\alpha \text{-lock}$ 到 $\xi \text{-lock}$ 的频繁操作可以由更复杂的协议以及将叶结点上的  $\xi \text{-lock}$ 转换为 $\alpha \text{-lock}$的不频繁的锁转换来代替。因此锁之间需要一个新的属性，即$\xi \text{-lock}$ 可以转换为 $\alpha \text{-lock}$。论文将在通用的方法中描述。

## 4. 通用方案
 使用4种类型的锁: $p_r \text{-lock}$, $p_u \text{-lock}$, $\alpha \text{-lock}$, $\xi \text{-lock}$. 锁的相容性如Fig.5, 意味着 $\alpha$ 和 $\xi$ 可以互相转换.
![[Pasted image 20220228101502.png]]
reader 协议和方案1一样, 区别是使用 $p_r$ 替换 $p$. 
给出updater协议前, 首先定义一些变量:
- $P$ 和 $\Xi$  表示 updater 可以放置 $p_u \text{-locks}$  和 $\xi \text{-locks}$ 的最大级别数.
- $H$ 的值 $h$  表示树的高度. 一般实现中, $(h,root)$ 存放在一起, $H$ 引用这个条目, root 可以看作是 $H$ 的后代. 

updaer 遵循下面的协议：
		**Begin**
			{Let variable $H$ always contain the height $h$ of the tree,  $h\ge 0$ }
			**procedure** process son of current;
					**begin** get son of current;
								current := son of current;
								**if** current is safe
								**then** remove locks on all ancestors of current;
					**end**;
					0)  **If** $P \ne 0$  **then** place $p_u \text{-lock}$  on $H$ **else** place $\alpha \text{-lock}$ on $H$;
							 current:= $H$; {root = son of $H$};
					1) $\overline {\Xi}$ := min{$h$, $\Xi$};
							$\overline{P}$ := min{$P$, $h-\overline{\Xi}$};
							$\overline{\alpha}$ := $h-\overline{\Xi} - \overline{P}$ 
					2) **for** $L$ := 1 **step** 1 **until** $\overline{P}$ **do**
								   **begin** place a $p_u \text{-lock}$ on son of current;
											    release  $p_u \text{-lock}$ on current;
											    get son of current;
												  current := son of current
								   **end**
					3) **for** $L$ := 1 **step** 1 **until** $\overline{\alpha}$ **do**
								   **begin** place a $\alpha \text{-lock}$ on son of current;
											    process son of current;
							 	**end**
					4) **for** $L$ := 1 **step** 1 **until** $\overline{\Xi}$ **do**
								   **begin** place a $\xi \text{-lock}$ on son of current;
											    process son of current;
									**end**						
					5) **if** $p_u \text{-lock}$ stil held
							   **then begin**  release all locks;
														  $P=0$; $\Xi=0$;
														  repeat protocol and exit.
											**end**										
					6)  **if** $\alpha \text{-locks}$ stil held
							**then begin** 6a): convert top-down all $\xi$ to $\alpha$;
													 6c): convert top-down all $\alpha$ to $\xi$;
					7)  MODIFY: modify all nodes with $\xi \text{-locks}$, requesting additional $\xi \text{-locks}$ for 
									   over flows, 	underflows, splits and merges as necessary; 
					8)  release all locks;
		**end**

在 Step6 后,updater 可以执行实际的改变. 当 Step7 结束后, 锁定的子路径上的所有节点使用 $\xi \text{-locks}$. 
B-tree 上的插入和删除规则决定了是否会获取额外的 $\xi \text{-lock}$。 当尝试overflow（或underflow）到兄弟时会发生这种情况. 这种情况如图6所示
![[Pasted image 20220228105632.png]]
在步骤 6 结束时，节点 $b$ 和 $c$ 持有 $\xi \text{-locks}$，并且要在节点 $c$ 上执行更新操作。
节点 $d$ 和 $e$ 是 $c$ 的直系兄弟. 由于 $b$ 有一个 $\xi \text{-lock}$, 我们知道 $c$ 上的更新将传播到$b$.  然后尝试对 $d$ 或 $e$ 进行 overflow（或underflow）操作.
为此, 在 $d$ 上请求 $\xi \text{-lock}$, 并且在授予时尝试将 $c$ 和 $d$ 组合在一起 . (如果这不可能，则尝试组合 c 和 e).  完成此级别所需的修改后, 更新节点 $b$. 请注意, 节点 $b$ 是安全的（因为节点 $a$ 上没有锁), 因此在修改后, 更新将终止.
![[Pasted image 20220228110736.png]]

我们从 updater 的协议中得到下面的观察: 
- **观察1:** 所有被进程锁定的节点形成一个 “梳子”. (树的梳子是具有以下限制的子树：如果一个节点有多个子树作为后代，则其中只有一个子树可以有多个节点.) 梳子的示例如图 7 所示。
- **观察2:** 如果一个进程持有 $p_r$, $p_u$  或 $\alpha \text{-locks}$, 那么它的梳子被简化为一条路径 (如Fig.7b 所示). 让这条路径是 $(P_1，P_2 \cdots, P_n)$.  那么
	- a) 进程没有被授予任何$\alpha$ 到 $\xi$ 的转换. 在这种情况下, 存在整数 $j$, $k$，其中 $0 \leqq j \leqq k \leqq n$  使得所有节点 $p_1,p_2 ..., p_j$ 都具有 $p_r \text{-locks}$  (如果是 reader, 在这种情况下$j=k=n$) 或 $p_u \text{-locks}$ (如果是updater), 所有 $p_{j+1}, .... , p_k$ 的节点都有 $\alpha \text{-locks}$, 所有 $p_{k+1}, .... , p_n$ 节点, 都有  $\xi \text{locks}$ .
	- b) 进程已被授予 $\alpha$ 到 $\xi$  锁的转换. 然后它不再持有 $p_u \text{-locks}$, $p_n$ 是一个叶子节点并且有一个整数 $k$ 使得 $1\leqq k \leqq n$ , $p_1, ..... p_k$ 有 $\xi \text{-locks}$ 和 $p_{k+l}, ... , p_n$ 有 $\alpha \text{-locks}$.

 在接下来的部分中, 我们将更仔细地研究这些协议. 我们将证明它们是无死锁的, 并将分析它们提供的并发性.

## 5. 通用方案无死锁
 
## Reference

> [11] Metzger, J. K.: Managing simultaneous operations in large ordered indexes. Technische Universit~it Miinchen, Institut f'tir Informatik, TUM-Math. Report, 1975
