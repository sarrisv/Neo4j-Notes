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
    * **MERGE**
        * `MATCH` but if there is not match it creates the object
        * Normal Ex: ` MERGE (p:Person {name:'Tom Hanks'}) `
            * Creates Person node with name 'Tom Hanks' if none is found.
        * `ON CREATE` Ex: ` MERGE (p:Person {name:'Tom Hanks'}) ON CREATE SET p.born = 'Concord' `
            * If Person *p* named 'Tom Hanks' does not already exist create on and give it the property *born* = 'Concord'
        * `ON MATCH` Ex: ` MERGE (p:Person {name:'Tom Hanks'}) ON MATCH SET p.born = 'Concord' `
            * If Person *p* named 'Tom Hanks' exists give it the property *born* = 'Concord'
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
        * Removes property from an object
        * Ex: ` MATCH (p:Person {name:'Tom Hanks'})-[r:ACTED_IN]-(:Movie) REMOVE p.born, r.roles `
            * Removes Person *p*'s born property and the all of *p*'s *roles* in movies which they acted in
    * **DELETE**
        * Removes an object from the graph
        * Node Ex: ` MATCH (p:Person {name:'Tom Hanks'}) DELETE p `
            * Removes Person *p* named 'Tom Hanks' from the graph
                * To delete a node it must have no relationships
        * Relationship Ex: ` MATCH (p:Person {name:'Tom Hanks'})-[r]->(:Movie) DELETE r`
            * Removes all the relationships Person *p* has with any Movie node
    * **DETATCH DELETE**
        * Removes a node and all of its relationships from the graph
        * Ex: ` MATCH (p:Person {name:'Tom Hanks'}) DETATCH DELETE p `
            * Removes all the relationships of Person *p* then deletes the node
## Importing Data (sorted by speed, slow->fast)
* **LOAD CSV**
    * CSV
    * Special handling needed for >1000k rows
        * use ` :auto USING PERIODIC COMMIT LOAD CSV ` to create import data batch sizes
        * Can't use 
            * `collect()`
            * `count()`
            * `ORDER BY`
            * `DISTINCT`
    * Server Online
    * General Step: 
        1) Determine how the CSV file will be structured.
        2) Determine if normalized or denormalized data.
        3) Ensure IDs to be used in the data are unique.
        4) Ensure data in CSV files is “clean”.
        5) Execute Cypher code to inspect the data.
        6) Determine if data needs to be transformed.
        7) Ensure constraints are created in the graph.
        8) Determine the size of the data to be loaded.
        9) Execute Cypher code to load the data.
        10) Add indexes to the graph.
    * Import CSV Ex: 
        * Create Constraints: 
            ```
            CREATE CONSTRAINT UniqueMovieIdConstraint ON (m:Movie) ASSERT m.id IS UNIQUE; 
            CREATE CONSTRAINT UniquePersonIdConstraint ON (p:Person) ASSERT p.id IS UNIQUE
            ```
        * Import Nodes: 
            ```
            :auto USING PERIODIC COMMIT 500 LOAD CSV WITH HEADERS FROM 'https://data.neo4j.com/v4.0-intro-neo4j/movies1.csv' as row
            MERGE (m:Movie {id:toInteger(row.movieId)})
            ON CREATE SET
              m.title = row.title,
              m.avgVote = toFloat(row.avgVote),
              m.releaseYear = toInteger(row.releaseYear),
              m.genres = split(row.genres,":")
            ```
         * Import Relationships:
            ```
            LOAD CSV WITH HEADERS FROM 'https://data.neo4j.com/v4.0-intro-neo4j/directors.csv' AS row
            MATCH (movie:Movie {id:toInteger(row.movieId)})
            MATCH (person:Person {id: toInteger(row.personId)})
            MERGE (person)-[:DIRECTED]->(movie)
            ON CREATE SET person:Director
            ```
         * Create Index Ex: 
            ```
            CREATE INDEX MovieTitleIndex ON (m:Movie) FOR (m.title)
            CREATE INDEX PersonNameIndex ON (p:Person) FOR (pname)
            ```
* **APOC**
    * CSV, XML, or JSON
    * No size limit
    * Server Online
    * Functions
        * `apoc.periodic.iterate()`
            * Sets batch sizes for data to be imported
        * `apoc.load.csv()`
            * Loads the RDBMS CSV/s
        * `apoc.do.when()`
            * Conditional programing 
        * `apoc.load.json()`
            * Loads JSON instead of CSV
    * CSV Ex:
        ```
        CALL apoc.periodic.iterate(
            "CALL apoc.load.csv('https://data.neo4j.com/v4.0-intro-neo4j/movies2.csv' )
            YIELD map AS row RETURN row",
            "WITH row.movieId as movieId, row.title AS title, row.genres AS genres,
                toInteger(row.releaseYear) AS releaseYear, toFloat(row.avgVote) AS avgVote,
                collect({id: row.personId, name:row.name, born: toInteger(row.birthYear),
                died: toInteger(row.deathYear),personType: row.personType,
                roles: split(coalesce(row.characters,''),':')}) AS people
            MERGE (m:Movie {id:movieId})
                ON CREATE SET m.title=title, m.avgVote=avgVote, m.releaseYear=releaseYear, m.genres=split(genres,':')
            WITH *
            UNWIND people AS person
            MERGE (p:Person {id: person.id})
                ON CREATE SET p.name = person.name, p.born = person.born, p.died = person.died
            WITH  m, person, p
            CALL apoc.do.when(person.personType = 'ACTOR',
                'MERGE (p)-[:ACTED_IN {roles: person.roles}]->(m)
                    ON CREATE SET p:Actor',
                'MERGE (p)-[:DIRECTED]->(m)
                    ON CREATE SET p:Director', {m:m, p:p, person:person}) 
            YIELD value AS value
            RETURN count(*)  ",
        {batchSize: 500})
        ```
    * JSON Ex: 
        ```
        WITH "https://api.stackexchange.com/2.2/search?page=1&pagesize=5&order=asc&sort=creation&tagged=neo4j&site=stackoverflow&filter=!5-i6Zw8Y)4W7vpy91PMYsKM-k9yzEsSC1_Uxlf" AS url
        CALL apoc.load.json(url)
        YIELD value AS data
        UNWIND data.items as q
        MERGE (question:Question {id: q.question_id})
          ON CREATE SET  question.title = q.title,
                         question.tags = q.tags,
                         question.is_answered = q.is_answered
        MERGE (user:User {name: q.owner.display_name})
        MERGE (user)-[:ANSWERED]->(question)
        ```
