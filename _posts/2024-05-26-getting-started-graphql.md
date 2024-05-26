---
title: "Getting Started with GraphQL"
date: 2024-05-26 09:39:00 +0000
categories: [Backend Topics, API]
tags: [graphql, query, language, overfetching, underfetching, data, control, flexibility, rest, api, web, applications, complex, efficient, performance, user, experience, solution, powerful, data deluge, manage]
---

In today's world, web applications are becoming increasingly complex, handling vast amounts of data from multiple sources. Traditional REST APIs, while effective, can sometimes fall short when it comes to managing this data deluge. Over-fetching and under-fetching are common issues that can lead to inefficient data transfer, slower load times, and a subpar user experience. But there is a solution: GraphQL, a powerful query language that puts you in control of your data fetching needs.

## Getting Started with GraphQL

GraphQL is a query language that provides an alternative to traditional REST APIs. It offers more flexibility and control over the data you fetch or mutate.

### Why Use GraphQL?

**Overfetching**

With REST APIs, you might fetch more data than you need. For example, an endpoint `/courses` could return a list of courses with unnecessary data like descriptions and video URLs if you only need the course ID, title, and thumbnail.

**Underfetching**

Conversely, you might not get all the data you need in a single request. You'd have to make multiple requests to different endpoints to gather the required information.

### How GraphQL Works

With GraphQL, you send queries to a single endpoint (e.g., `/graphql`). The query specifies exactly what data you need, preventing over- and underfetching.

For instance, to fetch course IDs, titles, and thumbnails, you could send a query like this:

```graphql
query {
  courses {
    id
    title
    thumbnailUrl
  }
}
```

To fetch a single course and its author's details, you can nest related data in the query:

```graphql
query {
  course(id: 1) {
    id
    title
    author {
      name
      otherCourses {
        id
        title
        thumbnailUrl
      }
    }
  }
}
```

This single query fetches the course details and the author's name and other courses, preventing underfetching.

GraphQL also supports mutations for adding, updating, or deleting data, similar to POST, PUT, and DELETE requests in REST APIs.

## TL;DR

GraphQL is a query language that lets you specify the exact data you need. This prevents overfetching (getting too much data) and underfetching (not getting enough data in a single request).

With GraphQL, you send queries to a single endpoint. The query defines the data you want, including nested related data. This way, you get just the right amount of data in one request.

GraphQL also supports mutations for adding, updating, or deleting data, like REST APIs.

