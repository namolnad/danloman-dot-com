---
layout: post
title: Introducing jt. A JSON trimmer.
date: 2023-01-15
---

Today I'd like to introduce you to `jt` — a small command-line tool that allows you to take a JSON blob as input and return a subset of that JSON blob based on a simple schema argument. For example:

> ``` js
// example.json
{"body":{"key1":[1,2,3],"key2":{"key3":3,"key5":"string"},"key4":"value","key5":[]}}
```
Given the above file, we can run this simple command:
``` bash
cat example.json | jt '{body{key1,key2{key3},key4}}'
```
This will produce the following output:
``` js
{"body":{"key1":[1,2,3],"key2":{"key3":3},"key4":"value"}}
```
> `jt` will take the JSON input and transform it based on the schema provided, outputting only the selected keys and values.

## So… Why jt?

JSON responses from REST endpoints often become bloated over time, and in many cases, mobile clients only need a small portion of the response — a response which may have been originally intended for a desktop interface. `jt` was created to address this issue by allowing clients to select only the portion of a network response that they need. This has the benefit of reducing the amount of data sent over the wire.

While GraphQL is a powerful tool that can also accomplish this, this is not its end goal in and of itself, and choosing GraphQL comes with its own costs and tradeoffs. It requires a significant investment of time and resources to convert a project or company's backend and clients to GraphQL, it has a steep learning curve, and queries from the client can quickly become difficult to optimize when numerous endpoints are stitched together (sometimes creating N+1 query issues that need to be addressed.)

`jt` is designed to be a simpler, targeted tool for trimming JSON responses. The client can send a schema for a given request as a custom header, or this data can be persisted server-side and versioned alongside client updates. While similar json manipulation tools, such as jq, exist, `jt` is specifically designed for deep, nested filtering.

Please note that `jt` is a new tool and may overlook many complexities, corner cases, and perhaps even support of the full JSON spec. That said, I'd love for you to try the tool and share your thoughts and suggestions for improvements.
