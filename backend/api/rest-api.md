# Understanding REST APIs

REST (Representational State Transfer) is an architectural style for designing networked applications. It relies on stateless, client-server communication over HTTP using standard methods and status codes. RESTful APIs are designed around resources, which can be anything from users and products to documents.

---

## Key Concepts

1. **Resources:** Everything that can be accessed via a RESTful API is considered a "resource." Each resource has a unique identifier (URI).

2. **Representations:** Resources are transferred in some representation like JSON or XML.

3. **Stateless Communication:** Each request from client to server contains all needed information; the server does not store any state about the client session.

4. **HTTP Methods:** REST APIs use standard HTTP methods to perform actions on resources.

5. **HTTP Status Codes:** Servers use HTTP status codes to indicate the outcome of a client's request.

---

# HTTP Methods

HTTP methods define the action you want to perform on a resource.

| Method | Description                                                                                                 | Idempotent? |
| ------ | ----------------------------------------------------------------------------------------------------------- | ----------- |
| GET    | Retrieves a resource or a list of resources. Should not modify data on the server.                          | Yes         |
| POST   | Creates a new resource. The request body contains the data for the new resource.                            | No          |
| PUT    | Updates a resource by replacing it with new data. Requires complete new representation in the request body. | Yes         |
| PATCH  | Updates a resource by partially modifying it. Requires only the modified fields in the request body.        | Yes         |
| DELETE | Deletes a resource.                                                                                         | Yes         |

---

## Important Notes on Methods

* **Idempotence:** An idempotent method produces the same result if called once or multiple times with the same request (e.g., GET, PUT, PATCH, DELETE). POST is generally not considered idempotent.

* **Safe methods:** A safe method should not modify server data (GET, HEAD, OPTIONS).

---

# HTTP Status Codes

Status codes are three-digit numbers the server uses to indicate the outcome of a client's request.

## Categories

| Range | Meaning                                                                   |
| ----- | ------------------------------------------------------------------------- |
| 1xx   | Informational (The request was received, continuing process)              |
| 2xx   | Success (The request was successfully received, understood, and accepted) |
| 3xx   | Redirection (Further action needs to be taken by the client)              |
| 4xx   | Client Error (Invalid request or cannot be fulfilled)                     |
| 5xx   | Server Error (Server failed to fulfill request)                           |

---

## Common Status Codes

| Code | Category     | Description                                                               | Use Case                  |
| ---- | ------------ | ------------------------------------------------------------------------- | ------------------------- |
| 200  | Success      | OK: The request was successful.                                           | A successful GET request  |
| 201  | Success      | Created: A new resource has been created.                                 | A successful POST request |
| 204  | Success      | No Content: The request was successful, but there's no content to return. | DELETE success            |
| 301  | Redirection  | Moved Permanently: The resource has moved to a new URL.                   | Redirect old URL          |
| 302  | Redirection  | Found: Temporary redirect.                                                | Temporary redirect        |
| 304  | Redirection  | Not Modified: Resource hasn't changed.                                    | Conditional GET           |
| 400  | Client Error | Bad Request: Invalid request.                                             | Missing/invalid data      |
| 401  | Client Error | Unauthorized: Not authenticated.                                          | Login required            |
| 403  | Client Error | Forbidden: No permission.                                                 | Access denied             |
| 404  | Client Error | Not Found: Resource not found.                                            | Invalid URL               |
| 405  | Client Error | Method Not Allowed                                                        | Wrong HTTP method         |
| 409  | Client Error | Conflict                                                                  | Duplicate or conflict     |
| 422  | Client Error | Unprocessable Entity                                                      | Validation error          |
| 500  | Server Error | Internal Server Error                                                     | Backend issue             |
| 501  | Server Error | Not Implemented                                                           | Feature not supported     |
| 503  | Server Error | Service Unavailable                                                       | Server down               |

---

# RESTful API Design Best Practices

* Use nouns to represent resources (e.g., /users, /products).
* Use plural nouns for collections (e.g., /users).
* Use HTTP methods according to their semantics.
* Use status codes appropriately.
* Keep APIs consistent and predictable.
* Design stateless APIs.

---

# Interview Questions and Answers

### Q1: What is a REST API?

A: REST (Representational State Transfer) is an architectural style for designing networked applications. It relies on stateless client-server communication using standard HTTP methods and status codes. RESTful APIs are designed around resources (like data), which can be accessed using methods like GET, POST, PUT, PATCH, DELETE.

---

### Q2: What are the common HTTP methods used in REST APIs?

A:

* GET: Retrieve data
* POST: Create data
* PUT: Replace data
* PATCH: Partial update
* DELETE: Delete data

---

### Q3: Difference between PUT and PATCH?

A:

* PUT: Completely replaces a resource
* PATCH: Partially updates a resource

---

### Q4: What are HTTP status codes?

A: HTTP status codes are three-digit numbers sent by the server to indicate the outcome of a request. They help in understanding success or failure.

---

### Q5: Examples of status codes?

A:

* 200 OK
* 201 Created
* 400 Bad Request
* 401 Unauthorized
* 403 Forbidden
* 404 Not Found
* 500 Internal Server Error

---

### Q6: What is idempotent method?

A: An idempotent method produces the same result no matter how many times it is called. GET, PUT, PATCH, DELETE are idempotent. POST is not.

---

### Q7: What are safe methods?

A: Safe methods do not modify server data. Examples: GET, HEAD, OPTIONS.

---

### Q8: Should API return error codes?

A: Yes. Always return proper status codes:

* 400 → invalid data
* 404 → resource not found
* 500 → server error

---

### Q9: Best practices for REST API?

A:

* Use nouns
* Use proper HTTP methods
* Use status codes
* Keep API consistent
* Make it stateless

---

### Q10: How to handle errors?

A:

* Use proper status codes
* Return descriptive error message (JSON)
* Log errors on server

---
