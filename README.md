<<<<<<< Updated upstream
# Comorbridity Networks Analysis in Deseases of the Circulatory System
### Setup and Install Neo4j Desktop
 - Download Neo4j Desktop [here](https://neo4j.com/download/)
 - About Neo4j [here](https://neo4j.com/product/#neo4j-desktop)
### Create New Project and Plugins
- Create new project and Local DBMS [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Create%20Project.pdf)
- Install and additional Plugin Graph Data Science Library  [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Plugins%20Graph%20Data%20Science.pdf)
### Dataset
- Dataset claim [here](https://github.com/phuritanc/git-snaneo4j/blob/main/DATAMTCLAIMALL.csv)
- Dataset car brand [here](https://github.com/phuritanc/git-snaneo4j/blob/main/CARBRAND.csv)
- Dateset car model [here](https://github.com/phuritanc/git-snaneo4j/blob/main/CARMODEL.csv)
### Import Data to Neo4j
- Import data to project [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Import%20Dataset%20to%20neo4j.pdf)
### Load Dataset into Local DBMS
- Load Dataset Sample Query [here](https://github.com/phuritanc/git-snaneo4j/blob/main/Load%20Data.pdf)
#### Cypher Query
Run all the example queries:
- Load Data claim to Local DBMS
```
\\Create Constraint
CREATE CONSTRAINT ON (c:claim) ASSERT c.ClaimNbr IS UNIQUE;
CREATE CONSTRAINT ON (a:agent) ASSERT a.AgentName IS UNIQUE;
CREATE CONSTRAINT ON (n:notifytype) ASSERT n.NotifyType IS UNIQUE;
CREATE CONSTRAINT ON (l:licenseplate) ASSERT l.LicensePlate IS UNIQUE;
CREATE CONSTRAINT ON (u:causeofloss) ASSERT u.CauseOfLoss IS UNIQUE;
CREATE CONSTRAINT ON (s:subpolicytype) ASSERT s.SubPolicyType IS UNIQUE;
CREATE CONSTRAINT ON (a:approval) ASSERT a.Approval IS UNIQUE;
```
```
\\Load csv dataset on import into local DB
LOAD CSV WITH HEADERS FROM 
'file:///DATAMTCLAIMALL.csv' AS line 
WITH line,SPLIT(line.AccidentDate,'-') as date

CREATE (claim:claim {ClaimNbr: line.ClaimNbr , NotifyType: line.NotifyType , CauseOfLoss: line.CauseOfLoss , Approval: line.Approval , LicensePlate: line.LicensePlate , SubPolicyType: line.SubPolicyType , CarBrandCode: line.CarBrandCode , CarModel: line.CarModel , AgentName: line.AgentName})
MERGE (agent:agent {AgentName: line.AgentName})
MERGE (notifytype:notifytype {NotifyType: line.NotifyType})
MERGE (licenseplate:licenseplate {LicensePlate: line.LicensePlate})
MERGE (causeofloss:causeofloss {CauseOfLoss: line.CauseOfLoss})
MERGE (subpolicytype:subpolicytype {SubPolicyType: line.SubPolicyType})
MERGE (approval:approval {Approval: line.Approval})

//Create Relationship
CREATE (claim)-[:agentowner]->(agent)
CREATE (claim)-[:vehicleclaim]->(licenseplate)
CREATE (claim)-[:approvedby]->(approval)
CREATE (claim)-[:type]->(notifytype)
CREATE (claim)-[:productgroup]->(subpolicytype)
CREATE (claim)-[:result]->(causeofloss)
;
```
- Load Data car brand to Local DBMS
```
\\Create Constraint
CREATE CONSTRAINT ON (b:carbrand) ASSERT b.CarBrand IS UNIQUE;
```
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


=======
# sna-project
sna project
pimmy
SNA !!! 
>>>>>>> Stashed changes
