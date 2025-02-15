---
layout: post
title: "System design - Twitter"
date: 2025-02-13
description: "System design"
tag: System Design
---

# System design - Twitter

## [common features](https://dev.to/karanpratapsingh/system-design-twitter-865)

1. Post Tweet (support media file - image, vedio, etc.)

2. User follow each other

3. homefeed (timeline)

4. search

5. favorite (favorite posts)

6. Retweet

7. metrics & analytic

## Traffic estimation & storage esitmation

Suppose 200 million DAU, with average 5 posts per day.

We have:

$$
200\;million * 5 = 1\;billion\;posts / day
$$

Say 10 percent of posts contains media file

We have:

$$
200 \; million * 5 * 10\% = 100 \; million \; media / day
$$

So the **RPS** would be

$$
{ 1\; billion \over 86400\;s } \approx 12k/sec
$$

---

For Storage, let's assume each post message size is 100 Bytes, then

$$
1\;billion\;*\;100\;bytes\;=\;100\;GB\;/\;day
$$

For media files, assume average size is 10k, then

$$
10\;k\;*\;100\;million\;=\;1\;TB\;/\;day
$$

For 5 years, it requires

$$
1.1\;TB\;*\;365\;days\;*\;5\;years\;=\;2\;PB
$$

---

For band width:

$$
{1\;TB\;\over\;86400\;s}\;\approx\;12\;mb\;/\;s
$$

## Api Design

1. Post Tweet (support media file - image, vedio, etc.)

```
    postTweet(userId, content, medialinks[]) --> boolean success or not
    parameters:
        userId: UUID, user who want to post tweet
        content: String, tweet content
        medialinks[]: String[], list of medialinks (Optional)
    return:
        boolean: success or not
```

2. User follow each other

```
    follow(followerId, followeeId) --> boolean success or not
    unFollow(followerId, followeeId) --> boolean success or not
    parameters:
        followeeId: UUID, from user
        followerId: UUID, to user
    return:
        boolean: success or not
```

3. homefeed (timeline)

```
    generateHomeFeed(userId) --> tweet[]
    para:
        userId: UUID
    return:
        tweet[]: list of tweet should be shown for homefeed
```

4. search

```
    search(content, type) --> tweet[] | user[]
```

5. favorite (favorite posts)

```
    favorite(userId, tweetId)
```

6. Retweet

```
    reTweet(userId, tweetId) --> boolean success
```

7. metrics & analytic

## Modeling

<img src='https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fraw.githubusercontent.com%2Fkaranpratapsingh%2Fportfolio%2Fmaster%2Fpublic%2Fstatic%2Fcourses%2Fsystem-design%2Fchapter-V%2Ftwitter%2Ftwitter-datamodel.png'>

## High Level Design

<img src='https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fraw.githubusercontent.com%2Fkaranpratapsingh%2Fportfolio%2Fmaster%2Fpublic%2Fstatic%2Fcourses%2Fsystem-design%2Fchapter-V%2Ftwitter%2Ftwitter-advanced-design.png'>

`User Service`

This service handles user-related concerns such as authentication and user information.

`Newsfeed Service`

This service will handle the generation and publishing of user newsfeeds. It will be discussed in detail separately.

`Tweet Service`

The tweet service will handle tweet-related use cases such as posting a tweet, favorites, etc.

`Search Service`

The service is responsible for handling search-related functionality. It will be discussed in detail separately.

`Media service`

This service will handle the media (images, videos, files, etc.) uploads. It will be discussed in detail separately.

`Notification Service`

This service will simply send push notifications to the users.

`Analytics Service`

This service will be used for metrics and analytics use cases.

### Features implementation

`HomeFeed Generation`

There are three models we can use for homefeed generation

1. Pull model

   When post a tweet, write to DB once. When userA generate home feed, userA will pull the tweets of users subsribed by userA

    <img src = 'https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fraw.githubusercontent.com%2Fkaranpratapsingh%2Fportfolio%2Fmaster%2Fpublic%2Fstatic%2Fcourses%2Fsystem-design%2Fchapter-V%2Ftwitter%2Fnewsfeed-pull-model.png'>

   - advantage:
     1. write load is good
   - disadvantage:
     1. read load is heave as each time, first got all followed users, then fetch the tweets.

2. Push model

   When post a tweet, fan out to users' pre-generated homefeed. follower can fetch the tweet from pre-generated homefeed directly to prevent heavy read load on database or cache.

    <img src='https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fraw.githubusercontent.com%2Fkaranpratapsingh%2Fportfolio%2Fmaster%2Fpublic%2Fstatic%2Fcourses%2Fsystem-design%2Fchapter-V%2Ftwitter%2Fnewsfeed-push-model.png'>

   - advantage:
     1. write load is heavy, not good for those celebrities.
   - disadvantage:
     1. read load is good

3. Hybrid model

   For celebrities, use pull model to reduce the write load

   For normal user, use push model to reduce the read load

The Hybrid model is more appropriate here as it balance the read and write load.

---

`Ranking Algorithm` (improve - machine learning)

As we discussed, we will need a ranking algorithm to rank each tweet according to its relevance to each specific user.

For example, Facebook used to utilize an EdgeRank algorithm, here, the rank of each feed item is described by:

$$
Rank\;=\;Affinity\;×\;Weight\;×\;Decay
$$

Where,
`Affinity`: is the "closeness" of the user to the creator of the edge. If a user frequently likes, comments, or messages the edge creator, then the value of affinity will be higher, resulting in a higher rank for the post.

`Weight`: is the value assigned according to each edge. A comment can have a higher weightage than likes, and thus a post with more comments is more likely to get a higher rank.

`Decay`: is the measure of the creation of the edge. The older the edge, the lesser will be the value of decay and eventually the rank.

---

**`Retweets`**

Retweets are one of our extended requirements. To implement this feature we can simply create a new tweet with the user id of the user retweeting the original tweet and then modify the type enum and content property of the new tweet to link it with the original tweet.

For example, the type enum property can be of type tweet, similar to text, video, etc and content can be the id of the original tweet. Here the first row indicates the original tweet while the second row is how we can represent a retweet.

| id                  | userID              | type  | content                      | createdAt     |
| ------------------- | ------------------- | ----- | ---------------------------- | ------------- |
| ad34-291a-45f6-b36c | 7a2c-62c4-4dc8-b1bb | text  | Hey, this is my first tweet… | 1658905644054 |
| f064-49ad-9aa2-84a6 | 6aa2-2bc9-4331-879f | tweet | ad34-291a-45f6-b36c          | 1658906165427 |

This is a very basic implementation, to improve this we can create a separate table itself to store retweets.

---

`Notification`

use FCM or APNS

`Mutual friends` (improve - machine learning)

For mutual friends, we can build a social graph for every user. Each node in the graph will represent a user and a directional edge will represent followers and followees. After that, we can traverse the followers of a user to find and suggest a mutual friend.

---
