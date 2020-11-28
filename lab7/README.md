## Tarefa de análises feitas no Cypher

## Exercício 1

Calcule o Pagerank do exemplo da Wikipedia em Cypher:

~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/network/pagerank/pagerank-wikipedia.csv' AS line
MERGE (p1:Page {name:line.source})
MERGE (p2:Page {name:line.target})
CREATE (p1)-[:LINKS]->(p2)


CALL gds.graph.create(
  'wikipediaGraph',
  'Page',
  'LINKS'
)

CALL gds.pageRank.stream('wikipediaGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC, name ASC
~~~

## Exercício 2

Departing from a Drug-Drug graph created in a previous lab, whose relationship determines drugs taken together, apply a community detection in it to see the results:

~~~cypher
CALL gds.graph.create(
  'drugGraph',
  'Drug',
  {
    Relates: {
      orientation: 'UNDIRECTED'
    }
  }
)


CALL gds.louvain.stream('drugGraph')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).name AS name, communityId
ORDER BY communityId ASC

CALL gds.louvain.stream('DrugGraph')
YIELD nodeId, communityId
MATCH (p:Drug {code: gds.util.asNode(nodeId).code})
SET p.community = communityId

CALL gds.louvain.stream('DrugGraph')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).name AS name, communityId

~~~

