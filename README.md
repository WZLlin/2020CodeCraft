# 2020CodeCraft
2020华为软挑初赛上合赛区第一，复赛总榜A榜第一解决方案

#### 队伍：但使龙城飞将在，六宫粉黛无颜色
#### 复赛A榜     1.7085    
#### 1963W数据   4.3s
#### 新数据      3s (构图方式对节点多但稀疏的数据不太友好，增加构图哈希桶数，加入拓扑排序后可达2.4s)



# 大致思路
## 输入
```
struct NodeInfor{    //输入时通过hash去重唯一记录各个节点的 长度、字符串信息。用于构造答案使用。
    char str[12];
    uint32_t num;
    int len;
};
struct Node_D{
    uint32_t num;
    int indegree;
    int outdegree;
};
```
（1）输入除了将数据读入数组外，同时通过Hash去重，统计所有有效节点(NodeInfor)，并记录所有节点的入度出度(Node_D)

（2）后续映射可以只对不重复的节点进行排序

（3）在构图时可以通过入度出度预先安排每个节点邻居的范围，从而不用对边排序，只对每个节点的邻居排序

## 构图
```
struct GVec{
//GVec 用于记录  head 邻居的位置 起始：s_index，结束 e_index
    int s_index;
    int e_index;
};


pair<uint32_t, uint32_t> forG[2000000];     //正向图
pair<uint32_t, uint32_t> revG[2000000];     //反向图
GVec forGIndex[MaxNodeNum];           //正向图索引
GVec revGIndex[MaxNodeNum]; 
```
(1)通过forG记录图，forGIndex[i]记录节点i在forG的位置
(2)构图时通过输入时统计的各个节点输出度，先计算出所有节点邻居在forG的位置范围，构图时得到一条边从对应位置放入
(3)采用4个线程 分别分正向反向奇数偶数构造同一个forG和revG

## 搜索
采用4+3的策略，通过bfs搜索。
反向通过一个数组和一个动态数组存储路径

(1)当起点到某个点只存在一条路径时，通过数组记录，当搜索超过1条路径时使用动态数组记录
这样记录当只有起点到某个搜索点只有一条路径时，访存十分快。实验表明，在1963W数据中，有1700W+的环只存在一条反向路径 

(2)反向搜索只记录路径，不对路径进行排序，但通过标志位确定是否有序

(3)正向搜索使用路径时，可通过标志位判断是否存在路径，路径是否为1条，路径是否有序。使用到反向路径时进行排序。

trick:搜索7环时，先判断路径标志，再进行金额，路径是否重复等判断会提升很多，因为路径标志命中率不高，可以避免后续操作。

trick:bfs比dfs搜索更快，线下加速0.6s左右，线上加速0.2s左右，应该是dfs搜索大量节点会使上层节点的邻居被替换出cache。

trick:尽可能压缩常用的数据，类似我们的标志位，从int到char，forGIndex从指针变为int，都提升很大，线上提升应该超过0.2


## 多线程划分
```
#define TaskSize 1024
struct Task_s{
    int len[8];    //环在ans 中的长度
    char *pans[8];
}Task[TaskSize];
```
对所有节点分段调度，我们对节点分成了1024段，每个任务计算完毕后，将计算结果存入Task_s，保存ans指针的起始位置和长度。
写文件按顺序取指针一次写入即可。


## 其他线下较明显的优化
对邻居左边界进行维护，继续压缩NodeInfor到16B，以及多线程邻居排序，这些线下1963W数据大概可以提升0.25s，但线上效果均为负。





# 感言
第一次参赛，其实算法很菜，其实当初也就像进个复赛- -。。。现在目标达成了。
队伍3个人，也是临时组队，来自不同的学校。比起开始一个人瞎搞，有了队友后真的提升了好多。
#### 感谢义哥，感谢振炬兄。




