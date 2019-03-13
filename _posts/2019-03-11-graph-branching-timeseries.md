---
layout: post
title: 'Git like Branching with Graph database'
# tags: [Neo4j, GraphDB, Git, Timeseries]
# featured_image_thumbnail: assets/images/posts/2019_03_08_git_branching_graph.png
featured_image: assets/images/posts/2019-git-like-branching/featured.png
featured_image_source:
    url: https://neo4j.com/blog/graph-databases-drupal-neo4j-module-rules-integration/
    name: Neo4j
featured: true
hidden: true
---
<!-- https://neo4j.com/blog/graph-databases-drupal-neo4j-module-rules-integration/ -->
Recently, I came across an interesting problem of product taxonomy. Essentially have categories, sub categories and assign products to them. Simple. However, there was a catch. 

<!--more-->

Users wanted their own private workspace, where they make all their changes, review it, and then make the changes live. Changes in private workspace had to be  **merged** with live changes. A workspace could live for short or go for weeks or even months, without having a need ot accept live changes. Also, there was no restriction on how many workspaces a user could have.

Does it remind you of Git and branches?

Before we talk about branches, lets go over timeseries first.
#### Timeseries in Graph database

The basis of this model is to maintain all relationships. Never delete, instead expire. There are several ways of doing this, but in this example, I am going to use relationship properties. I am using Neo4j to spike this out.

Lets create some categories and build a tree.

```
create (c:category {name: 'cat 1'});
create (c:category {name: 'cat 2'});
create (c:category {name: 'cat 3'});

match (a:category), (b:category)
where a.name = "cat 1" and b.name =  "cat 2"
create (a)-[:has {branch: 'master', from: 0, to: 2148530400000}]->(b);

match (a:category), (b:category)
where a.name = "cat 1" and b.name =  "cat 3"
create (a)-[:has {branch: 'master', from: 0, to: 2148530400000}]->(b);
```

{% include image-caption.html imageurl="assets/images/posts/2019-git-like-branching/initial-categories.png#small" 
title="Initial Categories" caption="Initial Categories Result" %}

Ignore `branch` property for now in relationships `[:has {branch: 'master', from: 0, to: 2148530400000}]`. 

Key is that it has `from` and `to` properties indicating creation time and expiration time respectivley. These properties with value `0` indicates begining of time and `2148530400000` indicates end of time. 

<strong>
Note: If you are going to use milliseconds as end of time, be aware of 
<a href="https://en.wikipedia.org/wiki/Year_2038_problem" target="_blank">year 2038 problem</a>
</strong>


Lets add some more categories to the tree at T-1 and T-2.



```
create (c:category {name: 'cat 4'});
create (c:category {name: 'cat 5'});

# At time T-1
match (a:category), (b:category)
where a.name = "cat 2" and b.name =  "cat 4"
create (a)-[:has {branch: 'master', from: 1, to: 2148530400000}]->(b);

# At time T-2
match (a:category), (b:category)
where a.name = "cat 2" and b.name =  "cat 5"
create (a)-[:has {branch: 'master', from: 2, to: 2148530400000}]->(b);

```

{% include image-caption.html imageurl="assets/images/posts/2019-git-like-branching/step-2.png#small" 
title="Additional Categories" caption="Additional Categories at Time T-1 and T-2" %}

To get everything at Time T1:

```
match (n:category)-[rel:has]-(:category) 
where rel.from <= 1 
return n
```

{% include image-caption.html imageurl="assets/images/posts/2019-git-like-branching/step-3.png#small" 
title="Graph at time T-1" caption="Graph at Time T-1" %}

To get everything that is not expired:

```
match (n:category)-[rel:has]-(:category) 
where rel.to = 2148530400000
return n
```

Having this model as a basis, you can now traverse the graph based on time.
Remember:
1. When you add a node, add a relationship between the node or start node, whose creation time is now and expiration time is infinity.
2. When you delete a node, only update expiration time for all the relationships associated with that node. Do not delete the node itself.

#### Branching

We added `branch` property to all our relationships. Lets say we have branch name `test-branch` created at time `T-4`. 

User added `category 6` as a child of `category 3` at T-4 on `test-branch`.

```
create (c:category {name: 'cat 6'});

match (a:category), (b:category)
where a.name = "cat 3" and b.name =  "cat 6"
create (a)-[:has {branch: 'test-branch', from: 4, to: 2148530400000}]->(b);
```

At `T-5`, user also added `category 7` as child of `category 3` on `master`.

```
create (c:category {name: 'cat 7'});

match (a:category), (b:category)
where a.name = "cat 3" and b.name =  "cat 7"
create (a)-[:has {branch: 'master', from: 5, to: 2148550400000}]->(b);
```

The query to find everything has changed. You will now need to include branch name in your query. 
To find everything on master:
```
match (c:category)-[rel:has]-(:category) 
where rel.branch = 'master' 
return c;
```

{% include image-caption.html imageurl="assets/images/posts/2019-git-like-branching/step-4.png#small" 
title="Everything on master" caption="Everything on master without test branch category 6" %}

However, to query branches is more complicated. Since, we want to include everything that has not expired on or before the time of branch creation, T-4, but also, everything after on test branch.

```
match (c:category)-[rel:has]-(:category) 
where rel.branch = 'test-branch'
or rel.from <= 4 and rel.branch = 'master'
WITH rel.branch as b, c, rel ORDER BY rel.from DESC # 1
WITH b, c,HEAD(COLLECT(rel)) as latestRel # 2
return c;
```

1. Grouping by branch name, and ordering by creating time. This will put branch relationships at top of selection. This is important, since users could have expired and added new relationships on master since branch was created.
2. Taking the lastest relationship, as the last one is latest from our branch or latest from master.

**Note: Expired relations are not taken into account as part of the query above. You will need to filter for expired ones as well based on your usecase.**

{% include image-caption.html imageurl="assets/images/posts/2019-git-like-branching/step-5.png#small" 
title="Everything on master" caption="Everything on test branch" %}

You will see that only `Category 6` is present under `Category 3`, `Category 7` is excluded.

#### Can the model be improved?

Absolutely. **This was a spike**, just to prove that branching can be accomplished. 
I will be posting an alternate way of doing this in future using relationship nodes.

#### Things to consider

There were some learnings.
- Use graph database to maintain the structure and not the values. Use a different store for that.
- Creating a node for state (values stored outside) can help answer if there is a difference between 2 branches. If not, will need to replicate timeseries in the other store as well.
- Queries for branches do get complex.
- Cleaning up branch information in this particular model is not performant, since you will be querying the entire graph to find relationships belonging to a particular branch. May want to consider relationship nodes between nodes. And have `branch` nodes associated with them.

It is a approach to solve branching model. Try it out. 

**Adios!!**
