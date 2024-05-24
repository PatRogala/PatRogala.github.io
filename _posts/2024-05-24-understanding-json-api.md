---
title: "Understanding JSON API: A Simple Explanation"
date: 2024-05-24 18:26:00 +0000
categories: [Backend Topics, API]
tags: [json, api, specification, client, server, fetch, resource, response, data, type, id, attributes, relationships, include, fields, create, update, delete, request, lms, web, app, courses, instructors, programming, web-development, html, css, javascript, efficiency, overhead]
---

## Understanding JSON API

JSON API is a specification that defines how clients can request resources and how servers should respond.

### Fetching Resources
Let's say we want to fetch a list of courses from our LMS web app. We send a GET request to `/courses`, and the server responds with:

```json
{
  "data": [
    {
      "type": "courses",
      "id": "1",
      "attributes": {
        "title": "Introduction to Programming",
        "description": "Learn the basics of programming.",
        "startDate": "2023-09-01"
      },
      "relationships": {
        "instructor": {
          "data": {
            "type": "instructors",
            "id": "23"
          }
        }
      }
    }
  ]
}
```

Here, we see the `type` is `courses`, and we have the `id`, `attributes` (title, description, start date), and a `relationships` object that links to the course instructor.

To get the instructor's name, we include the `instructor` relationship:

```
GET /courses?include=instructor
```

The server will now include the instructor data in the `included` section of the response.

We can also specify which fields we want for the instructor:

```
GET /courses?include=instructor&fields[instructors]=name
```

Now, we'll only get the instructor's name.

### Creating Resources

To create a new course, we send a POST request to `/courses` with the course data:

```json
{
  "data": {
    "type": "courses",
    "attributes": {
      "title": "Web Development Fundamentals",
      "description": "Learn HTML, CSS, and JavaScript.",
      "startDate": "2023-10-01"
    },
    "relationships": {
      "instructor": {
        "data": {
          "type": "instructors",
          "id": "42"
        }
      }
    }
  }
}
```

The server will respond with the newly created course data, including the assigned `id`.

### Updating Resources

To update a course, we send a PATCH request to `/courses/:id` with the updated data:

```json
{
  "data": {
    "type": "courses",
    "id": "3",
    "attributes": {
      "title": "Advanced Web Development"
    }
  }
}
```

This will update the title of the course with `id` 3.

### Deleting Resources

To delete a course, we send a DELETE request to `/courses/:id`. No request body is needed.

```
DELETE /courses/3
```

This will delete the course with `id` 3.

JSON API simplifies working with related resources and allows clients to request only the data they need, improving efficiency and reducing overhead.
