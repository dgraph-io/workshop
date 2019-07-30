# Querying in Flock
In this chapter, we'll be learning about querying in Dgraph. These include searching nodes, 
filtering, traversing, reverse traversing, date-time queries, and much more.

Dgraph's query language is called GraphQL+-. It's inspired by original GraphQL spec but slightly modified. The modifications were necessary to overcome the shortcomings of GraphQL as a query language for a database. 

Let's recap the graph model of the Flock,

![Graph](./assets/graph-3.JPG)

---

 ## Query 1: 
 **Search for hashtags**

 One of the standard operation in Dgraph is to select one or more node based on specific criteria. 

Dgraph's inbuilt functions like `has`, `ge`, `eq` help you express the criteria for the selection of nodes in the query. 

In our first query, we'll be selecting nodes based on the existence of a predicate/property using the `has` function. 


```sh
{
    hashtag_query(func: has(hashtags), first: 10, offset: 30) {
        hashtags
    }
}
```

Here is structure of the query, 
![query-structure](./assets/query-structure-2.JPG)


The query request to "**Select all nodes which have hashtags in it**". In a result, we request only for the `hashtags`. Hence the other predicates of the node will not be returned. This is similar **select a, b,c** in SQL. 

In the animation below, you could see that only the nodes with non-empty values for `hashtags` predicate/property are selected. 

