# Web Fundamentals — Class Notes
### Client-Server Architecture · HTTP Methods · Status Codes · JSON · REST Basics

---

## 1. Client-Server Architecture

### 1.1 Core Idea
Client-server architecture is a computing model where work is split between two roles:

| Role | Responsibility | Examples |
|------|----------------|----------|
| **Client** | Requests data/services, renders the result | Browser, mobile app, Postman |
| **Server** | Listens for requests, processes them, sends back a response | Web server, API server, database server |

The client **never talks directly to the server's internals** — it communicates only through a defined interface (usually an API over HTTP).

### 1.2 How a Request Flows

```
[Client]  --- HTTP Request --->  [Server]
   ^                                 |
   |                                 v
   +------- HTTP Response  <---------+
```

Step by step:
1. User opens browser and types `www.example.com`
2. Browser (client) resolves domain via **DNS** to an IP address
3. Client sends an **HTTP request** to the server at that IP
4. Server processes the request (may query a database)
5. Server sends back an **HTTP response** (HTML, JSON, etc.)
6. Client renders/uses the response

### 1.3 Key Characteristics
- **Statelessness**: Each request is independent; the server doesn't remember previous requests (unless sessions/tokens are used).
- **Separation of concerns**: Client handles presentation; server handles logic/data.
- **Many clients, one server (or cluster)**: A single server can handle thousands of client connections.
- **Request-response model**: Communication is always initiated by the client.

### 1.4 Real-World Example
- **Client**: Your phone's Instagram app
- **Server**: Instagram's backend API servers
- **Flow**: App requests your feed → server queries database → server returns JSON → app displays photos

---

## 2. HTTP (HyperText Transfer Protocol)

HTTP is the **protocol** (set of rules) that clients and servers use to communicate. It is:
- **Text-based** (human-readable)
- **Stateless**
- **Request-response** based

### 2.1 Anatomy of an HTTP Request

```
GET /api/users/42 HTTP/1.1
Host: example.com
Authorization: Bearer <token>
Accept: application/json

(optional body)
```

| Part | Meaning |
|------|---------|
| `GET` | HTTP Method |
| `/api/users/42` | Path/resource |
| `HTTP/1.1` | Protocol version |
| Headers | Metadata (auth, content type, etc.) |
| Body | Data sent (used in POST/PUT/PATCH) |

