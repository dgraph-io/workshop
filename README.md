# Building Flock with Dgraph
---

Welcome to "Building Flock with Dgraph."

Flock builds a graph model on Dgraph. It makes use of the tweets data obtained directly from Twitter API.

Here are the learning goals of the workshop,
-  Learn the approach towards designing and building applications on Dgraph.
- Understand queries and mutations in Dgraph.

We will be using Flock as an example to meet the learning goals.

# Software requirements
---

Dgraph is a distributed system with multiple processes interacting with each other. To make both Dgraph and Flock easier to run, this workshop provides easy-to-run configs with Docker Compose. Here are the installation links for Docker and Docker Compose:

- [Installing Docker](https://docs.docker.com/install/)
- [Installing Docker Compose](https://docs.docker.com/compose/install/)

# Chapters
---
The workshop is composed of three sections. The first one is optional, it's meant for absolute beginners to Dgraph.

- [Getting started with Dgraph](./1-getting-started/README.md)

 This chapter is the guide for getting started with Dgraph. It deals with basic operations like installation,
nodes, edges, queries, and mutations.

- [Designing Flock and its mutations](./2-flock-mutations/README.md)

In this chapter, you'll learn about Flock's design and schema. We'll decipher the process of designing the application graph. The chapter ends with upserts in Dgraph.

- [Querying Flock](./3-flock-querying/README.md)

In this chapter, we'll be learning about querying in Dgraph. These include searching nodes,
filtering, traversing, reverse traversing, date-time queries, and much more.


# Issues
---
This workshop is very new, so there's a possibility that some things might be missing or wrong.

If you find anything that seems broken, please file an issue. Alternatively, even better, send a pull request!

# Resources

- [Tour of Dgraph](https://tour.dgraph.io)
- [Documentation](https://docs.dgraph.io)
- [Discuss Forum](https://discuss.dgraph.io)
- [Community Slack](https://dgraph.slack.com)
- [Blog](https://blog.dgraph.io)
---
