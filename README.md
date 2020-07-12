# Neo4j Notes
Notes from the Introduction to Neo4j 4.0 course

## Nodes
* Parentheses represent nodes
    * () will access all node objects
    * (n) will set n as the varible to access node objects later in the query
* Not all nodes need to have the same properties
    * {} represent properties
  
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
()--()      (p:Person {name:})--(m:Movie)                               // 2 nodes have some type of relationship
()-->()     (p:Person)-[rel:ACTED_IN]->(m:Movie {title:'The Matrix'})   // the first node has a relationship to the second node
()<--()     (m:Movie {title:'The Matrix'})<-[rel:ACTED_IN]-(p:Person)   // the second node has a relationship to the first node

```

## Commands
* **MATCH**
    * Finds all nodes that match given pattern
    * Ex: ` MATCH (p:Person) `
        * Finds all the nodes with label *Person*
    * Ex: `MATCH (p:Person {born: 1970})`
        * Finds all the *Person* nodes with the property *born: 1970*
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
    * Ex: `MATCH (p:Person)-[r]->(m:Movie) RETURN p.name, type(r), m.title` 
        * type(r) returns the relationship *p* has with *m*

## Useful Queries
* Get visualization of how the database stores data
    * ` CALL db.schema.visualization() `
* List all node properties in the database
    * ` CALL db.propertykeys()`

## Other
* // = inline comment
