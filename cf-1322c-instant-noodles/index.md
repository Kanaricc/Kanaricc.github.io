# [CF 1322C] Instant Noodles

Wu got hungry after an intense training session, and came to a nearby store to buy his favourite instant noodles. After Wu paid for his purchase, the cashier gave him an interesting task.

You are given a bipartite graph with positive integers in all vertices of the right half. For a subset 𝑆 of vertices of the left half we define 𝑁(𝑆) as the set of all vertices of the right half adjacent to at least one vertex in 𝑆, and 𝑓(𝑆) as the sum of all numbers in vertices of 𝑁(𝑆). Find the greatest common divisor of 𝑓(𝑆) for all possible non-empty subsets 𝑆 (assume that GCD of empty set is 0).

Wu is too tired after his training to solve this problem. Help him!

## 分析

这道题的范围有500000之大，所以实在是无法从子集本身入手，只能考虑一下gcd是否存在性质。

假设现在有一个子集$a$，当我们为其加上一个$\Delta$后，这两个结果都会参与gcd的计算，更相减损后表现为$\Delta$参与gcd计算。

这个$\Delta$的增量直接贡献答案的特性可能是重要的一环。比赛时因为时间也好，别的也罢，被卡在这自闭不会了。

接下来确定增量是什么。

是否是单独的一个元素：对于一个右节点$v$，是否存在添加一个左节点后只增加其一个的情况。干脆把全部的左节点全部选中，然后删掉包含$v$的。接下来，观察是否存在一个左节点只增加$v$。这显然会有不存在的情况。

在不存在的情况时，我们有这些结论。假设这个捣乱的节点是$v'$

* $v'$由于删除所有包含$v$的节点而被共同删除。
* $v'$无法被不包含$v$的其他节点增加，那么能增加$v'$的节点定包含$v$。

此时有两种情况。
* 所有添加v的节点均包含$v'$。那么二者共进退，表现为捆绑在一起的一个节点。
* 并不是所有添加v的节点均包含$v'$。那么，我们就能通过添加别的节点率先加上v，从而能够单独添加v'，从而能够单独添加v。
所以，这个增量不是单个元素，而是这么一个东西：邻域相同的元素。

可以做了？……太菜，搞不明白这怎么在比赛期间短期折腾出来。


