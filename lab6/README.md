## Tarefa de Cypher e o FDA Adverse Event Reporting System (FAERS)

## Exercício 1

Escreva uma sentença em Cypher que crie o medicamento de nome `Metamizole`, código no DrugBank `DB04817`.

### Resolução
~~~cypher
 CREATE (:Drug {drugbank: "DB04817", name:"Metamizole"}
~~~

## Exercício 2

Considerando que a `Dipyrone` e `Metamizole` são o mesmo medicamento com nomes diferentes, crie uma aresta com o rótulo `:SameAs` que ligue os dois.

### Resolução
~~~cypher
 MATCH (d1:Drug {name:"Dipyrone"})
 MATCH (d2:Drug {name:"Metamizole"})
 CREATE (d1)-[:SameAs]->(d2)
~~~

## Exercício 3

Use o `DELETE` para excluir o relacionamento que você criou (apenas ele).

### Resolução
~~~cypher
 MATCH (:Drug {name:"Dipyrone"})-[same:SameAs]->(:Drug {name:"Metamizole"})
 DELETE same
~~~

## Exercício 4

Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
 MATCH (p1:Pathology)<-[a]-(d1:Drug)-[b]->(p2:Pathology)
 MERGE (p1)<-[r:Relates]->(p2)
 ON CREATE SET r.weight=1
 ON MATCH SET r.weight=r.weight+1

 MATCH (p1: Pathology)<-[r:Relates]->(p2: Pathology) 
 RETURN p1, p2
 LIMIT 30
~~~

## Exercício 5

Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
 LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
 CREATE(:DrugUse {idperson: line.idperson, codedrug: line.codedrug})

 LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
 CREATE(:SideEffect {idperson: line.idperson, codepathology: line.pathology})

 MATCH(d:DrugUse)
 MATCH(s:SideEffect)
 MATCH(droga:Drug)
 WHERE d.idperson = s.idperson AND d.codedrug = droga.code
 MERGE (d)-[r:relacao]->(s)
 ON CREATE SET r.weight=1
 ON MATCH SET r.weight=r.weight+1

 MATCH (d: DrugUse)-[r:relacao]->(s:SideEffect)
 WHERE r.weight > 20
 RETURN d,s
 LIMIT 50
~~~

## Exercício 6

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução

Podemos verificar todos os efeitos colaterais relatados para cada uma das drogas que tenha pelo menos peso 30, ou seja, fazer uma conexão entre dois efeitos colaterais ocasionados por aquele remédio.

~~~cypher
MATCH (s1: SideEffect)-[r1:relacao]->[d:Drug]<-[r2:relacao]-(s2:SideEffect)
WHERE r1.weight > 30 and r2.weight > 30
MERGE (s1)-[:Droga {drogaEfeito: d.name}]->(s2)
ON CREATE SET Droga.weight = 1
ON MATCH SET Droga.weight = Droga.weight + 1

MATCH (s1: SideEffect)-[:Droga]->(s2: SideEffect)
RETURN s1, s2
LIMIT 30
~~~


