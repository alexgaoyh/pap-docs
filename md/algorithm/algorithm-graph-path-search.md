# 图数据结构-路径查找

## 背景
&ensp;&ensp;现实场景中会有很多关于路径查找的场景，都可以用图算法进行解决，本文算法在有向图中进行查找，返回所有可能的路径，并且可以返回路径间的权重。

## 应用场景
1. 关联反查：查询从某一实体节点出发按照指定关系可以到达的节点；
2. 最短路径：查找两个实体节点之间的最短到达路径
3. 闭环搜索：查询是否存在从一个实体节点出发经过若干路径可以回到原点的路径
4. 共同邻居：用于查找多个节点共有的邻居节点

## 算法
&ensp;&ensp;本例定义了图数据结构，可以向图中增加边（含边的权重），返回起始点到终止点之间所有可能的路径，同时可以查找出每条路径中的权重。

```java
import java.util.*;

/**
 * 使用邻接表存储图，并返回指定节点间的所有可能路径。
 */
public class DAGPaths {
    private int V;
    private Map<String, Integer> indexMap;
    private Map<Integer, String> nameMap;
    private List<Edge>[] adj;

    private static class Edge {
        int to;
        String weight;

        public Edge(int to, String weight) {
            this.to = to;
            this.weight = weight;
        }
    }

    public DAGPaths(int V) {
        this.V = 0;
        indexMap = new HashMap<>();
        nameMap = new HashMap<>();
        adj = new ArrayList[V];
        for (int i = 0; i < V; i++) {
            adj[i] = new ArrayList<Edge>();
        }
    }

    public void addEdge(String from, String to, String weight) {
        BSTNode fromNode = findNode(from);
        BSTNode toNode = findNode(to);
        if (fromNode == null) {
            fromNode = new BSTNode(from, V++);
            indexMap.put(from, fromNode.index);
            nameMap.put(fromNode.index, from);
        }
        if (toNode == null) {
            toNode = new BSTNode(to, V++);
            indexMap.put(to, toNode.index);
            nameMap.put(toNode.index, to);
        }
        adj[fromNode.index].add(new Edge(toNode.index, weight));
    }

    private BSTNode findNode(String name) {
        int index = indexMap.getOrDefault(name, -1);
        if (index == -1) {
            return null;
        }
        return new BSTNode(name, index);
    }

    public List<List<String>> getAllPaths(String start, String end) {
        List<List<String>> paths = new ArrayList<>();
        int startIdx = indexMap.getOrDefault(start, -1);
        int endIdx = indexMap.getOrDefault(end, -1);
        if (startIdx == -1 || endIdx == -1) {
            return paths;
        }
        boolean[] visited = new boolean[V];
        Stack<Integer> stack = new Stack<>();
        stack.push(startIdx);
        visited[startIdx] = true;
        dfs(paths, stack, visited, endIdx);
        return paths;
    }

    private void dfs(List<List<String>> paths, Stack<Integer> stack, boolean[] visited, int end) {
        int curr = stack.peek();
        if (curr == end) {
            List<String> path = new ArrayList<>();
            for (int i : stack) {
                path.add(nameMap.get(i));
            }
            paths.add(path);
            return;
        }
        for (Edge e : adj[curr]) {
            if (!visited[e.to]) {
                stack.push(e.to);
                visited[e.to] = true;
                dfs(paths, stack, visited, end);
                visited[e.to] = false;
                stack.pop();
            }
        }
    }

    private List<String> getWeight(List<String> path) {
        List<String> weightList = new ArrayList<>();

        for(int left = 0; left < path.size() - 1; left++) {
            int leftIndex = findNode(path.get(left + 1)).index;
            String weight = adj[findNode(path.get(left)).index].stream().filter(e -> e.to == leftIndex).findFirst().get().weight;
            weightList.add(weight);
        }

        return weightList;
    }

    private static class BSTNode {
        String name;
        int index;

        public BSTNode(String name, int index) {
            this.name = name;
            this.index = index;
        }
    }

    public static void main(String[] args) {
        DAGPaths graph = new DAGPaths(8);
        graph.addEdge("A", "B", "2");
        graph.addEdge("A", "C", "3");
        graph.addEdge("B", "D", "1");
        graph.addEdge("C", "D", "5");
        graph.addEdge("C", "E", "4");
        graph.addEdge("D", "E", "1");
        graph.addEdge("E", "F", "2");
        graph.addEdge("E", "G", "3");
        graph.addEdge("F", "H", "2");
        graph.addEdge("G", "H","3");

        List<List<String>> paths = graph.getAllPaths("A", "H");
        for (List<String> path : paths) {
            System.out.println(path + " : " + graph.getWeight(path));
        }
    }
}
```

