---
title: How lucene's mlt works

---

###TL;DR
It takes the document, computes tf/idf, takes the top 25 (configurable) terms, and queries the DB with these terms with a boost. 

The long version is extremly interesting, go read it - [it right here](http://cephas.net/blog/2008/03/30/how-morelikethis-works-in-lucene/ "Lucene MLT")



