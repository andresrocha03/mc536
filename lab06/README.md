## Exercício

Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (p1:Pathology)<-[a]-(d:Drug)-[b]->(p2:Pathology)
MERGE (p1)<-[r:Relates]->(p2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1
~~~

# Trabalhando com Efeitos Colaterais

Considere o seguinte arquivo que indica um conjunto de pessoas (identificadas por código) e as drogas que elas usam:

[https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv]

Considere este outro arquivo que indica as mesmas pessoas e efeitos colaterais que elas experimentaram:

[https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv]

## Exercício

Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
CREATE(:Symptom {code: line.codePathology, person: line.idPerson})

CREATE INDEX FOR (n:Symptom) ON (n.code)

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MATCH (d:Drug {code: line.codedrug})
MATCH (p:Pathology {code: line.codepathology})
CREATE (d)-[:Cure {person: line.idperson}]->(p)

MATCH (d:Drug)-[c:Cure]->(p:Pathology)
MATCH (s:Symptom) 
WHERE (s.person) = c.person
MERGE (d)-[g:Generate]->(s)
ON CREATE
    SET g.weight = 1
ON MATCH 
    SET g.weight = g.weight + 1
~~~

## Exercício

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução
É possível analisar quais são os sintomas mais/menos comuns gerados por um dado medicamento. Segue a setença em cypher para observar os mais utilizados, o valor de 50 foi escolhido arbitrariamente como um limite significativo para determinar se o sintoma é altamente comum ou não.  

~~~cypher
MATCH (d)-[g:Generates]->(s)
WHERE g.weight > 50
RETURN d,s
~~~