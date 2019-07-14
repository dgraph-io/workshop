# Building Flock with Dgraph 
---
Howdy, Welcome to "Building Flock with Dgraph". 

Flock is a simple graph model build using the tweet data available from Twitter API. We build the graph in Dgraph using these tweets and run some fun and intersting queries on top of it.

Here are the learning goals of the workshop, by the end of it you should become familiar with,
- The approach towards building applications on Dgraph.
- Querying and mutations in Dgraph.   

We would be using Flock as an example to meet the learning goals. 

# Software requirements
---
Good news is that we would be running the code samples and the project with Docker containers and 
docker-compose, Here are the installation links, 

- [Installing docker](https://docs.docker.com/install/)
- [Installing docker compose](https://docs.docker.com/compose/install/)

# Chapters
---
The workshop is composed of three sections, the first one optional, which is meant for absolute beginners Dgraph. 

- [Getting started with Dgraph](./1-getting-gtarted/README.md)
  
 This chapter is the guide for getting started with Dgraph. Want to understand how to install and run Dgraph? 
 Curious to know fundamentals like creating nodes, edges, and traversing them? Explore them using Dgraph's Ratel-UI.

- [Designing flock and its mutations](./2-flock-mutations/README.md)
  
Here you'll learn about Flock's design and schema, the nuts and bolts of bringing your application
graph to life using mutations with upserts. Here is where Flock get's built. 

- [Querying Flock](./3-flock-querying/README.md)

In the last chapter, we've built Flock using data available from twitter's API. Let's construct some
interesting graph queries in this chapter. We've run through queries related to Searching nodes, 
filtering, traversing, reverse traversing, date-time queries, and much more.
  

# Issues
---
This workshop is very new, so there's a possibility that some things might be missing or wrong.

If you find anything that seems broken, please file an issue. Or even better, send a pull request! 

# Resources
- [Docs](https://docs.dgraph.io)
- [Discuss forum](https://discuss.dgraph.io)
- [Community slack](https://dgraph.slack.com)
- [Blog](https://blog.dgraph.io)
---

