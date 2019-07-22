# Flock Querying

In this chapter, we'll be learning about querying in Dgraph. These include searching nodes, 
filtering, traversing, reverse traversing, date-time queries, and much more.

Dgraph's query language is called GraphQL+-. It's inspired by original GraphQL spec, but slightly modified. The modifications were necessary to overcome the shortcomings of GraphQL as a query language for a database. 

Let's recap the graph model of flock,

![Graph](./assets/graph-3.JPG)
---

 # Node selection 
 ## Query 1: 
 Search for hashtags
 
 One of the common operation in Dgraph is to select one or more node based on certain criteria. 

Dgraph's inbuilt functions like `has`, `ge`, `eq` help you express the creteria for selection of nodes in the query. 

In our first query we'll be selecting nodes based on existance of a predicate/property using the `has` function. 



```sh
{
    hashtag_query(func: has(hashtags), first: 10, offset: 30) {
        hashtags
    }
}
```

Here is structure of the query, 
![query-structure](./assets/query-structure-2.JPG)


The query request to "**Select all nodes which has hashtags in it**". In the result we request only for the `hashtags`, hence the other properties of the node will not be returned in the result. This is similar **select a, b,c** in SQL. 

In the animation below you could see that only the nodes with non-empty values for `hashtags` predicate/property is selected. 

![Query in action](http://play.minio.io/my-test/query-gif-1.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=Q3AM3UQ867SPQQA43P2F%2F20190722%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190722T042355Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=3bb18c3f523dc94b1ec767ed6c47b85aa3264b09d40ac12b276cc832b96cf613)


Let's execute the query in Ratel. From the result, click on any of the nodes and copy a `hashtag`. We'll be using this `hashtag` to run the next query. 
In the image below you could see that we have chosen the `hastag` "BOOM". 

![Query-1](./assets/query-1.png)


It's important to notice that the above query is different from "**Give me all the nodes/tweets with specific hashtag #xyz**".

---

## Query 2: 
Find tweets with a specific hashtag.

Let's query for nodes/tweets with hashtag "BOOM". You should the hashtag which has been copied from the last query. 

The structure of the query is same as the last one. But, instead of `has` function, we'll be using the `eq` function. 


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

In this case, we are querying for a value of a predicate. This needs index to be set on `hashtags` to speed up the operation. 

Let's add the follow schema using Ratel's schema editor. 

```sh
hashtags: [string] @index(exact)
```

The [Getting started guide](../1-getting-started/Readme.md) has details about adding schema using Ratel.

Let's execute the query using Ratel, 


![Query 2](./assets/query-2.png)

---

## Query 3: 
Find authors of tweets of a given hashtag.

Let's extend the last query to find authors of the tweets with the hashtag "BOOM".

The root level query finds the tweets based on the the hashtag. The first level nested query does the graph traversal along the author edge. This gives us the final result. 



![query-structure](./assets/query-structure-3.JPG)

Every level of nested query further traverses the graph. Each level will be using the nodes selected in its previous level as the starting point.

![Query-animation](http://play.minio.io/my-test/query-gif-2.GIF?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=Q3AM3UQ867SPQQA43P2F%2F20190722%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190722T143319Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=b688b8406b36c2e656b87b318360cbbb46b5943be90264de0ca1c12762ead514)

---


## Query 4: 
Find all nodes/users with a user_id predicate.

This query is similar to Query 1. 


```sh
{
  dataquery(func: has(user_id), first: 10, offset: 40) {
    screen_name
  }
}

```

Let's execute the query and we should obtain the screen_names of users. Since we have set the `first` parameter to 10, the result should have maximum of 10 nodes. 



![Query 3](./assets/3.png)


Copy one of the `user_id` from the result.

---

## Query 5: 
Find the `Tweets of a user`. 

Let's use the `user_id` we just obtained and fetch all the tweets of the User. 

We need to modify the schema to add a `string` index to `user_id`. In the [last chapter](../2-flock-mutations/Readme.md) we added the `@upsert` directive to the `user_id` field. 

Add the following modification to the `user_id` field in the schema using Ratel, 

```sh
user_id: string @index(exact) @upsert .
```

Here is the query, 

```sh
{
  dataquery(func: eq(screen_name, "mzamra")) {
    uid
    screen_name
    user_id
    user_name
    profile_banner_url
    profile_image_url
    friends_count
    description
  }
}
```

Let's run this on Ratel.

![Query-4](./assets/4.png)

---

## Query 6: 
Find users with highest number of mentions. 

```sh
{
  var(func: has(<~mention>)) {
    ~mention @groupby(mention) {
      a as count(uid)
    }
  }
  dataquery(func: uid(a), orderdesc: val(a), first: 10, offset: 0) {
    uid
    screen_name
    user_id
    user_name
    profile_banner_url
    profile_image_url
    friends_count
    description
    total_mentions : val(a)
  }
}
```

![Query-6](./assets/6.png)

---

## Query 7: 
Find users with highest number tweets.

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
Find tweets which are created after given time.

```sh
{
  dataquery(func:has(hashtags), first: 3, offset: 0)
  @filter(ge(created_at, "2019-06-28T19:03:12Z")) {
    hashtags
    created_at
  }
}

```

---

## Query 9: 
Find tweets of user. Tweets are filtered by their date of creation.

```sh
{
  dataquery(func: has(screen_name), first: 3, offset: 0) @cascade {
    screen_name
    ~author @filter(ge(created_at, "2019-07-02T19:03:12Z")) {
      created_at
    }
  }
}
```
![Query-10](./assets/10.png)


---

## Query 10: 
Find authors ordered by number of tweets. Also fetch their tweets filtered by their date of creation. 


```sh
{
  var(func: has(user_id)) {
    a as count(~author) @filter(ge(created_at, "2019-04-28T19:03:12Z"))
  }
  dataquery(func: uid(a), orderdesc: val(a), first: 3, offset: 0) @cascade {
    uid
    screen_name
    user_id
    user_name
    profile_banner_url
    profile_image_url
    friends_count
    description
    total_tweets : val(a)
    ~author @filter(ge(created_at, "2019-04-28T19:03:12Z")) {
      created_at
    }
  }
}


```