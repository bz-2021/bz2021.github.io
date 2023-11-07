---
title: 【算法基础】BFS（Breadth First Search）
date: 2023-01-08 16:11:24
tags: [算法, BFS]
---

从LeetCode上的中等题开始，涉及多源BFS、双端队列BFS（0-1BFS）。
还涉及Flood-Fill、最短路模型、A*、双向广搜。

<!-- more -->

# BFS专题

## 🟡 **[934. 最短的桥](https://leetcode.cn/problems/shortest-bridge/)**

- 题目
    
    给你一个大小为 `n x n` 的二元矩阵 `grid` ，其中 `1` 表示陆地，`0` 表示水域。
    
    **岛** 是由四面相连的 `1` 形成的一个最大组，即不会与非组内的任何其他 `1` 相连。`grid` 中 **恰好存在两座岛** 。
    
    你可以将任意数量的 `0` 变为 `1` ，以使两座岛连接起来，变成 **一座岛** 。
    
    返回必须翻转的 `0` 的最小数目。
    
- 解答
    
    ```cpp
    #define x first
    #define y second
    typedef pair<int, int> PII;
    class Solution {
    public:
        queue<PII> q;
        int n;
        int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};
        void dfs(int i, int j, vector<vector<int>>& grid){
            if(i < 0 || i >= n || j < 0 || j >= n || grid[i][j] == 0 || grid[i][j] == 2) return;
            grid[i][j] = 2;
            q.emplace(i, j);
            for(int k = 0; k < 4; k ++){
                int a = i + dx[k], b = j + dy[k];
                dfs(a, b, grid);
            }
        }
        void init(vector<vector<int>>& grid){
            n = grid.size();
            for(int i = 0; i < n; i ++)
                for(int j = 0; j < n; j ++)
                    if(grid[i][j]) {
                        dfs(i, j, grid);
                        return;
                    }
        }
        int bfs(vector<vector<int>>& grid){
            int res = 0;
            while(!q.empty()){
                int sz = q.size();
                for(int i = 0; i < sz; i ++){
                    auto t = q.front();
                    q.pop();
                    if(grid[t.x][t.y] == 1) return res;
                    for(int k = 0; k < 4; k ++){
                        int a = t.x + dx[k], b = t.y + dy[k];
                        if(a < 0 || a >= n || b < 0 || b >= n || grid[a][b] == 2) continue;
                        if(grid[a][b]) return res;
                        if(grid[a][b] == 0) {
                            grid[a][b] = 2;
                            q.emplace(a, b);
                        }
                    }
                }
                res ++;
            }
            return res;
        }
        int shortestBridge(vector<vector<int>>& grid) {
            init(grid);
            return bfs(grid);
        }
    };
    ```
    

## 🟡 **[994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)**

