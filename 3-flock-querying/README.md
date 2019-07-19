# Flock Querying

In this chapter, we'll be learning about querying in Dgraph. These include searching nodes, 
filtering, traversing, reverse traversing, date-time queries, and much more.

---

## Find hashtags from the nodes
```sh
{
    hashtag_query(func: has(hashtags), first: 10, offset: 30) {
        hashtags
    }
}
```
![Query-1](./assets/1.png)

---

```sh
{
    dataquery(func: eq(hashtags, "SecretOfHappyLife")) {
        uid
        id_str
        retweet
        message
        hashtags
    }
}
```


![Query-2](./assets/2.png)

---

```sh
{
  dataquery(func: has(screen_name), first: 10, offset: 40) {
    screen_name
  }
}
```

![Query-3](./assets/3.png)


---


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

![Query-4](./assets/4.png)

---

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
```sh
{
  dataquery(func: has(user_id), first: 3, offset: 10) {
    user_id
  }
}
```
![Query-8](./assets/8.png)

---
```sh
{
  dataquery(func: eq(user_id, "1138630994339123201")) {
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
![Query-9](./assets/9.png)

---

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

![Query-11](./assets/11.png)

---

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