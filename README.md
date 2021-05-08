tobias & christian

Vi benytter en docker-compose for at sætte op vores cluster. Replicas kan dog ikke tilgåes. Docker-compose filen er lavet af kenneth fra klassen. Den inkludrer gds (graph data science library) for at ilustere nogle grafer som vi kan køre cypher queries på. Vi prøvede i ret lang tid at få det til at køre manuelt fra terminalen på windows samt WSL2 men endte så med docker som der virkede mest stabilt. Terminal udgaverne kunne vi ikke få at loade GDS, neo4jdesktop kunne godt loade GDS men så kunne den ikke køre flere instancer og heller ikke i et cluster.
##Cypher queries for at genoprette eksempler:

## diijstrka eksempeler med veje
    CREATE (a:Location {name: 'A'}),
       (b:Location {name: 'B'}),
       (c:Location {name: 'C'}),
       (d:Location {name: 'D'}),
       (e:Location {name: 'E'}),
       (f:Location {name: 'F'}),
       (a)-[:ROAD {cost: 50}]->(b),
       (a)-[:ROAD {cost: 50}]->(c),
       (a)-[:ROAD {cost: 100}]->(d),
       (b)-[:ROAD {cost: 40}]->(d),
       (c)-[:ROAD {cost: 40}]->(d),
       (c)-[:ROAD {cost: 80}]->(e),
       (d)-[:ROAD {cost: 30}]->(e),
       (d)-[:ROAD {cost: 80}]->(f),
       (e)-[:ROAD {cost: 40}]->(f);

    
CALL gds.graph.create(
    'myGraph',
    'Location',
    'ROAD',
    {
        relationshipProperties: 'cost'
    }
)

MATCH (source:Location {name: 'A'}), (target:Location {name: 'F'})
CALL gds.beta.shortestPath.dijkstra.write.estimate('myGraph', {
    sourceNode: id(source),
    targetNode: id(target),
    relationshipWeightProperty: 'cost',
    writeRelationshipType: 'PATH'
})
YIELD nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory
RETURN nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory
