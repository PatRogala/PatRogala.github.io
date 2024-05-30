---
title: "Implementing Basic Auth in Node.js"
date: 2024-05-28 12:16:00 +0000
categories: [Backend Topics, API]
tags: [basic authentication, node.js, web app, security, implementation, tutorial, guide, nodejs, express, lms, learning management system, http, browser, credentials, password, username, base64, encode, decode, www-authenticate, authorization header]
---

**Authentication is the gatekeeper to your web application's sensitive data.** Without proper authentication measures, anyone could potentially access and misuse your application's core functionality. Among the various authentication methods available, Basic Authentication stands out for its simplicity and widespread browser support. However, implementing it correctly is crucial to ensure the security of your application and its users' data. In this article, we'll explore the fundamentals of Basic Authentication and guide you through its implementation in Node.js, specifically for a Learning Management System (LMS) web app.

**Use Basic Auth only for something like staging environment**

## What is Basic Authentication?

Basic Authentication is a part of the HTTP specification (RFC 7617) and is natively supported by all major web browsers. When you visit a protected page, the browser displays a login popup asking for credentials (username and password).

The process works like this:

1. The user tries to access a protected page.
2. The server sends a `401 Unauthorized` response with a `WWW-Authenticate` header set to basic.
3. The browser shows the login popup.
4. After submitting the credentials, the browser encodes them using Base64 and sends them in the `Authorization` header.
5. The server decodes the header, verifies the credentials, and grants access if valid.

### Security Considerations
**Warning:** Basic Authentication sends credentials in plain text, so anyone intercepting the request can decode them. Always use HTTPS when implementing Basic Authentication.

### Implementation in Node.js
Let's create a simple Express.js server with Basic Authentication for an LMS platform.

```js
const express = require('express');
const app = express();

// Middleware to handle Basic Authentication
const basicAuth = require('./basicAuth');
app.use(basicAuth);

// Protected route
app.get('/courses', (req, res) => {
  res.send('Welcome to the LMS Course List!');
});

app.listen(3000, () => {
  console.log('Server started on port 3000');
});
```

In the `basicAuth` middleware, we'll check the `Authorization` header and verify the credentials.

```js
const credentialsAreValid = (username, password) => {
  // Replace with your authentication logic
  return username === 'admin' && password === 'password';
};

const basicAuth = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader) {
    // No Authorization header, send 401 Unauthorized
    res.setHeader('WWW-Authenticate', 'Basic realm="LMS"');
    return res.status(401).send('Authentication required');
  }

  const [authType, encodedCredentials] = authHeader.split(' ');

  if (authType !== 'Basic') {
    // Invalid auth type, send 401 Unauthorized
    res.setHeader('WWW-Authenticate', 'Basic realm="LMS"');
    return res.status(401).send('Invalid authentication type');
  }

  const decodedCredentials = Buffer.from(encodedCredentials, 'base64').toString().split(':');
  const [username, password] = decodedCredentials;

  if (!credentialsAreValid(username, password)) {
    // Invalid credentials, send 401 Unauthorized
    res.setHeader('WWW-Authenticate', 'Basic realm="LMS"');
    return res.status(401).send('Invalid credentials');
  }

  // Valid credentials, proceed to the next middleware
  next();
};

module.exports = basicAuth;
```

In this example, we check if the `Authorization` header exists and starts with `Basic`. If not, we send a `401 Unauthorized` response with the `WWW-Authenticate` header set to `Basic realm="LMS"`.

If the header is present, we decode the Base64-encoded credentials and verify them using the `credentialsAreValid` function (replace this with your authentication logic).

If the credentials are valid, we proceed to the next middleware. Otherwise, we send a `401 Unauthorized` response.

With this implementation, when a user tries to access the `/courses route`, the browser will prompt for a username and password if they haven't authenticated yet.

Remember, Basic Authentication should only be used over HTTPS to prevent credentials from being intercepted in plain text.

## TL;DR

Basic Authentication is part of the HTTP spec and natively supported by browsers. It involves sending a `401 Unauthorized` response with a `WWW-Authenticate: Basic` header to trigger a login popup. After submitting credentials, the browser encodes them in Base64 and sends them in the `Authorization` header. The server decodes and verifies the credentials, granting access if valid. However, Basic Authentication sends credentials as plain text, so always use HTTPS to prevent interception.
