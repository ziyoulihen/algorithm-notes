# 并查集

并查集，主要用来解决**元素分组**的问题。主要分成2个关键步骤：

1. 查询（Find）：查询元素的根节点
2. 合并（Union）：将2个不相交的集合合并成一个集合

## 代码套路

```java
class UnionFind {
    /**
     * 探测图中是否存在环
     *
     * @param edges    构成图的边集合
     * @param vertices 构成图的顶点数量
     */
    public boolean detectCycle(int[][] edges, int vertices) {
        // 首先，我们申请1个一位数组来存储每个节点对应的父节点。
        // 该数组下标表示节点，元素值表示父节点。
        int[] parent = new int[vertices];
        initialize(parent);

        for (int i = 0; i < edges.length; i++) {
            int[] edge = edges[i];
            int x = edge[0];
            int y = edge[1];
            // 若存在两个根节点相同的节点，则表示图中存在环。
            if (find(parent, x) == find(parent, y)) {
                return true;
            }
            union(parent, x, y);
        }

        return false;
    }

    /**
     * 初始化操作：每个节点的父节点初始为其本身。表示该节点单独一组。
     */
    private void initialize(int[] parent) {
        for (int i = 0; i < parent.length; i++) {
            parent[i] = i;
        }
    }

    /**
     * 合并操作
     */
    private void union(int[] parent, int x, int y) {
        // 分别先找到2个节点的根节点。
        int xRoot = find(parent, x);
        int yRoot = find(parent, y);
        // 然后合并成新树，以xRoot和yRoot为根节点都可以。
        // 个人习惯，以yRoot为根节点。
        parent[xRoot] = yRoot;
    }

    /**
     * 查找操作
     */
    private int find(int[] parent, int x) {
        int root = x;
        // 若某个节点为根节点，则其父节点必然是其本身。
        while (parent[root] != root) {
            root = parent[root];
        }
        return root;
    }

    public static void main(String[] args) {
        UnionFind solution = new UnionFind();
        System.out.println(solution.detectCycle(new int[][]{{0, 1}, {1, 2}, {2, 3}, {3, 4}, {4, 5}}, 6));
        System.out.println(solution.detectCycle(new int[][]{{0, 1}, {1, 2}, {2, 3}, {3, 4}, {4, 5}, {5, 0}}, 6));
        System.out.println(solution.detectCycle(new int[][]{{0, 1}, {1, 2}, {2, 3}, {3, 4}, {4, 5}, {3, 4}}, 6));
    }
}
```

上面所给出的示例中，查询操作的实现方式采用的是循环方式。也可以使用递归方式：

```java
class UnionFind {
    private int find(int[] parent, int x) {
        if (parent[x] == x) {
            return find(parent[x]);
        }
        return parent[x];
    }
}
```

查询操作（`find`）的作用是查找节点对应的根节点。所以，每次都需要从当前节点开始从下往上找。极端情况下，树的每一层都只有1个节点。此时，每次要找到叶子节点对应的根节点时，都需要将整棵树遍历一次。那么，都有哪些优化手段？

### 路径压缩

最常见的查处操作优化手段是**路径压缩**。其主要的逻辑是在查找时，将查找路径进行压缩，即将当前节点的父节点更新为**父节点的父节点**或**根节点**。如此，大大缩短了再次查询的查找路径。

**循环方式**

```java
class UnionFind {
    private int find(int[] parent, int x) {
        int root = x;
        while (parent[root] != root) {
            // 将父节点更新成父节点的父节点
            parent[root] = parent[parent[root]];
            root = parent[root];
        }
        return root;
    }
}
```

**递归方式：**

```java
class UnionFind {
    private int find(int[] parent, int x) {
        if (parent[x] != x) {
            // 将父节点更新成根节点
            parent[x] = find(parent, parent[x]);
        }
        return parent[x];
    }
}
```

### 按秩合并

**压缩路径**是在查询操作时进行优化，而**按秩合并**则是在合并操作时进行优化。我们知道如果树的高越低，则查找路径越短。所以，我们只要保证合并以后的树的高尽可能低，则查找路径必然更短。