* **ETF Tool**
    * Live RDBMS
    * No size limit
    * Server Online
* **neo4j-admin import tools**
    * CSV (special format)
    * No size limit
    * Server Offline
* Other Notes
    * Normalized vs Denormalized
        * Normalizd: CSVs for each relational table
        * Denormalized: One massive CSV

## Constraints
* Types 
    * Uniqueness
        * Constraint that ensures that a value for a property is unique for all nodes of that type
    * Existence
        * Constraint that ensures that when a node or relationship is created or modified, it must have certain properties set.
    * Node Key
        *  Uniqueness and Existence constraint for multiple properties of a node of a certain type
* Create Constraints
    * If all existing nodes achieve the constraint it is added to the graph
    * Uniqueness Ex: ` CREATE CONSTRAINT UniqueMovieTitleConstraint ON (m:Movie) ASSERT m.title IS UNIQUE `
    * Existence Ex: ` CREATE CONSTRAINT ExistsMovieTagline ON (m:Movie) ASSERT exists(m.tagline) `
    * Node Key Ex: ` CREATE CONSTRAINT UniqueNameBornConstraint ON (p:Person) ASSERT (p.name, p.born) IS NODE KEY `
* Drop Constraints
    * Removes constraint from the graph
    * Ex: ` DROP CONSTRAINT ExistsMovieTagline `

## Indexs
* Indexes are used to improve initial node lookup performance
* Single-property
    * Ex: ` CREATE INDEX MovieReleased FOR (m:Movie) ON (m.released) `
* Composite-property
    * Es: ` CREATE INDEX MovieReleasedVideoFormat FOR (m:Movie) ON (m.released, m.videoFormat)  `
* Full-text Schema
    * Node Ex: ` CALL db.index.fulltext.createNodeIndex('MovieTitlePersonName',['Movie', 'Person'], ['title', 'name']) `
    * Relationship Ex: ` CALL db.indexfulltext.createRelationshipIndex() `
* Use Indexes
    * Basic Ex: ` CALL db.index.fulltext.queryNodes('MovieTitlePersonName', 'Jerry') YIELD node RETURN node`
        * *node* returns every movie with either a title or person containing 'Jerry'
    * Score Ex: ` CALL db.index.fulltext.queryNodes('MovieTitlePersonName', 'Matrix') YIELD node, score RETURN node.title, score `
        * Returns all the nodes with movie titles or people with names that contain 'Matrix' but also returns a score of "closeness"
    * Property Ex: ` CALL db.index.fulltext.queryNodes('MovieTitlePersonName', 'name: Jerry') YIELD node RETURN node `
        * Returns nodes where the propety *name* contains 'Jerry'
* Drop Indexes
    * Property Index Ex: ` DROP INDEX MovieReleasedVideoFormat `
        * Drops the property index *MovieReleasedVideoFormat*
    * Schema Ex: ` CALL db.index.fulltext.drop('MovieTitlePersonName') `
        * Drops the entire full-text schema *MovieTitlePersonName*

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
* **EXPLAIN** 
    * Showcases the call process of a query
    * Ex: ` EXPLAIN MATCH (p:Person {name:'Tom Hanks'}) `
        * Outputs a infographic of the call procedure
* **PROFILE**
    * Runs a query with proformance metrics
    * Ex: ` PRPOFILE MATCH (p:Person {name:'Tom Hanks'}) `
        * Outputs infographic of call procedure with time and memory access information

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
    * **split()**
        * Splits a string based on a demlimiter (default: ',')
        * Ex: ` RETURN split(<some string>) `
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
    * **coalesce()**
        * Replaces null values with a given value
        * Ex: ` RETURN split(coalesce(line.genres,"") `
            * Replaces the null values in *lines.genres* with "" the splits them
* Conversion
    * **toInteger()**
    * **toFloat()**
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
* List all the graph's constraints
    * ` CALL db.constraints() `
* List all existing indexes
    * ` CALL db.indexes() `
* Clear all parameters
    * ` :params {} `
* Check on all queries being performed
    * ` :queries ` 
    * ` dbms.listQueries() `
* Remove everything
     * (using APOC)
     ```
     // Delete all constraints and indexes
     CALL apoc.schema.assert({},{},true);
     // Delete all nodes and relationships
     CALL apoc.periodic.iterate('MATCH (n) RETURN n','DETACH DELETE n', { batchSize:500 })
     
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

## Best Practices
* Parameters
    * Create parameters via `=>`
        * Single Ex: ` :param actorName => 'Tom Hanks' `
        * Mutiple Ex: ` :params {actorName: 'Tom Cruise', movieName: 'Top Gun'} `
    * Cypher is expensive to recompile, use `$` to parameterize code
        * Ex: ` WHERE p.name = $actorName `
    

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