### 2.2 Anatomy of an HTTP Response

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Alice"
}
```

---

## 3. HTTP Methods (Verbs)

HTTP methods indicate **what action** the client wants to perform on a resource.

| Method | Purpose | Idempotent? | Has Body? | Example |
|--------|---------|:-----------:|:---------:|---------|
| **GET** | Retrieve a resource | ✅ Yes | ❌ No | `GET /users/42` — fetch user 42 |
| **POST** | Create a new resource | ❌ No | ✅ Yes | `POST /users` — create a new user |
| **PUT** | Replace a resource entirely | ✅ Yes | ✅ Yes | `PUT /users/42` — overwrite user 42 |
| **PATCH** | Partially update a resource | ❌ No (usually) | ✅ Yes | `PATCH /users/42` — update just the email |
| **DELETE** | Remove a resource | ✅ Yes | ❌ Usually no | `DELETE /users/42` — delete user 42 |
| **HEAD** | Like GET, but headers only (no body) | ✅ Yes | ❌ No | Check if resource exists |
| **OPTIONS** | Discover allowed methods on a resource | ✅ Yes | ❌ No | Used in CORS pre-flight checks |

> **Idempotent** means: calling it once or 100 times produces the same server state. `GET`, `PUT`, and `DELETE` are idempotent. `POST` is not (calling it twice usually creates two resources).

### 3.1 Example: PUT vs PATCH
```http
PUT /users/42
{ "name": "Alice", "email": "alice@example.com", "age": 30 }
```
This **replaces the entire user record** — missing fields may be wiped out.

```http
PATCH /users/42
{ "email": "newalice@example.com" }
```
This **only updates the email**, leaving other fields untouched.

---

## 4. HTTP Status Codes

Status codes tell the client the **result** of its request. They are grouped into 5 classes by their first digit.

| Range | Class | Meaning |
|-------|-------|---------|
| **1xx** | Informational | Request received, processing continues |
| **2xx** | Success | Request succeeded |
| **3xx** | Redirection | Further action needed to complete request |
| **4xx** | Client Error | The client made a mistake |
| **5xx** | Server Error | The server failed to fulfill a valid request |

### 4.1 Most Common Codes

| Code | Name | When You'll See It |
|------|------|---------------------|
| `200` | OK | Standard success response |
| `201` | Created | New resource successfully created (after POST) |
| `204` | No Content | Success, but nothing to return (after DELETE) |
| `301` | Moved Permanently | Resource has a new permanent URL |
| `304` | Not Modified | Cached version is still valid |
| `400` | Bad Request | Malformed request (e.g., invalid JSON) |
| `401` | Unauthorized | Missing/invalid authentication |
| `403` | Forbidden | Authenticated but not allowed to access |
| `404` | Not Found | Resource doesn't exist |
| `405` | Method Not Allowed | e.g., using DELETE on a read-only endpoint |
| `409` | Conflict | Request conflicts with current state (e.g., duplicate entry) |
| `422` | Unprocessable Entity | Validation error on submitted data |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Generic server crash/bug |
| `502` | Bad Gateway | Upstream server sent an invalid response |
| `503` | Service Unavailable | Server temporarily overloaded/down |

### 4.2 Quick Memory Trick
- **2xx** → "Yes, it worked"
- **4xx** → "You (client) did something wrong"
- **5xx** → "We (server) did something wrong"

---

## 5. JSON (JavaScript Object Notation)

JSON is the standard **data format** used to exchange information between client and server. It's lightweight, human-readable, and language-independent.

### 5.1 Syntax Rules
- Data is in **key-value pairs**: `"key": value`
- Keys must be strings wrapped in double quotes
- Values can be: string, number, boolean, null, array, or object
- Objects are wrapped in `{ }`
- Arrays are wrapped in `[ ]`

### 5.2 Example

```json
{
  "id": 42,
  "name": "Alice",
  "isActive": true,
  "age": 30,
  "email": null,
  "roles": ["admin", "editor"],
  "address": {
    "city": "Dhaka",
    "zip": "1207"
  }
}
```

### 5.3 JSON Data Types

| Type | Example |
|------|---------|
| String | `"hello"` |
| Number | `42`, `3.14` |
| Boolean | `true`, `false` |
| Null | `null` |
| Array | `[1, 2, 3]` |
| Object | `{ "key": "value" }` |

### 5.4 Why JSON (and not XML)?
- Smaller, less verbose
- Maps directly to objects in most programming languages
- Faster to parse
- Native support in JavaScript (`JSON.parse()` / `JSON.stringify()`)

---

## 6. REST (Representational State Transfer) Basics

REST is an **architectural style** for designing APIs that use HTTP methods and URLs to operate on **resources**.

### 6.1 Core Principles

| Principle | Meaning |
|-----------|---------|
| **Stateless** | Server stores no client session; each request has all info needed |
| **Resource-based** | Everything is a "resource" identified by a URL (e.g., `/users`, `/orders/5`) |
| **Uniform Interface** | Standard HTTP methods (GET, POST, PUT, DELETE) act consistently on resources |
| **Client-Server separation** | Frontend and backend evolve independently |
| **Cacheable** | Responses should define if/how they can be cached |
| **Layered System** | Client doesn't need to know if it's talking to the actual server or an intermediary (load balancer, proxy) |

### 6.2 Designing REST Endpoints — Example: "Users" Resource

| Action | Method | Endpoint | Status on Success |
|--------|--------|----------|---------------------|
| Get all users | GET | `/users` | 200 |
| Get one user | GET | `/users/42` | 200 |
| Create a user | POST | `/users` | 201 |
| Replace a user | PUT | `/users/42` | 200 |
| Update part of a user | PATCH | `/users/42` | 200 |
| Delete a user | DELETE | `/users/42` | 204 |

### 6.3 Full Example Flow

**Request — Create a new user:**
```http
POST /users HTTP/1.1
Content-Type: application/json

{
  "name": "Bob",
  "email": "bob@example.com"
}
```

**Response:**
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 43,
  "name": "Bob",
  "email": "bob@example.com"
}
```

**Request — Fetch that user later:**
```http
GET /users/43 HTTP/1.1
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 43,
  "name": "Bob",
  "email": "bob@example.com"
}
```

### 6.4 REST Naming Conventions (Good Practice)
- Use **nouns**, not verbs: `/users` ✅ not `/getUsers` ❌
- Use **plural** resource names: `/products` not `/product`
- Nest resources logically: `/users/42/orders` → orders belonging to user 42
- Use query params for filtering/sorting: `/products?category=shoes&sort=price`

---

## 7. Putting It All Together — Mini Case Study

**Scenario**: A shopping app needs to display a product and let a user add a review.

1. **Client** sends: `GET /products/101`
2. **Server** responds: `200 OK` with JSON:
   ```json
   { "id": 101, "name": "Running Shoes", "price": 59.99 }
   ```
3. **Client** renders the product page.
4. User submits a review → **Client** sends:
   ```http
   POST /products/101/reviews
   { "rating": 5, "comment": "Great shoes!" }
   ```
5. **Server** validates, saves it, responds: `201 Created`
6. If the user wasn't logged in, server would instead respond: `401 Unauthorized`

This single scenario touches **every topic** in this guide: client-server roles, HTTP methods (GET/POST), status codes (200/201/401), JSON as the data format, and REST-style resource URLs.

---

## Quick Reference Cheat Sheet

```
GET     → Read           → 200
POST    → Create         → 201
PUT     → Replace        → 200
PATCH   → Partial Update → 200
DELETE  → Remove         → 204

2xx → Success
4xx → Client's fault
5xx → Server's fault

JSON = { "key": "value", "array": [1,2,3] }

REST = Resources (nouns) + HTTP verbs + Statelessness
```
