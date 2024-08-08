---
title: 'Git: Checking out only a specific folder'
author: yossale

---
I've been looking for a way to checkout only a specific folder from a git repository. Most answers I found to this problem stated that it's not feasible - but apparently, this feature was added to recent versions of git, and it's now quite easy, as described in the following blog post : <http://briancoyner.github.io/blog/2013/06/05/git-sparse-checkout/>.

One notice - when you run the"git pull origin master" command, it looks like it's going to checkout the entire repository - it's OK. It's only going through the repository, but the eventual result will be only your specified folder.

&nbsp;