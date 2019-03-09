---
layout: post
title: 'Git like Branching with Graph database'
# tags: [Neo4j, GraphDB, Git, Timeseries]
# featured_image_thumbnail: assets/images/posts/2019_03_08_git_branching_graph.png
featured_image: assets/images/posts/2019_03_08_git_branching_graph.png
featured_image_source_url: https://neo4j.com/blog/graph-databases-drupal-neo4j-module-rules-integration/
featured: false
hidden: true
---
<!-- https://neo4j.com/blog/graph-databases-drupal-neo4j-module-rules-integration/ -->
Recently, I came across an interesting problem of product management. Users wanted to group their
products into categories. There is virtually no limit to how many subcategories. Simple. However, there is a catch. 

<!--more-->

Users want their own private workspace, where they make all their changes, review it, and then make it live. 
Users can make direct changes to live version. Changes in private workspace will need to be **merged** with live changes.
User can also be working in their own workspace for a very long time without having a need to accept live changes. 
Also, there should not be a restriction on how many workspaces a user can have.

Reminds you of Git and branches, doesn't it?

Now, imagine **1.5-2 million** products, **10-20k** categories, **300-500 users**.
Given, not every user will create a workspace, but a system cannot assume so. Need a solution that will allow to add more products over time.

Options on table:
- Copy data for every workspace - Hell no!!
- Git as a database - Hmm, can solve merge problems, but has a lot of other problems!!
- Relational database - Doable, but maintaing relationships in tree will not be simple
- Document store - In itself won't be enough.
- Graph database - Lets talk.

To be continued....