![Query in action](http://play.minio.io/my-test/query-gif-1.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=Q3AM3UQ867SPQQA43P2F%2F20190729%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190729T204012Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=33c64ecf90c61423ad99bd48b7feb9eb2c962d6b088c39453c6ac15d12aaa637)


Let's execute the query in Ratel. From the result, click on any of the nodes and copy a `hashtag`. We'll be using this `hashtag` to run the next query. 
In the image below, you could see that we have chosen the hashtag `BOOM`. 

![Query-1](./assets/query-1.png)


It's essential to notice that the above query is different from "**Give me all the nodes/tweets with specific hashtag #xyz**".

---

## Query 2: 

We can also use filtering to add criteria for node selection. We can filter based on any predicate/properties of a node. Indexes have to be added to help improve the performance of these queries.

In the following query, let us filter the tweets based on their date and time of creation. Let us create a date-time index for the `created_at` predicate. 

```sh
created_at: dateTime @index(hour) .
```

The [Getting started guide](../1-getting-started/Readme.md) has details about adding schema using Ratel.

Here's the [reference to docs](https://docs.dgraph.io/query-language/#datetime-indices) to know more about DateTime indexes in Dgraph.

```sh
{
  dataquery(func:has(hashtags), first: 3, offset: 0)
@filter(ge(created_at, "2019-06-28T19:03:12Z")) {
    hashtags
    created_at
  }
}

```
Here's the [link to docs](https://docs.dgraph.io/query-language/#applying-filters) to know more about filtering. 


---

## Query 3: 
**Find tweets with a specific hashtag**

Let's query for nodes/tweets with hashtag "BOOM". You should use the hashtag which has been copied from the last query. 

The structure of the query is the same as the last one. However, instead of `has` function, we'll be using the `eq` function. 


```sh
{
    dataquery(func: eq(hashtags, "BOOM")) {
        uid
        id_str
        retweet
        message
        hashtags
    }
}
```

In this case, we are querying for the value of a predicate. This needs index to be set on `hashtags` to speed up the operation. 

Let's add the following schema using Ratel's schema editor. 

```sh
hashtags: [string] @index(exact)
```

Let's execute the query using Ratel, 


![Query 2](./assets/query-2.png)

---

## Query 4: 
**Find authors of tweets of a given hashtag**

Let's extend the last query to find authors of the tweets with the hashtag "BOOM".

The root-level query finds the tweets based on the hashtag. The first level nested query does the graph traversal along the author's edge. This gives us the final result. 

```sh
{
    dataquery(func: eq(hashtags, "BOOM")) {
        uid
        id_str
        retweet
        message
        hashtags
    }
}
```

Here is the structure of the query. 

![query-structure](./assets/query-structure-3.JPG)

Every level of the nested query further traverses the graph. Each level will be using the nodes selected at its previous level as the starting point.


![Query-animation](http://play.minio.io/my-test/query-gif-2.GIF?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=Q3AM3UQ867SPQQA43P2F%2F20190729%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190729T211826Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=0aee78370db4ba7b74d0151a95bcce23d464c841614685349f27baa29698e12e)

Run the query using Ratel.

---

## Note on the direction of the edges

Dgraph doesn't enforce direction to an edge. From our modeling, the `author` edge is supposed to point to a `User` node from a `Tweet` node. However, it would work the other way around too. The onus is on the application to enforce this. 

In Flock, we've ensured that all the `author` edges point from a `tweet` node to a `user` node. 
This makes it easier to select a `tweet` and traverse along the `author` edge to select the `users`. 

But, how about the other way around? Can we start from `author` node and traverse along the `author` edge in the `reverse` direction to reach their `tweets`? 

That's possible! It requires `@reverse` directive to be added to the schema, 

```sh
author: uid @reverse .`
```

In the next couple of queries, let us start with the author node and do a reverse edge traversal to find their tweets.

---

## Query 5: 
**Find all nodes/users with a screen_name predicate**

This query is similar to Query 1. 

```sh
{
  dataquery(func: has(screen_name), first: 10, offset: 40) {
    screen_name
  }
}

```

Let's execute the query, and we should obtain the screen_names of users. Since we have set the `first` parameter to 10, the result should have a maximum of 10 nodes. 


Copy one of the `screen_names` from the result. We'll be using it in our next query.

---

## Query 6: 
**Find the Tweets of a user.**

Let's use the `screen_name` we just obtained and fetch all the tweets of the User.  

We need to modify the schema to add a `string` index to `screen_name`.

Add the following modification to the schema using Ratel, 

```sh
screen_name: string @index(term) @upsert .
```

If you notice, we have used the term index for `screen_name`. A screen name on twitter could have multiple terms. Like `Francesc Campoy`, `Karthic Rao`, `Daniel Mai`! The `term` index lets one query for any term in the name. You could just search for `Francesc` and obtain all the users who have the term in their screen names. 


Here is the query, 

```sh
{
  dataquery(func: eq(screen_name, "DraiDrai12")) {
    uid
    screen_name
    user_id
    user_name
    profile_banner_url
    profile_image_url
    friends_count
    description
    ~author {
      uid
      message
    }
  }
}
```

As mentioned earlier, the `author` edge points from a `tweet` to a `user`. Hence, if you start a `user` node, the `author` edge has to be traversed in a reverse direction to obtain their tweets.

Using tilde (~) prefix will traverse an edge in reverse direction. 

![query](./assets/5.jpg)

Let's run this query on Ratel.

![Query-4](./assets/4.png)

Note: If you run Flock for a short duration, the user_id you've selected may not have tweets. They've existed because of the `mentions`. Use the next query to find  `user_ids` or `screen_names` with a lot of tweets.

---


## Query 7: 
Find users with the highest number of tweets. 

We'll be using the variable, count, and sort functionalities in this query.

Nodes matched at in one block of a query can be stored in a variable and used elsewhere, either in a child block or a different query block. 

Here's the link to [docs on variable blocks](https://docs.dgraph.io/query-language/#var-blocks).

We already know from our model that the `author` edge points from a `Tweet` node to an `User` node. Which means the `~author` would traverse from a `User` node to their tweets. 


The following query computes the number of `author` edge for each of the `User` nodes selected via the `has` function. What do you think the result would be?


```sh
{
  count_edges(func: has(hashtags)) {
    edge_count: count(author)
  }
}
```


![query-4](./assets/query-4.png)

You guys guessed it right. Flock by design and implementation ensure that every `tweet` would have only one author.
Hence, the result is always 1.

---

How about the query below?

```sh
{
  count_edges(func: has(user_id)) {
    edge_count: count(author)
  }
}
```
![Query-5](./assets/query-5.png)

### The count is 0! But Why!!!? 
The query above counts the number of `author` edge emerging out of `User` nodes. Again, Flock by its design and implementation ensure that all `author` edge points out of `Tweet` nodes and towards the `User` nodes. None of them emerge out of `User` nodes, and they point towards them. 

For each of the `User` node, if we could find the number of `author` edge pointing towards them, it would give us the number of `tweets` by each `User`. Just simple tilde (~) in the above query would do the trick. 

```sh
{
  count_edges(func: has(user_id)) {
    edge_count: count(~author)
  }
}
```


---

Let's define and decipher the query 7. That is to find the users with the highest number of tweets. 


![query](./assets/query-structure-4.jpg)

Let's run the query using Ratel.

```
{
  var(func: has(user_id)) {
    a as count(~author)
  }
  dataquery(func: uid(a), orderdesc: val(a), first: 3, offset: 0) {
    uid
    screen_name
    user_id
    user_name
    profile_banner_url
    profile_image_url
    friends_count
    description
    total_tweets : val(a)
  }
}
```

![Query-7](./assets/7.png)

---

## Query 8: 
Similarly, we could find the users who are most mentioned. This time, let us reverse traverse the `mention` edge and also find the tweets they are mentioned in.

This requires `@reverse` directive added to the schema, 

```sh
mention: uid @reverse .
```

Here's the query

```
{
  var(func: has(user_id)) {
    a as count(~mention)
  }
  dataquery(func: uid(a), orderdesc: val(a), first: 3, offset: 0) {
    uid
    screen_name
    user_id
    user_name
    profile_banner_url
    profile_image_url
    friends_count
    description
    total_tweets : val(a)
    ~mention {
      message 
    }
  }

}
```

![query-6](./assets/query-6.png)

---

## Query 9

We could add filters during edge traversals in the nested block. Let's use [query 6](#query-6) for the example below. We have added a filter to select only the tweets of a user, which are created after `2019-03-02T19:03:12Z`. 

```sh
{
  dataquery(func: eq(screen_name, "DraiDrai12")) {
    uid
    screen_name
    user_id
    user_name
    profile_banner_url
    profile_image_url
    friends_count
    description
    ~author @filter(ge(created_at, "2019-03-02T19:03:12Z")) {
      uid
      message
      created_at
    }
  }
}
```

![query-7](./assets/query-7.png)

---




