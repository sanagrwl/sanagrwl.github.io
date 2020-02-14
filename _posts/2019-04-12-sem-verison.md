---
layout: post
title: 'Automate Semantic Versioning'
# tags: ["git", "versioning"]
featured_image: assets/images/posts/sem-version-comic.jpg  
featured_image_source:
    url: https://medium.com/@elliotchance/semantic-versioning-d5b93bb4faf4
    name: Medium
featured: false
hidden: false
---

Every now and then, I come across teams who have developed a custom way of versioning their applications, artifacts etc. Some prefer manual tagging git commits with sem version, others prefer a file to be committed. There are several ways to accomplish the same, and folks usually go with a route that suits their needs. I want to showcase here an automated way of doing semantic versioning using `git tags and describe`.

<!--more-->

<strong>
I am only interested in knowing when it is a major or a minor release.
<strong>


What this means is that when a developer makes a change, only they know whether they added a new feature or it is no longer backward compatible. They don't really care if it was patch version 1 2,3 and so on. 

Even for a consumer of an artifact, it really doesn't matter if you released 2.1.0 and then jumped to 2.1.10 or 3.0.1 and skipped versions in the middle.

<strong>
Every commit is a patch unless specified.
<strong>

Any commit can go to production. With that opinion, you are making a change, and at the very least it is a patch release. 

So how do you specifiy it is not a patch release?

<strong>
Semantic Versioning
<strong>

Lets say we have the following git history:
```
d99640b (HEAD -> master, tag: v/1.1) commit 4
6031055 commit 3
3287fc8 commit 2
77b39e9 (tag: v/1.0) first commit
```

Here is a function that generates a sem version.

```
function sem_version() {
  # usage: sem_version ["v/*"]
  # Requires current commit or a previous commit to be tagged.
  # Tags should be of pattern v/d+.d+ eg: v/3.10
  IFS=-
  set -- `git describe --always --tags --match "$1"`
  unset IFS
  local major_minor=$1
  local patch=$2
  local v=`basename $major_minor.${patch:-0}`
  echo $v
}
```

If you were to execute this on the latest commit, you will get the following output:
```
    sem_version 'v/*'
    # output: 1.1.0
```

If the same thing was executed at the very first commit:
```
    git checkout 77b39e9
    sem_version 'v/*'
    # output: 1.0.0
```

If the same thing was executed at commit3:
```
    git checkout 6031055
    sem_version 'v/*'
    # output: 1.0.2
```

Lets see what `git describe` is doing.

```
    git checkout 6031055
    git describe --always --tags --match "v/*"
    # output: v/1.0-2-g6031055
```

The above output is giving you the latest tag that matches `v/*`, number of commits, `2`, since that tag and the current commit, `6031055`, prepended with letter `g`.

The `sem_version` method simply uses this to create a semantic version, with patch number is just the number of commits. The `basename` command simply removes `v/` from the `git describe` output. You can also use the commit sha to add metadata to your semantic version. Ex: `1.0.2+6031055`. Build metadata is ingored when determining precedence anyways.

<mark>Ensure that you have your very first commit tagged with 'v/' pattern.</mark>

This makes it easier and consistent way of automating sem versionig in your CI/CD tools. And as a human, one would only need to look at tags that matches pattern `v/*`. You can have your CI/CD tools push proper semantic versioned tags, but there will be always a clear distinction between automated vs manual tags. 


