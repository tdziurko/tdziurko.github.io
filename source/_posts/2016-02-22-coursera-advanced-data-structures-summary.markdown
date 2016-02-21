---
layout: post
title: My impressions after Advanced Data Structures at Coursera
date: 2016-02-22 10:00:10:11 +0200
comments: true
categories: 
- java
- learning
keywords: 
description: Short summary after I have finished "Advanced Data Structures in Java" at Coursera
---

Short summary after I have finished "Advanced Data Structures in Java" at Coursera

<!-- more -->

Online course ["Advanced Data Structures in Java"](https://www.coursera.org/learn/advanced-data-structures/) at Coursera have finished last week and I felt 
that after these cool five weeks it is worth to recap and summarise my impressions.

{% img center /images/blog/2016/02/graphs.png 900 600 %}
_image source: [http://lin-ear-th-inking.blogspot.com](http://lin-ear-th-inking.blogspot.com/2013/05/flight-paths-in-jeql-redux.html)_
 

# Course Content
 
At start, do not be mislead by course name. It is not about all advanced data structures what would be a very broad topic, but about graphs, 
searching algorithms and path calculations.

During the first week we have learnt about data structures that can be used to represent graphs - Adjacency Matrix and Adjacency 
List - and what are best use cases to choose each of them. Next we had to create a class hierarchy for map search engine to be used as 
a framework during the rest of our assignments. Assignment from this week also required to implement [Breadth-first Search](https://en.wikipedia.org/wiki/Breadth-first_search) 
(BFS) algorithm. [DFS](https://en.wikipedia.org/wiki/Depth-first_search) was also covered but we have not coded it 
in our assignments as it can not be used to find shortest paths.

Third week was about implementing [Dijkstra](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) and 
[A-Star](https://en.wikipedia.org/wiki/A*_search_algorithm) algorithms and how they can be created using BFS as a base. Using BFS as a starting point 
helped me to understand these algorithms much faster. Fourth week was theoretical one, only one without an assignment. So there were no coding but more lectures about 
[Travelling Salesman Problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem), NP Complete and NP Hard problems and different types 
of graphs (Eulerian, Hamiltonian).

Final week was the most creative one as we were asked to implement an extension to any of algorithms we have learnt about. We could extend A* 
or Dijkstra to use not only distance but also road type or speed limits to find the fastest route or we could try to improve speed of any of these algorithms or... 
or we could create completely original extension. I have chosen to implement something I called "Route Calculation Cache", quote from my submission:
 
{% blockquote  %} 

My extension remembers calculated routes and when another user wants to find similar route, extension will retrieve 
this stored route instead of starting duplicated and extensive calculation. This will decrease execution time 
and user will see routes in a shorter time.

{% endblockquote  %}

And after that course ended after five weeks with graphs.
 
# My impressions

* Sophisticated algorithms are not the thing we use frequently at work so it was very funny feeling to 
learn again stuff that I was forced to memorize several years ago at university. Obviously now I was more motivated 
so learning was more enjoyable process than when I was a young student. 

* Test datasets can be obtained for any area using provided application so everyone was able to test A* or Dijkstra 
using streets from their neighbourhood. You could check if your route to work is optimal or not ;) Below fragment of my home city 
with calculated example route:
  
{% img center /images/blog/2016/02/plock_dijkstra.png 700 520 %}

* Peer reviews. Our solutions for Week3 and (partly) Week5 were reviewed by other developers attending this course. At first 
I was not convinced that it will bring any value to my learning process, but it appeared that comparing how 
people from different countries and with different backgrounds designed classes hierarchy to solve same problem
was an interesting experience. I saw approaches similar to mine but also completely different ones but still valid.
Another topic is different approach to documenting code: some developers shared my view and used comments only where it
was really, really necessary and tried to write self-explanatory code. But in some people's solutions I saw 
as many lines of documentation as lines of code :)
 
* Good peer pressure: because I have advertised this course at [JVM Bloggers](https://github.com/tdziurko/jvm-bloggers/) 
newsletter I was motivated to graduate :) And the more people I know tweeted that they are joining, the more motivated 
I was to complete assignments week by week. Discussing about course with developers from my company helped a lot too. 
Generally this is one of the finest features of online courses: motivating others and getting motivation :)


# Wrap up

I am happy that I have enrolled "Advanced Data Structures in Java". I have refreshed some long forgotten 
knowledge about graph structures, learnt a lot too so definitely it was time well spent. Also a satisfaction after
successfully finishing a five-weeks long course is a nice feeling.
 
And what about you? What are you thoughts about this course or online courses in general?