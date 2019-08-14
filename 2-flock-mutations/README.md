## Designing flock and its mutations
---

In this chapter, you'll learn about Flock's design and schema. We'll decipher the process of designing the application graph. The chapter ends with upserts in Dgraph.

## Table of Contents
---
- [Twitter data](#Twitter-data)
- [Data Modeling](#Data-Modeling)
- [Schema and indices](#Schema-and-indices)
- [Upserts](#Upserts)
- [Building Flock](#Building-Flock)
- [Running Flock](#Running-Flock)

## Twitter data

Flock loads into Dgraph real tweet data from the Twitter stream using the official Twitter Developer API.

[Here is the gist](https://gist.github.com/hackintoshrao/365a6f1610e940999cb0ed162b82cc0d) containing sample JSON data of a tweet obtained from Twitter API.

---

## Data Modeling

Node and Edges are the fundamental units of storing data in the Dgraph.

Here are the steps to model your application on Dgraph,
- Identify the different objects or entities in your application data.
- Identify the relationships between them.
- Draw the application graph with,
  - Nodes representing the entities/objects.
  - Edges between the nodes to represent their relationship.
- Associate properties with the nodes.
- Create a Dgraph schema and index.

---

Let's go through these steps in detail,

**Step 1:** Identify the different objects or entities in your application data.

From the Twitter JSON data, we can see two different entities in the data: `Tweets` and `Users`.

These would be the nodes of the graph.

![Graph-1](./assets/graph-1.png)

---

**Step 2:** Identify the relationships between them.

Here's a tweet. Let's identify it's the relationship with a Twitter user.

![Graph-2](./assets/graph-2.jpg)

As you can see, a Twitter `User` could be related to a `Tweet` in two ways:

-  An `User` would be `Author` of a `Tweet`. In other words, a `Tweet` would have an `Author`.

-  One or more `Users` could be mentioned in a `Tweet` .

Here, we'll call the relationship between `Tweets` and `Users` as `Author` and `Mention` edges.

---

**Step 3:** Draw the application graph with Nodes and Edges

![Graph-3](./assets/graph-3.png)

---

**Step 3:** Associate properties with the nodes.

Let's revisit the fields obtained from the tweet and associate them with either of the nodes.

Let's identify the fields that would belong to a tweet,

![Graph-4](./assets/graph-4.JPG)

Let's identify the fields that would belong to a User,

![Graph-5](./assets/graph-5.JPG)

Le's add these to the graph,

![Graph-6](./assets/graph-6.png)

Though conceptually it's easy to visualize the graph with types (Tweets, Users...),
Dgraph doesn't support it yet.

However, the good news is that the type system on nodes will be introduced in Dgraph 1.1.
Here is the preview https://docs.dgraph.io/master/query-language/#type-system.

---

Here are questions one could have during the schema design process,

Q: Is it necessary to normalize the info in a tweet into a `Tweet` and `User` node?

A: Though it's not necessary, if not done, the model would be highly denormalized. The author's info would be duplicated in every tweet. So this hampers,

- Updates and deletes.
Let's say. The user wants to change the profile picture. With the denormalized approach, the new `profile_picture_url` update has to be propagated to all the tweets of the user.

-  Ability to discover insights
Graph databases are distinct because of their ability to traverse the relationships between
Nodes and discover exciting facts about the data. This is possible only when different
entities are separated as nodes and are connected by edges representing the relationship
between them.

---

## Schema and indices

Schema and indices in Dgraph are flexible. These can be altered as your application needs to evolve.

 It's not compulsory to add all the predicates into the schema. Indices are used
to enhance query performance (not mutations). In the next chapter, we'll learn about adding
an index to schema based on the query requirements.


However, the predicates those require `upserts` into the schema.


From the model, we saw that there are two edges.
- `Author`
- `Mention`

Edges are represented by `uid` keyword in the schema. Let's these entries to the schema using Ratel. Edges are also called as `uid predicates`.

```
mention: uid .
author: uid .
```


---

## Upserts

Dgraph doesn't support primary keys. The uniqueness of the value of a predicate has to ensured manually with the help of upserts.

In Flock, the Twitter handle of users has to be unique. Having more than a node for a User with a given
twitter_handle would be invalid.

Let's first list the predicates which need a  uniqueness constraint.

The `user_id` of Author nodes and `id_str` of a Tweet has to be unique.

Here are the steps to achieve uniqueness constraint for the predicates mentioned above:

**Step 1:** Add the `@upsert` directive for the predicates in the schema:

```sh

user_id: string @upsert .
id_str:  string @upsert .

```

The steps below have to be executed in a single transaction,

**Step 2:** Run a query to check whether a node with the given `user_id` exists.

**Step 3:** Create the new node only if the node doesn't exist.

Again, the `@upsert` doesn't ensure the uniqueness of the fields. It only helps ensure the safety of the two-step transaction by also checking the index values as part of the transaction conflict detection.

Refer to these docs to know more about upserts:

- https://docs.dgraph.io/query-language/#upsert-directive
- https://docs.dgraph.io/howto/#upserts

The new upsert block which would be introduced in v1.1 version release would help you achieve
upsert in one step, [click here](https://docs.dgraph.io/master/mutations/#upsert-block)
of a preview of the new feature.

---

## Building Flock

We use the following algorithm to build Flock:

- Fetch data from Twitter API
For every tweet fetched,
 - Organize the data based on the fields required for nodes.
 - Check whether the `Tweet` or the `User` node already exists using upserts.
 -  Create the nodes if they don't exist.
This applies to `Users` mentioned in the tweet too.
 - Create the `Author` and `Mention` edges from the `Tweet` node to the `User` node. It can be achieved in one mutation call.

---

## Running Flock
---

The setup instructions are described in [Flock's repo](https://github.com/dgraph-io/flock). Follow the instructions to get Flock up and to run.

We are using Docker Compose to run Flock.

---