- 题目
    
    在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：
    
    - 值 `0` 代表空单元格；
    - 值 `1` 代表新鲜橘子；
    - 值 `2` 代表腐烂的橘子。
    
    每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。
    
    返回 *直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`* 。
    
- 解答
    
    ```cpp
    #define x first
    #define y second
    typedef pair<int, int> PII;
    class Solution {
    public:
        int orangesRotting(vector<vector<int>>& grid) {
            int m = grid.size(), n = grid[0].size();
            int res = 0, cnt = 0, p = 0;
    				// cnt统计总共新鲜的橘子，p统计BFS过程中遇到的新鲜的橘子
            int dx[] = {-1, 0, 1, 0}, dy[] = {0, -1, 0, 1};
            queue<PII> q;
            for(int i = 0; i < m; i ++)
                for(int j = 0; j < n; j ++){
                    if(grid[i][j] == 2) q.emplace(i, j);
                    if(grid[i][j] == 1) cnt ++;
                }
            while(!q.empty()){
                int sz = q.size();
                for(int i = 0; i < sz; i ++){
                    auto t = q.front();
                    q.pop();
                    for(int i = 0; i < 4; i ++){
                        int a = t.x + dx[i], b = t.y + dy[i];
                        if(a < 0 || a >= m || b < 0 || b >= n) continue;
                        if(grid[a][b] == 1){
                            p ++;
                            grid[a][b] = 2;
                            q.emplace(a, b);
                        }
                    }
                }
                if(!q.empty()) res ++;
            }
            return p < cnt ? -1 : res;
        }
    };
    ```
    

## 🟡 **[1162. 地图分析](https://leetcode.cn/problems/as-far-from-land-as-possible/) (多源BFS)**

- 题目
    
    你现在手里有一份大小为 `n x n` 的 网格 `grid`，上面的每个 单元格 都用 `0` 和 `1` 标记好了。其中 `0` 代表海洋，`1` 代表陆地。
    
    请你找出一个海洋单元格，这个海洋单元格到离它最近的陆地单元格的距离是最大的，并返回该距离。如果网格上只有陆地或者海洋，请返回 `-1`。
    
    我们这里说的距离是「曼哈顿距离」（ Manhattan Distance）：`(x0, y0)` 和 `(x1, y1)` 这两个单元格之间的距离是 `|x0 - x1| + |y0 - y1|` 。
    
- 解答
    
    ```cpp
    #define INF 0x3f3f3f3f
    #define x first
    #define y second
    typedef pair<int, int> PII;
    class Solution {
    public:
        int maxDistance(vector<vector<int>>& grid) {
            int m = grid.size(), n = grid[0].size();
            vector<vector<int>> dist(m, vector<int>(n, INF));
            int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
            queue<PII> q;
            for(int i = 0; i < m; i ++)
                for(int j = 0; j < n; j ++)
                    if(grid[i][j]) {
                        dist[i][j] = 0;
                        q.emplace(i, j);
                    }
            while(!q.empty()){
                auto t = q.front();
                q.pop();
                for(int i = 0; i < 4; i ++){
                    int a = t.x + dx[i], b = t.y + dy[i];
                    if(a < 0 || a >= m || b < 0 || b >= n) continue;
                    if(dist[a][b] > dist[t.x][t.y] + 1){
                        dist[a][b] = dist[t.x][t.y] + 1;
                        q.emplace(a, b);
                    }
                }
            }
            int res = -1;
            for(int i = 0; i < m; i ++)
                for(int j = 0; j < n; j ++)
                    if(!grid[i][j]) res = max(res, dist[i][j]);
            return (res == INF) ? -1 : res;
    				// 若为INF则对应网格上只有陆地或者海洋的情况
        }
    };
    ```
    

## 🔴 **[127. 单词接龙](https://leetcode.cn/problems/word-ladder/description/)（双向BFS）**

- 题目
    
    字典 `wordList` 中从单词 `beginWord` **和 `endWord` 的 **转换序列** 是一个按下述规格形成的序列 `beginWord -> s1 -> s2 -> ... -> sk`：
    
    - 每一对相邻的单词只差一个字母。
    - 对于 `1 <= i <= k` 时，每个 `si` 都在 `wordList` 中。注意， `beginWord` **不需要在 `wordList` 中。
    - `sk == endWord`
    
    给你两个单词 **`beginWord` **和 `endWord` 和一个字典 `wordList` ，返回 *从 `beginWord` 到 `endWord` 的 **最短转换序列** 中的 **单词数目*** 。如果不存在这样的转换序列，返回 `0` 。
    
    **示例 1：**
    
    ```
    输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
    输出：5
    解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5。
    ```
    
- 解答
    
    ![两种算法效率的比较](img/bfs_compare.png)
    
    - **朴素BFS**
        
        ```cpp
        class Solution {
        public:
            int bfs(string& begin, string& end, unordered_map<string, int>& list){
                unordered_map<string, int> dist;
                if(!list.count(end)) return 0;
                queue<string> q;
                q.push(end);
                dist[end] = 1; // 由于题目要求序列的单词数目，故初始值为1
                while(q.size()){
                    auto t = q.front(); q.pop();
                    for(int i = 0; i < t.size(); i ++)
                        for(int j = 0; j < 26; j ++){
                            string tt = t;
                            tt[i] = 'a' + j;
                            if(tt == begin) return dist[t] + 1;
                            if(!list.count(tt)) continue;
                            if(!dist.count(tt)) {
                                q.push(tt);
                                dist[tt] = dist[t] + 1;
                            }
                        }
                }
                return 0;
            }
            int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
                unordered_map<string, int> list;
                for(auto x : wordList) list[x] ++;
                return bfs(beginWord, endWord, list);
            }
        };
        ```
        
    - **双向BFS**
        
        ```cpp
        class Solution {
        public:
            int extend(queue<string>& q, unordered_map<string, int>& da, 
         unordered_map<string, int>& db, unordered_map<string, int>& list){
                int d = da[q.front()];
                while(q.size() && da[q.front()] == d){
                    auto t = q.front();
                    q.pop();
                    for(int i = 0; i < t.size(); i ++){
                        for(int j = 0; j < 26; j ++){
                            string tt = t;
                            tt[i] = j + 'a';
                            if(db.count(tt)) return da[t] + db[tt] + 1;
                            if(!list.count(tt)) continue;
                            if(!da.count(tt)){
                                q.push(tt);
                                da[tt] = da[t] + 1;
                            }  
                        }
                    }
                }
                return -1;
            }
            int bfs(string& begin, string& end, unordered_map<string, int>& list){
                if(!list.count(end)) return 0;
                unordered_map<string, int> da, db;
                queue<string> qa, qb;
                qa.push(begin); da[begin] = 0;
                qb.push(end); db[end] = 1;
                while(qa.size() && qb.size()){
                    int t;
                    if(qa.size() <= qb.size()) t = extend(qa, da, db, list);
                    else t = extend(qb, db, da, list);
                    if(t != -1) return t;
                }
                return 0;
            }
            int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
                unordered_map<string, int> list;
                for(auto x : wordList) list[x] ++;
                return bfs(beginWord, endWord, list);
            }
        };
        ```
        
    

## 🟡 **[LCP 56. 信物传送](https://leetcode.cn/problems/6UEx57/)（0-1 BFS）**

- 题目
    
    欢迎各位勇者来到力扣城，本次试炼主题为「信物传送」。
    
    本次试炼场地设有若干传送带，`matrix[i][j]` 表示第 `i` 行 `j` 列的传送带运作方向，`"^","v","<",">"` 这四种符号分别表示 **上、下、左、右** 四个方向。信物会随传送带的方向移动。勇者**每一次**施法操作，可**临时**变更一处传送带的方向，在物品经过后传送带恢复原方向。
    
    通关信物初始位于坐标 `start` 处，勇者需要将其移动到坐标 `end` 处。
    
    请返回勇者施法操作的最少次数。
    
- 解答
    
    在以下两种做法中，使用初始值为INF的dist数组记录每个点到起点的最短距离。在扩展时，只有d小于dist时才会加入队列中，此时不需要bool数组记录是否被访问过，dist[m-1][n-1]即是最终答案。
    
    若不使用dist数组，也可用bool数组标记是否被访问，这时只需用res来记录遍历的总层数，即为最短路。
    
    时间复杂度：**O(mn)**
    
    - **使用Deque**
        
        ```cpp
        #define x first
        #define y second
        typedef pair<int, int> PII;
        class Solution {
        public:
            int conveyorBelt(vector<string>& matrix, vector<int>& start, vector<int>& end) {
                int m = matrix.size(), n = matrix[0].size();
                int res = 0, dist[m][n];
                bool st[m][n];
                memset(st, 0, sizeof st);
                deque<PII> q;
                q.emplace_front(start[0], start[1]);
                int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
                char c[] = {'^', '>', 'v', '<'};
                while(!q.empty()){
                    int sz = q.size();
                    for(int i = 0; i < sz; i ++){
                        auto t = q.front();
                        q.pop_front();
                        if(st[t.x][t.y]) continue;
                        st[t.x][t.y] = true;
                        if(t.x == end[0] && t.y == end[1]) return res;
                        for(int k = 0; k < 4; k ++){
                            int a = t.x + dx[k], b = t.y + dy[k];
                            if(a < 0 || a >= m || b < 0 || b >= n) continue;
                            if(c[k] == matrix[t.x][t.y]) {
                                i --;
                                q.emplace_front(a, b);
                            } else q.emplace_back(a, b);
                        }
                    }
                    res ++;
                }
                return res;
            }
        };
        ```
        
    - **使用临时队列**
        
        ```cpp
        #define x first
        #define y second
        typedef pair<int, int> PII;
        class Solution {
        public:
            int conveyorBelt(vector<string>& matrix, vector<int>& start, vector<int>& end) {
                int m = matrix.size(), n = matrix[0].size();
                int res = 0, dist[m][n];
                bool st[m][n];
                memset(st, 0, sizeof st);
                queue<PII> q;
                q.emplace(start[0], start[1]);
                int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
                char c[] = {'^', '>', 'v', '<'};
                while(!q.empty()){
                    queue<PII> qt;
                    while(!q.empty()){
                        auto t = q.front();
                        q.pop();
                        if(st[t.x][t.y]) continue;
                        st[t.x][t.y] = true;
                        if(t.x == end[0] && t.y == end[1]) return res;
                        for(int k = 0; k < 4; k ++){ 
                            int a = t.x + dx[k], b = t.y + dy[k];
                            if(a < 0 || a >= m || b < 0 || b >= n) continue;
                            if(c[k] != matrix[t.x][t.y])  qt.emplace(a, b); 
                            else q.emplace(a, b);
                        }
                    }
                    if(!qt.empty()) res ++;
                    while(!qt.empty()){
                        q.push(qt.front());
                        qt.pop();
                    }
                }
                return res;
            }
        };
        ```
        

## 🔴 **[2290. 到达角落需要移除障碍物的最小数目](https://leetcode.cn/problems/minimum-obstacle-removal-to-reach-corner/)（0-1 BFS）**

- 题目
    
    给你一个下标从 **0** 开始的二维整数数组 `grid` ，数组大小为 `m x n` 。每个单元格都是两个值之一：
    
    - `0` 表示一个 **空** 单元格，
    - `1` 表示一个可以移除的 **障碍物** 。
    
    你可以向上、下、左、右移动，从一个空单元格移动到另一个空单元格。
    
    现在你需要从左上角 `(0, 0)` 移动到右下角 `(m - 1, n - 1)` ，返回需要移除的障碍物的 **最小** 数目。
    
- 解答
    
    ```cpp
    #define x first
    #define y second
    typedef pair<int, int> PII;
    class Solution {
    public:
        int bfs(vector<vector<int>>& grid, int m, int n){
            int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
            int dist[m][n];
            memset(dist, 0x3f, sizeof dist);
            deque<PII> q;
            q.emplace_front(0, 0);
            dist[0][0] = 0;
            while(!q.empty()){
                auto t = q.front();
                q.pop_front();
                for(int i = 0; i < 4; i ++){
                    int a = t.x + dx[i], b = t.y + dy[i];
                    if(a < 0 || a >= m || b < 0 || b >= n) continue;
                    int d = dist[t.x][t.y] + grid[a][b];
                    if(d < dist[a][b]){
                        dist[a][b] = d;
                        grid[a][b] == 0 ? q.emplace_front(a, b) : q.emplace_back(a, b);
                    }
                }
            }
            return dist[m - 1][n - 1];
        }
        int minimumObstacles(vector<vector<int>>& grid) {
            int m = grid.size(), n = grid[0].size();
            int t;
            return bfs(grid, m, n);
        }
    };
    ```
    

## 🔴 **[1368. 使网格图至少有一条有效路径的最小代价](https://leetcode.cn/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/)（特殊的0-1 BFS）**

- 题目
    
    给你一个 m x n 的网格图 `grid` 。 `grid` 中每个格子都有一个数字，对应着从该格子出发下一步走的方向。 `grid[i][j]` 中的数字可能为以下几种情况：
    
    - **1** ，下一步往右走，也就是你会从 `grid[i][j]` 走到 `grid[i][j + 1]`
    - **2** ，下一步往左走，也就是你会从 `grid[i][j]` 走到 `grid[i][j - 1]`
    - **3** ，下一步往下走，也就是你会从 `grid[i][j]` 走到 `grid[i + 1][j]`
    - **4** ，下一步往上走，也就是你会从 `grid[i][j]` 走到 `grid[i - 1][j]`
    
    注意网格图中可能会有 **无效数字** ，因为它们可能指向 `grid` 以外的区域。
    
    一开始，你会从最左上角的格子 `(0,0)` 出发。我们定义一条 **有效路径** 为从格子 `(0,0)` 出发，每一步都顺着数字对应方向走，最终在最右下角的格子 `(m - 1, n - 1)` 结束的路径。有效路径 **不需要是最短路径** 。
    
    你可以花费 `cost = 1` 的代价修改一个格子中的数字，但每个格子中的数字 **只能修改一次** 。
    
    请你返回让网格图至少有一条有效路径的最小代价。
    
- 解答
    
    ```cpp
    #define x first
    #define y second
    typedef pair<int, int> PII;
    class Solution {
    public:
        int bfs(vector<vector<int>>& grid, int m, int n){
            int res = 0;
            int dx[] = {0, 0, 0, 1, -1}, dy[] = {0, 1, -1, 0, 0};
            vector<vector<bool>> st(m, vector<bool>(n, 0));
            deque<PII> q;
            q.emplace_front(0, 0);
            while(!q.empty()){
                int sz = q.size();
                for(int i = 0; i < sz; i ++){ // 每次遍历一整层
                    auto t = q.front();
                    q.pop_front();
                    st[t.x][t.y] = true;
                    if(t.x == m - 1 && t.y == n - 1) return res;
                    for(int j = 1; j <= 4; j ++){
                        int a = t.x + dx[j], b = t.y + dy[j];
                        if(a < 0 || a >= m || b < 0 || b >= n || st[a][b]) continue;
                        if(grid[t.x][t.y] == j){
                            i --; // 当前的一层新增一个节点故i--
                            q.emplace_front(a, b);
                        } else {
                            q.emplace_back(a, b);
                        }
                    }
                }
                res ++;
            }
            return res;
        }
        int minCost(vector<vector<int>>& grid) {
            int m = grid.size(), n = grid[0].size();
            return bfs(grid, m, n);
        }
    };
    ```
    

## 🟡 **[792. 匹配子序列的单词数](https://leetcode.cn/problems/number-of-matching-subsequences/)**

- 题目
    
    给定字符串 `s` 和字符串数组 `words`, 返回  *`words[i]` 中是`s`的子序列的单词个数* 。
    
    字符串的 **子序列** 是从原始字符串中生成的新字符串，可以从中删去一些字符(可以是none)，而不改变其余字符的相对顺序。
    
    - 例如， `“ace”` 是 `“abcde”` 的子序列。
    
    **示例 1:**
    
    ```
    输入: s = "abcde", words = ["a","bb","acd","ace"]
    输出: 3
    解释: 有三个是 s 的子序列的单词: "a", "acd", "ace"。
    ```
    
    **Example 2:**
    
    ```
    输入:s = "dsahjpjauf", words = ["ahjpjau","ja","ahbwzgqnuk","tnmlanowax"]
    输出: 2
    ```
    
    **提示:**
    
    - `1 <= s.length <= 5 * 104`
    - `1 <= words.length <= 5000`
    - `1 <= words[i].length <= 50`
    - `words[i]`和  都只由小写字母组成。
- 解答
    
    ```cpp
    typedef pair<int, int> PII;
    #define x first
    #define y second
    
    class Solution {
    public:
        int numMatchingSubseq(string s, vector<string>& words) {
            vector<PII> ps[26];
            for(int i = 0; i < words.size(); i ++)
                ps[words[i][0] - 'a'].push_back({i, 0});
            int res = 0;
            for(auto& c: s){
                vector<PII> buf;
                for(auto&p: ps[c - 'a'])
                    if(p.y + 1 == words[p.x].size()) res ++;
                    else buf.push_back({p.x, p.y + 1});
                ps[c - 'a'].clear();
                for(auto& p: buf)
                    ps[words[p.x][p.y] - 'a'].push_back(p);
            }
            return res;
        }
    };
    ```
