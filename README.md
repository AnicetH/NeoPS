# NeoPS
Objective of This document contains an exercise for Neo4j Professional Services Candidates, to be completed as part of the hiring process. Objectives of this exercise are to demonstrate: ● your capabilities to correctly grasp documented requirements ● your ability to rapidly get up to speed with new 


**the data model:**

![Image](https://github.com/user-attachments/assets/e08f3ce8-f11d-4842-a75d-d01b4c740261)

// ***********Task 1: Loading a dataset into Neo4j** **************

// Import Data with cypher to create a Person's Node and then relations between employees


// Load CSV file to create nodes and Subordination's relation between employees

LOAD CSV WITH HEADERS FROM 'https://drive.google.com/uc?export=download&id=14fuprkvrvtsLhg1l64QTyOYuPpHuQeR-' AS row
MERGE (employee:Employee {name: row.employee_name})
MERGE (boss:Employee {name: row.boss_name})
MERGE (employee)-[:REPORTS_TO]->(boss);


// Load CSV file to create friendship relation between employees

LOAD CSV WITH HEADERS FROM 'https://drive.google.com/uc?export=download&id=1KOGAEdFdA6arJHmJnFNKZL71QR5N5y6Z' AS row
MERGE (employee:Employee {name: row.employee_name})
MERGE (friend:Employee {name: row.friend_name})
MERGE (employee)<-[:IS_FRIENDS_WITH]->(friend);




// ********** **Task 2: Query your data** ***********************

// Query: 1) Show a hierarchy of all people working under “Darth Vader”.

MATCH path=(boss:Employee {name: 'Darth Vader'})<-[:REPORTS_TO*]-(employee:Employee)
RETURN employee.name, length(path) as len
ORDER BY  len


// Query: 2) Show all the people that work in the team of Jacob, but are not friends with Jacob.

MATCH (employee:Employee)-[:REPORTS_TO]->(boss:Employee {name: 'Jacob'})
WHERE NOT (employee:Employee)-[:IS_FRIENDS_WITH]->(boss:Employee)
RETURN employee.name
ORDER BY employee.name




// ********** **Bonus task 3 (for consultant / data scientists)*************************


************Show off how you’d use graph data science to gain insights on this data set:

// *******BonusCommunity Detection Using Neo4j Algorithms********

Community Detection Using Neo4j Algorithms

1- Louvain Modularity:
This algorithm can help to identifie strongly connected groups or clusters within the graph.
We can use it to find communities based on the strongest friendships.

//********* Cypher code using graph algorithm:
CALL gds.louvain.stream($generatedName, $config)
YIELD nodeId, communityId AS community, intermediateCommunityIds AS communities
WITH gds.util.asNode(nodeId) AS node, community, communities
WITH community, communities, collect(node) AS nodes
RETURN community, communities, nodes[0..$communityNodeLimit] AS nodes, size(nodes) AS size
ORDER BY size DESC
LIMIT toInteger($limit);

//********* Result of the Louvain algo*******

Communit/Communities/Size/Nodes
13	117,13	12	BradleyScottJacobAdamBillPeterJoshHenryIkeLexusConnerPete
29	122,29	10	AaronDennisDannyHeatherSteveIsiahTuckerHenriettaLukeGregory
139	35,139	10	DanielNateMauryDavinLindaEmmettDezirayLeroyBrianBeth
13	149,13	9	HeidiAlexandriaMissyJennyDouglasKellyMillieDougieJessica
32	47,32	9	LauraKipCaroleLenoraAlexFelipeCosetteCoreyLouis
13	28,13	8	SarahWilsonKennyNicholeSamanthaNancyGaryVernon
13	13,13	8	SylvesterMorganIslaEricLisaNellyFredKatie
139	130,139	7	AnnieMilanJoseGordonAllisonLannieKal
32	70,32	7	DianeBillyMikeFrancesJasonLolaConnor
32	88,32	7	ThadJessieTadKyleClarkAmirLizzy
13	131,13	6	ElmerDaveyFosseJakeKevinSally
122	9,122	6	MindyMelissaTrentLeslieShellyEmily
32	32,32	5	WendyAllenTheresaNathanFrancis
139	109,139	4	TommyTriciaFloraTammy
122	145,122	4	TerrencePattyEstherMark
139	20,139	4	ClydeHannahMelitaEstella
13	139,13	3	GreeleyMeganChantal
122	62,122	3	DavidQuaidMandy
32	136,32	3	NelsonMonicaJimmy
139	0,139	3	MeaganSimonAllie
122	84,122	3	TimmyChrisFilmore
32	60,32	2	JenniferAshley
139	10,139	2	MariaStan
57	57,57	1	Elvis
6	6,6	1	Gavin
44	44,44	1	Frederick
40	40,40	1	Bridon
5	5,5	1	Ferrari
2	2,2	1	Wayne
33	33,33	1	Karen
30	30,30	1	Charlotte
123	123,123	1	Stephen
29	29,29	1	Larry
105	105,105	1	Craig
93	93,93	1	Bobby
82	82,82	1	Thomas
75	75,75	1	Ryan
106	106,106	1	Carlos
74	74,74	1	Stacy
69	69,69	1	Herbert


2- Triangle Counting :
This algoritm is useful for detecting smaller friendship circles.s.

//********* Cypher code using graph Triangle count algorithm:
CALL gds.triangleCount.stream($generatedName, $config)
YIELD nodeId, triangleCount AS triangles
WITH gds.util.asNode(nodeId) AS node, triangles
RETURN node, triangles
ORDER BY triangles DESC
LIMIT toInteger($limit);

//********* Result of the Triangle count algo*******

Node / Triangles
Estella	288
Danny	260
Mandy	244
Millie	243
Leroy	241
Kenny	232
Jacob	230
Alex	210
Steve	207
Aaron	205
Kip	200
Filmore	188
Peter	179
Clark	175
Conner	172
Isiah	170
Emmett	166
Hannah	165
Lola	157
Allie	157
Nelly	156
Louis	154
Dougie	152
Wilson	146
Morgan	140
Lizzy	138
Pete	135
Cosette	134
Linda	133
Jessica	132


3- Label Propagation Algorithm (LPA):
This algorithm spreads labels across the graph, resulting in clusters representing both formal and informal relationships.
It's ideal for finding communities based on informal ties

//********* Cypher code using graph LPA algorithm:
CALL gds.labelPropagation.stream($generatedName, $config) YIELD nodeId, communityId AS community
WITH gds.util.asNode(nodeId) AS node, community
WITH collect(node) AS allNodes, community
RETURN community, allNodes[0..$communityNodeLimit] AS nodes, size(allNodes) AS size
ORDER BY size DESC
LIMIT toInteger($limit);

//********* Result of the LPA algo*******
Community	Size	Nodes
194	126	BradleyMeaganAnnieSylvesterDianeMorganClydeThadMelissaTommyBillyGreeleyEricTerrenceWendyScottHeidiAlexandriaLauraHannahAaronSarahJacobDennisMaria
191	2	IslaMissy
180	1	Wayne
183	1	Ferrari
184	1	Gavin
187	1	Mindy
207	1	Larry
208	1	Charlotte
210	1	Allen
211	1	Karen
213	1	Daniel
218	1	Bridon
222	1	Frederick
225	1	Carole
235	1	Elvis
238	1	Jennifer
240	1	Quaid
247	1	Herbert
252	1	Stacy
253	1	Ryan
260	1	Thomas
266	1	Kyle
271	1	Bobby
283	1	Craig
284	1	Carlos


//**** Suppose you were to advise the HR team at this company; can you find employees and/or
teams that could be of special interest to the HR team? You can deploy any algorithm of your
choice? ***\\

// Key Graph Algorithms to apply to advise a relevant insights to the HR team

1. Betweenness Centrality:

This algorithm identifies key individuals who serve as bridges between different groups or teams. They might be crucial connectors in formal and informal networks.

*** Insights: High-centrality employees could be mentors, mediators, or potential risks (if overloaded).

*** Top 10 Employees and/or teams list who serve as bridges in this case:
Node	Score
Millie	2420.9395761101623
Leroy	2262.953014159903
Estella	1917.2772474601425
Jacob	1546.7986905681175
Alex	1479.3627172019583
Linda	1106.8537230914935
Kip	1088.3188652850415
Steve	1077.8618358335852
Ashley	1051.2505716505725
Heidi	1030.7562119253303

*** Advise: Recommend programs to support these key individuals, like leadership training or burnout prevention.


2. Community Detection with Louvain:

Detect communities or clusters in the graph to identify tightly-knit teams or informal groups.

*** Insights: HR could explore why some groups are isolated and encourage cross-team collaboration.

*** Top3 Employees community:

Community	Size	Nodes
13		46	Bradley/Sylvester/Morgan/Isla/Greeley/Eric/Scott/Heidi/Alexandria/Sarah/Missy/Jacob/Jenny/Adam/Lisa/Elmer/Megan/Davey/Douglas/Bill/Fosse/Kelly/Millie/Peter/Jake
32		33	Diane/Thad/Billy/Wendy/Laura/Allen/Kip/Carole/Lenora/Nelson/Jennifer/Alex/Mike/Theresa/Jessie/Tad/Frances/Jason/Kyle/Monica/Clark/Amir/Nathan/Lizzy/Lola
139		30	Meagan/Annie/Clyde/Tommy/Hannah/Maria/Daniel/Nate/Stan/Maury/Melita/Davin/Linda/Simon/Tricia/Milan/Emmett/Jose/Flora/Deziray/Gordon/Allison/Allie/Leroy/Tammy
122		16	Mindy/Melissa/Terrence/Trent/Leslie/David/Quaid/Mandy/Shelly/Timmy/Patty/Esther/Emily/Chris/Mark/Filmore
29		11	Aaron/Larry/Dennis/Danny/Heather/Steve/Isiah/Tucker/Henrietta/LukeGregory


*** Advise:Highlight under-connected or over-connected groups for team-building activities.


3. Closeness Centrality:

This measures how quickly an individual can reach others in the network.

*** Insights: Employees with high closeness scores could be excellent internal communicators or leaders.

*** Top10 Employees list with high closeness score:
Node	Score
Millie	0.39622641509433965
Morgan	0.39375
Estella	0.38769230769230767
Jacob	0.3853211009174312
Leroy	0.38181818181818183
Steve	0.3673469387755102
Isiah	0.3652173913043478
Heidi	0.3539325842696629
Bill	0.3452054794520548
Jessie	0.33962264150943394

*** Advise:Recognize these individuals for their potential influence in connecting teams.


4. Triangle Count:

Measures the number of closed triangles (friendship circles) each individual is part of.

*** Insights: Individuals in multiple triangles might be key players in informal networks.

*** Top10 Employees with closed triangles
Node	Triangles
Leroy	39
Millie	31
Estella	28
Steve	18
Alex	17
Jacob	16
Morgan	16
Kip	15
Linda	12
Isiah	12

*** Advise:Leverage these individuals to foster collaboration and cohesion.



//************************ **SUMMARY OF HR-FOCUSED INSIGHT** *************************************\\

1- Key Employees to Support: Employees with high betweenness or closeness scores may need additional support or recognition to prevent burnout and encourage their development.

2- Isolated Teams: Louvain clustering can highlight isolated communities. HR could initiate efforts to integrate these groups

3- High-Connectivity Employees: Individuals with many connections or triangles could be utilized for mentorship or team-building roles.

4- Potential Risks: Low-scoring individuals or isolated groups may need engagement initiatives.







//*********Bonus task 4 (for architects / data modellers):** 
//***********extend the dataset and data model to show more (potential) functionality *******\\

The dataset and the data model can be extended to enrich its functionality by introducing new types of nodes, relationships, and properties to capture additional dimensions of relationships or organizational insights. 

****************** Extended Data Model *************************

**1. Nodes**

*** Employee: Represents employees in the organization.
--> Properties: (name, department, job_title, location, hire_date)

*** Friend: Represents informal relationships.
--> Properties: (name, interaction_frequency, shared_interests)

*** Project: Represents projects employees are involved in.
--> Properties: (project_name, project_budget, deadline, priority)

*** Training: Represents skills training programs employees participate in.
--> Properties: (training_name, skill_type, completion_status)

*** HR_Events: Represents HR-related concerns or events.
--> Properties: (event_type, event_date, location).


**2. Relationships**

*** (employee)-[:REPORTS_TO]->(boss): Links employees to their managers.

*** (employee)<-[:IS_FRIENDS_WITH]->(friend): Define friendship between employees

*** (employee)-[:WORKS_ON]->(project): Links Employee to Project.

*** (employee)-[:PARTICIPATED_IN]->(training): Links Employee to Training.

*** (employee)-[:ATTENDS_TO]->(event): Links Employee to HR_EVENTS.