```java
class RankUnionFind {
    public boolean detectCircle(int[][] edges, int vertices) {

        int[] parent = new int[vertices];
        // 申明秩数组，表示以该节点为根节点的树的高度。秩初始值为0。
        int[] rank = new int[vertices];
        initialize(parent);

        for (int i = 0; i < edges.length; i++) {
            int[] edge = edges[i];
            int x = edge[0];
            int y = edge[1];
            if (find(parent, x) == find(parent, y)) {
                return true;
            }
            union(parent, rank, x, y);
        }

        return false;
    }

    private void initialize(int[] parent) {
        for (int i = 0; i < parent.length; i++) {
            parent[i] = i;
        }
    }

    private int find(int[] parent, int x) {
        if (parent[x] != x) {
            return find(parent, parent[x]);
        }
        return parent[x];
    }

    private void union(int[] parent, int[] rank, int x, int y) {
        int xRoot = find(parent, x);
        int yRoot = find(parent, y);
        // 以秩大（树的高度更高）的树根节点为新树的更节点
        // 即秩小指向秩大。
        // 合并的新树高度不变，不需要更新秩。
        if (rank[xRoot] > rank[yRoot]) {
            parent[yRoot] = xRoot;
        } else if (rank[xRoot] < rank[yRoot]) {
            parent[xRoot] = yRoot;
        } else {
            // 秩相同，以xRoot或yRoot为新树的根节点都可以。
            // 个人习惯以，yRoot为根节点。
            // 合并成新树后，需要更新根节点的秩。
            parent[xRoot] = yRoot;
            ++rank[yRoot];
        }
    }

    public static void main(String[] args) {
        RankUnionFind solution = new RankUnionFind();
        System.out.println(solution.detectCircle(new int[][]{
                {0, 1}, {1, 2}, {2, 3}, {3, 4}, {4, 5}, {4, 1}
        }, 6));
    }
}
```

## 经典例题

