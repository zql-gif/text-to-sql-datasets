1. CoSQL原文对template-base的方法只做简单描述，并未给出源码或者参考的论文
2. [[1809.05255] SQL-to-Text Generation with Graph-to-Sequence Model](https://arxiv.org/abs/1809.05255)一文中提到"Earlier attempts for SQL-to-text task are rulebased and template-based (Koutrika et al., 2010; Ngonga Ngomo et al., 2013).", 两篇文章如下：
* [Explaining structured queries in natural language | IEEE Conference Publication | IEEE Xplore](https://ieeexplore.ieee.org/document/5447824)
* [Sorry, i don't speak SPARQL | Proceedings of the 22nd international conference on World Wide Web](https://dl.acm.org/doi/abs/10.1145/2488388.2488473)

# links
https://arxiv.org/abs/2208.10099

# 0 CoSQL: Template-based

1. Create a list of SQL query patterns without values, column and table names from SQL and NL response pairs in the training set : mask variable values to form SQL-response **templates** 
2. Manual mapping : manually changed the patterns and their corresponding responses to make sure that table, column, and value slots in the responses have one-to-one map to the slots in the SQL query
3. Match a new SQL quary with the templates :  A score will be computed to represent the similarity between the SQL and each template, which is based on the number of each SQL key components existing in the SQL and each template (原文描述为**rule-based approach** to select the closest SQL-response template pair)
4. Fill in the selected response template

# 1 Explaining Structured Queries in Natural Language

## 1.1 Task
Given a query q over a database D, we would like to generate a narrative that captures the intended meaning or objective of q.
## 1.2 Challenges
 1. Insufficient SQL semantics
 2. Complexity of the queries
 3. sql具有多种可替代的表述 ：**Several alternative expressions of a query in a formal language that are equivalent**
 4. 语义自然：Capture the query elements in the right order so that the corresponding textual expression is natural and meaningful independent of the way the user has expressed the query
## 1.3 Overview
1. A graph-based approach for **representing structured queries as directed graphs**
2. Annotate the graph elements with labels using **an extensible template mechanism**
3. Three translation strategies
* BST algorithm : the translation consists of a composition of clauses each one focusing on specific query semantics
* MRP algorithm : the translation is realized in a holistic manner, where information from all parts of the query graph is blended in the translation as we traverse the graph. 
* TMT algorithm : enables the use of predefined, richer, templates for query parts in an effort to produce more concise translations.
## 1.4 Query Graph Representation
### A Database Graph
符号解释如下：
* D : a database comprises a set of relations
* Ri : a relation with a set of attributes in database D
* Aij(上下标) : an attribute of Ri
* Database graph G(V, E) : a directed graph corresponding to the schema of D 

关于G(V,E)的点集V中点有 ：
* R : relation nodes
* A: attribute node

关于G(V,E)的边集E中边有 ：
* membership edges, Eμ : connect an attribute node to its container relation node. 从属性点 Aij指向它的关系点Ri
* selection edges, Eσ : connect each relation to each of its attributes. 这种边是为了体现select语句中关系relation和所选择的属性之间的联系，是双向的 
* predicate edges, Eθ : emanate from an attribute node and ending at another attribute node. 代表两个不同关系之间的join，以各自的属性作为作为join的共同属性

下面是一个例子，分别给出了database和对应表达了join关系的G(V,E)图：
![example_course_database](attachment/example_course_database.png)


![A_join_on_database_graph](attachment/A_join_on_database_graph.png)

本文的方法以这种图的形式捕捉sql的语义

### B Query Graphs
符号解释如下：
* q : SPJ Query q是指一种由 **选择（Selection, S）**、**投影（Projection, P）** 和 **连接（Join, J）** 这三种关系操作组成的关系查询
* Gq (Vq , Eq) : a directed graph that is an extension of the database graph G(V,E)

关于Gq (Vq , Eq)的点集V中点有 ：
* R : relation nodes
* A: attribute node