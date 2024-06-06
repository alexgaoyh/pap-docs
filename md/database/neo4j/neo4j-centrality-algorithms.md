# [图数据库]Neo4j中心性算法-以红楼梦为例

## 背景

&ensp;&ensp;近期由于知识图谱的原因，又一次将关注点转移到了图数据库Neo4j中，之所以选择Neo4j，还是因为前些年已经对其进行了调研，上手较快，不同的应用场景可以会有其他的技术选型。

&ensp;&ensp;本文期望提供知识图谱增强下的资源推荐功能，其他使用中心性算法帮助发现用户的兴趣点和社交圈子，从而提供更加个性化的推荐内容。

## 介绍

&ensp;&ensp;由于作者使用的是 Neo4j 3.5版本，所以引入了插件进行了增强 [graph-algorithms-algo-3.5.4.0.jar](https://github.com/neo4j-contrib/neo4j-graph-algorithms)，如果是较新的版本，引入的插件包发生了变化，本文不做过多介绍。

&ensp;&ensp;本文对中心性算法： 度中心度、紧密中心度、中介中心度、特征向量中心度 分别进行了介绍。

## 代码

&ensp;&ensp;首先是针对不同中心性算法的 Cypher 查询语句。

```sql
-- 度中心度
CALL algo.degree.stream("HLM", "RELATIONSHIP", {direction: "Both"}) 
YIELD nodeId, score 
RETURN algo.asNode(nodeId).name AS name, score AS degree 
ORDER BY degree DESC

-- 紧密中心度
CALL algo.closeness.stream("HLM", "RELATIONSHIP") 
YIELD nodeId, centrality 
MATCH (n:HLM) WHERE id(n) = nodeId 
RETURN n.name AS node, centrality 
ORDER BY centrality DESC 
LIMIT 20;

-- 中介中心度
MATCH (c:HLM) 
WITH collect(c) as characters 
CALL algo.betweenness.stream("HLM", "RELATIONSHIP") 
YIELD nodeId, centrality 
MATCH (c) WHERE id(c) = nodeId 
RETURN c.name as name, centrality 
ORDER BY centrality desc

-- 特征向量中心度
CALL algo.pageRank.stream('HLM', 'RELATIONSHIP', {iterations:20, dampingFactor:0.85}) 
YIELD nodeId, score 
RETURN algo.asNode(nodeId).name AS name, score 
ORDER BY score DESC

```

&ensp;&ensp;其次是针对如上查询语句，在处理知识图谱数据过程中对应的节点(HLM)和关系(RELATIONSHIP)的设置。

```java
@Node("HLM")
public class HLMEntity implements Serializable {

    @Id
    private String name;

    @Relationship(type = "RELATIONSHIP", direction = Relationship.Direction.OUTGOING)
    private Set<HLMRelationshipEntity> relationships = new HashSet<>();
}

@RelationshipProperties
public class HLMRelationshipEntity implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Relationship(type = "relation", direction = Relationship.Direction.OUTGOING)
    private HLMEntity startNode;

    @Relationship(type = "relation", direction = Relationship.Direction.INCOMING)
    @TargetNode
    private HLMEntity endNode;

    private String type;
}

```

## 执行效果

&ensp;&ensp;第一张图是红楼梦人物数据的展示图，后面四张图是四种中心性算法的执行结果。

![hlm.jpg](https://s2.loli.net/2024/06/06/4Y3GLAn9NKoX1db.jpg)

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/06/06/U2ziueG3hD1VgPa.jpg" alt="Neo4j中心性算法-以红楼梦为例-度中心度" style="width: 45%;">
    <img src="https://s2.loli.net/2024/06/06/91cnMYpPSixwGa5.jpg" alt="Neo4j中心性算法-以红楼梦为例-紧密中心度" style="width: 45%;">
</div>

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/06/06/AHio8y4aRTDuNX2.jpg" alt="Neo4j中心性算法-以红楼梦为例-中介中心度" style="width: 45%;">
    <img src="https://s2.loli.net/2024/06/06/oOgt6ZyFuWeSKTd.jpg" alt="Neo4j中心性算法-以红楼梦为例-特征向量中心度" style="width: 45%;">
</div>

## 总结

&ensp;&ensp; 使用Neo4j的graph-algorithms-algo插件，调用了针对红楼梦人物关系数据的四种中心度算法。

## 参考

2. http://pap-docs.pap.net.cn/
3. https://gitee.com/alexgaoyh