-   [LeetCode990 等式方程的可满足性](#leetcode990-等式方程的可满足性)
-   [LeetCode547 省份数量](#leetcode547-省份数量)
-   [LeetCode200 岛屿数量](#leetcode200-岛屿数量)
-   [LeetCode684 冗余连接](#leetcode684-冗余连接)
-   [LeetCode1319 连通网络的操作次数](#leetcode-连通网络的操作次数)
-   [LeetCode399-](#leetcode399-)
-   [LeetCode952-](#leetcode952-)

## [LeetCode990 等式方程的可满足性](https://leetcode-cn.com/problems/satisfiability-of-equality-equations/)

### 解法一：合并集（循环查找）

```java
class Solution {
    public boolean equationsPossible(String[] equations) {
        int[] parent = new int[26];
        for (int i = 0; i < parent.length; i++) {
            parent[i] = i;
        }
        for (String equation : equations) {
            if (equation.charAt(1) == '=') {
                int index1 = equation.charAt(0) - 'a';
                int index2 = equation.charAt(3) - 'a';
                union(parent, index1, index2);
            }
        }
        for (String equation : equations) {
            if (equation.charAt(1) == '!') {
                int index1 = equation.charAt(0) - 'a';
                int index2 = equation.charAt(3) - 'a';
                if (find(parent, index2) == find(parent, index1)) {
                    return false;
                }
            }
        }
        return true;
    }

    private int find(int[] parent, int index) {
        while (parent[index] != index) {
            parent[index] = parent[parent[index]];
            index = parent[index];
        }
        return index;
    }

    private void union(int[] parent, int index1, int index2) {
        parent[find(parent, index1)] = find(parent, index2);
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：0 ms, 在所有 Java 提交中击败了 100.00%的用户
-   内存消耗：38.2 MB, 在所有 Java 提交中击败了 12.47%的用户
:::

### 解法二：合并集（递归查找）

```java
class Solution {
    public boolean equationsPossible(String[] equations) {
        int[] parent = new int[26];
        for (int i = 0; i < parent.length; i++) {
            parent[i] = i;
        }
        for (String equation : equations) {
            if (equation.charAt(1) == '=') {
                int index1 = equation.charAt(0) - 'a';
                int index2 = equation.charAt(3) - 'a';
                union(parent, index1, index2);
            }
        }
        for (String equation : equations) {
            if (equation.charAt(1) == '!') {
                int index1 = equation.charAt(0) - 'a';
                int index2 = equation.charAt(3) - 'a';
                if (find(parent, index2) == find(parent, index1)) {
                    return false;
                }
            }
        }
        return true;
    }

    private int find(int[] parent, int index) {
        if (parent[index] == index) {
            return index;
        } else {
            parent[index] = find(parent, parent[index]);
            return parent[index];
        }
    }

    private void union(int[] parent, int index1, int index2) {
        parent[find(parent, index1)] = find(parent, index2);
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：1 ms, 在所有 Java 提交中击败了 99.72%的用户
-   内存消耗：37.8 MB, 在所有 Java 提交中击败了 84.47%的用户
:::

## [LeetCode547 省份数量](https://leetcode-cn.com/problems/number-of-provinces/)

### 解法一：DFS

```java
class Solution {
  public int findCircleNum(int[][] isConnected) {
        int circles = 0;
        int size = isConnected.length;
        boolean[] visited = new boolean[size];
        for (int i = 0; i < size; i++) {
            if (!visited[i]) {
                bfs(isConnected, visited, size, i);
                ++circles;
            }
        }
        return circles;
    }

    private void bfs(int[][] isConnected, boolean[] visited, int size, int i) {
        visited[i] = true;

        for (int j = 0; j < size; j++) {
            if (isConnected[i][j] == 1 && !visited[j]) {
                bfs(isConnected, visited, size, j);
            }
        }
    }
}
```

### 解法二：BFS

```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int circles = 0;
        int size = isConnected.length;
        boolean[] visited = new boolean[size];
        Deque<Integer> queue = new LinkedList<>();
        for (int i = 0; i < size; i++) {
            if(!visited[i]) {
                queue.offer(i);
                while (!queue.isEmpty()) {
                    int j = queue.poll();
                    visited[j] = true;
                    for (int k = 0; k < size; k++) {
                        if (isConnected[j][k] == 1 && !visited[k]) {
                            queue.offer(k);
                        }
                    }
                }
                ++circles;
            }
        }
        return circles;
    }
}
```

### 解法三：合并集

```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int size = isConnected.length;
        int[] parent = new int[size];
        for (int i = 0; i < size; i++) {
            parent[i] = i;
        }
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                if (isConnected[i][j] == 1) {
                    union(parent, i, j);
                }
            }
        }
        int circles = 0;
        for (int i = 0; i < size; i++) {
            if (parent[i] == i) {
                circles++;
            }
        }
        return circles;
    }

    private void union(int[] parent, int i, int j) {
        parent[find(parent, i)] = find(parent, j);
    }

    private int find(int[] parent, int i) {
        while (parent[i] != i) {
            parent[i] = parent[parent[i]];
            i = parent[i];
        }
        return i;
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：1 ms, 在所有 Java 提交中击败了 99.14%的用户
-   内存消耗：39.5 MB, 在所有 Java 提交中击败了 31.86%的用户
:::

## [LeetCode200 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

### 解法一：合并集

```java
public class Solution {
    private int islands;
    private int[] parent;

    public int numIslands(char[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;
        islands = 0;
        parent = new int[rows * cols];
        for (int row = 0; row < rows; row++) {
            for (int col = 0; col < cols; col++) {
                if (grid[row][col] == '1') {
                    parent[row * cols + col] = row * cols + col;
                    ++islands;
                }
            }
        }

        for (int row = 0; row < rows; row++) {
            for (int col = 0; col < cols; col++) {
                if (grid[row][col] == '1') {
                    grid[row][col] = '0';
                    if (row - 1 >= 0 && grid[row - 1][col] == '1') {
                        union(row * cols + col, (row - 1) * cols + col);
                    }
                    if (row + 1 < rows && grid[row + 1][col] == '1') {
                        union(row * cols + col, (row + 1) * cols + col);
                    }
                    if (col - 1 >= 0 && grid[row][col - 1] == '1') {
                        union(row * cols + col, row * cols + col - 1);
                    }
                    if (col + 1 < cols && grid[row][col + 1] == '1') {
                        union(row * cols + col, row * cols + col + 1);
                    }
                }
            }
        }

        return islands;
    }

    private int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    private void union(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX != rootY) {
            parent[rootX] = rootY;
            --islands;
        }
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：2 ms, 在所有 Java 提交中击败了 88.69%的用户
-   内存消耗：41 MB, 在所有 Java 提交中击败了 23.64%的用户
:::

### 解法二：合并集

```java
class Solution {
  private static class UnionFind {
        private int count;
        /**
         * parent[i]表示第i个元素的根节点
         */
        private final int[] parent;
        /**
         * rank[i]表示第i个元素是否是根节点
         */
        private final int[] rank;

        public UnionFind(char[][] grid, int rows, int cols) {
            count = 0;
            parent = new int[rows * cols];
            rank = new int[rows * cols];
            for (int row = 0; row < rows; row++) {
                for (int col = 0; col < cols; col++) {
                    if (grid[row][col] == '1') {
                        parent[row * cols + col] = row * cols + col;
                        ++count;
                    }
                    rank[row * cols + col] = 0;
                }
            }
        }

        public int find(int i) {
            if (parent[i] != i) {
                parent[i] = find(parent[i]);
            }
            return parent[i];
        }

        public void union(int x, int y) {
            int rootX = find(x);
            int rootY = find(y);
            if (rootX != rootY) {
                if (rank[rootX] > rank[rootY]) {
                    parent[rootY] = rootX;
                } else if (rank[rootX] < rank[rootX]) {
                    parent[rootX] = rootY;
                } else {
                    parent[rootY] = rootX;
                    rank[rootX] += 1;
                }
                --count;
            }
        }

        public int getCount() {
            return this.count;
        }
    }

    public int numIslands(char[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;
        UnionFind unionFind = new UnionFind(grid, rows, cols);
        for (int row = 0; row < rows; row++) {
            for (int col = 0; col < cols; col++) {
                if (grid[row][col] == '1') {
                    grid[row][col] = '0';
                    if (row - 1 >= 0 && grid[row - 1][col] == '1') {
                        unionFind.union(row * cols + col, (row - 1) * cols + col);
                    }
                    if (row + 1 < rows && grid[row + 1][col] == '1') {
                        unionFind.union(row * cols + col, (row + 1) * cols + col);
                    }
                    if (col - 1 >= 0 && grid[row][col - 1] == '1') {
                        unionFind.union(row * cols + col, row * cols + col - 1);
                    }
                    if (col + 1 < cols && grid[row][col + 1] == '1') {
                        unionFind.union(row * cols + col, row * cols + col + 1);
                    }
                }
            }
        }
        return unionFind.getCount();
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：8 ms, 在所有 Java 提交中击败了 5.25%的用户
-   内存消耗：40.8 MB, 在所有 Java 提交中击败了 67.95%的用户
:::

### 解法三：BFS

```java
class Solution {
    public int numIslands(char[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;
        int islands = 0;

        Deque<int[]> queue = new LinkedList<>();
        for (int row = 0; row < rows; row++) {
            for (int col = 0; col < cols; col++) {
                if (grid[row][col] == '1') {
                    queue.offer(new int[]{row, col});
                    grid[row][col] = '0';
                    ++islands;
                    while (!queue.isEmpty()) {
                        int[] position = queue.poll();
                        int r = position[0];
                        int c = position[1];
                        if (r + 1 < rows && grid[r + 1][c] == '1') {
                            queue.offer(new int[]{r + 1, c});
                            grid[r + 1][c] = '0';
                        }
                        if (r - 1 >= 0 && grid[r - 1][c] == '1') {
                            queue.offer(new int[]{r - 1, c});
                            grid[r - 1][c] = '0';
                        }
                        if (c + 1 < cols && grid[r][c + 1] == '1') {
                            queue.offer(new int[]{r, c + 1});
                            grid[r][c + 1] = '0';
                        }
                        if (c - 1 >= 0 && grid[r][c - 1] == '1') {
                            queue.offer(new int[]{r, c - 1});
                            grid[r][c - 1] = '0';
                        }
                    }
                }
            }
        }

        return islands;
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：5 ms, 在所有 Java 提交中击败了 17.74%的用户
-   内存消耗：40.8 MB, 在所有 Java 提交中击败了 61.37%的用户
:::

### 解法四：DFS

```java
class Solution {
    private static final int[][] diffs = new int[][]{{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
    private int rows;
    private int cols;

    public int numIslands(char[][] grid) {
        rows = grid.length;
        cols = grid[0].length;
        int islands = 0;
        for (int row = 0; row < rows; row++) {
            for (int cols = 0; cols < this.cols; cols++) {
                if (grid[row][cols] == '1') {
                    bfs(grid, row, cols);
                    ++islands;
                }
            }
        }
        return islands;
    }

    private void bfs(char[][] grid, int row, int col) {
        grid[row][col] = '0';

        for (int[] diff : diffs) {
            int nextRow = row + diff[0];
            int nextCol = col + diff[1];
            if (isInvalid(nextRow, nextCol) || grid[nextRow][nextCol] == '0') {
                continue;
            }
            bfs(grid, nextRow, nextCol);
        }
    }

    private boolean isInvalid(int nextRow, int nextCol) {
        return nextRow < 0 || nextRow >= rows || nextCol < 0 || nextCol >= cols;
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：1 ms, 在所有 Java 提交中击败了 100.00%的用户
-   内存消耗：40.9 MB, 在所有 Java 提交中击败了 39.64%的用户
:::

## [LeetCode684 冗余连接](https://leetcode-cn.com/problems/redundant-connection/)

```java
public class Solution {
    public int[] findRedundantConnection(int[][] edges) {
        int nodesCount = edges.length;
        int[] parent = new int[nodesCount + 1];
        for (int i = 1; i <= nodesCount; i++) {
            parent[i] = i;
        }
        for (int[] edge : edges) {
            int node1 = edge[0];
            int node2 = edge[1];
            if (find(parent, node1) != find(parent, node2)) {
                union(parent, node1, node2);
            } else {
                return edge;
            }
        }
        return new int[0];
    }

    private void union(int[] parent, int node1, int node2) {
        parent[find(parent, node1)] = find(parent, node2);
    }

    private int find(int[] parent, int node) {
        if (parent[node] != node) {
            parent[node] = find(parent, parent[node]);
        }
        return parent[node];
    }
}

```

:::tip

-   执行结果：通过
-   执行用时：0 ms, 在所有 Java 提交中击败了 100.00%的用户
-   内存消耗：38.3 MB, 在所有 Java 提交中击败了 95.69%的用户
:::

## [LeetCode1319 连通网络的操作次数](https://leetcode-cn.com/problems/number-of-operations-to-make-network-connected/)

```java
public class Solution {
    public int makeConnected(int n, int[][] connections) {
        int edgesCount = connections.length;
        if (edgesCount < n - 1) {
            return -1;
        }
        int[] parent = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
        int count = n;
        for (int[] connection : connections) {
            int node1 = connection[0];
            int node2 = connection[1];
            if (find(parent, node1) != find(parent, node2)) {
                union(parent, node1, node2);
                --count;
            }
        }
        return count - 1;
    }

    private void union(int[] parent, int node1, int node2) {
        parent[find(parent, node1)] = find(parent, node2);
    }

    private int find(int[] parent, int node) {
        if (parent[node] != node) {
            parent[node] = find(parent, parent[node]);
        }
        return parent[node];
    }
}
```

:::tip

-   执行结果：通过
-   执行用时：3 ms, 在所有 Java 提交中击败了 99.50%的用户
-   内存消耗：52.2 MB, 在所有 Java 提交中击败了 82.73%的用户
:::