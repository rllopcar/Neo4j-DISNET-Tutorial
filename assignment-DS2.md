<center>
# Disease understanding: Dealing with complex and unstructured big data in biomedical domain (Dr. Alehjandro Rodríguez)
</center>
-

- Authors: 
	- Roberto Llop Cardenal - robertollocardenal@gmail.com
	- Ignacio Rodríguez - ignacio.rodriguez.burgos@gmail.com

## Data visualization tutorial for DISNET platform using Neo4j

For this data science seminar we chosen the proposal **API Data visualization** where we are going to create a step by step guide on how to query and visualize the relationships between disease and their phenotopic characteristics.

### Introduction

[Neo4j](https://neo4j.com/) is a graph database management system described by its developers as an ACID-compliant transactional database woth native graph storage and processing. Graph databases are an exceptional tool when it comes to find relationships between item or nodes, which is why we have chosen it in the first place for this projects.

For us, both diseases and phenomatic characteristics or symptoms are going to be represented as nodes and will be linked by a relationship that will indicate the symptoms that a diseases HAS. Once we hace the database populated by the different nodes and relationships we will show how to perform interesting queries to the database that will help the user discover/find new knowledge about the relationship between disease and their relationships with their symptoms.

Every step and command used is written in the tutorial along with a .gif file showing how to perform the operation in the Neo4j Desktop environment.

### Requirements

- Having installed Neo4j Desktop.

### Step 0 - Setting up environment

Once we have Neo4j Desktop version install, the first step is to launch the application. Next we will create a new project by clicking on the icom "New Project.
<center>
<img src="./gifs/newProject.gif" width="450" height="300" />
</center>

The Next step is to create a new graph by clicking on the icon "New Graph", when indicated, chose "Local graph" and set a password.

<center>
<img src="./gifs/newGraph.gif" width="450" height="300" />
</center>

In order to import data from an API, we will use the plugin APOC, which includes some functions to perform this process in a simple and efficient way. I order to install the plugin we need to click on the icon "Add Plugin" and select APOC (it might take a minute to install, depending on your internet conexion).

<center>
<img src="./gifs/apoc.gif" width="450" height="300" />
</center>

Once we have the APOC plugin installed we have to indicate in the configuration file of our graph that we allow external inputs through APOC functions. We do this by writing in our configuration file the following text:

~~~
# apoc configuration
apoc.import.file.enabled=true
~~~
<center>
<img src="./gifs/configuration.gif" width="450" height="300" />
</center>

After setting up our environmet we are ready to import some date into our graph, but first we need to start our graph and acces it via the neo4j browser.
<center>
<img src="./gifs/startGraph.gif" width="450" height="300" />
</center>

This process takes sometime, specially if is the first time. After is started just click the icon "Neo4j Browser". After clicking a browser window will appear and from now on we will work from this window.
<center>
<img src="./gifs/browser.gif" width="450" height="300" />
</center>


### Step1 - Retrieve data from the DISNET platform
Neo4j has the feature that allow us to import data, in this case in JSON format, directly from endpoint API.

The DISNET platform has a lot possibilities when it comes to retrieving information and it depends to the interest and uses case for each user. So, in this tutorial we are going to explain the url that we have use for this tutorial. This is the query we have used:

~~~
"http://disnet.ctb.upm.es/api/disnet/query/diseaseWithMoreDisnetConcepts?source=pubmed&version=2018-04-03&limit=5000&forceSemanticTypes=sosy&token=eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJncmFyZG9sYWdhckBnbWFpbC5jb20iLCJhdWQiOiJ3ZWIiLCJuYW1lIjoiR2VyYXJkbyBMYWd1bmVzIEdhcmPDrWEiLCJ1c2VyIjp0cnVlLCJpYXQiOjE1MTM4MDY4MjB9.vGaQE6HHucr8DOM8MNnBWOCnV2dPE0r2qu_9NsYFEI-PCn5J6_iyhTUH4pwBoWCIVyvIjpNwk9vOnjQmS0-wXQ"
~~~

Looking closely to the parameters we can differentiate the next elements

 - **source**=pubmed: We have selected the data source pubmed for retrieving our data.
 - **versio**=2018-04-03: From the pubmed datasource we have selected the snapshot 2018-04-03, which in the case of pubmed is the only one that has.
 - **limit**=5000: We have selected this treshold because the default value if not specified is 10, which is not representative enough for this use. 
 - **forceSemanticTypes=sosy**: It force in the query the semantic types or types defined. If there are more than one, they are separated by commas.
 - **token**: Access key that allows the query to be excuted.

 For more information about how to make a proper and efficient queries, visit [DISNET](http://disnet.ctb.upm.es/apis/disnet)

The next is to retrieve the data and check if the petition to the data source works correctly. In order to do this copy and paste the following command into the console of your Neo4j browser.

~~~
call apoc.load.json("http://disnet.ctb.upm.es/api/disnet/query/diseaseWithMoreDisnetConcepts?source=pubmed&version=2018-04-03&limit=5000&forceSemanticTypes=sosy&token=eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJncmFyZG9sYWdhckBnbWFpbC5jb20iLCJhdWQiOiJ3ZWIiLCJuYW1lIjoiR2VyYXJkbyBMYWd1bmVzIEdhcmPDrWEiLCJ1c2VyIjp0cnVlLCJpYXQiOjE1MTM4MDY4MjB9.vGaQE6HHucr8DOM8MNnBWOCnV2dPE0r2qu_9NsYFEI-PCn5J6_iyhTUH4pwBoWCIVyvIjpNwk9vOnjQmS0-wXQ") yield value
~~~
<center>
<img src="./gifs/tryUrl.gif" width="450" height="300" />
</center>

If everything works like is supposed too you should have got a result (first lilnes) similar to this one:

~~~
{
  "authorizationMessage": "Authorized!",
  "authorized": true,
  "diseaseCount": 10,
  "errorsFound": [],
  "responseMessage": "OK",
  "diseaseList": [
    {
      "name": "hypercalcemia",
      "disnetConceptsCount": 22,
      "disnetConceptList": [
        {
  ...
~~~
Perfect, let's continue with the next step where we are goingt to populat our graph.


### Step 2 - Populate the graph

In the previus step we have check that the url works but we have not populated our graph. In order to do that we have to create the nodes and relationships from the json file that we got as a response from the DISNET API.

For this tutorial we are going to have two different types of nodes:

 - **Diseases**
 - **Symptoms**

One type of relationship or link that works as a nexus between our nodes. That relationship is going to be defined with the term "HAS" and is going to be a directional relationship, going from a **disease** to a **symptom**. 

We achieve this with the following command:

~~~
call apoc.load.json("http://disnet.ctb.upm.es/api/disnet/query/diseaseWithMoreDisnetConcepts?source=pubmed&version=2018-04-03&limit=5000&forceSemanticTypes=sosy&token=eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJncmFyZG9sYWdhckBnbWFpbC5jb20iLCJhdWQiOiJ3ZWIiLCJuYW1lIjoiR2VyYXJkbyBMYWd1bmVzIEdhcmPDrWEiLCJ1c2VyIjp0cnVlLCJpYXQiOjE1MTM4MDY4MjB9.vGaQE6HHucr8DOM8MNnBWOCnV2dPE0r2qu_9NsYFEI-PCn5J6_iyhTUH4pwBoWCIVyvIjpNwk9vOnjQmS0-wXQ") YIELD value
UNWIND value.diseaseList AS dL
UNWIND dL.disnetConceptList as sosy
merge (symptom:Symptom {cui:sosy.cui}) ON CREATE SET symptom.name=sosy.name
merge (disease:Disease {name:dL.name}) 
merge (disease)-[:HAS]->(symptom)
~~~

Again this step takes about a minute to execute and if succesful, you should get a similar message saying:

~~~
Added 2595 labels, created 2595 nodes, set 3011 properties, created 19003 relationships, completed after 38118 ms.
~~~


### Step 3 - Perform interesting queries

Alright! Now that we have some data in our database let's make our first query.

**Query 1**: Let's retrieve all the Disease nodes. (Limiting the result to the first 100 diseases)

~~~
MATCH (n:Disease)
RETURN (n) LIMIT 100
~~~

**Query 2**: Let's retrieve all the Symptoms nodes. (Limiting the result to the the first 100 symptoms)

~~~
MATCH (n:Symptom) 
RETURN n LIMIT 100
~~~

***Query 3**: Query that shows some disease and their symtoms with a limit of 100 nodes for the first type of node that satisffy that condition. 

~~~
MATCH (d:Disease)-[:HAS]->(s) RETURN s, d LIMIT 100
~~~

<center>
<img src="./gifs/query1.gif" width="450" height="300" />
</center>

- **Query 4**: Disease and symptoms for a given disease.

~~~
MATCH (d:Disease {name:"Agnosia"})-[r:HAS]->(s:Symptom)
RETURN d,s
~~~
<center>
<img src="./gifs/query4.gif" width="450" height="300" />
</center>


- **Query 5 **: The two diseases with more symptoms and the symptoms they have in common. The result should be the disease beriberi and encephalomyelitis with nine symptom nodes in common.

~~~
MATCH (d)-[:HAS]->(s)
RETURN d, COLLECT(s) as symptom
ORDER BY SIZE(symptom) DESC LIMIT 2
~~~

<center>
<img src="./gifs/query5.gif" width="450" height="300" />
</center>

- **Query 6** : Symptoms in common given two diseases. The result should be 5 symptom nodes that they have in common.

~~~
MATCH (d1:Disease {name:"beriberi"})-[HAS]->(s:Symptom)<-[:HAS]-(d2:Disease {name:"Agnosia"})
RETURN d1, d2, s
~~~

<center>
<img src="./gifs/query6.gif" width="450" height="300" />
</center>

- **Query 7**: Given a disease returns the disease that shares the most symptoms and those symptoms. In this case we obtain that the disease with more symptoms in common is Wernicke encephalopathy.

~~~
MATCH (d1:Disease {name:"beriberi"})-[HAS]->(s:Symptom)<-[:HAS]-(d2:Disease)
RETURN d1,d2, COLLECT(s) AS symptom
ORDER BY SIZE(symptom) DESC LIMIT 2
~~~

<center>
<img src="./gifs/query7.gif" width="450" height="300" />
</center>

- **Query 8**: Given a disease returns the two disease that shares the most symptoms and those symptoms. In this case for the disease beriberi we obtain that the two disease with more symptoms in common are Wernicke encephalopathy and post-vaccinal encephalitis. In this query we can visualize the symptoms that each pair of disease has in common as well as the symptoms that all diseases in th query have in common.

~~~
MATCH (d1:Disease {name:"beriberi"})-[HAS]->(s:Symptom)<-[:HAS]-(d2:Disease)
RETURN d1,d2, COLLECT(s) AS symptom
ORDER BY SIZE(symptom) DESC LIMIT 2
~~~

<center>
<img src="./gifs/query8.gif" width="450" height="300" />
</center>

### Conclusion

This tutorial is aimed for those interested professionals that are looking for new ways of interacting with their data. Of course, graph databases are not the panagea for every data science problem, but they provide a series of unique features that can help find new relationships and interactions between the entities of a database. These new relationships provides new knowledge that would be difficult to find with a relational or NoSQL type of database.

