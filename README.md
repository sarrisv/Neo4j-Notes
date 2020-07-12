# Neo4j Notes
Notes from the Introduction to Neo4j 4.0 course

## Syntax
* () -> nodes
    * (n) will set n as the varible to access node objects later in the query
* {} -> properties
    * Not all nodes need to have the same properties
* [] -> relationships between nodes
  
### Node Representation (Syntax - Example)
```
()                          ()
(variable)                  (p)
(:Label)                    (:Person)
(variable:Label)            (p:Person)
(:Label1:Label2)            (:Actor:Director)
(variable:Label1:Label2)    (p:Actor:Director)
```

### Property Representation
```
(variable {propertyKey: propertyValue})                                         (p {born:1970})
(variable:Label {propertyKey: propertyValue})                                   (p:Person {born:1970})
(variable {propertyKey1: propertyValue1, propertyKey2: propertyValue2})         (m {released: 1970, rating: 70}) 
(variable:Label {propertyKey: propertyValue, propertyKey2: propertyValue2})     (m:Movie {released: 1970, rating: 70})
```

### ASCII ART - Node Relationships
```
() = nodes, [] = relationships

()--()              (p:Person {name:})--(m:Movie)                                                       // 2 nodes have some type of relationship
()-->()             (p:Person)-[rel:ACTED_IN]->(m:Movie {title:'The Matrix'})                           // the first node has a relationship to the second node
()<--()             (m:Movie {title:'The Matrix'})<-[rel:ACTED_IN]-(p:Person)                           // the second node has a relationship to the first node
()-[]->()-[]->()    (p:Person)-[:FOLLOWS]->(:Person)-[:FOLLOWS]->(:Person {name:'Jessica Thompson'})    // traversing multiple relationships
```

## Commands
* **MATCH**
    * Finds all nodes that match given pattern
    * Ex: ` MATCH (p:Person) `
        * Finds all the nodes with label *Person*
    * Ex: ` MATCH (p:Person {born: 1970})`
        * Finds all the *Person* nodes with the property *born: 1970*
    * Ex: ` MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)`
        * Finds the Actors *a* and Directors *d* for each Movie *m*
* **RETURN** 
    * Returns whatever its given
    * Ex: ` MATCH (m:Movie) RETURN m `
        * Returns all the nodes descibed by *m*
    * Ex: ` MATCH (m:Movie {released: 1970}) RETURN m.title, m.released `
        * Returns the movies released in 1970 as a table with 2 columns named after the property
    * Ex: ` MATCH (m:Movie {released: 1970}) RETURN m.title AS Title `
        * Returns the movies released in 1970 as a table with 1 column named "Title", for longer strings use backticks (\`Title\`) not apostrophes ('Title') or quotation mark ("Title")
    * Ex: ` MATCH (p:Person {name: 'Tom Hanks'})-[r:ACTED_IN|DIRECTED]->(m:Movie) RETURN m.title, type(r) , m.released `
        * Returns all the Tom Hanks acted or directed as a table of [table, Hanks' relationship (acted/directed), year released]
        
## Functions
* **type()**
    * Returns the type of relationship given
    * Ex: ` MATCH (p:Person)-[r]->(m:Movie) RETURN p.name, type(r), m.title` 
        * type(r) returns the relationship *p* has with *m*

## Useful Queries
* Get visualization of how the database stores data
    * ` CALL db.schema.visualization() `
* List all node properties in the database
    * ` CALL db.propertykeys()`

## Other
* Can assign variables to path or multiple paths
    * Single Path Ex:   ` path = (:Person)-[:FOLLOWS]->(:Person)-[:FOLLOWS]->(:Person {name:'Jessica Thompson'})`
    * Multiple Path Ex: ` path = (:Person)-[:ACTED_IN]->(:Movie)<-[:DIRECTED]-(:Person {name:'Ron Howard'})`
* inline comment -> //

## Cypher Conventions
* Node labels are CamelCase 
    * Ex: *Person*, *NetworkAddress*
* Property keys, variables, parameters, aliases, and functions are camelCase 
    * Ex: *businessAddress*, *title*
* Relationship types are in upper-case and can use the underscore
    * Ex: *ACTED_IN*, *FOLLOWS* 
    * Note you cannot use the “-” character in a relationship type
* Cypher keywords are upper-case 
    * Ex: `MATCH`, `RETURN`
* String constants are in single quotes, unless the string contains a quote or apostrophe 
    * Ex: ‘The Matrix’, “Something’s Gotta Give”
* Specify variables only when needed for use later in the Cypher statement
* Place named nodes and relationships (that use variables) before anonymous nodes and relationships in your `MATCH` clauses when possible
* Specify anonymous relationships with `-->`, `--`, or `<--`

