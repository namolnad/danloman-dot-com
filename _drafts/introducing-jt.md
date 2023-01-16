---
layout: post
title: Introducing jt, a JSON trimmer.
date: 2023-01-15
---
# Introducing `jt` (JSONTrimmer)
Today I'm introducing a small tool I recently wrote called `jt`, short for JSONTrimmer. Like UNIX tools of old, `jt` is a very targeted tool in its functionality, focusing only on taking a JSON blob as input and returning a subset of that JSON blob as instructed via a simple schema argument. For example:

``` swift
// Given a file named example.json which contains the below:
// {"body":{"key1":[1,2,3],"key2":{"key3":3,"key5":"string"},"key4":"value","key5":[]}}
cat example.json | jt '{body{key1,key2{key3},key4}}'
// The below output is produced:
// {"body":{"key4":"value","key1":[1,2,3],"key2":{"key3":3}}}
```

## Why `jt`?
We've probably all seen a given REST endpoint's response gets bloated over time, and been in situations where mobile clients may consume only a small portion of the response which was originally intended for a desktop interface. To this end, the need for `jt` was inspired by many conversations discussing the benefits of various tools which allow a client to select only the portion of a network response that it cares about. There are obvious benefits to this — including simply sending less data over the wire. One highly-utilized example of this type of tooling is GraphQL. While I'll readily admit that GraphQL is great, and does far more than this tool sets out to do, it has to be recognized that it has its own tradeoffs. It's a massive undertaking to convert your project/company's backend and clients to GraphQL, it has a significant learning curve, and queries from the client can quickly become difficult to optimize when numerous endpoints are stitched together (sometimes creating N+1 query issues your team may need to scramble on.)

I kept finding myself wondering why there wasn't a tool which was designed exclusively to trim a JSON response to a subset of the original data — potentially to be inserted as a new middleware layer where it could modify your existing REST responses prior to returning them to the client. The client could send a schema for a given request as a custom header, or this data could be persisted server-side and could be versioned to coincide with client updates. While exploring some similar json manipulation tools to power this functionality (e.g. `jq`, an amazing tool which does far more than `jt`), I was unable to perform the deep, nested filtering which I felt would be required to perform this task.

This idea nagged at me enough where I decided to devote a portion of my MLK weekend to seeing if I could put together an initial version of this tool. Which leads us to `jt`'s initial launch today! It should of course be recognized that `jt` is young (roughly 1 day old at time of writing), likely overlooks many complexities/corner cases/full JSON spec, and can almost certainly be made far more efficient. However, I did not want that to stop me from getting it out into the world, seeing peoples' initial reactions, and hearing their suggestions for improvements. I'm excited to see what you all think!
