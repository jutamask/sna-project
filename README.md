# Comorbidity Networks Analysis in Diseases of the Circulatory System
### Setup and Install Neo4j Desktop
 - Download Neo4j Desktop [Click here](https://neo4j.com/download/)
 - About Neo4j [Click here](https://neo4j.com/product/#neo4j-desktop)
### Create New Project and Plugins
- Create new project and Local DBMS [Click here](https://github.com/jutamask/sna-project/blob/main/Create%20new%20project%20.pdf)
- Install and additional Plugin Graph Data Science Library  [Click here](https://github.com/jutamask/sna-project/blob/main/Install%20-%20Graph%20Data%20Science%20Library.pdf)
### Dataset
- Dataset Diagnosis-Related  [Click here](https://github.com/jutamask/sna-project/blob/main/data_sna.csv)
- Dataset Description-ICD10 [Click here](https://github.com/jutamask/sna-project/blob/main/descript_icd_dx.csv)

### Import Data to Neo4j
- Import data to project [Click here](https://github.com/jutamask/sna-project/blob/main/Import%20Data%20to%20Neo4j.pdf)
### Load Dataset into Local DBMS
- Load Dataset Sample Query [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Load%20Data.pdf)
#### Cypher Query
Run all the example queries:
- Load Data Diagnosis-Related to Local DBMS
```
\\Create Constraint
CREATE CONSTRAINT ON (i:id) ASSERT i.id IS UNIQUE;
```
```
\\Load csv dataset on import into local DB and Create Node 'id'
LOAD CSV WITH HEADERS FROM 
'file:///data_sna.csv' AS line 
CREATE (id:id {id: line.id, pdx_id: line.Pdx, sdx_id: line.Sdx, n_id: line.number_of_patient});
```
- Load Data Description-ICD10 to Local DBMS
```
\\Create Constraint
CREATE CONSTRAINT ON (d:dx) ASSERT d.dx IS UNIQUE;
```
```
\\Load csv dataset on import into local DB and Create Node 'dx'
LOAD CSV WITH HEADERS FROM 'file:///descript_icd_dx.csv' AS line 
CREATE (dx:dx {dx: line.dx, term_d: line.TERM, chapter_d: line.CHAPTER, block_d: line.BLOCK});
```
- Create Relationship
```
\\To determine which Node id has the primary diagnosis (PDx), establish a HasPDx relationship.
MATCH (i:id) UNWIND i.pdx_id as pdx MATCH (d:dx {dx:pdx}) MERGE (i)-[:HasPDx]-(d) ;
```
```
\\To determine which Node id has the secondary diagnosis (SDx), establish a HasSDx relationship.
MATCH (i:id) UNWIND i.sdx_id as sdx MATCH (d:dx {dx:sdx}) MERGE (i)-[:HasSDx]-(d) ;
```
```
\\Create a RISK relationship to determine the risk of each disease.
MATCH (x:dx)-[:HasSDx]-(i:id)-[:HasPDx]-(a:dx) MERGE (x)-[:Risk]-(a) ;
```

### Explore data
- Explore Data use cypher query [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Explore%20Data%20Node%20and%20Relationship.pdf)
#### Cypher Query for Explore Data
Run all the example queries:
- Node id
```
MATCH (n:id) RETURN n LIMIT 50
```
- Node dx
```
MATCH (n:dx) RETURN n LIMIT 50
```
- Relationship of primary diagnosis (PDx)
```
MATCH p=()-[r:HasPDx]->() RETURN p LIMIT 50
```
- Relationship of secondary diagnosis (SDx)
```
MATCH p=()-[r:HasSDx]->() RETURN p LIMIT 50
```
- Relationship of risk for each disease
```
MATCH p=()-[r:Risk]->() RETURN p LIMIT 50
```
### Explore the highest number of diseases
- Insight volumn claim [here](https://github.com/phuritanc/git-snaneo4j/blob/main/largest%20number%20of%20claim.pdf)
#### Cypher Query for highest number of diseases
Run all the example queries:
- Top5 Diseases with the highest number of Principal Diagnosis (PDx)
``` 
MATCH (n:id)-[:HasPDx]-(a:dx)
RETURN a.dx as PDx,a.term_d as PDx_Name,
sum(ToInteger(n.n_id)) as number_of_patient
ORDER BY number_of_patient DESC
LIMIT 5 
```
- Top5 Diseases with the highest number of Secondary Diagnosis (SDx)
```
MATCH (n:id)-[:HasSDx]-(a:dx)
RETURN a.dx as SDx,a.term_d as SDx_Name,
sum(ToInteger(n.n_id)) as number_of_patient
ORDER BY number_of_patient DESC
LIMIT 5 
```
- Graph data shows the network of Comorbidity
```
MATCH p=(x:dx)-[:Risk]-(a:dx) RETURN p LIMIT 25
```
- Graph data shows the network of Dyslipidemia(DLP)
```
MATCH p=(x:dx)-[:Risk]-(a:dx) WHERE a.dx = "E789" RETURN p LIMIT 10
```
- Graph data shows the network of Type 2 Diabetes mellitus (DM Type2)
```
MATCH p=(x:dx)-[:Risk]-(a:dx) WHERE a.dx = "E119" RETURN p LIMIT 25
```
- Graph data shows the network of Atherosclerotic heart disease
```
MATCH p=(x:dx)-[:Risk]-(a:dx) WHERE a.dx = "I251" RETURN p LIMIT 25
```
### Centrality Algorithms
- Algorithm Community Detection-Modularity Optimization [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Algorithm.pdf)
#### Cypher Query for Degree Centrality Algorithm
- Setting parameters
```
:param limit => ( 42); 
:param config => ({ nodeProjection: '*', relationshipProjection: { relType: { type: '*', orientation: 'NATURAL', properties: { n_id: { property: 'n_id', defaultValue: 1 }}}}, relationshipWeightProperty: 'n_id'}); :param communityNodeLimit => ( 10);
```
- Anonymous Graph
```
CALL gds.beta.modularityOptimization.stream($config) YIELD nodeId, communityId AS community
WITH gds.util.asNode(nodeId) AS node, community
WITH collect(node) AS allNodes, community
RETURN community, allNodes[0..$communityNodeLimit] AS nodes, size(allNodes) AS size
ORDER BY size DESC
LIMIT toInteger($limit);
```
#### Cypher Query for Betweenness Centrality Algorithm
- Setting parameters
```
:param limit => ( 42); 
:param config => ({ nodeProjection: '*', relationshipProjection: { relType: { type: '*', orientation: 'NATURAL', properties: { n_id: { property: 'n_id', defaultValue: 1 }}}}, relationshipWeightProperty: 'n_id'}); :param communityNodeLimit => ( 10);
```
- Anonymous Graph
```
CALL gds.beta.modularityOptimization.stream($config) YIELD nodeId, communityId AS community
WITH gds.util.asNode(nodeId) AS node, community
WITH collect(node) AS allNodes, community
RETURN community, allNodes[0..$communityNodeLimit] AS nodes, size(allNodes) AS size
ORDER BY size DESC
LIMIT toInteger($limit);
```

