[TOC]

核心思想：由根节点状态 => 子节点状态 或者 子节点状态 => 父节点状态

### 834. 树中距离之和

给定一个无向、连通的树。树中有 N 个标记为 0...N-1 的节点以及 N-1 条边 。

第 i 条边连接节点 edges[i] [0] 和 edges[i] [1] 。返回一个表示节点 i 与其他所有节点距离之和的列表 ans。

示例：

> 输入: N = 6, edges = [[0,1],[0,2],[2,3],[2,4],[2,5]]
> 输出: [8,12,6,10,10,10]
> 解释: 
> 如下为给定的树的示意图：
> 0
> / \
> 1   2
>     /|\
>   3 4 5
>
> 我们可以计算出 dist(0,1) + dist(0,2) + dist(0,3) + dist(0,4) + dist(0,5) 
> 也就是 1 + 1 + 2 + 2 + 2 = 8。 因此，answer[0] = 8，以此类推。

思考：树中其他节点到一个节点的距离包含两个部分

1、以这个节点为根节点的==树==内部节点到根节点的距离x

dist[root]的值包含两部分：

①每个以子节点为根的子树内部节点到子节点的距离，dist[childe]

②因为子节点到父节点的距离为1，因此子树内部所有节点距离父节点距离需要再加1，即node[child]

因此可以子节点推出父节点的状态，很容易想到dfs自底向上，写出**递推**公式：

distSum[root] = ΣdistSun[i] + Σnode[i]; i ∈ neighbor，

distSum[j]表示：以j为根节点的树内节点到节点j的距离之和。

node[j]表示：以j为根节点的树的节点数量

结束条件就是当该**节点**为**叶子节点**

```java
    private void postOrder(int root,int parent,int[] distSum,int[] nodeSum) {
        List<Integer> neighbor = neighbors.get(root);

        for (int num : neighbor) {
            if (num == parent) {
                continue;
            }
            postOrder(num,root,distSum,nodeSum);
            nodeSum[root] = nodeSum[root] + nodeSum[num];
            distSum[root] = distSum[root] + distSum[num] + nodeSum[num];
        }
    }
```

2、树外节点到这个节点距离 y，这个不好求，因此可以想办法通过其他途径**递推**出来

由题意可知dist[0]可以为节点0的答案，

子节点的dist，包括子树内部和子树外部，

子树内部节点到0的距离 - 1就是子树内部节点到该子树节点的距离；- node[child]

0节点到该子节点的的距离是1，因此外部节点到该节点的距离需要加1；N - node[child]

dist[child] = dist[parent] - node[child] + N - node[child];

因此可以采用自顶向下的bfs遍历，由父节点状态推出子节点的状态。

```java
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{0,-1});
        while (!queue.isEmpty()) {
            int size = queue.size();

            for (int i = 0; i < size; i++) {
                int[] root = queue.poll();
                List<Integer> neighbor = neighbors.get(root[0]);
                for (int num : neighbor) {
                    if (num == root[1]) {
                        continue;
                    }
                    distSum[num] = distSum[root[0]] - nodeSum[num] + N - nodeSum[num];
                    queue.offer(new int[]{num,root[0]});
                }
            }
        }
```

### 968. 监控二叉树

给定一个二叉树，我们在树的节点上安装摄像头。节点上的每个摄影头都可以监视**其父对象、自身及其直接子对象。**

计算监控树的所有节点所需的最小摄像头数量。

示例1：

> ![](../picture/968监控二叉树-示例.png)
>
> 输入：[0,0,null,0,0]
> 输出：1
> 解释：如图所示，一台摄像头足以监控所有节点。

核心思路：

1、如果从父状态推导子状态，需要保存每个父状态，同时子状态越来越多，推导比较繁琐

2、会思考从子状态推父状态，因此使用dfs的后序遍历，从左子树和右子树推出父状态。

* 假设当前root节点安装监控，只要以子节点为根的子树都被监控到即可
* 当前不安装监控，则要保证左子树或者右子树的根节点安装了监控

3、从上述可以思考推出需要几个状态：

* a：当前树根节点安装监控，且监控树所有节点所需要的最小监控数量
* b：无论当前树是否安装监控，都需要保证当前树所有节点都被监控的最小监控数量
* c：当前根节点子树的左右子树被监控的最少监控数量

定义状态类：

```java
    private class Status{
        //root节点安装监控，且保证整个子树都能覆盖
        int a;
        //保证以root为根节点的树都能覆盖，无论root节点是否安装监控
        int b;
        //root节点的两个子树都能被监控，无论子树的根节点是否安装监控
        int c;

        public Status(int a, int b, int c) {
            this.a = a;
            this.b = b;
            this.c = c;
        }
    }
```

假设左子树的状态为：L~a~，L~b~，L~c~，右子树的状态为：R~a~，R~b~，R~c~，写出状态转移方程：

a = L~c~ + R~c~ + 1

> L~c~+R~c~：表示两个子树的左右子树一定被监控，但子树的根节点不一定有监控，因此根节点被监控的时候，可以保证整个树一定被监控

b = Math.min(a, Math.min(L~a~+R~b~，L~b~+R~a~)) 

> 解释：b的值要么为a，要么为左子树必须安装监控，右子树可能安装监控（或者右子树必须安装监控，左子树可能安装监控）

c = Math.max(a，L~b~ + R~b~)

> 解释：c的值含义是保证两个子树必定监控，
>
> * a状态下两个子树一定被完全监控
> * L~b~+R~b~状态表示两个子树都覆盖

```java
    public int minCameraCover(TreeNode root) {
        return dfs(root).b;
    }

    private Status dfs(TreeNode node) {
        if (node == null) {
            return new Status(Integer.MAX_VALUE/2,0,0);
        }

        Status left = dfs(node.left);
        Status right = dfs(node.right);
        //状态转移方程
        Status status = new Status(0,0,0);
        status.a = left.c + right.c + 1;
        status.b = Math.min(status.a,Math.min(left.a + right.b,left.b + right.a));
        status.c = Math.min(status.a,left.b + right.b);
        return status;
    }
```

