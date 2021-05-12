# Comorbidity Networks Analysis in Diseases of the Circulatory System
### Setup and Install Neo4j Desktop
 - Download Neo4j Desktop [here](https://neo4j.com/download/)
 - About Neo4j [here](https://neo4j.com/product/#neo4j-desktop)
### Create New Project and Plugins
- Create new project and Local DBMS [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Create%20Project.pdf)
- Install and additional Plugin Graph Data Science Library  [Click here](https://github.com/jutamask/sna-project/blob/main/Install%20-%20Graph%20Data%20Science%20Library.pdf)
### Dataset
- Dataset Diagnosis-Related  [Click here](https://github.com/jutamask/sna-project/blob/main/data_sna.csv)
- Dataset Description-ICD10 [Click here](https://github.com/jutamask/sna-project/blob/main/descript_icd_dx.csv)

### Import Data to Neo4j
- Import data to project [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Import%20Dataset%20to%20neo4j.pdf)
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
//To determine which Node id has the primary diagnosis (PDx), establish a HasPDx relationship.
MATCH (i:id) UNWIND i.pdx_id as pdx MATCH (d:dx {dx:pdx}) MERGE (i)-[:HasPDx]-(d) ;
```
```
//To determine which Node id has the secondary diagnosis (SDx), establish a HasSDx relationship.
MATCH (i:id) UNWIND i.sdx_id as sdx MATCH (d:dx {dx:sdx}) MERGE (i)-[:HasSDx]-(d) ;
```
```
//Create a RISK relationship to determine the risk of each disease.
MATCH (x:dx)-[:HasSDx]-(i:id)-[:HasPDx]-(a:dx) MERGE (x)-[:Risk]-(a) ;
```

- ถึงตรงนี้

```
\\Load csv dataset on import into local DB
LOAD CSV WITH HEADERS FROM 
'file:///CARBRAND.csv' AS line
MATCH(claim:claim {ClaimNbr: line.ClaimNbr})
MERGE(carbrand:carbrand {CarBrandCode: line.CarBrandCode})

\\Create Relationship
CREATE (claim)-[:claimbrand]->(carbrand)
;
```
- Load Data car model to Local DBMS
```
\\Create Constraint
CREATE CONSTRAINT ON (m:carmodel) ASSERT m.CarModel IS UNIQUE;
```
```
\\Load csv dataset on import into local DB
LOAD CSV WITH HEADERS FROM 
'file:///CARMODEL.csv' AS line
MATCH(claim:claim {ClaimNbr: line.ClaimNbr})
MATCH(claim)-[:claimbrand]->(carbrand)
CREATE(carmodel:carmodel {CarModelCode: line.CarModel ,CarBrandCode: line.CarBrandCode })

\\Create Relationship
CREATE (claim)-[:claimmodel]->(carmodel)
MERGE (carmodel)-[:brand]->(carbrand)
;
```
### Explore data
- Explore Data use cypher query [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Explore%20Data%20Node%20and%20Relationship.pdf)
#### Cypher Query for Explore Data
Run all the example queries:
- Node Claim
```
MATCH (n:claim) RETURN n LIMIT 25
```
- Node Car Model
```
MATCH (n:carmodel) RETURN n LIMIT 25
```
- Node cause of loss
```
MATCH (n:causeofloss) RETURN n LIMIT 25
```
- Node Agent
```
MATCH (n:agent) RETURN n LIMIT 25
```
- Relationship of agent owner
```
MATCH p=()-[r:agentowner]->() RETURN p LIMIT 25
```
- Relationship of claim model
```
MATCH p=()-[r:claimmodel]->() RETURN p LIMIT 300
```
- Relationship of product group
```
MATCH p=()-[r:productgroup]->() RETURN p LIMIT 25
```
### Explore the highest number of claims made
- Insight volumn claim [here](https://github.com/phuritanc/git-snaneo4j/blob/main/largest%20number%20of%20claim.pdf)
#### Cypher Query for highest number of claims
Run all the example queries:
- Top10 Car Model with the highest number of claims
``` 
MATCH (c:claim)-[:claimmodel]->(m:carmodel)
RETURN m.CarModelCode as model , COUNT(c.ClaimNbr) as no_of_claim
ORDER BY no_of_claim DESC
LIMIT 10
```
- Top10 Agent with the highest number of claims
```
MATCH (a:agent)-[:agentowner]-(c:claim)
RETURN a.AgentName as agentname , COUNT(c.ClaimNbr) as no_of_claim
ORDER BY no_of_claim DESC
LIMIT 10
```
- Top10 License Plate with the highest number of claims
```
MATCH (l:licenseplate)-[:vehicleclaim]-(c:claim)
RETURN l.LicensePlate as licenseplate , COUNT(c.ClaimNbr) as no_of_claim
ORDER BY no_of_claim DESC
LIMIT 10
```
- Top10 Cause of loss with the highest number of claims
```
MATCH (r:causeofloss)-[:result]-(c:claim)
RETURN r.CauseOfLoss as result , COUNT(c.ClaimNbr) as no_of_claim
ORDER BY no_of_claim DESC
LIMIT 10
```
### Algorithm Community Detection-Modularity Optimization
- Algorithm Community Detection-Modularity Optimization [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Algorithm.pdf)
#### Cypher Query
- Setting parameters
```
:param limit=>( 100);
:param config=>({
 nodeProjection: '*',
 relationshipProjection: {
 relType: {
 type: 'brand',
 orientation: 'UNDIRECTED',
 properties: {}
 }
 },
 relationshipWeightProperty: null,
 seedProperty: '',
 maxIterations: 10,
 tolerance: 0.0001
});
:param communityNodeLimit=>( 100);
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

