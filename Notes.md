## Cypher Syntax
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
()--()                      (p:Person {name:})--(m:Movie)                                                       // 2 nodes have some type of relationship
()-->()                     (p:Person)-[rel:ACTED_IN]->(m:Movie {title:'The Matrix'})                           // the first node has a relationship to the second node
()<--()                     (m:Movie {title:'The Matrix'})<-[rel:ACTED_IN]-(p:Person)                           // the second node has a relationship to the first node
()-[]->()-[]->()            (p:Person)-[:FOLLOWS]->(:Person)-[:FOLLOWS]->(:Person {name:'Jessica Thompson'})    // traversing multiple relationships
()-[:rel*n]-()              (follower:Person)-[:FOLLOWS*2]->(p:Person)                                          // traversing multiple times
()-[:rel*lower..upper]-()   (follower:Person)-[:FOLLOWS*1..3]->(p:Person)                                       // traversing paths of length lower-upper (ex 1-3)
```

## Graph Creation/Modification
* Commands
   * **CREATE**
      * Adds a new object to the graph
      * Node Ex: ` CREATE (:Person {name:'Tom Hanks'}) ` 
         * Creates node with *Person* label and *name* property
      * Relationship Ex: ` MATCH (p:Person {name:'Tom Hanks'}), (m:Movie {name:'The Matrix'})  CREATE (p)-[:LOVES]->(m)`
         * Person *p* named 'Tom Hanks' now has a *LOVES* relationship with Movie *m* 'The Matrix'
            * Relationships created using `CREATE` *must* have a direction
      * Relationships with Properties Ex: 
         ```
         MATCH (a:Person), (m:Movie)
         WHERE a.name = 'Katie Holmes' AND m.title = 'Batman Begins'
         CREATE (a)-[rel:ACTED_IN {roles: ['Rachel','Rachel Dawes']}->(m)
         ```
      * Node + Relationship Ex: 
         ```
         MATCH (m:Movie {name:'Batman Begins'}) 
         CREATE (p:Person {name:'Gary Oldman', born:1958})-[:ACTED_IN {role:['James Gorden','Commish']}]->(m)` 
         ```
         * Created a Person *p* named 'Gary Oldman', born in 1958 whose roles as an actor were 'James Gorden' and 'Commish' in Movie *m*
   * **SET**
      * Changes properties of an object
      * Add Label Ex: ` MATCH (p:Person {name:'Tom Hanks'}) SET p:Actor:Director `
         * Finds Person *p* named 'Tom Hanks' then gives it the labels *Actor* and *Director*
            * *p* now has three labels
      * New Property 1 Ex: ` MATCH (p:Person {name:'Tom Hanks'}) SET p.address = '123 Road', p.zipCode = 123456 `
         * Add the properties *address* and *zipCode* to Person *p* named 'Tom Hanks'
      * Remove Properties Ex: ` MATCH (p:Person {name:'Tom Hanks'}) SET p.born = null `
         * Person *p* no longer has the *born* property
      * Overwrite Properties Ex: ` MATCH (p:Person {name:'Tom Hanks'}) SET p = {name:'Matt Bomer', born:1977} `
         * Person *p* that had the name 'Tom Hanks' now is named 'Matt Bomer' and only has two properties *name* and *born*
      * New Properties 2 Ex: ` MATCH (p:Person {name:'Tom Hanks'}) SET p += {height:'1.8m'} `
         * Person *p* named 'Tom Hanks' now has the additional property *height*
   * **REMOVE**
      * Removes obejct or property from the graph
      * Node Ex:  ` MATCH (p:Person {name:'Tom Hanks'}) REMOVE p `
         * Removes Person *p* named 'Tom Hanks' from the graph
      * Property Ex: ` MATCH (p:Person {name:'Tom Hanks'})-[r:ACTED_IN]-(:Movie) REMOVE p.born, r.roles `
         * Removes Person *p*'s born property and the all of *p*'s *roles* in movies which they acted in

## Query Commands
* **MATCH**
    * Finds all nodes that match given pattern
    * Node Type Ex: ` MATCH (p:Person) `
        * Finds all the nodes with label *Person*
    * Property Ex: ` MATCH (p:Person {born: 1970}) `
        * Finds all the *Person* nodes with the property *born: 1970*
    * Relationship Ex: ` Match (p:Person)-[:ACTED_IN|:DIRECTED]->(m:Movie) `
        * Finds all the People *p* that either acted in or directed a Movie *m*
    * Multiple Relationship Ex: ` MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person) `
        * Finds the Actors *a* and Directors *d* for each Movie *m*
* **OPTIONAL MATCH**
   * `MATCH` but if it fails it returns `null`
   * Ex: 
        ```
        MATCH (p:Person)
        WHERE p.name STARTS WITH 'James'
        OPTIONAL MATCH (p)-[r:REVIEWED]->(m:Movie)
        ```
        * Finds all the Person *p* named 'James' then checks what Movie *m* they have reviewed *r*, if they have not reviewed any movies *r* will be `null`
* **WHERE**
    *  Modification of `MATCH` for testing equality, multiple values, ranges, labels, existence of a property, string values, regular expressions, patterns in the graph, inclusion in a list
        * `WHERE` can do everthing `MATCH` can plus more
    * Equality Ex: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie) WHERE m.released = 2008 ` 
        * Finds all of the of Person *p* who acted in Movies *m* where *m* was released in 2008
        * MATCH equivalent: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie {released: 2008}) `
    * Multiple Values Ex: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie) WHERE m.released = 2008 OR m.released = 2009 `
        * Finds all the Person *p* that acted in a Movie *m* released in either 2008 or 2009
    * Ranges Ex: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie) WHERE m.released >= 2003 AND m.released <= 2004 `
        * Finds all the Person *p* that acted in a Movie *m* released between 2003 and 2004
        * Alternatively: `WHERE 2003 <= m.released <= 2004 `
    * Label Ex: ` MATCH (p)-[:ACTED_IN]->(m) WHERE p:Person AND m:Movie AND m.title='The Matrix' `
        * Finds all the nodes *p* that acted in any node *m* then tests if *p* has the *Person* label, *m* has the *Movie* label, and that if it does is *m* titled 'The Matrix'
        * MATCH equivalent: ` MATCH (p:Person)-[:ACTED_IN]->(:Movie {title: 'The Matrix'}) `
        * Performs better than `MATCH` if nodes have multiple labels
    * Existence Ex: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie) WHERE p.name='Jack Nicholson' AND exists(m.tagline) `
        * Finds all the Person *p* that acted in a Movie *m* then tests if the *p* is named 'Jack Nicholson' and *m* has a tagline
    * Strings Ex: ` MATCH (p:Person)-[:ACTED_IN]->() WHERE p.name STARTS WITH 'Michael' `
        * Finds all the Person *p* that have acted in anything then checks if the String `p.name` starts with 'Michael'
        * Commands to test String properties: `STARTS WITH`, `ENDS WITH`, and `CONTAINS`
        * Case-Sensitive
    * Regex Ex: ` MATCH (p:Person) WHERE p.name =~'Tom.*' `
        * Finds all the Person *p* then checks if their name starts with 'Tom'
        * `=~` indicates Regex
    * Pattern Ex: 
        ```
        MATCH (gene:Person)-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(other:Person) 
        WHERE gene.name= 'Gene Hackman' 
        AND exists( (other)-[:DIRECTED]->(m))
        ```
        * Find all the Person *gene* which acted in any Movie *m* which another Person *other* also acted in then checks if *gene* is named 'Gene Hackman' and the *other* also directed the *m* they both acted in. 
            * Basically *m* is any movie where Gene Hackman acted with another person that also directed the movie
            
    * List Ex: ` MATCH (p:Person) WHERE p.born IN [1965, 1970] `
        * Finds all the Person *p* which were born in any year given in the list
    * List 2 Ex: ` MATCH (p:Person)-[r:ACTED_IN]->(m:Movie) WHERE 'Neo' IN r.roles AND m.title='The Matrix' ` 
        * Checks if 'Neo' is in the `r`'s list property `roles` and if 
    * Relationship Traversal Ex: 
        ```
        MATCH (follower:Person)-[:FOLLOWS]->(reviewer:Person)-[:REVIEWED]->(m:Movie)
        WHERE m.title = 'The Replacements'
        ```
        * Finds all the Person *follower* of Person *reviewer* which reviewed the Movie *m* titled 'The Replacements' by traversing the relationships
* **RETURN** 
    * Returns whatever its given
    * Basic Ex: ` MATCH (m:Movie) RETURN m `
        * Returns all the nodes descibed by *m*
    * Property Ex: ` MATCH (m:Movie {released: 1970}) RETURN m.title, m.released `
        * Returns the movies released in 1970 as a table with 2 columns named after the property
    * Alias Ex: ` MATCH (m:Movie {released: 1970}) RETURN m.title AS Title `
        * Returns the movies released in 1970 as a table with 1 column named "Title", for longer strings use backticks (\`Title\`) not apostrophes ('Title') or quotation mark ("Title")
    * Relationship Ex: ` MATCH (p:Person {name: 'Tom Hanks'})-[r]->(m:Movie) RETURN m.title, type(r) , m.released `
        * Returns all the movies Tom Hanks was involved in as a table of [table, Hanks' relationship to movie, year released]
* **DISTINCT**
   * Eliminates duplicates in the data
   * `RETURN` Ex: ` MATCH (p:Person {name:'Tom Hanks'})-[:DIRECTED | ACTED_IN]->(m:Movie) RETURN DISTINCT m.title, m.released `
      * There are Movies *m* that Person *p* named 'Tom Hanks' has acted in and direceted, `DISTINCT` removes the duplicates before we `RETURN`
   * List Ex: `MATCH (p:Person)-[:ACTED_IN | DIRECTED | WROTE]->(m:Movie {released:2003}) RETURN m.title, collect(DISTINCT p.name) `
      * Collects all the Person *p* which acted in, directed, or wrote for a Movie *m* then returns the title and a list of the *p* involved without duplicates
   * `WITH` Ex: `MATCH (p:Person {name:'Tom Hanks'})-[:DIRECTED | ACTED_IN]->(m:Movie) WITH DISTINCT m RETURN m.title `
      * `WITH DISTINCT` continues the query with *m* without the duplicates for Movies *m* he both acted and directed
* **ORDER BY** 
   * Sorts data (default: ascending)
   * Ex: ` MATCH (:Person {name:'Tom Hanks'})-[:ACTED_IN]->(m:Movie) RETURN m.title ORDER BY m.released `
      * Returns Movies *m* in ascending order via the year they were released
   * `DESC` Ex: ` MATCH (:Person {name:'Tom Hanks'})-[:ACTED_IN]->(m:Movie) RETURN m.title ORDER BY m.released DESC `
      * Returns Movies *m* in decending order via the year they were released
   * Ex: ` MATCH (:Person {name:'Tom Hanks'})-[:ACTED_IN]->(m:Movie) RETURN m.title ORDER BY m.released DESC, m.title `
      * Returns Movies *m* in decending order via the year they were released but movies with the same release year are sorted in ascending order based on title
* **LIMIT**
   * Limits the # of table rows an expression outputs
   * `RETURN` Ex: ` MATCH (m:Movie) RETURN m.title, m.released ORDER BY m.released LIMIT 10 `
      * Reduces output of the `RETURN` to just the first 10 results
   * `WITH` Ex: `MATCH (p:Person)-[:ACTED_IN]->(m:Movie) WITH m, p LIMIT 6 RETURN collect(p.name), m.title `
      * Limits the number of Person *p* in the rest of the query to the first 6 in the table at that time
* **IN**
    * Returns boolean of whether or not an element is in a list
    * Ex: ` MATCH (p:Person) WHERE p.born IN [1970,1971,1972] RETURN p `
        * Returns all Person *p* that were born in the list of years provided
* **WITH**
    * Used to Process the results of a match before continuing a query
    * Ex: 
        ```
        MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
        WITH  a, count(a) AS numMovies, collect(m.title) as movies
        WHERE 1 < numMovies < 4
        RETURN a.name, numMovies, movies
        ```
        * `WITH` processes the `MATCH` then continues the query with Person *a*, the number of movies the *a* has acted in as *numMovies*, and a list of those movies as *movies*
            * The *numMovies* and *movies* would not be possible to create in the `MATCH` statement
* **UNWIND**
    * Contructs rows from a list
    * Ex: 
        ```
        MATCH (m:Movie)<-[:ACTED_IN]-(p:Person)
        WITH collect(p) AS actors, count(p) AS actorCount, m
        UNWIND actors AS actor
        RETURN m.title, actorCount, actor.name
        ```
        * `UNWIND` creates a table with each row being a node in the list *actors*
* **CALL** 
    * Performs a subquery which typically returns a set up nodes
    * Ex: 
        ```
        CALL { MATCH (p:Person)-[:REVIEWED]->(m:Movie) RETURN  m }
        MATCH (m) WHERE m.released=2000
        RETURN m.title, m.released
        ```
        *   `CALL` performs the `MATCH (p:Person)-[:REVIEWED]->(m:Movie) RETURN  m` subquery, making *m* useable, before continuing onto the rest of the query

## Functions
* Node
   * **labels()**
      * Returns the labels of a given node
      * Ex: ` MATCH (p:Person {name:'Tom Hanks'}) RETURN labels(p) `
   * **properties()**
      * Returns the properties of a given node
      * Ex: ` MATCH (p:Person {name:'Tom Hanks'}) RETURN properties(p) ` 
* Relationship
    * **type()**
        * Returns the type of relationship given
        * Ex: ` MATCH (p:Person)-[r]->(m:Movie) RETURN p.name, type(r), m.title ` 
            * type(r) returns the relationship *p* has with *m*
    * **shortestPath()** 
        * Returns the path with the least relationship traversals that satisfies the condition
        * Ex: ` MATCH path = shortestPath((m1:Movie {title:'A Few Good Men'})-[*]-(m2:Movie {title:'The Matrix'})) `
            * path is the shortest relationship traversal between 'A Few Good Men' and 'The Matrix'
* String
    * **toLower()** & **toUpper()** 
        * Returns all lower-case / upper-case version of given string
        * Ex: ` MATCH (p:Person)-[:WROTE]->(:Movie) RETURN toLower(p.name) `
            * Returns the lower-case version of all the Person *p* which wrote a Movie *m*
* Aggregating  
    * **collect()**
        * Returns an aggregated list object of a given object
        * Ex: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(p) RETURN size(collect(m)) `
            * Returns a list of the Movie *m* which a Person *p* both acted and directed
    * **size()**
        * Returns the # of elements in a given list
        * Ex: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(p) RETURN size(collect(m)) `
            * Returns the # of elements in the list of provided by the collect function 
    * **count()** 
        * Returns # of given object type
        * Ex: ` MATCH (p:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(p)  RETURN count(m) `
            * Returns the # of the Movie *m* which a Person *p* both acted and directed
* Conversion
    * **toInteger()**
    * **toString()**
* Time
    * **date()**
        * Returns a new Date object
    * **time()**
        * Returns a new Time object
    * **datetime()**
        * Returns a new DateTime object
    * **timestamp()**
        * Returns a long integer of the milliseconds since midnight, January 1, 1970 UTC

## Useful Queries
* Get visualization of how the database stores data
    * ` CALL db.schema.visualization() `
* List all node properties in the database
    * ` CALL db.propertykeys() `

## Types
* Property
    * Number
        * **Integer**
        * **Float**
    * **String**
    * **Boolean**
    * Spatial
        * **Point**
    * Temporal
        * **Date**
        * **Time**
        * **LocalTime**
        * **DateTime**
        * **LocalDateTime**
        * **Duration**
* Structural
    * **Node**
        * ID
        * Labels
        * Properties
    * **Relationship**
        * ID
        * Type
        * Properties
        * ID of start and end nodes
* Composite (Data Structures)
    * **List**
        * Can access list elements via index
            * list[*index*]
    * **Map**
        * Key -> Value
        * Can access map elements via key
            * map[*key*]
        * Projections are modified versions of existing nodes 
            * Useful for creating additional or removing properties
            * ` MATCH (m:Movie {title:'The Matrix'}) RETURN m {.title, .released} `
                * Creates map of *m* only containing the title and released properties

## Other
* Can assign variables to path or multiple paths
    * Single Path Ex:   ` path = (:Person)-[:FOLLOWS]->(:Person)-[:FOLLOWS]->(:Person {name:'Jessica Thompson'}) `
    * Multiple Path Ex: ` path = (:Person)-[:ACTED_IN]->(:Movie)<-[:DIRECTED]-(:Person {name:'Ron Howard'}) `
* inline comment -> //
* Uses Python range syntax -> start..end
* Conditionals (`WHERE`, etc.) can use the boolean operators `AND`, `OR`, `XOR`, and `NOT`
* Can use commas to seperate multiple of a command 
    * Ex: ` MATCH (a:Person)-[:ACTED_IN]->(m:Movie), (m)<-[:DIRECTED]-(d:Person) `
* Cypher automatically groups return data based on the node type with the least # of nodes

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
