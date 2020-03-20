---
title: neo4j 学习笔记
date: 2018-02-17 09:59:18
tags: neo4j
categories: db
---
## 一些基本概念，从官网copy过来的，如下：

### Graph Fundamentals
Basic concepts to get you going.

A graph database can store any kind of data using a few simple concepts:

1. Nodes - graph data records
2. Relationships - connect nodes
3. Properties - named data values


### A Graph Database
Neo4j stores data in a Graph, with records called Nodes.

The simplest graph has just a single node with some named values called Properties. Let's draw a social graph of our friends on the Neo4j team:

1. Start by drawing a circle for the node
2. Add the name Emil
3. Note that he is from Sweden

* Nodes are the name for data records in a graph
* Data is stored as Properties
* Properties are simple name/value pairs


### Labels
Associate a set of nodes.

Nodes can be grouped together by applying a Label to each member. In our social graph, we'll label each node that represents a Person.

1. Apply the label "Person" to the node we created for Emil
2. Color "Person" nodes red

* A node can have zero or more labels
* Labels do not have any properties


### More Nodes
Schema-free, nodes can have a mix of common and unique properties.

Like any database, storing data in Neo4j can be as simple as adding more records. We'll add a few more nodes:

1. Emil has a klout score of 99
2. Johan, from Sweden, who is learning to surf
3. Ian, from England, who is an author
4. Rik, from Belgium, has a cat named Orval
5. Allison, from California, who surfs

* Similar nodes can have different properties
* Properties can be strings, numbers, or booleans
* Neo4j can store billions of nodes


### Consider Relationships
Connect nodes in the graph

The real power of Neo4j is in connected data. To associate any two nodes, add a Relationship which describes how the records are related.

In our social graph, we simply say who KNOWS whom:

1. Emil KNOWS Johan and Ian
2. Johan KNOWS Ian and Rik
3. Rik and Ian KNOWS Allison

* Relationships always have direction
* Relationships always have a type
* Relationships form patterns of data


### Relationship properties
Store information shared by two nodes.

In a property graph, relationships are data records that can also contain properties. Looking more closely at Emil's relationships, note that:

* Emil has known Johan since 2001
* Emil rates Ian 5 (out of 5)
* Everyone else can have similar relationship properties


可以使用这种方式访问neo4j数据库：
``` bash
http://localhost:7474/browser/
```

### 清空neo4j数据库
``` bash
match(n)
optional match(n)-[r]-()
delete n,r
```
执行上面的命令后，会删除节点及与该节点相关的关系，但是property keys删不干净

未完，待续……
