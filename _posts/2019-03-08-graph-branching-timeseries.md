---
layout: post
title: 'Git like Branching with Graph database'
# tags: [Neo4j, GraphDB, Git, Timeseries]
# featured_image_thumbnail: assets/images/posts/2019_03_08_git_branching_graph.png
featured_image: assets/images/posts/2019_03_08_git_branching_graph.png
featured_image_source:
    url: https://neo4j.com/blog/graph-databases-drupal-neo4j-module-rules-integration/
    name: Neo4j
featured: false
hidden: true
---
<!-- https://neo4j.com/blog/graph-databases-drupal-neo4j-module-rules-integration/ -->
Recently, I came across an interesting problem of product taxonomy. Essentially have categories, sub categories and assign products to them. Simple. However, there was a catch. 

<!--more-->

Users wanted their own private workspace, where they make all their changes, review it, and then make the changes live. A use could move products or categories to other part of the tree, delete them, split categories etc.
Direct changes to live version were allowed at any time. Changes in private workspace had to be  **merged** with live changes. A workspace could be short or go for weeks or even months, without having a need ot accept live changes. Also, there was no restriction on how many workspaces a user could have.

Does it reminds you of Git and branches?

Now, imagine **1.5-2 million** products, **10-20k** categories, **300-500 users**.
Given, not every user will create a workspace, but a system cannot assume so. Need a solution that will allow to add more products over time.

Options on table:
- Copy data for every workspace - Hell no!!
- Git as a database - Hmm, can solve merge problems, but has a lot of other problems!!
- Relational database - Doable, but maintaing relationships in tree will not be simple, don't have to use it as pure relational database either.
- Document store - In itself won't be enough.
- Graph database - Lets talk.

#### Timeseries in Graph database

The basis of this model is to maintain all relationships. Never delete, instead expire. There are several ways of doing this, but in this example, I am going to use relationship properties. I am using Neo4j to spike this out.

Lets create some categories, products and build a tree

```
// Add Cypher Query to create categories and products
```

1. query line 1 explanation 
2.
3. Ignore the **branch** property for now. I'll cover that in branching section.

Result:
// add graph database screenshot to show relationships



Explanation of above query and screenshot of graph

```
// Add Cypher Query to fetch based on time
```

Result:
// add graph screenshot for result

Having this model as a basis, you can now traverse the graph based on time.
Remember:
1. When you add a node, create a relationship whose creation time is now and expiration time is infinity.
2. When you delete a node, only update expiration time t now for all the relationships associated with that node. Do not delete the node itself.

#### Branching



#### Can the model be improved?

Absolutely. **This was a spike**, just to prove that branching can be accomplished. 
I will be posting an alternate way of doing this in future using relationship nodes.

#### Things to consider

There were some learnings.
- Use graph database to maintain the structure and not the values. Use a different store for that.
- Creating a node for state (values stored outside) can help answer if there is a difference between 2 branches. If not, will need to replicate timeseries in the other store as well.
- Queries are complex, but performant.
- Cleaning up branch information in this particular model is not performant, since you will be querying the entire graph to find relationships belonging to a particular branch.

It is an approach to solve branching model. Try it out. 

**Adios!!**
