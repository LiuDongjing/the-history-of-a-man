# leetcode解题思路
## 435(Non overlapping Intervals)
### Brute Force
将Intervals按start point从小到大排序，然后从左到右处理Interval，
针对每个Interval，有两种情况，属于最终的Interval或者不属于，对这
两种情况分别讨论，选择最小移除数的那种情况。处理完一个，再处理下一个。

复杂度O(2^n)，会超时。

### 贪心算法
也是先排序，从小到大处理每个Interval。针对某个Interval，和下一个Interval有如图三种关系：

![相交情况](images/435_NonOverlapping_greedy1.JPG)

很显然，前两种情况是保留第一个Interval。主要是看最后一种情况。
蓝色的Interval对后续选择的只是相交的那一段，而绿色Interval还包
含后一段，也就是说与前者相交的Interval也必然与后者相交，这种情
况下自然要选择前者。

