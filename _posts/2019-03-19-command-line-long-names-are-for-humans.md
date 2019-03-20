---
layout: post
title: 'Code is for humans - Chapter 1'
# tags: []
featured_image: assets/images/posts/code-is-for-humans.jpg
featured_image_source:
    url: https://techcrunch.com/2016/05/10/please-dont-learn-to-code/
    name: Techcrunch
featured: false
hidden: false
---

When you execute a command with options, do you use short or long option? And when you write a script, do you use short or long option?
<!--more-->

Here is an example of a bash script:
```
#!/bin/bash
set -eE
```
What is the difference between `e` and `E`? Error? Echo? Eject? Exit? Which one is what?

What if we wrote it as:
```
#!/bin/bash
set -o errexit -o errtrace
```

More descriptive, yes. Think of someone other than you reading it, someone who may not be as experienced as you are or new to that tech. A person won't have to reference another manual to deciper your script. And even if they have to lookup, **they have to search specific thing and not <a href="https://stackoverflow.com/questions/9952177/whats-the-meaning-of-the-parameter-e-for-bash-shell-command-line" target="_blank">what is -e in bash script</a>**

Yes it may get verbose. One can also argue that people can lookup and learn. You will have to decided on what to use.

However, I have come to an opinion that if I am exectuing a command, use short options. But if I am scripting, and someone else have to read that code, use long options.

Code is for humans, Computers are <a href="https://www.youtube.com/watch?v=4r7wHMg5Yjg" target="_blank">honey badgers</a>.

Later!

