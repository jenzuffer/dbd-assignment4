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

##
    CALL gds.graph.create(
        'myGraph',
        'Location',
        'ROAD',
        {
            relationshipProperties: 'cost'
        }
    )

##
    MATCH (source:Location {name: 'A'}), (target:Location {name: 'F'})
    CALL gds.beta.shortestPath.dijkstra.write.estimate('myGraph', {
        sourceNode: id(source),
        targetNode: id(target),
        relationshipWeightProperty: 'cost',
        writeRelationshipType: 'PATH'
    })
    YIELD nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory
    RETURN nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory


##overlap similarity algorith eksempel
    CREATE
    (fahrenheit451:Book {title:'Fahrenheit 451'}),
    (dune:Book {title:'Dune'}),
    (hungerGames:Book {title:'The Hunger Games'}),
    (nineteen84:Book {title:'1984'}),
    (gatsby:Book {title:'The Great Gatsby'}),

    (scienceFiction:Genre {name: "Science Fiction"}),
    (fantasy:Genre {name: "Fantasy"}),
    (dystopia:Genre {name: "Dystopia"}),
    (classics:Genre {name: "Classics"}),

    (fahrenheit451)-[:HAS_GENRE]->(dystopia),
    (fahrenheit451)-[:HAS_GENRE]->(scienceFiction),
    (fahrenheit451)-[:HAS_GENRE]->(fantasy),
    (fahrenheit451)-[:HAS_GENRE]->(classics),

    (hungerGames)-[:HAS_GENRE]->(scienceFiction),
    (hungerGames)-[:HAS_GENRE]->(fantasy),

    (nineteen84)-[:HAS_GENRE]->(scienceFiction),
    (nineteen84)-[:HAS_GENRE]->(dystopia),
    (nineteen84)-[:HAS_GENRE]->(classics),

    (dune)-[:HAS_GENRE]->(scienceFiction),
    (dune)-[:HAS_GENRE]->(fantasy),
    (dune)-[:HAS_GENRE]->(classics),

     (gatsby)-[:HAS_GENRE]->(classics)


    MATCH (book:Book)-[:HAS_GENRE]->(genre)
    WITH {item:id(genre), categories: collect(id(book))} AS userData
    WITH collect(userData) AS data
    CALL gds.alpha.similarity.overlap.stream({
      data: data,
      similarityCutoff: 0.75
    })
    YIELD item1, item2, count1, count2, intersection, similarity
    RETURN gds.util.asNode(item1).name AS from, gds.util.asNode(item2).name AS to,
           count1, count2, intersection, similarity
    ORDER BY similarity DESC


## collapse path algorithm eksempel

    CREATE
  (Dan:Person),
  (Annie:Person),
  (Matt:Person),
  (Jeff:Person),

  (Guitar:Instrument),
  (Flute:Instrument),

  (Dan)-[:PLAYS]->(Guitar),
  (Annie)-[:PLAYS]->(Guitar),

  (Matt)-[:PLAYS]->(Flute),
  (Jeff)-[:PLAYS]->(Flute)


    (p1:Person)-[:PLAYS]->(:Instrument)-[:PLAYED_BY]->(p2:Person)



    CALL gds.graph.create(
      'persons',
      ['Person', 'Instrument'],
      {
        PLAYS: {
          type: 'PLAYS',
          orientation: 'NATURAL'
        },
        PLAYED_BY: {
          type: 'PLAYS',
          orientation: 'REVERSE'
        }
    }   ) 


    CALL gds.alpha.collapsePath.mutate(
      'persons',
      {
        relationshipTypes: ['PLAYS', 'PLAYED_BY'],
        allowSelfLoops: false,
        mutateRelationshipType: 'PLAYS_SAME_INSTRUMENT'
      }
    ) YIELD relationshipsWritten

    

##Harmonic Centrality algorithm eksempel


